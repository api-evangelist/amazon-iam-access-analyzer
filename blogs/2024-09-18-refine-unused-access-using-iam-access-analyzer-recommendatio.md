---
title: "Refine unused access using IAM Access Analyzer recommendations"
url: "https://aws.amazon.com/blogs/security/refine-unused-access-using-iam-access-analyzer-recommendations/"
date: "Wed, 18 Sep 2024 19:09:34 +0000"
author: "Stéphanie Mbappe"
feed_url: "https://aws.amazon.com/blogs/security/tag/iam-access-analyzer/feed/"
---
<p>As a security team lead, your goal is to manage security for your organization at scale and ensure that your team follows <a href="https://aws.amazon.com/iam" rel="noopener" target="_blank">AWS Identity and Access Management (IAM)</a> <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html" rel="noopener" target="_blank">security best practices</a>, such as the principle of least privilege. As your developers build on AWS, you need visibility across your organization to make sure that teams are working with only the required privileges. Now, <a href="https://aws.amazon.com/iam/access-analyzer" rel="noopener" target="_blank">AWS Identity and Access Management Analyzer</a> offers prescriptive recommendations with actionable guidance that you can share with your developers to quickly refine unused access.</p> 
<p>In this post, we show you how to use IAM Access Analyzer recommendations to refine unused access. To do this, we start by focusing on the recommendations to refine unused permissions and show you how to generate the recommendations and the actions you can take. For example, we show you how to filter unused permissions findings, generate recommendations, and remediate issues. Now, with IAM Access Analyzer, you can include step-by-step recommendations to help developers refine unused permissions quickly.</p> 
<h2>Unused access recommendations</h2> 
<p>IAM Access Analyzer continuously analyzes your accounts to identify unused access and consolidates findings in a centralized dashboard. The dashboard helps review findings and prioritize accounts based on the volume of findings. The findings highlight unused IAM roles and unused access keys and passwords for IAM users. For active IAM roles and users, the findings provide visibility into unused services and actions. You can learn more about unused access analysis through the&nbsp;<a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/what-is-access-analyzer.html#what-is-access-analyzer-unused-access-analysis" rel="noopener" target="_blank">IAM Access Analyzer documentation</a>.</p> 
<p>For unused IAM roles, access keys, and passwords, IAM Access Analyzer provides quick links in the console to help you delete them. You can use the quick links to act on the recommendations or use export to share the details with the AWS account owner. For overly permissive IAM roles and users, IAM Access Analyzer provides policy recommendations with actionable steps that guide you to refine unused permissions. The recommended policies retain resource and condition context from existing policies, helping you update your policies iteratively.</p> 
<p>Throughout this post, we use <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create.html" rel="noopener" target="_blank">an IAM role</a> in an AWS account and configure the permissions by doing the following:</p> 
<ol> 
 <li>Attaching the AWS managed policy <a href="https://us-east-1.console.aws.amazon.com/iam/home?region=us-east-1#/policies/details/arn%3Aaws%3Aiam%3A%3Aaws%3Apolicy%2FAmazonBedrockReadOnly" rel="noopener" target="_blank">AmazonBedrockReadOnly</a>.</li> 
 <li>Attaching the AWS managed policy <a href="https://us-east-1.console.aws.amazon.com/iam/home?region=us-east-1#/policies/details/arn%3Aaws%3Aiam%3A%3Aaws%3Apolicy%2FAmazonS3ReadOnlyAccess" rel="noopener" target="_blank">AmazonS3ReadOnlyAccess</a>.</li> 
 <li>Embedding an inline policy with the permissions described in the following code and named <a href="https://us-east-1.console.aws.amazon.com/iam/home?region=us-east-1#/roles/details/IAMRole_IA2_Blog_EC2Role/editPolicy/InlinePolicyListLambda?step=addPermissions" rel="noopener" target="_blank">InlinePolicyListLambda</a>.</li> 
