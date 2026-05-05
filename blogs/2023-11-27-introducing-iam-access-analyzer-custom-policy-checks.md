---
title: "Introducing IAM Access Analyzer custom policy checks"
url: "https://aws.amazon.com/blogs/security/introducing-iam-access-analyzer-custom-policy-checks/"
date: "Mon, 27 Nov 2023 14:00:04 +0000"
author: "Mitch Beaumont"
feed_url: "https://aws.amazon.com/blogs/security/tag/iam-access-analyzer/feed/"
---
<blockquote>
 <p><strong>July 12, 2024:</strong> AWS has extended custom policy checks to include a new check called Check No Public Access. This new check determines whether a resource policy grants public access to a specified resource type. In addition to this new check, there has been an update to the existing Check Access Not Granted check. The Check Access Not Granted check can now be used to determine whether a given policy grants permission to one or more customer-defined AWS resources. The examples in this blog post’s <a href="https://github.com/aws-samples/access-analyzer-automated-policy-analysis-blog" rel="noopener" target="_blank">accompanying sample repository on GitHub</a> have been updated to reflect these updates.</p>
</blockquote> 
<hr /> 
<p><a href="https://aws.amazon.com/iam/" rel="noopener" target="_blank">AWS Identity and Access Management (IAM)</a> <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/what-is-access-analyzer.html" rel="noopener" target="_blank">Access Analyzer</a> was launched in late 2019. Access Analyzer guides customers toward least-privilege permissions across <a href="https://aws.amazon.com/" rel="noopener" target="_blank">Amazon Web Services (AWS)</a> by using analysis techniques, such as <a href="https://aws.amazon.com/what-is/automated-reasoning/" rel="noopener" target="_blank">automated reasoning</a>, to make it simpler for customers to set, verify, and refine IAM permissions. Today, we are excited to announce the general availability of IAM Access Analyzer custom policy checks, a new IAM Access Analyzer feature that helps customers accurately and proactively check IAM policies for critical permissions and increases in policy permissiveness.</p> 
<p>In this post, we’ll show how you can integrate custom policy checks into builder workflows to automate the identification of overly permissive IAM policies and IAM policies that contain permissions that you decide are sensitive or critical.</p> 
<h2>What is the problem?</h2> 
<p>Although security teams are responsible for the overall security posture of the organization, developers are the ones creating the applications that require permissions. To enable developers to move fast while maintaining high levels of security, organizations look for ways to safely delegate the ability of developers to author IAM policies. Many AWS customers implement manual IAM policy reviews before deploying developer-authored policies to production environments. Customers follow this practice to try to prevent excessive or unwanted permissions finding their way into production. Depending on the volume and complexity of the policies that need to be reviewed; these reviews can be intensive and take time. The result is a slowdown in development and potential delay in deployment of applications and services. Some customers write custom tooling to remove the manual burden of policy reviews, but this can be costly to build and maintain.</p> 
<h2>How do custom policy checks solve that problem?</h2> 
<p>Custom policy checks are a new IAM Access Analyzer capability that helps security teams accurately and proactively identify critical permissions in their policies. Custom policy checks can also tell you if a new version of a policy is more permissive than the previous version. Custom policy checks use automated reasoning, a form of static analysis, to provide a higher level of security assurance in the cloud. For more information, see <a href="https://d1.awsstatic.com/Security/pdfs/Formal_Reasoning_About_The_Security_of_Amazon_Web_Services.pdf" rel="noopener" target="_blank">Formal Reasoning About the Security of Amazon Web Services.</a></p> 
<p>Custom policy checks can be embedded in a continuous integration and continuous delivery (CI/CD) pipeline so that checks can be run against policies without having to deploy the policies. In addition, developers can run custom policy checks from their local development environments and get fast feedback about whether or not the policies they are authoring are in line with your organization’s security standards.</p> 
<h2>How to analyze IAM policies with custom policy checks</h2> 
<p>In this section, we provide step-by-step instructions for using custom policy checks to analyze IAM policies.</p> 
<h3>Prerequisites</h3> 
<p>To complete the examples in our walkthrough, you will need the following:</p> 
<ol> 
 <li>An AWS account, and an identity that has permissions to use the AWS services, and create the resources, used in the following examples. For more information, see the full sample code used in this blog post <a href="https://github.com/aws-samples/access-analyzer-automated-policy-analysis-blog.git" rel="noopener" target="_blank">on GitHub</a>.</li> 
 <li>An installed and configured AWS CLI. For more information, see <a href="https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html" rel="noopener" target="_blank">Configure the AWS CLI</a>.</li> 
 <li>The <a href="https://aws.amazon.com/cdk/" rel="noopener" target="_blank">AWS Cloud Development Kit</a> (AWS CDK). For installation instructions, refer to <a href="https://docs.aws.amazon.com/cdk/v2/guide/getting_started.html" rel="noopener" target="_blank">Install the AWS CDK</a>.</li> 
