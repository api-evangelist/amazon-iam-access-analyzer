---
title: "How to visualize IAM Access Analyzer policy validation findings with QuickSight"
url: "https://aws.amazon.com/blogs/security/how-to-visualize-iam-access-analyzer-policy-validation-findings-with-quicksight/"
date: "Mon, 13 Feb 2023 20:11:31 +0000"
author: "Mostefa Brougui"
feed_url: "https://aws.amazon.com/blogs/security/tag/iam-access-analyzer/feed/"
---
<p>In this blog post, we show you how to create an <a href="https://aws.amazon.com/quicksight" rel="noopener" target="_blank">Amazon QuickSight</a> dashboard to visualize the policy validation findings from <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/what-is-access-analyzer.html" rel="noopener" target="_blank">AWS Identity and Access Management (IAM) Access Analyzer</a>. You can use this dashboard to better understand your policies and how to achieve <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#grant-least-privilege" rel="noopener" target="_blank">least privilege</a> by periodically validating your <a href="https://aws.amazon.com/iam/" rel="noopener" target="_blank">IAM</a> roles against IAM best practices. This blog post walks you through the deployment for a multi-account environment using <a href="https://aws.amazon.com/organizations/" rel="noopener" target="_blank">AWS Organizations</a>.</p> 
<p>Achieving least privilege is a continuous cycle to grant only the permissions that your users and systems require. To achieve least privilege, you start by setting fine-grained permissions. Then, you verify that the existing access meets your intent. Finally, you refine permissions by removing unused access. To learn more, see <a href="https://aws.amazon.com/blogs/security/iam-access-analyzer-makes-it-easier-to-implement-least-privilege-permissions-by-generating-iam-policies-based-on-access-activity/" rel="noopener" target="_blank">IAM Access Analyzer makes it easier to implement least privilege permissions by generating IAM policies based on access activity</a>.</p> 
<p><a href="https://aws.amazon.com/blogs/aws/iam-access-analyzer-update-policy-validation/" rel="noopener" target="_blank">Policy validation</a> is a feature of IAM Access Analyzer that guides you to author and validate secure and functional policies with more than 100 policy checks. You can use these checks when creating new policies or to validate existing policies. To learn how to use IAM Access Analyzer policy validation APIs when creating new policies, see <a href="https://aws.amazon.com/blogs/security/validate-iam-policies-in-cloudformation-templates-using-iam-access-analyzer/" rel="noopener" target="_blank">Validate IAM policies in CloudFormation templates using IAM Access Analyzer</a>. In this post, we focus on how to validate existing IAM policies.</p> 
<h2>Approach to visualize IAM Access Analyzer findings</h2> 
<p>As shown in Figure 1, there are four high-level steps to build the visualization.</p> 
<div class="wp-caption aligncenter" id="attachment_28510" style="width: 730px;">
 <img alt="Figure 1: Steps to visualize IAM Access Analyzer findings" class="size-large wp-image-28510" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/02/08/img1-1-1024x88.png" width="760" />
 <p class="wp-caption-text" id="caption-attachment-28510">Figure 1: Steps to visualize IAM Access Analyzer findings</p>
