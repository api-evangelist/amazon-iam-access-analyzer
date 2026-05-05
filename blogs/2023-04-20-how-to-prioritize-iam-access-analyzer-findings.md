---
title: "How to prioritize IAM Access Analyzer findings"
url: "https://aws.amazon.com/blogs/security/how-to-prioritize-iam-access-analyzer-findings/"
date: "Thu, 20 Apr 2023 15:14:27 +0000"
author: "Swara Gandhi"
feed_url: "https://aws.amazon.com/blogs/security/tag/iam-access-analyzer/feed/"
---
<p><a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/what-is-access-analyzer.html" rel="noopener" target="_blank">AWS Identity and Access Management (IAM) Access Analyzer</a>&nbsp;is an important tool in your journey towards <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#grant-least-privilege" rel="noopener" target="_blank">least privilege access</a>. You can use IAM Access Analyzer access previews to <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-access-preview.html" rel="noopener" target="_blank">preview and validate public and cross-account access</a> before deploying permissions changes in your environment.</p> 
<p>For the permissions already in place, one of IAM Access Analyzer’s capabilities is that it helps you identify resources in your <a href="https://aws.amazon.com/organizations/" rel="noopener" target="_blank">AWS Organizations</a> organization and AWS accounts that are shared with an external entity.</p> 
<p>For each external entity that has access to a resource in your account, IAM Access Analyzer generates a&nbsp;<a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-work-with-findings.html" rel="noopener" target="_blank">finding</a>. Findings display information about the resource and the policy statement that generated the finding, with details such as the list of actions in the policy granting access, level of access, and conditions that allow the access. You can review the findings to determine if the access is intended or unintended.</p> 
<p>As your use of AWS services grows and the number of accounts in your organization increases, the number of findings that you have might also increase. To help reduce noise and allow you to focus on unintended access findings, you can filter findings and create archive rules for intended access.</p> 
<p>This blog post provides step-by-step guidance on how to get started with IAM Access Analyzer findings by using different filtering techniques that can help you filter approved use cases that result in access findings. For example, you might see a finding generated for an S3 bucket that hosts images for your website and thus allows public access, as approved by your organization, apply a filter so that you can concentrate on unintended access. IAM Access Analyzer offers a wide range of filters; for a complete list, see the <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-findings-filter.html" rel="noopener" target="_blank">IAM documentation</a>.</p> 
<p>In this post, we also share example archive rules for approved use cases that result in access findings. Archive rules automatically archive new findings that meet the criteria you define when you create the rule. You can also apply archive rules retroactively to archive existing findings that meet the archive rule criteria. Finally, we have included an example implementation of <a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-accessanalyzer-analyzer-archiverule.html" rel="noopener" target="_blank">archive rules</a> using an <a href="https://aws.amazon.com/cloudformation/" rel="noopener" target="_blank">AWS CloudFormation</a> template.</p> 
<h2>IAM Access Analyzer findings overview</h2> 
<p>To get started, create an analyzer for your entire organization or your account. The organization or account that you choose is known as the <em>zone of trust</em> for the analyzer. The zone of trust determines the type of access that IAM Access Analyzer considers to be trusted. IAM Access Analyzer continuously monitors to identify resource policies, access control lists, and other access controls that grant public or cross-account access from outside the zone of trust, and generates findings. For this blog post, we’ll demonstrate an organization as the zone of trust,&nbsp;showcasing findings from&nbsp;a large-scale, multi-account AWS deployment.</p> 
<h2>Prerequisites</h2> 
<p>This blog post assumes that you have the following in place:</p> 
<ul> 
 <li>IAM Access Analyzer is enabled in your organization or account in the AWS Regions where you operate. For more details on how to enable IAM Access Analyzer, see <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-getting-started.html#access-analyzer-enabling" rel="noopener" target="_blank">Enabling IAM Access Analyzer</a>.</li> 
 <li>Access to the <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-getting-started.html#access-analyzer-permissions-awsmanpol" rel="noopener" target="_blank">AWS Organizations management account</a> or to a member account in the organization with delegated administrator access for creating and updating IAM Access Analyzer resources.</li> 