</ol> 
<h3>Example 1: Use custom policy checks to compare two IAM policies and check that one does not grant more access than the other</h3> 
<p>In this example, you will create two IAM identity policy documents, <span style="font-family: courier;">NewPolicyDocument</span> and <span style="font-family: courier;">ExistingPolicyDocument</span>. You will use the new <span style="font-family: courier;">CheckNoNewAccess</span> API to compare these two policies and check that <span style="font-family: courier;">NewPolicyDocument</span> does not grant more access than <span style="font-family: courier;">ExistingPolicyDocument</span>.</p> 
<h4>Step 1: Create two IAM identity policy documents</h4> 
<ol> 
 <li>Use the following command to create <span style="font-family: courier;">ExistingPolicyDocument</span>. 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">cat &lt;&lt; EOF &gt; existing-policy-document.json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:StartInstances",
                "ec2:StopInstances"
            ],
            "Resource": "arn:aws:ec2:*:*:instance/*",
            "Condition": {
                "StringEquals": {
                    "aws:ResourceTag/Owner": "\${aws:username}"
                }
            }
        }
    ]
}
EOF</code></pre> 
  </div> </li> 
 <li>Use the following command to create <span style="font-family: courier;">NewPolicyDocument</span>. 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">cat &lt;&lt; EOF &gt; new-policy-document.json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:StartInstances",
                "ec2:StopInstances"
            ],
            "Resource": "arn:aws:ec2:*:*:instance/*"
        }
    ]
}
EOF</code></pre> 
  </div> </li> 
</ol> 
<p>Notice that <span style="font-family: courier;">ExistingPolicyDocument</span> grants access to the <span style="font-family: courier;">ec2:StartInstances</span> and <span style="font-family: courier;">ec2:StopInstances</span> actions if the condition key <span style="font-family: courier;">aws:ResourceTag/Owner</span> resolves to true. In other words, the value of the tag matches the policy variable <span style="font-family: courier;">aws:username</span>. <span style="font-family: courier;">NewPolicyDocument</span> grants access to the same actions, but does not include a condition key.</p> 
<h4>Step 2: Check the policies by using the AWS CLI</h4> 
<ol> 
 <li>Use the following command to call the <span style="font-family: courier;">CheckNoNewAccess</span> API to check whether <span style="font-family: courier;">NewPolicyDocument</span> grants more access than <span style="font-family: courier;">ExistingPolicyDocument</span>. 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">aws accessanalyzer check-no-new-access \
--new-policy-document file://<strong>new-policy-document.json</strong> \
--existing-policy-document file://<strong>existing-policy-document.json</strong> \
--policy-type IDENTITY_POLICY</code></pre> 
  </div> </li> 