</div> 
<ol> 
 <li><span style="font-size: 1.2em;">Collect IAM policies</span> <p>To validate your IAM policies with IAM Access Analyzer in your organization, start by periodically sending the content of your IAM policies (inline and customer-managed) to a central account, such as your <a href="https://docs.aws.amazon.com/prescriptive-guidance/latest/security-reference-architecture/security-tooling.html" rel="noopener" target="_blank">Security Tooling account</a>.</p> </li> 
 <li><span style="font-size: 1.2em;">Validate IAM policies</span> <p>After you collect the IAM policies in a central account, run an IAM Access Analyzer ValidatePolicy API call on each policy. The API calls return a list of findings. The findings can help you identify issues, provide actionable recommendations to resolve the issues, and enable you to author functional policies that can meet security best practices. The findings are stored in an <a href="https://aws.amazon.com/s3/" rel="noopener" target="_blank">Amazon Simple Storage Service (Amazon S3)</a> bucket. To learn about different findings, see <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-reference-policy-checks.html" rel="noopener" target="_blank">Access Analyzer policy check reference</a>.</p> </li> 
 <li><span style="font-size: 1.2em;">Visualize findings</span> <p>IAM Access Analyzer policy validation findings are stored centrally in an S3 bucket. The S3 bucket is owned by the central (hub) account of your choosing. You can use <a href="https://aws.amazon.com/athena/" rel="noopener" target="_blank">Amazon Athena</a> to query the findings from the S3 bucket, and then create a QuickSight analysis to visualize the findings.</p> </li> 
 <li><span style="font-size: 1.2em;">Publish dashboards</span> <p>Finally, you can publish a shareable QuickSight dashboard. Figure 2 shows an example of the dashboard.</p> 
  <div class="wp-caption aligncenter" id="attachment_28474" style="width: 690px;">
   <img alt="Figure 2: Dashboard overview" class="size-large wp-image-28474" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/02/06/img2-1024x742.png" style="border: 1px solid #bebebe;" width="680" />
   <p class="wp-caption-text" id="caption-attachment-28474">Figure 2: Dashboard overview</p>
  </div> </li> 
</ol> 
<h2>Design overview</h2> 
<p>This implementation is a serverless job initiated by <a href="https://aws.amazon.com/eventbridge/" rel="noopener" target="_blank">Amazon EventBridge</a> rules. It collects IAM policies into a hub account (such as your Security Tooling account), validates the policies, stores the validation results in an S3 bucket, and uses Athena to query the findings and QuickSight to visualize them. Figure 3 gives a design overview of our implementation.</p> 
<div class="wp-caption aligncenter" id="attachment_28476" style="width: 770px;">
 <img alt="Figure 3: Design overview of the implementation" class="size-large wp-image-28476" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/02/06/img3-1-1024x465.png" style="border: 1px solid #bebebe;" width="760" />
 <p class="wp-caption-text" id="caption-attachment-28476">Figure 3: Design overview of the implementation</p>
</div> 
<p>As shown in Figure 3, the implementation includes the following steps:</p> 
<ol> 
 <li>A time-based rule is set to run daily. The rule triggers an <a href="http://aws.amazon.com/lambda" rel="noopener" target="_blank">AWS Lambda</a> function that lists the IAM policies of the AWS account it is running in.</li> 
 <li>For each IAM policy, the function sends a message to an <a href="http://aws.amazon.com/sqs" rel="noopener" target="_blank">Amazon Simple Queue Service (Amazon SQS)</a> queue. The message contains the IAM policy <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_identifiers.html#identifiers-arns" rel="noopener" target="_blank">Amazon Resource Name (ARN)</a>, and the policy document.</li> 
 <li>When new messages are received, the Amazon SQS queue initiates the second Lambda function. For each message, the Lambda function extracts the policy document and validates it by using the IAM Access Analyzer <span style="font-family: courier;">ValidatePolicy</span> API call.</li> 
 <li>The Lambda function stores validation results in an S3 bucket.</li> 
 <li>An <a href="https://aws.amazon.com/glue" rel="noopener" target="_blank">AWS Glue</a> table contains the schema for the IAM Access Analyzer findings. Athena natively uses the <a href="https://docs.aws.amazon.com/prescriptive-guidance/latest/serverless-etl-aws-glue/aws-glue-data-catalog.html" rel="noopener" target="_blank">AWS Glue Data Catalog</a>.</li> 
 <li>Athena queries the findings stored in the S3 bucket.</li> 
 <li>QuickSight uses Athena as a data source to visualize IAM Access Analyzer findings.</li> 