</ol> 
<p>Content of inline policy <a href="https://us-east-1.console.aws.amazon.com/iam/home?region=us-east-1#/roles/details/IAMRole_IA2_Blog_EC2Role/editPolicy/InlinePolicyListLambda?step=addPermissions" rel="noopener" target="_blank">InlinePolicyListLambda</a>:</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-text">{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "InlinePolicyLambda",
            "Effect": "Allow",
            "Action": [
                "lambda:ListFunctions",
                "lambda:ListLayers",
                "lambda:ListAliases",
                "lambda:ListFunctionUrlConfigs"
            ],
            "Resource": "*",
            "Condition": {
                "NotIpAddress": {
                    "aws:SourceIp": "1.100.150.200/32"
                }
            }
        }
    ]
}</code></pre> 
</div> 
<p>We use an <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_managed-vs-inline.html#inline-policies" rel="noopener" target="_blank">inline policy</a> to demonstrate that IAM Access Analyzer unused access recommendations are applicable for that use case. The recommendations are also applicable when using AWS managed policies and customer managed policies.</p> 
<p>In your AWS account, after you have configured an unused access analyzer, you can select an IAM role that you have used recently and see if there are unused access permissions findings and recommendations.</p> 
<h2>Prerequisites</h2> 
<p>Before you get started, you must create an unused access analyzer for your organization or account. Follow the instructions in <a href="https://aws.amazon.com/blogs/security/iam-access-analyzer-simplifies-inspection-of-unused-access-in-your-organization/" rel="noopener" target="_blank">IAM Access Analyzer simplifies inspection of unused access in your organization</a> to create an unused access analyzer.</p> 
<h2>Generate recommendations for unused permissions</h2> 
<p>In this post we explore three options for generating recommendations for IAM Access Analyzer unused permissions findings: the console, AWS CLI, and AWS API.</p> 
<h3>Generate recommendations for unused permissions using the console</h3> 
<p>After you have created an unused access analyzer as described in the prerequisites, wait a few minutes to see the analysis results. Then use the <a href="https://aws.amazon.com/console/" rel="noopener" target="_blank">AWS Management Console</a> to view the proposed recommendations for the unused permissions.</p> 
<h4>To list unused permissions findings</h4> 
<ol> 
 <li>Go to the IAM console and under Access Analyzer, choose <strong>Unused access</strong> from the navigation pane.</li> 
 <li>Search for active findings with the type <strong>Unused permissions</strong> in the search box. 
  <ol> 
   <li>Select <strong>Active</strong> from the <strong>Status</strong> drop-down list.</li> 
   <li>In the search box, select <strong>Findings type</strong> under <strong>Properties</strong>.</li> 
   <li>Select <strong>Equals</strong> as <strong>Operators</strong>.</li> 
   <li>Select <strong>Findings Type = Unused permissions</strong>.</li> 
   <li>This list shows the active findings for IAM resources with unused permissions.</li> 
  </ol> <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_35738" style="width: 708px;">
   <img alt="Figure 1: Filter on unused permissions in the IAM console" class="size-full wp-image-35738" height="171" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/09/13/img1.jpg" style="border: 1px solid #bebebe;" width="698" />
   <p class="wp-caption-text" id="caption-attachment-35738">Figure 1: Filter on unused permissions in the IAM console</p>
  </div><p></p> </li> 
 <li>Select a finding to learn more about the unused permissions granted to a given role or user.</li> 
</ol> 
<h4>To obtain recommendations for unused permissions</h4> 
<ol> 
 <li>On the findings detail page, you will see a list of the unused permissions under <strong>Unused permissions</strong>.</li> 
 <li>Following that, there is a new section called <strong>Recommendations</strong>. The Recommendations section presents two steps to remediate the finding: 
  <ol> 
   <li>Review the existing permissions on the resource.</li> 
   <li>Create new policies with the suggested refined permissions&nbsp;and detach the existing policies.</li> 
  </ol> <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_35739" style="width: 750px;">
   <img alt="Figure 2: Recommendations section" class="size-full wp-image-35739" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/09/13/img2-2.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-35739">Figure 2: Recommendations section</p>
  </div><p></p> </li> 
 <li>The generation of recommendations is on-demand and is done in the background when you’re using the console. The message <strong>Analysis in progress</strong> indicates that recommendations are being generated. The recommendations exclude the unused actions from the recommended policies.</li> 
 <li>When an IAM principal, such as an IAM role or user, has multiple permissions policies attached, an analysis of unused permissions is made for each of permissions policies: 
  <ul> 
   <li>If no permissions have been used, the recommended action is to detach the existing permissions policy.</li> 
   <li>If some permissions have been used, only the used permissions are kept in the recommended policy, helping you apply the principle of least privilege.</li> 
  </ul> </li> 
 <li>The recommendations are presented for each existing policy in the column <strong>Recommended policy</strong>. In this example, the existing policies are: 
  <ul> 
   <li>AmazonBedrockReadOnly</li> 
   <li>AmazonS3ReadOnlyAccess</li> 
   <li>InlinePolicyListLambda</li> 
  </ul> <p>And the recommended policies are:</p> 
  <ul> 
   <li>None</li> 
   <li>AmazonS3ReadOnlyAccess-recommended</li> 
   <li>InlinePolicyListLambda-recommended</li> 
  </ul> <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_35740" style="width: 750px;">
   <img alt="Figure 3: Recommended policies" class="size-full wp-image-35740" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/09/13/img3-2.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-35740">Figure 3: Recommended policies</p>
  </div><p></p> </li> 
 <li>There is no recommended policy for <code style="color: #000000;">AmazonBedrockReadOnly</code> because the recommended action is to detach it. When hovering over <strong>None</strong>, the following message is displayed: <strong>There are no recommended policies to create for the existing permissions policy</strong>.</li> 
 <li><code style="color: #000000;">AmazonS3ReadOnlyAccess</code> and <code style="color: #000000;">InlinePolicyListLambda</code> and their associated recommended policy can be previewed by choosing <strong>Preview policy</strong>.</li> 