</ol> 
<p>After a moment, you will see a response from Access Analyzer. The response will look similar to the following.</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-text">{
    "result": "FAIL",
    "message": "The modified permissions grant new access compared to your existing policy.",
    "reasons": [
        {
            "description": "New access in the statement with index: 1.",
            "statementIndex": 1
        }
    ]
}</code></pre> 
</div> 
<p>In this example, the validation returned a result of <em>FAIL</em>. This is because <span style="font-family: courier;">NewPolicyDocument</span> is missing the condition key, potentially granting any principal with this identity policy attached more access than intended or needed.</p> 
<h3>Example 2: Use custom policy checks to check that an IAM policy does not contain sensitive permissions</h3> 
<p>In this example, you will create an IAM identity-based policy that contains a set of permissions. You will use the <span style="font-family: courier;">CheckAccessNotGranted</span> API to check that the new policy does not give permissions to disable <a href="https://aws.amazon.com/cloudtrail/" rel="noopener" target="_blank">AWS CloudTrail</a> or delete any associated trails.</p> 
<h4>Step 1: Create a new IAM identity policy document</h4> 
<ul> 
 <li>Use the following command to create <span style="font-family: courier;">IamPolicyDocument</span>. 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">cat &lt;&lt; EOF &gt; iam-policy-document.json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "cloudtrail:StopLogging",
                "cloudtrail:Delete*"
            ],
            "Resource": ["*"] 
        }
    ]
}
EOF</code></pre> 
  </div> </li> 
</ul> 
<h4>Step 2: Check the policy by using the AWS CLI</h4> 
<ul> 
 <li>Use the following command to call the <span style="font-family: courier;">CheckAccessNotGranted</span> API to check if the new policy grants permission to the set of sensitive actions. In this example, you are asking Access Analyzer to check that <span style="font-family: courier;">IamPolicyDocument</span> does not contain the actions <span style="font-family: courier;">cloudtrail:StopLogging</span> or <span style="font-family: courier;">cloudtrail:DeleteTrail</span> (passed as a list to the access parameter). 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">aws accessanalyzer check-access-not-granted \
--policy-document file://<strong>iam-policy-document.json</strong> \
--access actions=cloudtrail:StopLogging,cloudtrail:DeleteTrail \
--policy-type IDENTITY_POLICY</code></pre> 
  </div> </li> 
</ul> 
<p>Because the policy that you created contains both <span style="font-family: courier;">cloudtrail:StopLogging</span> and <span style="font-family: courier;">cloudtrail:DeleteTrail</span> actions, Access Analyzer returns a FAIL.</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-text">{
    "result": "FAIL",
    "message": "The policy document grants access to perform one or more of the listed actions.",
    "reasons": [
        {
            "description": "One or more of the listed actions in the statement with index: 0.",
            "statementIndex": 0
        }
    ]
}</code></pre> 
</div> 
<h3>Example 3: Integrate custom policy checks into the developer workflow</h3> 
<p>Building on the previous two examples, in this example, you will automate the analysis of the IAM policies defined in an <a href="https://aws.amazon.com/cloudformation/" rel="noopener" target="_blank"><strong>AWS CloudFormation</strong></a> template. Figure 1 shows the workflow that will be used. The workflow will initiate each time a pull request is created against the main branch of an <a href="https://aws.amazon.com/codecommit/" rel="noopener" target="_blank"><strong>AWS CodeCommit</strong></a> repository called <span style="font-family: courier;">my-iam-policy</span> (the commit stage in Figure 1). The first check uses the <span style="font-family: courier;">CheckNoNewAccess</span> API to determine if the updated policy is more permissive than a reference IAM policy. The second check uses the <span style="font-family: courier;">CheckAccessNotGranted</span> API to automatically check for critical permissions within the policy (the validation stage in Figure 1). In both cases, if the updated policy is more permissive, or contains critical permissions, a comment with the results of the validation is posted to the pull request. This information can then be used to decide whether the pull request is merged into the main branch for deployment (the deploy stage is shown in Figure 1).</p> 
<div class="wp-caption aligncenter" id="attachment_32345" style="width: 790px;">
 <img alt="Figure 1: Diagram of the pipeline that will check policies" class="size-full wp-image-32345" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/11/20/img1-2.jpg" style="border: 1px solid #bebebe;" width="780" />
 <p class="wp-caption-text" id="caption-attachment-32345">Figure 1: Diagram of the pipeline that will check policies</p>
</div> 
<h4>Step 1: Deploy the infrastructure and set up the pipeline</h4> 
<ol> 
 <li>Use the following command to download and unzip the Cloud Development Kit (CDK) project associated with this blog post. 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">git clone https://github.com/aws-samples/access-analyzer-automated-policy-analysis-blog.git