</ol> 
<h3>Benefits of the implementation</h3> 
<p>By implementing this solution, you can achieve the following benefits:</p> 
<ul> 
 <li>Store your IAM Access Analyzer policy validation results in a scalable and cost-effective manner with Amazon S3.</li> 
 <li>Add scalability and fault tolerance to your validation workflow with Amazon SQS.</li> 
 <li>Partition your evaluation results in Athena and restrict the amount of data scanned by each query, helping to improve performance and reduce cost.</li> 
 <li>Gain insights from IAM Access Analyzer policy validation findings with QuickSight dashboards. You can use the dashboard to identify IAM policies that don’t comply with AWS best practices and then take action to correct them.</li> 
</ul> 
<h3>Prerequisites</h3> 
<p>Before you implement the solution, make sure you’ve completed the following steps:</p> 
<ol> 
 <li>Install a Git client, such as <a href="https://desktop.github.com/" rel="noopener" target="_blank">GitHub Desktop</a>.</li> 
 <li>Install the <a href="http://aws.amazon.com/cli" rel="noopener" target="_blank">AWS Command Line Interface (AWS CLI)</a>. For instructions, see <a href="https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html" rel="noopener" target="_blank">Installing or updating the latest version of the AWS CLI</a>.</li> 
 <li>If you plan to deploy the implementation in a multi-account environment using Organizations,&nbsp;<a href="https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_org_support-all-features.html" rel="noopener" target="_blank">enable all features</a> and <a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/stacksets-orgs-enable-trusted-access.html" rel="noopener" target="_blank">enable trusted access</a> with Organizations to operate a service-managed stack set.</li> 
 <li>Get a QuickSight subscription to the Enterprise edition. When you first subscribe to the Enterprise edition, you get a free trial for four users for 30 days. Trial authors are automatically converted to month-to-month subscription upon trial expiry. For more details, see&nbsp;<a href="https://docs.aws.amazon.com/quicksight/latest/user/signing-up.html" rel="noopener" target="_blank">Signing up for an Amazon QuickSight subscription</a>, <a href="https://docs.aws.amazon.com/quicksight/latest/user/upgrading-subscription.html" rel="noopener" target="_blank">Amazon QuickSight Enterprise edition</a> and the <a href="https://calculator.aws/#/createCalculator/QuickSight" rel="noopener" target="_blank">Amazon QuickSight Pricing Calculator</a>.</li> 
</ol> 
<blockquote>
 <p><strong>Note</strong>: This implementation works in accounts that don’t have <a href="https://aws.amazon.com/lake-formation/" rel="noopener" target="_blank">AWS Lake Formation</a> enabled. If Lake Formation is enabled in your account, you might need to grant Lake Formation permissions in addition to the implementation IAM permissions. For details, see <a href="https://docs.aws.amazon.com/lake-formation/latest/dg/access-control-overview.html" rel="noopener" target="_blank">Lake Formation access control overview</a>.</p>
</blockquote> 
<h3>Walkthrough</h3> 
<p>In this section, we will show you how to deploy an <a href="https://aws.amazon.com/cloudformation/" rel="noopener" target="_blank">AWS CloudFormation</a> template to your central account (such as your Security Tooling account), which is the hub for IAM Access Analyzer findings. The central account collects, validates, and visualizes your findings.</p> 
<h4>To deploy the implementation to your multi-account environment</h4> 
<ol> 
 <li>Deploy the CloudFormation stack to your central account.<br /> 
  <blockquote>
   <p><strong>Important</strong>: Do not deploy the template to the organization’s management account; see <a href="https://docs.aws.amazon.com/whitepapers/latest/organizing-your-aws-environment/design-principles-for-organizing-your-aws-accounts.html#avoid-deploying-workloads-to-the-organizations-management-account" rel="noopener" target="_blank">design principles for organizing your AWS accounts</a>. You can choose the <a href="https://docs.aws.amazon.com/prescriptive-guidance/latest/security-reference-architecture/security-tooling.html" rel="noopener" target="_blank">Security Tooling account</a> as a hub account.</p>
  </blockquote> <p>In your central account, run the following commands in a terminal. These commands clone the GitHub repository and deploy the CloudFormation stack to your central account.</p> 
  <div class="hide-language"> 
   <div class="code-toolbar"> 
    <pre class="unlimited-height-code language-bash"><code class=" language-bash"><span style="font-style: italic;"># A) Clone the repository</span>