</ul> 
<h2>How to filter the findings</h2> 
<p>To start filtering your findings and create archive rules, you should complete the following steps:</p> 
<ol> 
 <li>Review public access findings</li> 
 <li>Filter by removing permissions errors</li> 
 <li>Filter for known identity providers</li> 
 <li>Filter cross-account access from trusted external accounts</li> 
</ol> 
<p>We’ll walk you through each step.</p> 
<h2>1. Review public access findings</h2> 
<p>Some AWS resources allow public access on the resource by means of a resource-based policy—for example, an <a href="https://docs.aws.amazon.com/AmazonS3/latest/userguide/bucket-policies.html" rel="noopener" target="_blank">Amazon Simple Storage Service (Amazon S3) bucket policy</a> that has the <span style="font-family: courier;">“Principal:*”</span> permission added to its bucket policy.&nbsp;For resources such as <a href="https://aws.amazon.com/ebs/" rel="noopener" target="_blank">Amazon Elastic Block Store (Amazon EBS)</a> snapshots, you can share these by using a flag on the resource permission. IAM Access Analyzer looks for such sharing and reports it in the findings.</p> 
<p>From the global report, you can generate a list of resources that allow public access by using the <strong>Public access: true</strong> query in the IAM console.</p> 
<p>The following is an example of an <a href="https://aws.amazon.com/cli/" rel="noopener" target="_blank">AWS Command Line Interface (AWS CLI)</a> command with <a href="https://awscli.amazonaws.com/v2/documentation/api/latest/reference/accessanalyzer/list-findings.html" rel="noopener" target="_blank">public access as “true”</a>. Replace <span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;AccessAnalyzerARN&gt;</span> with the Amazon Resource Name (ARN) of your <a href="https://us-east-1.console.aws.amazon.com/access-analyzer/home?region=us-east-1#/analyzers" rel="noopener" target="_blank">analyzer</a>.</p> 
<div>
 <code style="color: #000000;">aws accessanalyzer list-findings --analyzer-arn <span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;AccessAnalyzerARN&gt;</span> --filter isPublic={"eq"="true"}</code>
</div> 
<h3>Is the public access intended?</h3> 
<p>If the access is intended, you can archive the findings by creating an archive rule using the <a href="https://aws.amazon.com/console/" rel="noopener" target="_blank">AWS Management Console</a>, <a href="https://docs.aws.amazon.com/cli/latest/reference/accessanalyzer/create-archive-rule.html" rel="noopener" target="_blank">AWS CLI</a>, or <a href="https://docs.aws.amazon.com/access-analyzer/latest/APIReference/API_CreateArchiveRule.html" rel="noopener" target="_blank">API.</a> When you archive a security finding, IAM Access Analyzer removes it from the <strong>Active</strong> findings list and changes its status to <strong>Archived</strong>. For instructions on how to automatically archive expected findings, see <a href="https://aws.amazon.com/blogs/security/how-to-automatically-archive-expected-iam-access-analyzer-findings/" rel="noopener" target="_blank">How to automatically archive expected IAM Access Analyzer findings</a>.</p> 
<h3>Example: Known S3 bucket that hosts public website images</h3> 
<p>If you have resources for which public access is expected, such as an S3 bucket that hosts images for your website, you can add an archive rule with <strong>Resource</strong> criteria equal to the bucket name, as shown in Figure 1.</p> 
<div class="wp-caption aligncenter" id="attachment_29176" style="width: 770px;">
 <img alt="Figure 1: Create IAM Access Analyzer archive rule using the console" class="size-large wp-image-29176" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/04/17/img1-4-1024x683.png" style="border: 1px solid #bebebe;" width="760" />
 <p class="wp-caption-text" id="caption-attachment-29176">Figure 1: Create IAM Access Analyzer archive rule using the console</p>