cd ./access-analyzer-automated-policy-analysis-blog</code></pre> 
  </div> </li> 
 <li>Create a virtual Python environment to contain the project dependencies by using the following command. 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">python3 -m venv .venv</code></pre> 
  </div> </li> 
 <li>Activate the virtual environment with the following command. 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">source .venv/bin/activate</code></pre> 
  </div> </li> 
 <li>Install the project requirements by using the following command. 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">pip install -r requirements.txt</code></pre> 
  </div> </li> 
 <li>Use the following command to update the CDK CLI to the latest major version. 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">npm install -g aws-cdk@2 --force</code></pre> 
  </div> </li> 
 <li>Before you can deploy the CDK project, use the following command to bootstrap your AWS environment. Bootstrapping is the process of creating resources needed for deploying CDK projects. These resources include an <a href="https://aws.amazon.com/s3/" rel="noopener" target="_blank">Amazon Simple Storage Service (Amazon S3)</a> bucket for storing files and IAM roles that grant permissions needed to perform deployments. 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">cdk bootstrap</code></pre> 
  </div> </li> 
 <li>Finally, use the following command to deploy the pipeline infrastructure. 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">cdk deploy --require-approval never</code></pre> 
  </div> <p>The deployment will take a few minutes to complete. Feel free to grab a coffee and check back shortly.</p> <p>When the deployment completes, there will be two stack outputs listed: one with a name that contains <span style="font-family: courier;">CodeCommitRepo</span> and another with a name that contains <span style="font-family: courier;">ConfigBucket</span>. Make a note of the values of these outputs, because you will need them later.</p> <p>The deployed pipeline is displayed in the <a href="https://console.aws.amazon.com/codesuite/codepipeline/pipelines/" rel="noopener" target="_blank">AWS CodePipeline console</a> and should look similar to the pipeline shown in Figure 2.</p> 
  <div class="wp-caption aligncenter" id="attachment_32346" style="width: 750px;">
   <img alt="Figure 2: AWS CodePipeline and CodeBuild Management Console view" class="size-full wp-image-32346" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/11/20/img2-7.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-32346">Figure 2: AWS CodePipeline and CodeBuild Management Console view</p>
  </div> <p>In addition to initiating when a pull request is created, the newly deployed pipeline can also be initiated when changes to the main branch of the <a href="https://aws.amazon.com/codecommit/" rel="noopener" target="_blank">AWS CodeCommit</a> repository are detected. The pipeline has three stages, <em>CheckoutSources</em>, <em>IAMPolicyAnalysis</em>, and <em>deploy</em>. The <em>CheckoutSource</em> stage checks out the contents of the <span style="font-family: courier;">my-iam-policy</span> repository when the pipeline is triggered due to a change in the main branch.</p> <p>The <em>IAMPolicyAnalysis</em> stage, which runs after the <em>CheckoutSource</em> stage or when a pull request has been created against the main branch, has two actions. The first action, <strong>Check no new access,</strong> verifies that changes to the IAM policies in the CloudFormation template do not grant more access than a pre-defined <em>reference</em> policy. The second action, <strong>Check access not granted</strong>, verifies that those same updates do not grant access to API actions that are deemed sensitive or critical. Finally, the <em>Deploy</em> stage will deploy the resources defined in the CloudFormation template, if the actions in the <em>IAMPolicyAnalysis</em> stage are successful.</p> <p>To analyze the IAM policies, the <strong>Check no new access</strong> and <strong>Check access not granted</strong> actions depend on a <em>reference</em> policy and a predefined list of API actions, respectively.</p> </li> 
 <li>Use the following command to create the <em>reference</em> policy. 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">cd ../ 