git clone https://github.com/aws-samples/visualize-iam-access-analyzer-policy-validation-findings.git<br />&nbsp;
<span style="font-style: italic;"># B) Switch to the repository's directory</span>
cd visualize-iam-access-analyzer-policy-validation-findings<br />&nbsp;
<span style="font-style: italic;"># C) Deploy the CloudFormation stack to your central security account (hub). For</span> <span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;AWSRegion&gt;</span> <span style="font-style: italic;">enter your AWS Region without quotes.</span>
make deploy-hub aws-region=<span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;AWSRegion&gt;</span></code></pre> 
   </div> 
  </div> <p>If you want to send IAM policies from other member accounts to your central account, you will need to make note of the CloudFormation stack outputs for <span style="font-family: courier;">SQSQueueUrl</span> and <span style="font-family: courier;">KMSKeyArn</span> when the deployment is complete.</p> 
  <div class="hide-language"> 
   <div class="code-toolbar"> 
    <pre class="unlimited-height-code language-bash"><code class=" language-bash">make describe-hub-outputs aws-region=<span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;AWSRegion&gt;</span></code></pre> 
   </div> 
  </div> </li> 
 <li>Switch to your organization’s management account and deploy the stack sets to the member accounts. For <span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;SQSQueueUrl&gt;</span> and <span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;KMSKeyArn&gt;</span>, use the values from the previous step. 
  <div class="hide-language"> 
   <div class="code-toolbar"> 
    <pre class="unlimited-height-code language-bash"><code class=" language-bash"><span style="font-style: italic;"># Create a CloudFormation stack set to deploy the resources to the member accounts.</span><br />&nbsp;
make deploy-members SQSQueueUrl=<span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;SQSQueueUrl&gt;</span> KMSKeyArn=<span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;KMSKeyArn&lt;</span> aws-region=<span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;AWSRegion&gt;</span></code></pre> 
   </div> 
  </div> </li> 
</ol> 
<h4>To deploy the QuickSight dashboard to your central account</h4> 
<ol> 
 <li>Make sure that QuickSight is using the IAM role <span style="font-family: courier;">aws-quicksight-service-role</span>. 
  <ol> 
   <li>In QuickSight, in the navigation bar at the top right, choose your account (indicated by a person icon) and then choose <strong>Manage QuickSight</strong>.</li> 
   <li>On the <strong>Manage QuickSight</strong> page, in the menu at the left, choose <strong>Security &amp; Permissions</strong>.</li> 
   <li>On the <strong>Security &amp; Permissions</strong> page, under <strong>QuickSight access to AWS services</strong>, choose <strong>Manage</strong>.</li> 
   <li>For <strong>IAM role</strong>, choose <strong>Use an existing role</strong>, and then do one of the following: 
    <ul> 
     <li>If you see a list of existing IAM roles, choose the role <p><span style="font-family: courier;">arn:aws:iam::</span><span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;account-id&gt;</span><span style="font-family: courier;">:role/service-role/aws-quicksight-service-role.</span></p> </li> 
     <li>If you don’t see a list of existing IAM roles, enter the IAM ARN for the role in the following format: <p><span style="font-family: courier;">arn:aws:iam::</span><span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;account-id&gt;</span><span style="font-family: courier;">:role/service-role/aws-quicksight-service-role.</span></p> </li> 
    </ul> </li> 
   <li>Choose <strong>Save</strong>.</li> 
  </ol> </li> 
 <li>Retrieve the QuickSight users. 
  <div class="hide-language"> 
   <div class="code-toolbar"> 
    <pre class="unlimited-height-code language-bash"><code class=" language-bash"># <span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;aws-region&gt;</span> your Quicksight main Region, for example eu-west-1