</div> 
<h3>Is the public access unintended?</h3> 
<p>If the finding results from policies that were misconfigured to allow unintended public access, you can constrain the access by using <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html" rel="noopener" target="_blank">AWS global condition context keys</a> or a specific IAM principal ARN.&nbsp;The findings show the account and resource that contain the policy.</p> 
<p>For example, if the finding shows a misconfigured S3 bucket, the following policy shows how you can modify the S3 bucket policy to only allow IAM principals from your organization to access the bucket by using the <span style="font-family: courier;">PrincipalOrgID</span> condition key. Replace <span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;DOC-EXAMPLE-BUCKET&gt;</span> with the name of your S3 bucket, and <span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;ORGANIZATION_ID&gt;</span> with your organization ID.</p> 
<div class="hide-language"> 
 <pre><code class="lang-text">{
   "Version":"2008-10-17",
   "Id":"Policy1335892530063",
   "Statement":[
      {
         "Sid":"AllowS3Access",
         "Effect":"Allow",
         "Principal":"*",
         "Action":"s3:*",
         "Resource":[
            "arn:aws:s3:::<span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;DOC-EXAMPLE-BUCKET&gt;</span>",
            "arn:aws:s3:::<span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;DOC-EXAMPLE-BUCKET&gt;</span>/*"
         ],
         "Condition":{
            "StringEquals":{
               "aws:PrincipalOrgID":"<span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;ORGANIZATION_ID&gt;</span>"
            }
         }
      }
   ]
}</code></pre> 
</div> 
<h2>2. Filter by&nbsp;removing permissions errors</h2> 
<p>Before you further investigate the IAM Access Analyzer findings, you should make sure that IAM Access Analyzer has enough permissions to access the resources in your accounts to be able to provide the analysis.</p> 
<p>IAM Access Analyzer uses an AWS <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_terms-and-concepts.html#iam-term-service-linked-role" rel="noopener" target="_blank">service-linked role</a>&nbsp;to call other AWS services on your behalf. When IAM Access Analyzer analyzes a resource, it reads resource metadata, such as a resource-based policy, access control lists, and other access controls that grant public or cross-account access. If the policies don’t allow an IAM Access Analyzer role to read the resource metadata, it generates an <strong>Access Denied</strong> error finding, as shown in Figure 2.</p> 
<div class="wp-caption aligncenter" id="attachment_29177" style="width: 770px;">
 <img alt="Figure 2: IAM Access Analyzer access denied error example" class="size-large wp-image-29177" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/04/17/img2-4-1024x120.png" style="border: 1px solid #bebebe;" width="760" />
 <p class="wp-caption-text" id="caption-attachment-29177">Figure 2: IAM Access Analyzer access denied error example</p>
</div> 
<p>To view these error findings from the <a href="https://us-east-1.console.aws.amazon.com/access-analyzer/home?region=us-east-1#/findings" rel="noopener" target="_blank">IAM Access Analyzer console</a>, filter the findings by using the <strong>Error: Access Denied</strong> property.</p> 
<h3>Resolution</h3> 
<p>To resolve the access issue, make sure that the IAM Access Analyzer&nbsp;<a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-using-service-linked-roles.html#create-slr" rel="noopener" target="_blank">service-linked role</a> is not denied access. Review the resource-based policy attached to the resource that IAM Access Analyzer isn’t able to access. For a list of services that support resource-based policies, see the <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_aws-services-that-work-with-iam.html" rel="noopener" target="_blank">IAM documentation</a>.</p> 
<p>For example, if the analyzer can’t access an <a href="https://aws.amazon.com/kms/" rel="noopener" target="_blank">AWS Key Management Service (AWS KMS)</a> key because of an explicit deny, add an exception for the IAM Access Analyzer service-linked role to the policy statement, similar to the following. Make sure that you change the <span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;ACCOUNT_ID&gt;</span> to your account id.</p> 
<table style="border-collapse: collapse; border: 1px solid #808080;" width="100%"> 
 <tbody>
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%"><strong>Before</strong></td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%"><strong>After</strong></td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080; vertical-align: top;" width="50%"> 
    <div class="hide-language"> 
     <pre class="unlimited-height-code" style="width: 390px;"><code class="lang-text">{
   "Sid":"Deny unintended access to KMS key",
   "Effect":"Deny",
   "Principal":"*",
   "Action":[
      "kms:DescribeKey",
      "kms:GetKeyPolicy",
      "kms:List*"
   ],
   "Resource":"*",
   "Condition":{
      "ArnNotLikeIfExists":{
         "aws:PrincipalArn":[
            "arn:aws:iam::*:role/<span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;YOUR-ADMIN-ROLE&gt;</span>"
         ]
      }
   }
}</code></pre> 
    </div> </td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%"> 
    <div class="hide-language"> 
     <pre class="unlimited-height-code" style="width: 390px;"><code class="lang-text">{
   "Sid":"Deny unintended access to KMS key",
   "Effect":"Deny",
   "Principal":"*",
   "Action":[
      "kms:DescribeKey",
      "kms:GetKeyPolicy",
      "kms:List*"
   ],
   "Resource":"*",
   "Condition":{
      "ArnNotLikeIfExists":{
         "aws:PrincipalArn":[
            "arn:aws:iam::<span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;ACCOUNT_ID&gt;</span>:role/aws-service-role/access-analyzer.amazonaws.com/AWSServiceRoleForAccessAnalyzer",
"arn:aws:iam::*:role/<span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;YOUR-ADMIN-ROLE&gt;</span>"
         ]
      }
   }
}</code></pre> 
    </div> </td> 
  </tr> 
 </tbody>