</ol> 
<h4>To preview a recommended policy</h4> 
<p>IAM Access Analyzer has proposed two recommended policies based on the unused actions.</p> 
<ol> 
 <li>To preview each recommended policy, choose <strong>Preview policy</strong> for that policy to see a comparison between the existing and recommended permissions. 
  <ol> 
   <li>Choose <strong>Preview policy</strong> for <strong>AmazonS3ReadOnlyAccess-recommended</strong>. 
    <ol> 
     <li>The existing policy has been analyzed and the broad permissions—<code style="color: #000000;">s3:Get*</code> and <code style="color: #000000;">s3:List*</code>—have been scoped down to detailed permissions in the recommended policy.</li> 
     <li>The permissions <code style="color: #000000;">s3:Describe*</code>, <code style="color: #000000;">s3-object-lambda:Get*</code>, and <code style="color: #000000;">s3-object-lambda:List*</code> can be removed because they weren’t used.</li> 
    </ol> <p style="line-height: 1.25em;"></p>
    <div class="wp-caption aligncenter" id="attachment_35741" style="width: 710px;">
     <img alt="Figure 4: Preview of the recommended policy for AmazonS3ReadOnlyAccess" class="size-full wp-image-35741" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/09/13/img4-2.png" style="border: 1px solid #bebebe;" width="700" />
     <p class="wp-caption-text" id="caption-attachment-35741">Figure 4: Preview of the recommended policy for AmazonS3ReadOnlyAccess</p>
    </div><p></p> </li> 
   <li>Choose <strong>Preview policy</strong> for <strong>InlinePolicyListLambda-recommended</strong> to see a comparison between the existing inline policy <code style="color: #000000;">InlinePolicyListLambda</code> and its recommended version. 
    <ol> 
     <li>The existing permissions, <code style="color: #000000;">lambda:ListFunctions</code> and <code style="color: #000000;">lambda:ListLayers</code>, are kept in the recommended policy, as well as the existing condition.</li> 
     <li>The permissions in <code style="color: #000000;">lambda:ListAliases</code> and <code style="color: #000000;">lambda:ListFunctionUrlConfigs</code> can be removed because they weren’t used.</li> 
    </ol> 
    <div class="wp-caption aligncenter" id="attachment_35742" style="width: 710px;">
     <img alt="Figure 5: Preview the recommended policy for the existing inline policy InlinePolicyListLambda" class="size-full wp-image-35742" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/09/13/img5-1.png" style="border: 1px solid #bebebe;" width="700" />
     <p class="wp-caption-text" id="caption-attachment-35742">Figure 5: Preview the recommended policy for the existing inline policy InlinePolicyListLambda</p>
    </div> </li> 
  </ol> </li> 
</ol> 
<h4>To download the recommended policies file</h4> 
<ol> 
 <li>Choose <strong>Download JSON</strong> to download the suggested recommendations locally. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_35743" style="width: 750px;">
   <img alt="Figure 6: Download the recommended policies" class="size-full wp-image-35743" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/09/13/img6-2.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-35743">Figure 6: Download the recommended policies</p>
  </div><p></p> </li> 
 <li>A .zip file that contains the recommended policies in JSON format will be downloaded. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_35744" style="width: 609px;">
   <img alt="Figure 7: Downloaded recommended policies as JSON files" class="size-full wp-image-35744" height="192" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/09/13/img7-1.png" style="border: 1px solid #bebebe;" width="599" />
   <p class="wp-caption-text" id="caption-attachment-35744">Figure 7: Downloaded recommended policies as JSON files</p>
  </div><p></p> </li> 
 <li>The content of the <code style="color: #000000;">AmazonS3ReadOnlyAccess-recommended-1-2024-07-22T20/08/44.793Z.json</code> file the same as the recommended policy shown in Figure 4.</li> 