cat &lt;&lt; EOF &gt; cnna-reference-policy.json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "*",
            "Resource": "*"
        },
        {
            "Effect": "Deny",
            "Action": "iam:PassRole",
            "Resource": "arn:aws:iam::*:role/my-sensitive-roles/*"
        }
    ]
}	
EOF</code></pre> 
  </div> <p>This reference policy sets out the maximum permissions for policies that you plan to validate with custom policy checks. The <span style="font-family: courier;">iam:PassRole</span> permission is a permission that allows an IAM principal to pass an IAM role to an AWS service, like <a href="https://aws.amazon.com/ec2/" rel="noopener" target="_blank">Amazon Elastic Compute Cloud (Amazon EC2)</a> or <a href="https://aws.amazon.com/lambda/" rel="noopener" target="_blank">AWS Lambda</a>. The reference policy says that the only way that a policy is more permissive is if it allows <span style="font-family: courier;">iam:PassRole</span> on this group of sensitive resources: <span style="font-family: courier;">arn:aws:iam::*:role/my-sensitive-roles/*”</span>.</p> <p>Why might a reference policy be useful? A reference policy helps ensure that a particular combination of actions, resources, and conditions is not allowed in your environment. Reference policies typically allow actions and resources in one statement, then deny the problematic permissions in a second statement. This means that a policy that is more permissive than the reference policy allows access to a permission that the reference policy has denied.</p> <p>In this example, a developer who is authorized to create IAM roles could, intentionally or unintentionally, create an IAM role for an AWS service (like EC2 for AWS Lambda) that has permission to pass a privileged role to another service or principal, leading to an escalation of privilege.</p> </li> 
 <li>Use the following command to create a list of sensitive actions. This list will be parsed during the build pipeline and passed to the <span style="font-family: courier;">CheckAccessNotGranted</span> API. If the policy grants access to one or more of the sensitive actions in this list, a result of FAIL will be returned. To keep this example simple, add a single API action, as follows. 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">cat &lt;&lt; EOF &gt; sensitive-actions.file
dynamodb:DeleteTable
EOF</code></pre> 
  </div> </li> 
 <li>So that the CodeBuild projects can access the dependencies, use the following command to copy the <span style="font-family: courier;">cnna-reference-policy.file</span> and <span style="font-family: courier;">sensitive-actions.file</span> to an S3 bucket. Refer to the stack outputs you noted earlier and replace <span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;ConfigBucket&gt;</span> with the name of the S3 bucket created in your environment. 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">aws s3 cp ./cnna-reference-policy.json s3://<span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;ConfgBucket&gt;</span>/cnna-reference-policy.json
aws s3 cp ./sensitive-actions.file s3://<span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;ConfigBucket&gt;</span>/sensitive-actions.file</code></pre> 
  </div> </li> 
</ol> 
<h4>Step 2: Create a new CloudFormation template that defines an IAM policy</h4> 
<p>With the pipeline deployed, the next step is to clone the repository that was created and populate it with a CloudFormation template that defines an IAM policy.</p> 
<ol> 
 <li>Install <strong>git-remote-codecommit </strong>by using the following command. 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">pip install git-remote-codecommit</code></pre> 
  </div> <p>For more information on installing and configuring <strong>git-remote-codecommit</strong>, see the <a href="https://docs.aws.amazon.com/codecommit/latest/userguide/setting-up-git-remote-codecommit.html#setting-up-git-remote-codecommit-prereq" rel="noopener" target="_blank">AWS CodeCommit User Guide</a>.</p> </li> 
 <li>With <strong>git-remote-codecommit</strong> installed, use the following command to clone the my-iam-policy repository from AWS CodeCommit. 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">git clone codecommit://my-iam-policy &amp;&amp; cd ./my-iam-policy</code></pre> 
  </div> <p>If you’ve configured a named profile for use with the AWS CLI, use the following command, replacing <span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;profile&gt;</span> with the name of your named profile.</p> 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">git clone codecommit://<span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;profile&gt;</span>@my-iam-policy &amp;&amp; cd ./my-iam-policy</code></pre> 
  </div> </li> 
 <li>Use the following command to create the CloudFormation template in the local clone of the repository. 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">cat &lt;&lt; EOF &gt; ec2-instance-role.yaml
---
AWSTemplateFormatVersion: 2010-09-09
Description: CloudFormation Template to deploy base resources for access_analyzer_blog
Resources:
  EC2Role:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: 2012-10-17
        Statement:
        - Effect: Allow
          Principal:
            Service: ec2.amazonaws.com
          Action: sts:AssumeRole
      Path: /
      Policies:
      - PolicyName: my-application-permissions
        PolicyDocument:
          Version: 2012-10-17
          Statement:
          - Effect: Allow
            Action:
              - 'ec2:RunInstances'
              - 'lambda:CreateFunction'
              - 'lambda:InvokeFunction'
              - 'dynamodb:Scan'
              - 'dynamodb:Query'
              - 'dynamodb:UpdateItem'
              - 'dynamodb:GetItem'
            Resource: '*'
          - Effect: Allow
            Action:
              - iam:PassRole 
            Resource: "arn:aws:iam::*:role/my-custom-role"
        
  EC2InstanceProfile:
    Type: AWS::IAM::InstanceProfile
    Properties:
      Path: /
      Roles:
        - !Ref EC2Role
EOF</code></pre> 
  </div> </li> 
</ol> 
<p>The actions in the <em>IAMPolicyValidation</em> stage are run by a CodeBuild project. CodeBuild environments run arbitrary commands that are passed to the project using a buildspec file. Each project has already been configured to use an inline buildspec file.</p> 
<p>You can inspect the buildspec file for each project by opening the project’s <strong>Build details</strong> page as shown in Figure 3.</p> 
<div class="wp-caption aligncenter" id="attachment_32347" style="width: 790px;">
 <img alt="Figure 3: AWS CodeBuild console and build details" class="size-full wp-image-32347" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/11/20/img3-6.png" style="border: 1px solid #bebebe;" width="780" />
 <p class="wp-caption-text" id="caption-attachment-32347">Figure 3: AWS CodeBuild console and build details</p>
</div> 
<h4>Step 3: Run analysis on the IAM policy</h4> 
<p>The next step involves checking in the first version of the CloudFormation template to the repository and checking two things. First, that the policy does not grant more access than the reference policy. Second, that the policy does not contain any of the sensitive actions defined in the <span style="font-family: courier;">sensitive-actions.file</span>.</p> 
<ol> 
 <li>To begin tracking the CloudFormation template created earlier, use the following command. 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">git add ec2-instance-role.yaml </code></pre> 
  </div> </li> 
 <li>Commit the changes you have made to the repository. 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">git commit -m 'committing a new CFN template with IAM policy'</code></pre> 
  </div> </li> 
 <li>Finally, push these changes to the remote repository. 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">git push</code></pre> 
  </div> </li> 
 <li>Pushing these changes will initiate the pipeline. After a few minutes the pipeline should complete successfully. To view the status of the pipeline, do the following: 
  <ol> 
   <li>Navigate to https://<span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;region&gt;</span>.console.aws.amazon.com/codesuite/codepipeline/pipelines (replacing <span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;region&gt;</span> with your AWS Region).</li> 
   <li>Choose the pipeline called <strong>accessanalyzer-pipeline</strong>.</li> 
   <li>Scroll down to the <strong>IAMPolicyValidation </strong>stage of the pipeline.</li> 
   <li>For both the <strong>check no new access</strong> and <strong>check access not granted</strong> actions, choose <strong>View Logs</strong> to inspect the log output.</li> 
  </ol> </li> 
 <li>If you inspect the build logs for both the <strong>check no new access</strong> and <strong>check access not granted</strong> actions within the pipeline, you should see that there were no blocking or non-blocking findings, similar to what is shown in Figure 4. This indicates that the policy was validated successfully. In other words, the policy was not more permissive than the reference policy, and it did not include any of the critical permissions. 
  <div class="wp-caption aligncenter" id="attachment_32348" style="width: 510px;">
   <img alt="Figure 4: CodeBuild log entry confirming that the IAM policy was successfully validated" class="size-full wp-image-32348" height="154" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/11/20/img4-5.png" style="border: 1px solid #bebebe;" width="500" />
   <p class="wp-caption-text" id="caption-attachment-32348">Figure 4: CodeBuild log entry confirming that the IAM policy was successfully validated</p>
  </div> </li> 
</ol> 
<h4>Step 4: Create a pull request to merge a new update to the CloudFormation template</h4> 
<p>In this step, you will make a change to the IAM policy in the CloudFormation template. The change deliberately makes the policy grant more access than the reference policy. The change also includes a critical permission.</p> 
<ol> 
 <li>Use the following command to create a new branch called <span style="font-family: courier;">add-new-permissions</span> in the local clone of the repository. 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">git checkout -b add-new-permissions</code></pre> 
  </div> </li> 
 <li>Next, edit the IAM policy in <span style="font-family: courier;">ec2-instance-role.yaml</span> to include an additional API action, <span style="font-family: courier;">dynamodb:Delete*</span> and update the resource property of the inline policy to use an IAM role in the <span style="font-family: courier;">/my-sensitive-roles/*”</span> path. You can copy the following example, if you’re unsure of how to do this. 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">---
AWSTemplateFormatVersion: 2010-09-09
Description: CloudFormation Template to deploy base resources for access_analyzer_blog
Resources:
  EC2Role:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: 2012-10-17
        Statement:
        - Effect: Allow
          Principal:
            Service: ec2.amazonaws.com
          Action: sts:AssumeRole
      Path: /
      Policies:
      - PolicyName: my-application-permissions
        PolicyDocument:
          Version: 2012-10-17
          Statement:
          - Effect: Allow
            Action:
              - 'ec2:RunInstances'
              - 'lambda:CreateFunction'
              - 'lambda:InvokeFunction'
              - 'dynamodb:Scan'
              - 'dynamodb:Query'
              - 'dynamodb:UpdateItem'
              - 'dynamodb:GetItem'
              - 'dynamodb:Delete*'
            Resource: '*'
          - Effect: Allow
            Action:
              - iam:PassRole 
            Resource: "arn:aws:iam::*:role/my-sensitive-roles/my-custom-admin-role"
        
  EC2InstanceProfile:
    Type: AWS::IAM::InstanceProfile
    Properties:
      Path: /
      Roles:
        - !Ref EC2Role</code></pre> 
  </div> </li> 
 <li>Commit the policy change and push the updated policy document to the repo by using the following commands. 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">git add ec2-instance-role.yaml 
git commit -m "adding new permission and allowing my ec2 instance to assume a pass sensitive IAM role"</code></pre> 
  </div> </li> 
 <li>The <span style="font-family: courier;">add-new-permissions</span> branch is currently a local branch. Use the following command to push the branch to the remote repository. This action will not initiate the pipeline, because the pipeline only runs when changes are made to the repository’s main branch. 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">git push -u origin add-new-permissions</code></pre> 
  </div> </li> 
 <li>With the new branch and changes pushed to the repository, follow these steps to create a pull request: 
  <ol> 
   <li>Navigate to https://console.aws.amazon.com/codesuite/codecommit/repositories (don’t forget to the switch to the correct Region).</li> 
   <li>Choose the repository called <span style="font-family: courier;">my-iam-policy</span>.</li> 
   <li>Choose the branch <span style="font-family: courier;">add-new-permissions</span> from the drop-down list at the top of the repository screen. 
    <div class="wp-caption aligncenter" id="attachment_32349" style="width: 710px;">
     <img alt="Figure 5: my-iam-policy repository with new branch available" class="size-full wp-image-32349" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/11/20/img5-4.png" style="border: 1px solid #bebebe;" width="700" />
     <p class="wp-caption-text" id="caption-attachment-32349">Figure 5: my-iam-policy repository with new branch available</p>
    </div> </li> 
   <li>Choose <strong>Create pull request</strong>.</li> 
   <li>Enter a title and description for the pull request.</li> 
   <li>(Optional) Scroll down to see the differences between the current version and new version of the CloudFormation template highlighted.</li> 
   <li>Choose <strong>Create pull request</strong>.</li> 
  </ol> </li> 
 <li>The creation of the pull request will Initiate the pipeline to fetch the CloudFormation template from the repository and run the <strong>check no new access</strong> and <strong>check access not granted</strong> analysis actions.</li> 
 <li>After a few minutes, choose the <strong>Activity</strong> tab for the pull request. You should see a comment from the pipeline that contains the results of the failed validation. 
  <div class="wp-caption aligncenter" id="attachment_32350" style="width: 750px;">
   <img alt="Figure 6: Results from the failed validation posted as a comment to the pull request" class="size-full wp-image-32350" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/11/20/img6-4.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-32350">Figure 6: Results from the failed validation posted as a comment to the pull request</p>
  </div> </li> 
</ol> 
<h3>Why did the validations fail?</h3> 
<p>The updated IAM role and inline policy failed validation for two reasons. First, the reference policy said that no one should have more permissions than the reference policy does. The reference policy in this example included a <strong>deny</strong> statement for the <span style="font-family: courier;">iam:PassRole</span> permission with a resource of <span style="font-family: courier;">/my-sensitive-role/*</span>. The new created inline policy included an allow statement for the <span style="font-family: courier;">iam:PassRole</span> permission with a resource of <span style="font-family: courier;">arn:aws:iam::*:role/my-sensitive-roles/my-custom-admin-role</span>. In other words, the new policy had more permissions than the reference policy.</p> 
<p>Second, the list of critical permissions included the <span style="font-family: courier;">dynamodb:DeleteTable</span> permission. The inline policy included a statement that would allow the EC2 instance to perform the <span style="font-family: courier;">dynamodb:DeleteTable</span> action.</p> 
<h2>Cleanup</h2> 
<p>Use the following command to delete the infrastructure that was provisioned as part of the examples in this blog post.</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-text">cdk destroy </code></pre> 
</div> 
<h2>Conclusion</h2> 
<p>In this post, I introduced you to two new IAM Access Analyzer APIs: <span style="font-family: courier;">CheckNoNewAccess</span> and <span style="font-family: courier;">CheckAccessNotGranted</span>. The main example in the post demonstrated one way in which you can use these APIs to automate security testing throughout the development lifecycle. The example did this by integrating both APIs into the developer workflow and validating the developer-authored IAM policy when the developer created a pull request to merge changes into the repository’s main branch. The automation helped the developer to get feedback about the problems with the IAM policy quickly, allowing the developer to take action in a timely way. This is often referred to as <a href="https://aws.amazon.com/what-is/devsecops/#seo-faq-pairs" rel="noopener" target="_blank">shifting security left</a> — identifying misconfigurations early and automatically supporting an iterative, fail-fast model of continuous development and testing. Ultimately, this enables teams to make security an inherent part of a system’s design and architecture and can speed up product development workflow. </p> 
<p>You can find the full sample code used in this blog post <a href="https://github.com/aws-samples/access-analyzer-automated-policy-analysis-blog.git" rel="noopener" target="_blank">on GitHub</a>.</p> 
<p>To learn more about IAM Access Analyzer and the new custom policy checks feature, see the <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/what-is-access-analyzer.html" rel="noopener" target="_blank">IAM Access Analyzer documentation</a>.</p> 
<p>If you have feedback about this post, submit comments in the<strong> Comments</strong> section below. If you have questions about this post, <a href="https://console.aws.amazon.com/support/home" rel="noopener noreferrer" target="_blank">contact AWS Support</a>.</p> 
<p><strong>Want more AWS Security news? Follow us on <a href="https://twitter.com/AWSsecurityinfo" rel="noopener noreferrer" target="_blank" title="Twitter">Twitter</a>.</strong></p> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image"> 
   <img alt="Mitch Beaumont" class="alignnone size-full wp-image-30551" height="160" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/08/16/beaumonm.jpg" width="120" /> 
  </div> 
  <h3 class="lb-h4">Mitch Beaumont</h3> 
  <p>Mitch is a Principal Solutions Architect for AWS, based in Sydney, Australia. Mitch works with some of Australia’s largest financial services customers, helping them to continually raise the security bar for the products and features that they build and ship. Outside of work, Mitch enjoys spending time with his family, photography, and surfing.</p> 
  <p></p>
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image"> 
   <img alt="Author" class="aligncenter size-full wp-image-22385" height="160" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2021/09/28/Matt-Luttrell-Author.jpg" width="120" /> 
  </div> 
  <h3 class="lb-h4">Matt Luttrell</h3> 
  <p>Matt is a Principal Solutions Architect on the AWS Identity Solutions team. When he’s not spending time chasing his kids around, he enjoys skiing, cycling, and the occasional video game.</p> 
  <p></p>
 </div> 
</footer>