</table> 
<h2>3. Filter for known identity providers</h2> 
<p>With <a href="https://wiki.oasis-open.org/security" rel="noopener" target="_blank">SAML 2.0</a> or <a href="https://openid.net/connect/" rel="noopener" target="_blank">Open ID Connect (OIDC)</a>—which are open federation standards that many identity providers (IdPs) use—users can log in to the console or call the AWS API operations without you having to create an IAM user for everyone in your organization.</p> 
<p>To set up federation,&nbsp;you must perform a one-time configuration so that your organization’s IdP and your account trust each other.&nbsp;To configure this trust, you must register AWS as a service provider (SP) with the IdP of your organization and set up metadata and key exchange.</p> 
<p>The role or roles that you create in IAM define what the federated users from your organization are allowed to use on AWS. When you create the trust policy for the role, you specify the SAML or OIDC provider as the <code style="color: #000000;">Principal</code>. To only allow users that match certain attributes to access the role, you can scope the trust policy with a <code style="color: #000000;">Condition.</code></p> 
<h3>Example 1: Federation with Okta </h3> 
<p>Let’s walk through an example that uses Okta as the IdP. Although access to a trusted IdP is intended, IAM Access Analyzer creates a finding for an IAM role that has trust policy granting access to a SAML provider because the trust policy allows access outside of the known zone of trust for the analyzer. You will see findings created for the IAM role granting access to Okta using the IAM trust policy, as shown in Figure 3.</p> 
<div class="wp-caption aligncenter" id="attachment_29187" style="width: 770px;">
 <img alt="Figure 3: IAM Access Analyzer identity provider finding example" class="size-full wp-image-29187" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/04/18/img3.jpg" style="border: 1px solid #bebebe;" width="760" />
 <p class="wp-caption-text" id="caption-attachment-29187">Figure 3: IAM Access Analyzer identity provider finding example</p>
</div> 
<h3>Resolution&nbsp;</h3> 
<p>Setting access through SAML providers is a privileged operation, so we recommend that you analyze each finding to decide if an exception is acceptable. If you approve of the SAML-provided access setup, you can implement an archive rule to archive such findings with conditions for federation used in combination with your SAML provider.&nbsp;The filter for the <strong>Federated User</strong> rule depends on the name that you gave to the SAML IdP in your federation setup. For example, if your SAML IdP name is Okta, the rule should have a filter for <span style="font-family: courier;">arn:aws:iam::<span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;ACCOUNT_ID&gt;</span>:saml-provider/Okta</span>, where <span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;ACCOUNT_ID&gt;</span> is your account number, as shown in Figure 4.</p> 
<div class="wp-caption aligncenter" id="attachment_29188" style="width: 770px;">
 <img alt="Figure 4: Archive rule example for using an IdP-related finding" class="size-full wp-image-29188" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/04/18/img4-4.png" style="border: 1px solid #bebebe;" width="760" />
 <p class="wp-caption-text" id="caption-attachment-29188">Figure 4: Archive rule example for using an IdP-related finding</p>