</ol> 
<h3>Generate recommendations for unused permissions using AWS CLI</h3> 
<p>In this section, you will see how to generate recommendations for unused permissions using <a href="https://aws.amazon.com/cli/" rel="noopener" target="_blank">AWS Command Line Interface (AWS CLI)</a>.</p> 
<h4>To list unused permissions findings</h4> 
<ol> 
 <li>Use the following code to refine the results by filtering on the type UnusedPermission and selecting only the <strong>active</strong> findings. Copy the <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/reference-arns.html" rel="noopener" target="_blank">Amazon Resource Name (ARN)</a> of your unused access analyzer and use it to replace the ARN in the following code: 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">aws accessanalyzer list-findings-v2 \
  --analyzer-arn "arn:aws:access-analyzer:&lt;region&gt;:&lt;123456789012&gt;:analyzer/&lt;analyzer_name&gt;" \
  --region &lt;region&gt; \
  --filter '{"findingType": {"eq": ["UnusedPermission"]}, "status": {"eq": ["ACTIVE"]}}'</code></pre> 
  </div> </li> 
 <li>You will obtain results similar to the following. 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">{
    "findings": [
        {
            "analyzedAt": "2024-05-29T07:25:34+00:00",
            "createdAt": "2024-05-23T19:20:59+00:00",
            "id": "0fa3f5a1-bd92-4193-8ca4-aba12cd91370",
            "resource": "arn:aws:iam::123456789012:user/demoIAMUser",
            "resourceType": "AWS::IAM::User",
            "resourceOwnerAccount": "123456789012",
            "status": "ACTIVE",
            "updatedAt": "2024-05-29T07:25:35+00:00",
            "findingType": "UnusedPermission"
        },
        {
            "analyzedAt": "2024-05-29T07:25:34+00:00",
            "createdAt": "2024-05-23T19:20:59+00:00",
            "id": "1e952245-bcf3-48ad-a708-afa460df794b",
            "resource": "arn:aws:iam::123456789012:role/demoIAMRole",
            "resourceType": "AWS::IAM::Role",
            "resourceOwnerAccount": "123456789012",
            "status": "ACTIVE",
            "updatedAt": "2024-05-29T07:25:37+00:00",
            "findingType": "UnusedPermission"
        },
        ...
    ]
}</code></pre> 
  </div> </li> 
</ol> 
<h4>To generate unused permissions finding recommendations</h4> 
<p>After you have a list of findings for unused permissions, you can generate finding recommendations.</p> 
<ol> 
 <li>Run the following, replacing the analyzer ARN and the finding ID to generate the suggested recommendations. 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">aws accessanalyzer generate-finding-recommendation \
  --analyzer-arn "arn:aws:access-analyzer:&lt;region&gt;:&lt;123456789012&gt;:analyzer/&lt;analyzer_name&gt;" \
  --region &lt;region&gt; \
  --id "ab123456-bcd0-78ab-a012-afa460df794b"</code></pre> 
  </div> </li> 
 <li>You will get an <strong>empty</strong> response if your command ran successfully. The process is running in the background.</li> 
</ol> 
<h4>To obtain the generated recommendations</h4> 
<p>After the recommendations are generated, you need to make a separate API call to view the recommendations details.</p> 
<ol> 
 <li>The following command returns the recommended remediation. 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">aws accessanalyzer get-finding-recommendation \
  --analyzer-arn "arn:aws:access-analyzer:&lt;region&gt;:&lt;123456789012&gt;:analyzer/&lt;analyzer_name&gt;" \
  --region &lt;region&gt; \
  --id "ab123456-bcd0-78ab-a012-afa460df794b"</code></pre> 
  </div> </li> 
 <li>This command provides the following results. For more information about the meaning and structure of the recommendations, see <a href="#Anatomy_recommendation">Anatomy of a recommendation</a> later in this post.<br /> 
  <blockquote>
   <p><strong>Note</strong>: The recommendations consider AWS managed policies, customer managed policies, and inline policies. The IAM conditions in the initial policy are maintained in the recommendations if the actions they’re related to are used.</p>
  </blockquote> <p>The remediations suggested are to do the following:</p> 
  <ol> 
   <li>Detach <code style="color: #000000;">AmazonBedrockReadOnly</code> policy because it is unused: DETACH_POLICY</li> 
   <li>Create a new recommended policy with scoped down permissions from the managed policy <code style="color: #000000;">AmazonS3ReadOnlyAccess:</code> CREATE_POLICY</li> 
   <li>Detach <code style="color: #000000;">AmazonS3ReadOnlyAccess:</code> DETACH_POLICY</li> 
   <li>Embed a new recommended policy with scoped down permissions from the inline policy: CREATE_POLICY</li> 
   <li>Delete the inline policy.</li> 
  </ol> 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">{
    "recommendedSteps": [
        {
            "unusedPermissionsRecommendedStep": {
                "policyUpdatedAt": "2023-12-06T15:48:19+00:00",
                "recommendedAction": <span style="font-weight: 900;">"</span><span style="font-weight: 900;">DETACH_POLICY</span><span style="font-weight: 900;">"</span>,
                "existingPolicyId": "arn:aws:iam::aws:policy/AmazonBedrockReadOnly"
            }
        },
        {
            "unusedPermissionsRecommendedStep": {
                "policyUpdatedAt": "2023-08-10T21:31:39+00:00",
                "recommendedAction": <span style="font-weight: 900;">"CREATE_POLICY"</span>,
                "recommendedPolicy": "{\"Version\":\"2012-10-17\",\"Statement\":[{\"Effect\":\"Allow\",\"Action\":[\"s3:GetBucketObjectLockConfiguration\",\"s3:GetBucketOwnershipControls\",\"s3:GetBucketTagging\",\"s3:GetBucketVersioning\",\"s3:GetJobTagging\",\"s3:GetObject\",\"s3:GetObjectAcl\",\"s3:GetObjectLegalHold\",\"s3:GetObjectRetention\",\"s3:GetObjectTagging\",\"s3:GetObjectTorrent\",\"s3:GetObjectVersion*\",\"s3:GetStorage*\",\"s3:ListAllMyBuckets\",\"s3:ListBucket\",\"s3:ListBucketVersions\",\"s3:ListMultipartUploadParts\",\"s3:ListStorageLensGroups\",\"s3:ListTagsForResource\"],\"Resource\":\"*\"}]}",
                "existingPolicyId": "arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess"
            }
        },
        {
            "unusedPermissionsRecommendedStep": {
                "policyUpdatedAt": "2023-08-10T21:31:39+00:00",
                "recommendedAction": <span style="font-weight: 900;">"DETACH_POLICY"</span>,
                "existingPolicyId": "arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess"
            }
        },
        {
            "unusedPermissionsRecommendedStep": {
                "recommendedAction": <span style="font-weight: 900;">"CREATE_POLICY"</span>,
                "recommendedPolicy": "{\"Version\":\"2012-10-17\",\"Statement\":[{\"Sid\":\"InlinePolicyLambda\",\"Effect\":\"Allow\",\"Action\":[\"lambda:ListFunctions\",\"lambda:ListLayers\"],\"Resource\":\"*\",\"Condition\":{\"NotIpAddress\":{\"aws:SourceIp\":\"1.100.150.200/32\"}}}]}",
                "existingPolicyId": "InlinePolicyListLambda"
            }
        },
        {
            "unusedPermissionsRecommendedStep": {
                "recommendedAction": <span style="font-weight: 900;">"DETACH_POLICY"</span>,
                "existingPolicyId": "InlinePolicyListLambda"
            }
        }
    ],
    "status": "SUCCEEDED",
    "error": null,
    "completedAt": "2024-07-22T20:40:58.413698+00:00",
    "recommendationType": "UNUSED_PERMISSION_RECOMMENDATION",
    "resourceArn": "arn:aws:iam::123456789012:role/IAMRole_IA2_Blog_EC2Role",
    "startedAt": "2024-07-22T20:40:54+00:00"
}</code></pre> 
  </div> </li> 