# <span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;account-id&gt;</span> The ID of your account, for example 123456789012
# <span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;namespace-name&gt;</span> Quicksight namespace, for example default.
# You can list the namespaces by using <span style="font-style: italic;">aws quicksight list-namespaces --aws-account-id</span> <span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;account-id&gt;</span><br />&nbsp;
aws quicksight list-users --region <span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;aws-region&gt;</span> --aws-account-id <span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;account-id&gt;</span> --namespace <span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;namespace-name&gt;</span></code></pre> 
   </div> 
  </div> </li> 
 <li>Make a note of the user’s ARN that you want to grant permissions to list, describe, or update the QuickSight dashboard. This information is found in the <span style="font-family: courier;">arn</span> element. For example, <span style="font-family: courier;">arn:aws:quicksight:us-east-1:111122223333:user/default/User1</span></li> 
 <li>To launch the deployment stack for the QuickSight dashboard, run the following command. Replace <span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;quicksight-user-arn&gt;</span> with the user’s ARN from the previous step. 
  <div class="hide-language"> 
   <div class="code-toolbar"> 
    <pre class="unlimited-height-code language-bash"><code class=" language-bash">make deploy-dashboard-hub aws-region=<span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;AWSRegion&gt;</span> quicksight-user-arn=<span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;quicksight-user-arn&gt;</span></code></pre> 
   </div> 
  </div> </li> 
</ol> 
<h3>Publish and share the QuickSight dashboard with the policy validation findings</h3> 
<p>You can publish your QuickSight dashboard and then share it with other QuickSight users for reporting purposes. The dashboard preserves the configuration of the analysis at the time that it’s published and reflects the current data in the datasets used by the analysis.</p> 
<h4>To publish the QuickSight dashboard</h4> 
<ol> 
 <li>In the <a href="https://quicksight.aws.amazon.com/" rel="noopener" target="_blank">QuickSight console</a>, choose&nbsp;<strong>Analyses</strong> and then choose <strong>access-analyzer-validation-findings.</strong></li> 
 <li>(Optional) Modify the visuals of the analysis. For more information, see <a href="https://docs.aws.amazon.com/quicksight/latest/user/example-modify-visuals.html" rel="noopener" target="_blank">Tutorial: Modify Amazon QuickSight visuals</a>.</li> 
 <li>Share the QuickSight dashboard. 
  <ol> 
   <li>In your analysis, in the application bar at the upper right, choose&nbsp;<strong>Share</strong>, and then choose&nbsp;<strong>Publish dashboard</strong>.</li> 
   <li>On the&nbsp;<strong>Publish dashboard</strong>&nbsp;page, choose&nbsp;<strong>Publish new dashboard as</strong> and enter <span style="font-family: courier;">IAM Access Analyzer Policy Validation</span>.</li> 
   <li>Choose&nbsp;<strong>Publish dashboard</strong>. The dashboard is now published. </li> 
  </ol> </li> 
 <li>On the QuickSight start page, choose&nbsp;<strong>Dashboards</strong>.</li> 
 <li>Select the <strong>IAM Access Analyzer Policy Validation </strong>dashboard. IAM Access Analyzer policy validation findings will appear within the next 24 hours.<br /> 
  <blockquote>
   <p><strong>Note</strong>: If you don’t want to wait until the Lambda function is initiated automatically, you can invoke the function that lists customer-managed policies and inline policies by using the aws lambda invoke AWS CLI command on the hub account and wait one to two minutes to see the policy validation findings:</p>
  </blockquote> <p style="font-family: courier;">aws lambda invoke –function-name access-analyzer-list-iam-policy –invocation-type Event –cli-binary-format raw-in-base64-out –payload {} response.json</p> </li> 
 <li>(Optional) To export your dashboard as a PDF, see <a href="https://docs.aws.amazon.com/quicksight/latest/user/export-dashboard-to-pdf.html" rel="noopener" target="_blank">Exporting Amazon QuickSight analyses or dashboards as PDFs</a>.</li> 