</div> 
<blockquote>
 <p><strong>Note</strong>: To include additional values for a multi-account setup, use the <strong>Add another value</strong> filter.</p>
</blockquote> 
<h3>Example 2:&nbsp;IAM Identity Center</h3> 
<p>With <a href="https://aws.amazon.com/iam/identity-center/" rel="noopener" target="_blank">AWS IAM Identity Center (successor to AWS Single Sign-On)</a>, you can manage sign-in security for your workforce. IAM Identity Center provides a central place to define your<a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html" rel="noopener" target="_blank"> permission sets</a>, assign them to your users and groups, and give your users a portal where they can access their assigned accounts.</p> 
<p>With IAM Identity Center, you manage access to accounts by creating and assigning<a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/permissionsetsconcept.html" rel="noopener" target="_blank"> permission sets</a>. These are <a href="https://aws.amazon.com/iam/" rel="noopener" target="_blank">IAM</a> role templates that define (among other things) which policies to include in a role. When you create a permission set in IAM Identity Center and associate it to an account, IAM Identity Center creates a role in that account with a trust policy that allows a federated IdP as a principal — in this case, IAM Identity Center.</p> 
<p>IAM Access Analyzer generates a finding for this setup because the allowed access is outside of the known zone of trust for the analyzer, as shown in Figure 5.</p> 
<div class="wp-caption aligncenter" id="attachment_29189" style="width: 770px;">
 <img alt="Figure 5: IAM Access Analyzer finding example for IAM Identity Center" class="size-full wp-image-29189" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/04/18/img5.jpg" style="border: 1px solid #bebebe;" width="760" />
 <p class="wp-caption-text" id="caption-attachment-29189">Figure 5: IAM Access Analyzer finding example for IAM Identity Center</p>
</div> 
<p>To filter this finding, you need to implement an archive rule.</p> 
<h3>Resolution</h3> 
<p>You can implement an archive rule with conditions for federation used in combination with IAM Identity Center as the SAML provider.&nbsp;The roles created by IAM Identity Center in member accounts use a reserved path on AWS: <span style="font-family: courier;">arn:aws:iam::<span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;ACCOUNT_ID&gt;</span>:role/aws-reserved/sso.amazonaws.com/</span>. Hence, you can create an archive rule with a filter that contains <span style="font-family: courier;">:saml-provider/AWSSSO</span> in the <strong>Federated User</strong> name and <span style="font-family: courier;">aws-reserved/sso.amazonaws.com/</span> in the <strong>Resource</strong>, as shown in Figure 6.</p> 
<div class="wp-caption aligncenter" id="attachment_29190" style="width: 770px;">
 <img alt="Figure 6: Archive rule example for IAM Identity Center generated findings" class="size-full wp-image-29190" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/04/18/img6.jpg" style="border: 1px solid #bebebe;" width="760" />
 <p class="wp-caption-text" id="caption-attachment-29190">Figure 6: Archive rule example for IAM Identity Center generated findings</p>
</div> 
<h2>4. Filter cross-account access findings from trusted external accounts</h2> 
<p>We recommend that you identify and document accounts and principals that should be allowed access outside of the zone of trust for IAM Access Analyzer.</p> 
<p>When a resource-based policy attached to a resource allows cross-account access from outside the zone of trust, IAM Access Analyzer generates cross-account access findings.</p> 
<h3>Is the cross-account access intended?</h3> 
<p>When you review cross-account access findings, you need to determine whether the access is intended or not. For example, you might have access provided to your auditor’s account or a partner account for visibility and monitoring of your AWS applications.</p> 
<p>For trusted external accounts, you can create an archive rule that includes the AWS account in the criteria for the rule. Figure 7 shows an example of how to create the archive rule for a trusted external account (<span style="font-family: courier; color: #ff0000; font-style: italic;">EXTERNAL_ACCOUNT_ID</span>). In your own rule, replace <span style="font-family: courier; color: #ff0000; font-style: italic;">EXTERNAL_ACCOUNT_ID</span> with the trusted account id.</p> 
<div class="wp-caption aligncenter" id="attachment_29191" style="width: 770px;">
 <img alt="Figure 7: Archive rule example for trusted account findings" class="size-full wp-image-29191" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/04/18/img7.jpg" style="border: 1px solid #bebebe;" width="760" />
 <p class="wp-caption-text" id="caption-attachment-29191">Figure 7: Archive rule example for trusted account findings</p>