</ol> 
<h3>Generate recommendations for unused permissions using EventBridge and AWS API</h3> 
<p>We have described how to use AWS CLI and the console to find unused permissions findings and to generate recommendations.</p> 
<p>In this section, we show you how to use an <a href="https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-rules.html" rel="noopener" target="_blank">Amazon EventBridge rule</a> to find the active unused permissions findings from IAM Access Analyzer. Then we show you how to generate recommendations using two <a href="https://docs.aws.amazon.com/access-analyzer/latest/APIReference/API_Operations.html" rel="noopener" target="_blank">IAM Access Analyzer APIs</a> to <a href="https://docs.aws.amazon.com/access-analyzer/latest/APIReference/API_GenerateFindingRecommendation.html">generate the finding recommendations</a> and <a href="https://docs.aws.amazon.com/access-analyzer/latest/APIReference/API_GetFindingRecommendation.html" rel="noopener" target="_blank">get the finding recommendations</a>.</p> 
<h4>To create an EventBridge rule to detect unused permissions findings</h4> 
<p>Create an <a href="https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-create-rule.html" rel="noopener" target="_blank">EventBridge rule</a> to detect new unused permissions findings from IAM Access Analyzer.</p> 
<ol> 
 <li>Go to the Amazon EventBridge console.</li> 
 <li>Choose <strong>Rules</strong>, and then choose <strong>Create rule</strong>.</li> 
 <li>Enter a name for your rule. Leave the <strong>Event bus value</strong> as the default.</li> 
 <li>Under <strong>Rule type</strong>, select <strong>Rule with an event pattern</strong>.</li> 
 <li>In the <strong>Event Source</strong> section, select <strong>AWS events or EventBridge partner events</strong>.</li> 
 <li>For <strong>Creation method</strong>, select <strong>Use pattern form</strong>.</li> 
 <li>Under <strong>Event pattern</strong>: 
  <ol> 
   <li>For <strong>Event source</strong>, select <strong>AWS services</strong>.</li> 
   <li>For <strong>AWS service</strong>, select <strong>Access Analyzer</strong>.</li> 
   <li>For <strong>Event type</strong>, select <strong>Unused Access Finding for IAM entities</strong>.</li> 
  </ol> 
  <blockquote>
   <p><strong><u>Note</u></strong>: There is no event for generated recommendations, only for unused access findings.</p>
  </blockquote> <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_35748" style="width: 750px;">
   <img alt="Figure 8: Listing unused permissions by filtering events using an EventBridge rule" class="size-full wp-image-35748" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/09/13/img8.jpg" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-35748">Figure 8: Listing unused permissions by filtering events using an EventBridge rule</p>
  </div><p></p> </li> 
 <li>Configure the <strong>Event pattern</strong> by changing the default values to the following: 
  <ol> 
   <li><code style="color: #000000;">resources</code>: Enter the ARN of your unused access analyzer.</li> 
   <li><code style="color: #000000;">status</code>: ACTIVE indicates that you are only looking for active findings.</li> 
   <li><code style="color: #000000;">findingType</code>: UnusedPermission.</li> 
  </ol> </li> 
 <li>You can select a target <a href="https://aws.amazon.com/sns/" rel="noopener" target="_blank">Amazon Simple Notification Service (Amazon SNS)</a> to be notified of new active findings for a specific analyzer for unused permissions.</li> 