</ol> 
<h4>To share the QuickSight dashboard</h4> 
<ol> 
 <li>In the <a href="https://quicksight.aws.amazon.com/" rel="noopener" target="_blank">QuickSight console</a>, choose&nbsp;<strong>Dashboards</strong> and then choose <strong>IAM Access Analyzer Policy Validation.</strong></li> 
 <li>In your dashboard, in the application bar at the upper right, choose&nbsp;<strong>Share</strong>, and then choose <strong>Share dashboard</strong>.</li> 
 <li>On the <strong>Share dashboard</strong> page that opens, do the following: 
  <ol> 
   <li>For <strong>Invite users and groups to dashboard</strong> on the left pane, enter a user email or group name in the search box. Users or groups that match your query appear in a list below the search box. Only active users and groups appear in the list.</li> 
   <li>For the user or group that you want to grant access to the dashboard, choose <strong>Add</strong>. Then choose the level of permissions that you want them to have.</li> 
  </ol> </li> 
 <li>After you grant users access to a dashboard, you can <a href="https://docs.aws.amazon.com/quicksight/latest/user/sharing-a-dashboard.html#share-a-dashboard-share-link" rel="noopener" target="_blank">copy a link to it</a> and send it to them.</li> 
</ol> 
<p>For more details, see <a href="https://docs.aws.amazon.com/quicksight/latest/user/sharing-a-dashboard.html" rel="noopener" target="_blank">Sharing dashboards</a> or <a href="https://docs.aws.amazon.com/quicksight/latest/user/share-dashboard-view.html" rel="noopener" target="_blank">Sharing your view of a dashboard</a>.</p> 
<p>Your teams can use this dashboard to better understand their IAM policies and how to move&nbsp;toward least-privilege&nbsp;permissions, as outlined in the section <em>Validate your IAM roles</em> of the blog post <a href="https://aws.amazon.com/blogs/security/top-10-security-items-to-improve-in-your-aws-account/" rel="noopener" target="_blank">Top 10 security items to improve in your AWS account</a>.</p> 
<h2>Clean up</h2> 
<p>To avoid incurring additional charges in your accounts, remove the resources that you created in this walkthrough.</p> 
<p>Before deleting the CloudFormation stacks and stack sets in your accounts, make sure that the S3 buckets that you created are empty. To delete everything (including old versioned objects) in a versioned bucket, we recommend <a href="https://docs.aws.amazon.com/AmazonS3/latest/userguide/empty-bucket.html" rel="noopener" target="_blank">emptying the bucket through the console</a>. Before deleting the CloudFormation stack from the central account, <a href="https://docs.aws.amazon.com/athena/latest/ug/workgroups-create-update-delete.html#deleting-workgroups" rel="noopener" target="_blank">delete the Athena workgroup</a>.</p> 
<h4>To delete remaining resources from your AWS accounts</h4> 
<ol> 
 <li>Delete the CloudFormation stack from your central account by running the following command. Make sure to replace <span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;AWSRegion&gt;</span> with your own Region. 
  <div class="hide-language"> 
   <div class="code-toolbar"> 
    <pre class="unlimited-height-code language-bash"><code class=" language-bash">make delete-hub aws-region=<span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;AWSRegion&gt;</span></code></pre> 
   </div> 
  </div> </li> 
 <li><a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/stackinstances-delete.html" rel="noopener" target="_blank">Delete the CloudFormation stack set instances</a> and <a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/stacksets-delete.html" rel="noopener" target="_blank">stack sets</a> by running the following command using your organization’s management account credentials. Make sure to replace <span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;AWSRegion&gt;</span> with your own Region. 
  <div class="hide-language"> 
   <div class="code-toolbar"> 
    <pre class="unlimited-height-code language-bash"><code class=" language-bash">make delete-stackset-instances aws-region=<span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;AWSRegion&gt;</span><br />&nbsp;