</div> 
<h3>Is the cross-account access unintended?</h3> 
<p>After you have archived the intended access findings, you can start analyzing the findings initiated from unintended access. When you confirm that the findings show unintended access, you should take steps to remove the access by altering or deleting the policy or access control that granted access. You can expand the solution outlined in the blog post <a href="https://aws.amazon.com/blogs/security/automate-resolution-for-iam-access-analyzer-cross-account-access-findings-on-iam-roles/" rel="noopener" target="_blank">Automate resolution for IAM Access Analyzer cross-account access findings on IAM roles</a> by adding an explicit deny statement.</p> 
<p>You can also use&nbsp;<a href="https://aws.amazon.com/cloudtrail" rel="noopener" target="_blank">AWS CloudTrail</a>&nbsp;to track API calls that could have changed access configuration on your AWS resources.</p> 
<h2>Deploy IAM Access Analyzer and archive rules with a CloudFormation template</h2> 
<p>In this section, we demonstrate a sample <a href="https://aws.amazon.com/cloudformation/" rel="noopener" target="_blank">CloudFormation</a> template that creates an IAM access analyzer and archive rules for findings that are created for identified intended access to resources.</p> 
<blockquote>
 <p><strong>Important:</strong>&nbsp;When you create&nbsp;an archive rule using the AWS console, the existing findings and new findings that match criteria mentioned in the rules will be archived. However, archive rules created through CloudFormation or the AWS CLI will only archive the new findings&nbsp;that meet the criteria defined.&nbsp;You need to perform the <a href="https://docs.amazonaws.cn/en_us/access-analyzer/latest/APIReference/API_ApplyArchiveRule.html" rel="noopener" style="font-family: courier; background-color: #fofofo;" target="_blank">access-analyzer:ApplyArchiveRule</a>&nbsp;API after you create the archive rule to archive existing findings as well.</p>
</blockquote> 
<p>The sample CloudFormation template takes the following values as inputs and creates archive rules for findings that are created for identified intended access to resources shared outside of your zone of trust for the specified analyzer:</p> 
<ul> 
 <li>Analyzer name</li> 
 <li>Zone of trust</li> 
 <li>Known public S3 buckets, if you have any (for example, a bucket that hosts public website images).<br /> 
  <blockquote>
   <p><strong>Note</strong>: We use S3 buckets as an example. You can edit the rule to include resource types that are supported by IAM Access Analyzer, if public access is intended.</p>
  </blockquote> </li> 
 <li>Trusted accounts — AWS accounts&nbsp;that don’t belong to your organization, but you trust them to have access to resources in your organization</li> 
 <li>SAML provider — The SAML provider approved to have access to your resources<br /> 
  <blockquote>
   <p><strong>Note</strong>: If you don’t use federation, you can remove the rule <span style="font-family: courier;">SAMLFederatedUsers</span>.</p>
  </blockquote> </li> 
</ul> 
<pre class="unlimited-height-code"><code class="lang-yaml">AWSTemplateFormatVersion: 2010-09-09
Description: &gt;+
  Sample CloudFormation template creates archive rules for findings
  created for resources shared outside of your zone of trust for specified
  analyzer. 
   
Metadata:
  AWS::CloudFormation::Interface:
    ParameterGroups:
      - Label:
          default: Define Configuration
        Parameters:
          - AccessAnalyzerName
          - ZoneOfTrust
          - KnownPublicS3Buckets
          - TrustedAccounts
          - SAMLProvider