</ol> 
<h4>To generate recommendations for unused permissions using the IAM Access Analyzer API</h4> 
<p>The findings are generated on-demand. For that purpose, IAM Access Analyzer API&nbsp;<a href="https://docs.aws.amazon.com/access-analyzer/latest/APIReference/API_GenerateFindingRecommendation.html" rel="noopener" target="_blank">GenerateFindingRecommendation</a> can be called with two parameters: the ARN of the analyzer and the finding ID.</p> 
<ol> 
 <li>You can use <a href="https://aws.amazon.com/sdk-for-python/" rel="noopener" target="_blank">AWS Software Development Kit (SDK) for Python</a>(<a href="https://github.com/boto/boto3" rel="noopener" target="_blank">boto3</a>) for the API call.</li> 
 <li>Run the call as follows: 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">ia2_client = boto3.client('accessanalyzer')
response = ia2_client.generate_finding_recommendation(
    analyzerArn=analyzer,
    id=findingId
    )</code></pre> 
  </div> </li> 
</ol> 
<h4>To obtain the finding recommendations</h4> 
<ol> 
 <li>After the recommendations are generated, they can be obtained by calling the API&nbsp;<a href="https://docs.aws.amazon.com/access-analyzer/latest/APIReference/API_GetFindingRecommendation.html">GetFindingRecommendation</a> with the same parameters: the ARN of the analyzer and the finding ID.</li> 
 <li>Use AWS SDK for Python (<a href="https://aws.amazon.com/fr/sdk-for-python/" rel="noopener" target="_blank">boto3</a>) for the API call as follows: 
  <div class="hide-language"> 
   <pre class="unlimited-height-code">ia2_client = boto3.client('accessanalyzer')
response = ia2_client.get_finding_recommendation(
    analyzerArn=analyzer,
    id=findingId
)<code class="lang-text"></code></pre> 
  </div> </li> 
</ol> 
<h2>Remediate based on the generated recommendations</h2> 
<p>The recommendations are generated as actionable guidance that you can follow. They propose new IAM policies that exclude the unused actions, helping you rightsize your permissions.</p> 
<h3 id="Anatomy_recommendation">Anatomy of a recommendation</h3> 
<p>The recommendations are usually presented in the following way:</p> 
<ul> 
 <li><strong>Date and time</strong>: <code style="color: #000000;">startedAt, completedAt</code>. Respectively when the API call was made and when the analysis was completed and the results were provided.</li> 
 <li><strong>Resource ARN</strong>: The ARN of the resource being analyzed.</li> 
 <li><strong>Recommended steps</strong>: The recommended steps, such as creating a new policy based on the actions used and detaching the existing policy.</li> 
 <li><strong>Recommendation type</strong>:&nbsp;UNUSED_PERMISSION_RECOMMENDATION.</li> 
 <li><strong>Status</strong>: The status of retrieving the finding recommendation. The <a href="https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/accessanalyzer/paginator/GetFindingRecommendation.html" rel="noopener" target="_blank">status</a> values include SUCCEEDED, FAILED, and IN_PROGRESS.</li> 
</ul> 
<p>For more information about the structure of recommendations, see the output section of <a href="https://docs.aws.amazon.com/cli/latest/reference/accessanalyzer/get-finding-recommendation.html" rel="noopener" target="_blank">get-finding-recommendation</a>.</p> 
<h3>Recommended policy review</h3> 
<p>You must review the recommended policy. The recommended actions depend on the original policy. The original policy will be one of the following:</p> 
<ul> 
 <li><strong>An AWS managed policy</strong>: You need to create a new IAM policy using <code style="color: #000000;">recommendedPolicy</code>. Attach this newly created policy to your IAM role. Then detach the former policy.</li> 
 <li><strong>A customer managed policy or an inline policy</strong>: Review the policy, verify its scope, consider how often it’s attached to other principals (customer managed policy only), and when you are confident to proceed, use the recommended policy to create a new policy and detach the former policy.</li> 