<span style="font-style: italic;"># Wait for the operation to finish. You can check its progress on the CloudFormation console.</span><br />&nbsp;
make delete-stackset aws-region=<span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;AWSRegion&gt;</span></code></pre> 
   </div> 
  </div> </li> 
 <li>Delete the QuickSight dashboard by running the following command using the central account credentials. Make sure to replace <span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;AWSRegion&gt;</span> with your own Region. 
  <div class="hide-language"> 
   <div class="code-toolbar"> 
    <pre class="unlimited-height-code language-bash"><code class=" language-bash">make delete-dashboard aws-region=<span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;AWSRegion&gt;</span></code></pre> 
   </div> 
  </div> </li> 
 <li>To cancel your QuickSight subscription and close the account, see <a href="https://docs.aws.amazon.com/quicksight/latest/user/closing-account.html" rel="noopener" target="_blank">Canceling your Amazon QuickSight subscription and closing the account</a>.</li> 
</ol> 
<h2>Conclusion</h2> 
<p>In this post, you learned how to validate your existing IAM policies by using the IAM Access Analyzer <span style="font-family: courier;">ValidatePolicy</span> API and visualizing the results with AWS analytics tools. By using the implementation, you can better understand your IAM policies and work to reach least privilege in a scalable, fault-tolerant, and cost-effective way. This will help you identify opportunities to tighten your permissions and to grant the right fine-grained permissions to help enhance your overall security posture.</p> 
<p>To learn more about IAM Access Analyzer, see <a href="https://aws.amazon.com/blogs/security/tag/iam-access-analyzer/" rel="noopener" target="_blank">previous blog posts on IAM Access Analyzer</a>.</p> 
<p>To download the CloudFormation templates, see the <a href="https://github.com/aws-samples/visualize-iam-access-analyzer-policy-validation-findings" rel="noopener" target="_blank">visualize-iam-access-analyzer-policy-validation-findings</a> GitHub repository. For information about pricing, see <a href="https://aws.amazon.com/sqs/pricing/" rel="noopener" target="_blank">Amazon SQS pricing</a>, <a href="https://aws.amazon.com/lambda/pricing/" rel="noopener" target="_blank">AWS Lambda pricing</a>, <a href="https://aws.amazon.com/athena/pricing/" rel="noopener" target="_blank">Amazon Athena pricing</a> and <a href="https://aws.amazon.com/quicksight/pricing/" rel="noopener" target="_blank">Amazon QuickSight pricing</a>.</p> 
<p>If you have feedback about this post, submit comments in the Comments section below. If you have questions about this post, start a new thread on the <a href="https://repost.aws/topics/TAEEfW2o7QS4SOLeZqACq9jA/security-identity-compliance" rel="noopener" target="_blank">AWS Security, Identity, &amp; Compliance re:Post</a>.</p> 
<p><strong>Want more AWS Security news? Follow us on <a href="https://twitter.com/AWSsecurityinfo" rel="noopener noreferrer" target="_blank" title="Twitter">Twitter</a>.</strong></p> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image"> 
   <img alt="Mostefa Brougui" class="aligncenter size-full wp-image-28494" height="160" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/02/06/Mostefa-Brougui.jpg" width="120" /> 
  </div> 
  <h3 class="lb-h4">Mostefa Brougui</h3> 
  <p>Mostefa is a Sr. Security Consultant in Professional Services at Amazon Web Services. He works with AWS enterprise customers to design, build, and optimize their security architecture to drive business outcomes.</p> 
  <p></p>
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image"> 
   <img alt="Tobias Nickl" class="aligncenter size-full wp-image-28495" height="160" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/02/06/Tobias-Nickl.jpg" width="120" /> 
  </div> 
  <h3 class="lb-h4">Tobias Nickl</h3> 
  <p>Tobias works in Professional Services at Amazon Web Services as a Security Engineer. In addition to building custom AWS solutions, he advises AWS enterprise customers on how to reach their business objectives and accelerate their cloud transformation.</p> 
  <p></p>
 </div> 
</footer>