Parameters:
  AccessAnalyzerName:
    Description: Provide name of the analyzer you would like to create archive rules for.
    Type: String
  ZoneOfTrust:
    Description: Select the zone of trust of AccessAnalyzer
    AllowedValues:
      - ACCOUNT
      - ORGANIZATION
    Type: String
  KnownPublicS3Buckets:
    Description: List of comma-separated known S3 bucket arns, that should allow
      public access Example -
      arn:aws:s3:::DOC-EXAMPLE-BUCKET,arn:aws:s3:::DOC-EXAMPLE-BUCKET2
    Type: CommaDelimitedList
  TrustedAccounts:
    Description: List of comma-separated account IDs, that do not belong to your
      organization but you trust them to have access to resources in your
      organization. [Example - Your auditor’s AWS account]
    Type: List&lt;Number&gt;
  TrustedFederationPrincipals:
    Description: List of comma-separated trusted federated principals that are able
      to assume roles in your accounts. [Example -
      arn:aws:iam::012345678901:saml-provider/Okta,
      arn:aws:iam::1111222233334444:saml-provider/Okta]
    Type: CommaDelimitedList
Resources:
  AccessAnalyzer:
    Type: AWS::AccessAnalyzer::Analyzer
    Properties:
      AnalyzerName: ${AccessAnalyzerName}-${AWS::Region}
      Type: ZoneOfTrust
      ArchiveRules:
        - RuleName: ArchivePublicS3BucketsAccess
          Filter:
            - Property: resource
              Eq: KnownPublicS3Buckets
        - RuleName: AccountAccessNecessaryForBusinessProcesses
          Filter:
            - Property: principal.AWS
              Eq: TrustedAccounts
            - Property: isPublic
              Eq:
                - "false"
        - RuleName: SAMLFederatedUsers
          Filter:
            - Property: principal.Federated
              Eq: TrustedFederationPrincipals</code></pre> 
<p>To download this sample template, download the file <a href="https://aws-security-blog-content.s3.amazonaws.com/public/sample/1565-prioritize-access-analyzer-findings/IAMAccessAnalyzer.yaml" rel="noopener" style="font-family: courier;" target="_blank">IAMAccessAnalyzer.yaml</a> from Amazon S3.</p> 
<h2>Conclusion</h2> 
<p>In this blog post, you learned how to start with IAM Access Analyzer findings, filter them based on the level of access given outside of your zone of trust, and create archive rules for intended access findings. By using different filtering techniques to remediate intended access findings, you can concentrate on unintended access.</p> 
<p>To take this solution further, we recommend that you consider&nbsp;<a href="https://aws.amazon.com/blogs/security/automate-resolution-for-iam-access-analyzer-cross-account-access-findings-on-iam-roles/" rel="noopener" target="_blank">automating the resolution of unintended cross-account IAM roles found by IAM Access Analyzer by adding a deny statement to the IAM role’s trust policy</a>. You can also include capabilities like an approval workflow to resolve the finding to suit your organization’s process requirements.</p> 
<p>Lastly, we suggest that you use IAM Access Analyzer access previews to <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-access-preview.html" rel="noopener" target="_blank">preview and validate public and cross-account access</a> before deploying permissions changes in your environment.</p> 
<p>&nbsp;<br />If you have feedback about this post, submit comments in the<strong> Comments</strong> section below. If you have questions about this post, <a href="https://console.aws.amazon.com/support/home" rel="noopener noreferrer" target="_blank">contact AWS Support</a>.</p> 
<p><strong>Want more AWS Security news? Follow us on <a href="https://twitter.com/AWSsecurityinfo" rel="noopener noreferrer" target="_blank" title="Twitter">Twitter</a>.</strong></p> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image"> 
   <img alt="Swara Gandhi" class="aligncenter size-full wp-image-25347" height="2662" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2022/05/25/ganswara.jpg" width="1854" /> 
  </div> 
  <h3 class="lb-h4">Swara Gandhi</h3> 
  <p>Swara is a solutions architect on the AWS Identity Solutions team. She works on building secure and scalable end-to-end identity solutions. She is passionate about everything identity, security, and cloud.</p> 
  <p></p>
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image"> 
   <img alt="" class="aligncenter size-full wp-image-24337" height="160" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2021/09/30/Nitin-Kulkarni-Profile.jpg" width="120" /> 
  </div> 
  <h3 class="lb-h4">Nitin Kulkarni</h3> 
  <p>Nitin is a Solutions Architect on the AWS Identity Solutions team. He helps customers build secure and scalable solutions on the AWS platform. He also enjoys hiking, baseball and linguistics.</p> 
  <p></p>
 </div> 
</footer>