</ul> 
<h3>Use cases to consider when reviewing recommendations</h3> 
<p>During your review process, keep in mind that the unused actions are determined based on the time defined in your tracking period. The following are some use cases you might have where a necessary role or action might be identified as unused (this is not an exhaustive list of use cases). It’s important to review the recommendations based on your business needs. You can also archive some findings related to the use cases such as the ones that follow:</p> 
<ul> 
 <li><strong>Backup activities</strong>: If your tracking period is 28 days and you have a specific role for your backup activities running at the end of each month, you might discover that after 29 days some of the permissions for that backup role are identified as unused.</li> 
 <li><strong>IAM permissions associated to an infrastructure as code deployment pipeline</strong>: You should also consider the permissions associated to specific IAM roles such an IAM for infrastructure as code (IaC) deployment pipeline. Your pipeline can be used to deploy <a href="https://aws.amazon.com/s3" rel="noopener" target="_blank">Amazon Simple Storage Service (Amazon S3)</a> buckets based on your internal guidelines. After deployment is complete, the pipeline permissions can become unused after your tracking period, but removing those unused permissions can prevent you from updating your S3 buckets configuration or from deleting it.</li> 
 <li><strong>IAM roles associated with disaster recovery activities</strong>: While it’s recommended to have a disaster recovery plan, the IAM roles used to perform those activities might be flagged by IAM Access Analyzer for having unused permissions or being unused roles.</li> 
</ul> 
<h3>To apply the suggested recommendations</h3> 
<p>Of the three original policies attached to <code style="color: #000000;">IAMRole_IA2_Blog_EC2Role</code>, <code style="color: #000000;">AmazonBedrockReadOnly</code> can be detached and <code style="color: #000000;">AmazonS3ReadOnlyAccess</code> and <code style="color: #000000;">InlinePolicyListLambda</code> can be refined.</p> 
<ol> 
 <li><strong>Detach</strong> <code style="color: #000000;">AmazonBedrockReadOnly</code> <p>No permissions are used in this policy, and the recommended action is to detach it from your IAM role. To detach it, you can use the <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_manage-attach-detach.html#remove-policies-console" rel="noopener" target="_blank">IAM console</a>, the <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_manage-attach-detach.html#remove-policy-cli" rel="noopener" target="_blank">AWS CLI</a>, or the <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_manage-attach-detach.html#remove-policy-api" rel="noopener" target="_blank">AWS API</a>.</p> </li> 
 <li>Create a new policy called <code style="color: #000000;">AmazonS3ReadOnlyAccess-recommended</code> and detach <code style="color: #000000;">AmazonS3ReadOnlyAccess</code>. <p>The unused access analyzer has identified unused permissions in the managed policy <a href="https://docs.aws.amazon.com/aws-managed-policy/latest/reference/AmazonS3ReadOnlyAccess.html" rel="noopener" target="_blank">AmazonS3ReadOnlyAccess</a> and proposed a new policy <code style="color: #000000;">AmazonS3ReadOnlyAccess-recommended</code> that contains only the used actions. This is a step towards least privilege because the unused actions can be removed by using the recommended policy.</p> 
  <ol> 
   <li><a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_create.html" rel="noopener" target="_blank">Create a new IAM policy</a> named <strong>AmazonS3ReadOnlyAccess-recommended</strong> that contains only the following recommended policy or one based on the downloaded JSON file. 
    <div class="hide-language"> 
     <pre class="unlimited-height-code"><code class="lang-text">{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:getbuckettagging",
        "s3:getjobtagging",
        "s3:getobject",
        "s3:getobjectacl",
        "s3:getobjectlegalhold",
        "s3:getobjectretention",
        "s3:getobjecttagging",
        "s3:getobjecttorrent",
        "s3:getobjectversion",
        "s3:getobjectversionacl",
        "s3:getobjectversionattributes",
        "s3:getobjectversionforreplication",
        "s3:getobjectversiontagging",
        "s3:getobjectversiontorrent",
        "s3:getstoragelensconfigurationtagging",
        "s3:getstoragelensgroup",
        "s3:listbucket",
        "s3:listbucketversions",
        "s3:listmultipartuploadparts",
        "s3:liststoragelensgroups",
        "s3:listtagsforresource"
      ],
      "Resource": "*"
    }
  ]
}</code></pre> 
    </div> </li> 
   <li><strong>Detach</strong> the managed policy <code style="color: #000000;">AmazonS3ReadOnlyAccess</code>.</li> 
  </ol> </li> 
 <li>Embed a new inline policy <code style="color: #000000;">InlinePolicyListLambda-recommended</code> and delete <code style="color: #000000;">InlinePolicyListLambda</code>. This inline policy lists <a href="https://aws.amazon.com/lambda">AWS Lambda</a> aliases, functions, layers, and function URLs only when coming from a specific source IP address. 
  <ol> 
   <li><strong>Embed</strong> the recommended inline policy. <p>The recommended policy follows. You can embed an inline policy for the IAM role using the <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_manage-attach-detach.html#add-policies-console" rel="noopener" target="_blank">console</a>, <a href="https://docs.aws.amazon.com/cli/latest/reference/iam/put-role-policy.html" rel="noopener" target="_blank">AWS CLI</a>, or the <a href="https://docs.aws.amazon.com/IAM/latest/APIReference/API_PutRolePolicy.html" rel="noopener" target="_blank">AWS API PutRolePolicy</a>.</p> 
    <div class="hide-language"> 
     <pre class="unlimited-height-code"><code class="lang-text">{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "InlinePolicyLambda",
      "Effect": "Allow",
      "Action": [
        "lambda:ListFunctions",
        "lambda:ListLayers"
      ],
      "Resource": "*",
      "Condition": {
        "NotIpAddress": {
          "aws:SourceIp": "1.100.150.200/32"
        }
      }
    }
  ]
}</code></pre> 
    </div> </li> 
   <li><strong>Delete</strong> the inline policy.</li> 
  </ol> </li> 
 <li>After updating the policies based on the <strong>Recommended policy</strong> proposed, the finding <strong>Status</strong> will change from <strong>Active</strong> to <strong>Resolved</strong>. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_35754" style="width: 750px;">
   <img alt="Figure 9: The finding is resolved" class="size-full wp-image-35754" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/09/16/img9-1.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-35754">Figure 9: The finding is resolved</p>
  </div><p></p> </li> 
</ol> 
<h2>Pricing</h2> 
<p>There is no additional pricing for using the prescriptive recommendations after you have enabled unused access findings.</p> 
<h2>Conclusion</h2> 
<p>As a developer writing policies, you can use the actionable guidance provided in recommendations to continually rightsize your policies to include only the roles and actions you need. You can export the recommendations through the console or set up automated workflows to notify your developers about new IAM Access Analyzer findings.</p> 
<p>This new IAM Access Analyzer unused access recommendations feature streamlines the process towards least privilege by selecting the permissions that are used and retaining the resource and condition context from existing policies. It saves an impressive amount of time by the actions used by your principals and guiding you to refine them.</p> 
<p>By using the IAM Access Analyzer findings and access recommendations, you can quickly see how to refine the permissions granted. We have shown in this blog post how to generate&nbsp;prescriptive recommendations with actionable guidance for unused permissions using AWS CLI, API calls, and the console.</p> 
<ul> 
 <li>To learn more about IAM Access Analyzer Unused Access, see <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-findings.html" rel="noopener" target="_blank">Findings for external and unused access</a>.</li> 
 <li>To learn about the creation of unused access findings, see <a href="https://aws.amazon.com/blogs/security/iam-access-analyzer-simplifies-inspection-of-unused-access-in-your-organization/" rel="noopener" target="_blank">IAM Access Analyzer simplifies inspection of unused access in your organization</a>.</li> 
 <li>To learn about the strategies for achieving least privilege at scale, see Strategies for achieving least privilege at scale <a href="https://aws.amazon.com/blogs/security/strategies-for-achieving-least-privilege-at-scale-part-1/" rel="noopener" target="_blank">Part 1</a> and <a href="https://aws.amazon.com/blogs/security/strategies-for-achieving-least-privilege-at-scale-part-2/">Part 2</a>.</li> 
 <li>To practice, use the <a href="https://catalog.workshops.aws/refining-iam-permissions-like-a-pro/en-US" rel="noopener" target="_blank">Refining IAM Permissions Like A Pro the</a> workshop.</li> 
</ul> 
<p>&nbsp;<br />If you have feedback about this post, submit comments in the<strong> Comments</strong> section below. If you have questions about this post, <a href="https://console.aws.amazon.com/support/home" rel="noopener noreferrer" target="_blank">contact AWS Support</a>.</p> 
<footer> 
 <div class="blog-author-box"> 
  <img alt="P. Stéphanie Mbappe" class="alignleft size-full wp-image-33350" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2023/05/26/StephMbappe_288x384.png" style="margin-left: 12px; margin-right: 18px; margin-top: 12px; margin-bottom: 6px; width: 93.750px; height: 125px;" />
  <p></p> 
  <p><span class="lb-h4" style="line-height: 0.5; padding-top: 0px;">P. Stéphanie Mbappe</span><br /> Stéphanie is a Security Consultant with Amazon Web Services. She delights in assisting her customers at any step of their security journey. Stéphanie enjoys learning, designing new solutions, and sharing her knowledge with others.</p> 
  <p></p>
 </div> 
 <div class="blog-author-box"> 
  <img alt="Mathangi Ramesh" class="alignleft size-full wp-image-33350" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2019/11/14/Mathangi-Ramesh.jpg" style="margin-left: 12px; margin-right: 18px; margin-top: 12px; margin-bottom: 6px; width: 93.750px; height: 125px;" />
  <p></p> 
  <p><span class="lb-h4" style="line-height: 0.5; padding-top: 0px;">Mathangi Ramesh</span><br /> Mathangi is the product manager for AWS Identity and Access Management. She enjoys talking to customers and working with data to solve problems. Outside of work, Mathangi is a fitness enthusiast and a Bharatanatyam dancer. She holds an MBA degree from Carnegie Mellon University.</p> 
  <p></p>
 </div> 
</footer>
