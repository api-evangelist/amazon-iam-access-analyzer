---
title: "How to implement IAM policy checks with Visual Studio Code and IAM Access Analyzer"
url: "https://aws.amazon.com/blogs/security/how-to-implement-iam-policy-checks-with-visual-studio-code-and-iam-access-analyzer/"
date: "Tue, 14 Jan 2025 17:02:04 +0000"
author: "Anshu Bathla"
feed_url: "https://aws.amazon.com/blogs/security/tag/iam-access-analyzer/feed/"
---
<p>In a previous blog post, we introduced the <a href="https://aws.amazon.com/blogs/security/introducing-iam-access-analyzer-custom-policy-checks/" rel="noopener" target="_blank">IAM Access Analyzer custom policy check feature</a>, which allows you to validate your policies against custom rules. Now we’re taking a step further and bringing these policy checks directly into your development environment with the <a href="https://docs.aws.amazon.com/toolkit-for-vscode/latest/userguide/setup-toolkit.html" rel="noopener" target="_blank">AWS Toolkit for Visual Studio Code (VS Code)</a>.</p> 
<p>In this blog post, we show how you can integrate IAM Access Analyzer custom policy check capability into VS Code, so you can identify overly permissive IAM policies and fine-tune access controls early in the development process. This proactive approach to security and compliance helps to ensure that your IAM policies are validated before they are deployed, reducing the risk of introducing misconfigurations or granting unintended access. It also saves developer time by providing fast feedback to developers when they write a policy that does not meet organizational standards.</p> 
<h2 id="what-is-the-problem">What is the problem?</h2> 
<p>Although security teams oversee an organization’s overall security posture, developers create applications that require specific permissions. To enable developers to work efficiently while maintaining high security standards, organizations often seek ways to safely delegate the authoring of <a href="https://aws.amazon.com/iam/" rel="noopener" target="_blank">AWS Identity and Access Management (IAM)</a> policies to developers. Many AWS customers manually review developer-authored IAM policies before deploying them to production environments to help prevent <a href="https://docs.aws.amazon.com/wellarchitected/latest/framework/sec_permissions_least_privileges.html" rel="noopener" target="_blank">granting excessive or unintended permissions</a>. However, depending on the volume and complexity of policies, these manual reviews can be time-consuming, leading to development delays and potential bottlenecks in the deployment of applications and services. Organizations need to balance secure access management with the agility required for rapid application development and deployment.</p> 
<h2 id="how-to-use-iam-access-analyzer-custom-policy-checks-in-vs-code">How to use IAM Access Analyzer custom policy checks in VS Code</h2> 
<p>Custom policy checks are a feature in IAM Access Analyzer that are designed to help security teams proactively identify and analyze critical permissions within their IAM policies. In this section, we provide step-by-step instructions for using custom policy checks directly in VS Code.</p> 
<h3 id="prerequisites">Prerequisites</h3> 
<p>To complete the examples in our walkthrough, you first need to do the following:</p> 
<ol type="1"> 
 <li>Install Python version 3.6 or later.</li> 
 <li>Assuming you are already using the VS Code Integrated Development Environment (IDE), search for and install the <a href="https://docs.aws.amazon.com/toolkit-for-vscode/latest/userguide/setup-toolkit.html" rel="noopener" target="_blank"><u>AWS Toolkit</u></a> extension.</li> 
 <li>Configure your AWS role credentials to <a href="https://docs.aws.amazon.com/toolkit-for-vscode/latest/userguide/connect.html" rel="noopener" target="_blank"><u>connect the toolkit to AWS</u></a>.</li> 
 <li>Install the <a href="https://github.com/awslabs/aws-cloudformation-iam-policy-validator" rel="noopener" target="_blank"><u>IAM Policy Validator for AWS CloudFormation</u></a>, available on GitHub. Alternatively, you can install the <a href="https://github.com/awslabs/terraform-iam-policy-validator" rel="noopener" target="_blank"><u>IAM Policy Validator for Terraform</u></a> from GitHub if you are using Terraform as infrastructure-as-code in your organization.</li> 
 <li>So that you can open IAM Access Analyzer policy checks in the VS Code editor, open the VS Code Command Palette by pressing <strong>Ctrl+Shift+P</strong>, search for <strong>IAM Policy Checks</strong>, and then choose <strong>AWS: Open IAM Policy Checks</strong> as shown in Figure 1.<br /> 
  <div class="wp-caption aligncenter" id="attachment_37040" style="width: 1387px;">
   <img alt="Figure 1: Search for the AWS: Open IAM Policy Checks option" class="size-full wp-image-37040" height="240" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/01/09/Implement-IAM-policy-checks-1.png" width="1377" />
   <p class="wp-caption-text" id="caption-attachment-37040">Figure 1: Search for the AWS: Open IAM Policy Checks option</p>
  </div> </li> 
</ol> 
<p>By using the IAM policy checks option in VS Code, you can perform four types of checks:</p> 
<ul> 
 <li><a href="https://docs.aws.amazon.com/access-analyzer/latest/APIReference/API_ValidatePolicy.html" rel="noopener" target="_blank">ValidatePolicy</a></li> 
 <li><a href="https://docs.aws.amazon.com/access-analyzer/latest/APIReference/API_CheckNoPublicAccess.html" rel="noopener" target="_blank">CheckNoPublicAccess</a></li> 
 <li><a href="https://docs.aws.amazon.com/access-analyzer/latest/APIReference/API_CheckAccessNotGranted.html" rel="noopener" target="_blank">CheckAccessNotGranted</a></li> 
 <li><a href="https://docs.aws.amazon.com/access-analyzer/latest/APIReference/API_CheckNoNewAccess.html" rel="noopener" target="_blank">CheckNoNewAccess</a></li> 
</ul> 
<p>We’ll walk through examples of each of these checks in the sections that follow.</p> 
<h3 id="example-1-validatepolicy">Example 1: ValidatePolicy</h3> 
<p>In this example, we use the <code style="color: #000000;">ValidatePolicy</code> option provided by the IAM policy check plugin to validate IAM policies against <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_grammar.html" rel="noopener" target="_blank"><u>IAM policy grammar</u></a> and <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html" rel="noopener" target="_blank"><u>AWS best practices</u></a>. When you run this check, you can view policy validation check findings that include security warnings, errors, general warnings, and suggestions for your policy. These actionable recommendations help you author policies that are aligned with AWS best practices.</p> 
<p><strong>To run the ValidatePolicy check</strong></p> 
<ol type="1"> 
 <li>Let’s use the following IAM policy for illustration purposes. You can see that resource <code style="color: #000000;">*</code> (a wildcard) is being used in the first statement, which indicates that the <code style="color: #000000;">iam:PassRole</code> action is allowed for all resources. 
  <div class="hide-language"> 
   <pre><code class="lang-text">{
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Action": "iam:PassRole",	
        "Resource": "*"
      },
      {
        "Effect": "Allow",
        "Action": ["s3:GetObject", "s3:PutObject"],
        "Resource": "arn:aws:s3:::amzn-s3-demo-bucket/*"
      }
    ]
  }
</code></pre> 
  </div> </li> 
 <li>In the VS Code editor, navigate to the <strong>IAM Policy Checks</strong> pane. Choose the document type <strong>JSON Policy Language</strong> and policy type <strong>Identity</strong>. Then choose <strong>Run Policy Validation</strong>. 
  <div class="wp-caption aligncenter" id="attachment_37041" style="width: 1255px;">
   <img alt="Figure 2: IAM Access Analyzer ValidatePolicy check results" class="size-full wp-image-37041" height="891" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/01/09/Implement-IAM-policy-checks-2.png" width="1245" />
   <p class="wp-caption-text" id="caption-attachment-37041">Figure 2: IAM Access Analyzer ValidatePolicy check results</p>
  </div> <p> You can see that Access Analyzer has detected an issue, which is shown in the <strong>PROBLEMS</strong> pane. </p> 
  <div class="wp-caption aligncenter" id="attachment_37042" style="width: 2970px;">
   <img alt="Figure 3: Problems pane with finding details for the ValidatePolicy check " class="size-full wp-image-37042" height="258" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/01/09/Implement-IAM-policy-checks-3.png" width="2960" />
   <p class="wp-caption-text" id="caption-attachment-37042">Figure 3: Problems pane with finding details for the ValidatePolicy check</p>
  </div> <p> The security warning shown in Figure 3 states that the <code style="color: #000000;">iam:PassRole</code> action with a wildcard (<code style="color: #000000;">*</code>) in the resource can be overly permissive because it allows the ability to pass any IAM role in that account.</p></li> 
 <li>Now, let’s modify the IAM policy by replacing the wildcard (<code style="color: #000000;">*</code>) with a specific role Amazon Resource Name (ARN). 
  <div class="hide-language"> 
   <pre><code class="lang-text">{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "iam:PassRole",
      "Resource": "arn:aws:iam::111122223333:role/sample_role"
    },
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:PutObject"],
      "Resource": "arn:aws:s3:::amzn-s3-demo-bucket/*"
    }
  ]
}
</code></pre> 
  </div> </li> 
 <li>Verify the policy again by running the <code style="color: #000000;">ValidatePolicy</code> check to make sure that it doesn’t generate findings after you updated the IAM policy.<br /> 
  <div class="wp-caption aligncenter" id="attachment_37043" style="width: 886px;">
   <img alt="Figure 4: Results of the ValidatePolicy check after IAM policy correction" class="size-full wp-image-37043" height="225" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/01/09/Implement-IAM-policy-checks-4.png" width="876" />
   <p class="wp-caption-text" id="caption-attachment-37043">Figure 4: Results of the ValidatePolicy check after IAM policy correction</p>
  </div> </li> 
</ol> 
<h3 id="example-2-checknopublicaccess">Example 2: CheckNoPublicAccess</h3> 
<p>With the <code style="color: #000000;">CheckNoPublicAccess</code> option, you can verify whether your resource policy grants public access for <a href="https://docs.aws.amazon.com/cli/latest/reference/accessanalyzer/check-no-public-access.html" rel="noopener" target="_blank"><u>supported resource types</u></a>.</p> 
<p><strong>To run the CheckNoPublicAccess check</strong></p> 
<ol type="1"> 
 <li>To test whether a policy does not allow public access, create a new bucket using a CloudFormation template and attach a resource policy that grants access to any principal to see the objects in this bucket.<br /> 
  <blockquote>
   <p><strong>WARNING:</strong> This sample bucket policy should not be used in production. Using a wildcard in the principal element of a bucket policy would allow any IAM principal to view the contents of the bucket.</p>
  </blockquote> 
  <div class="hide-language"> 
   <pre><code class="lang-text">Resources:
          MyBucket:
            Type: 'AWS::S3::Bucket'
            Properties:
              BucketName: amzn-s3-demo-bucket

          MyBucketPolicy:
            Type: 'AWS::S3::BucketPolicy'
            Properties:
              Bucket:
                Ref: 'MyBucket'
              PolicyDocument:
                Version: '2012-10-17'
                Statement:
                  - Effect: Allow
                    Principal: "*"
                    Action: 's3:GetObject'
                    Resource:
                      Fn::Join:
                        - ''
                        - - 'arn:aws:s3:::'
                          - Ref: 'MyBucket'
                          - '/*'
</code></pre> 
  </div> </li> 
 <li>Select the document type <strong>CloudFormation template</strong> and then choose <strong>Run Custom Policy Check</strong> to see whether this resource policy passes the <code style="color: #000000;">CheckNoPublicAccess</code> check. 
  <div class="wp-caption aligncenter" id="attachment_37044" style="width: 880px;">
   <img alt="Figure 5: IAM Access Analyzer CheckNoPublicAccess check results" class="size-full wp-image-37044" height="383" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/01/09/Implement-IAM-policy-checks-5.png" width="870" />
   <p class="wp-caption-text" id="caption-attachment-37044">Figure 5: IAM Access Analyzer CheckNoPublicAccess check results</p>
  </div> <p> The policy check returns a failed result because this bucket does allow public access. </p> <p> </p>
  <div class="wp-caption aligncenter" id="attachment_37045" style="width: 1333px;">
   <img alt="Figure 6: Problems pane finding details for CheckNoPublicAccess check" class="size-full wp-image-37045" height="104" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/01/09/Implement-IAM-policy-checks-6.png" width="1323" />
   <p class="wp-caption-text" id="caption-attachment-37045">Figure 6: Problems pane finding details for CheckNoPublicAccess check</p>
  </div> </li> 
 <li>Next, fix this policy to allow access from a role within the same account by restricting the policy to a specific role ARN. 
  <div class="hide-language"> 
   <pre><code class="lang-text">Resources:
          MyBucket:
            Type: 'AWS::S3::Bucket'
            Properties:
              BucketName: amzn-s3-demo-bucket

          MyBucketPolicy:
            Type: 'AWS::S3::BucketPolicy'
            Properties:
              Bucket:
                Ref: 'MyBucket'
              PolicyDocument:
                Version: '2012-10-17'
                Statement:
                  - Effect: Allow
                    Principal: 
                      "AWS": 'arn:aws:iam::111122223333:role/sample_role'
                    Action: 's3:GetObject'
                    Resource:
                      Fn::Join:
                        - ''
                        - - 'arn:aws:s3:::'
                          - Ref: 'MyBucket'
                          - '/*'
</code></pre> 
  </div> </li> 
 <li>Re-run the <code style="color: #000000;">CheckNoPublicAccess</code> check. The resource policy no longer grants public access and the status of the policy check is <strong>PASS</strong>.</li> 
</ol> 
<h3 id="example-3-checkaccessnotgranted">Example 3: CheckAccessNotGranted</h3> 
<p>The <code style="color: #000000;">CheckAccessNotGranted</code> option allows you to check whether a policy allows access to a list of IAM actions and resource ARNs. You can use this check to give developers fast feedback that certain permissions or access to certain resources are not allowed.</p> 
<p><strong>To run the CheckAccessNotGranted check</strong></p> 
<ol type="1"> 
 <li>Identify sensitive actions and resources. <p> In the VS Code editor, under <strong>Custom Policy Checks</strong>, choose the check type <code style="color: #000000;">CheckAccessNotGranted</code>. Using a comma-separated list, create a list of actions and resource ARNs that you don’t want to allow in your IAM policy. You can also create a JSON file with your actions and resources by using the syntax shown in Figure 7. For this example, set the <code style="color: #000000;">s3:PutBucketPolicy</code> and <code style="color: #000000;">dynamodb:DeleteTable</code> IAM actions to “not allowed” in the IAM policy.</p> <p> </p>
  <div class="wp-caption aligncenter" id="attachment_37047" style="width: 902px;">
   <img alt="Figure 7: Configure the CheckAccessNotGranted check" class="size-full wp-image-37047" height="574" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/01/09/Implement-IAM-policy-checks-7.png" width="892" />
   <p class="wp-caption-text" id="caption-attachment-37047">Figure 7: Configure the CheckAccessNotGranted check</p>
  </div> </li> 
 <li>Create a sample CloudFormation template that contains an IAM policy attached to an IAM role, as follows. This policy grants access to some of the actions that you deemed sensitive in Figure 7. 
  <div class="hide-language"> 
   <pre><code class="lang-text">Resources:
  CreateTagsLambdaRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
        - Effect: Allow
          Principal:
            Service: lambda.amazonaws.com
          Action: sts:AssumeRole
      Policies:
      - PolicyName: my-application-access
        PolicyDocument:
          Version: '2012-10-17'
          Statement:
          - Effect: Allow
            Action:
            - ec2:DescribeInstances
            Resource: "*"
          - Effect: Allow
            Action:
            - s3:GetObject
            - s3:PutBucketPolicy
            - dynamodb:DeleteTable
            Resource: "*"            
          
      RoleName: sample-role
</code></pre> 
  </div> </li> 
 <li>In the VS Code editor, choose <strong>Run Custom Policy Check</strong> to identify whether one of the sensitive actions or resources is allowed in the IAM policy. The policy check returns <strong>FAIL</strong> because the policy has the actions <code style="color: #000000;">s3:PutBucketPolicy</code> and <code style="color: #000000;">dynamodb:DeleteTable</code>, which you marked as actions that you don’t want developers to grant access to. Remove the restricted actions from the policy and run the check again to see a <strong>PASS</strong> result for the policy check.</li> 
</ol> 
<h3 id="example-4-checknonewaccess">Example 4: CheckNoNewAccess</h3> 
<p>The <code style="color: #000000;">CheckNoNewAccess</code> option is a custom policy check that verifies whether your policy grants new access compared to a reference policy.</p> 
<p>You use a reference policy to check whether a candidate policy allows more access than the reference policy does. In other words, the check passes if the candidate policy is a subset of the reference policy. A reference policy typically starts by allowing all access. You then add a statement or statements that deny the access that you want the reference policy to check for. For more details and examples of reference policies, see the <a href="https://github.com/aws-samples/iam-access-analyzer-custom-policy-check-samples" rel="noopener" target="_blank"><u>iam-access-analyzer-custom-policy-check-samples</u></a> repository on GitHub.</p> 
<p>The ability to use a reference policy provides you with the flexibility to look for almost anything in an IAM policy. This is useful when you have custom requirements for your organization that may not be met with some of the other custom policy checks.</p> 
<p><strong>To run the CheckNoNewAccess check</strong></p> 
<ol type="1"> 
 <li>Create a reference policy: In your project, create a new JSON policy document that will serve as your reference policy. <p> The following reference policy checks that an IAM role trust policy only grants access to an allowlisted set of AWS services. This enables you to allow builders to create roles, but constrain the use of those roles to the set of AWS services specified. </p> <p> In this reference policy, only the specified AWS service principals <code style="color: #000000;">ec2.amazonaws.com</code>, <code style="color: #000000;">lambda.amazonaws.com</code>, and <code style="color: #000000;">ecs-tasks.amazonaws.com</code> are allowed to assume the role.</p> 
  <div class="hide-language"> 
   <pre><code class="lang-text">{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowThisSetOfServicePrincipals",
      "Effect": "Allow",
      "Principal": {
        "Service": [
          "ec2.amazonaws.com",
          "lambda.amazonaws.com",
          "ecs-tasks.amazonaws.com"
        ]
      },
      "Action": "sts:AssumeRole"
    },
    {
      "Sid": "AllowOtherSTSActions",
      "Effect": "Allow",
      "Principal": "*",
      "NotAction": "sts:AssumeRole"
    }
  ]
}
</code></pre> 
  </div> </li> 
 <li>Enter the reference policy in the VS Code editor. In the <strong>IAM Policy Checks</strong> pane, select the check type <strong>CheckNoNewAccess</strong>. Then set the reference policy type to <strong>Resource</strong>, because this is a trust policy that defines which principals can assume the role. In addition, provide the path of the reference policy that you created in Step 1. You can also directly enter the reference policy as a JSON policy document, as shown in Figure 8.<br /> 
  <div class="wp-caption aligncenter" id="attachment_37048" style="width: 1036px;">
   <img alt="Figure 8: Enter the reference policy for the CheckNoNewAccess check" class="size-full wp-image-37048" height="896" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/01/09/Implement-IAM-policy-checks-8.png" width="1026" />
   <p class="wp-caption-text" id="caption-attachment-37048">Figure 8: Enter the reference policy for the CheckNoNewAccess check</p>
  </div> </li> 
 <li>Create a CloudFormation template, as follows. This template creates an IAM role that allows the AWS service principals <code style="color: #000000;">lambda.amazonaws.com</code> and <code style="color: #000000;">glue.amazonaws.com</code> to assume the <code style="color: #000000;">sample-application-role</code> IAM role. 
  <div class="hide-language"> 
   <pre><code class="lang-text">Resources:
  SampleApplicationRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
        - Effect: Allow
          Principal:
            Service: 
            - lambda.amazonaws.com
            - glue.amazonaws.com
          Action: sts:AssumeRole
      Policies:
      - PolicyName: my-application-access
        PolicyDocument:
          Version: '2012-10-17'
          Statement:
          - Effect: Allow
            Action:
            - s3:GetObject
            Resource: "arn:aws:s3::111122223333:amzn-s3-demo-bucket/*"            
      RoleName: sample-application-role
</code></pre> 
  </div> </li> 
 <li>In the VS Code editor, choose <strong>Run Custom Policy Check</strong> to check your CloudFormation template against the reference policy you configured in Step 1. The check will return <strong>FAIL</strong> and you will see a security warning in the editor in the <strong>PROBLEMS</strong> pane. 
  <div class="wp-caption aligncenter" id="attachment_37049" style="width: 1389px;">
   <img alt="Figure 9: Problems pane finding details for the CheckNoNewAccess check " class="size-full wp-image-37049" height="100" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/01/09/Implement-IAM-policy-checks-9.png" width="1379" />
   <p class="wp-caption-text" id="caption-attachment-37049">Figure 9: Problems pane finding details for the CheckNoNewAccess check</p>
  </div> <p> The issue is that <code style="color: #000000;">glue.amazonaws.com</code> was not listed as a service principal that was allowed to assume a role in your reference policy. You can remove <code style="color: #000000;">glue.amazonaws.com</code> from the CloudFormation template and re-run the check to receive a <strong>PASS</strong> result.</p></li> 
</ol> 
<h2 id="conclusion">Conclusion</h2> 
<p>In this post, we explored how you can use the integration of VS Code with IAM Access Analyzer in your development workflow to make sure that your IAM policies align with best practices and adhere to your organization’s security requirements. The four critical checks provided by IAM Access Analyzer can be summarized as follows:</p> 
<ul> 
 <li>The <code style="color: #000000;">ValidatePolicy</code> check provides actionable recommendations that help you author policies that are aligned with AWS best practices.</li> 
 <li>The <code style="color: #000000;">CheckNoPublicAccess</code> check helps protect resources from being exposed publicly and mitigates the risk of unauthorized public access.</li> 
 <li>The <code style="color: #000000;">CheckAccesNotGranted</code> check looks for specific IAM actions and resource ARNs to help enforce access restrictions and help prevent unauthorized access to critical data or services.</li> 
 <li>The <code style="color: #000000;">CheckNoNewAccess</code> check validates that the permissions granted in your IAM policies remain within the intended scope, as defined by your organization’s requirements.</li> 
</ul> 
<p>Install or update the <a href="https://aws.amazon.com/visualstudiocode/" rel="noopener" target="_blank">AWS Toolkit for VS Code</a> today, and make sure that you have the <a href="https://github.com/awslabs/aws-cloudformation-iam-policy-validator" rel="noopener" target="_blank">CloudFormation Policy Validator</a> or <a href="https://github.com/awslabs/terraform-iam-policy-validator" rel="noopener" target="_blank">Terraform Policy Validator</a>, to take advantage of these features.</p> 
<p>If you have feedback about this post, submit comments in the <strong>Comments</strong> section below. </p> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image"> 
   <img alt="Anshu Bathla" class="aligncenter size-full wp-image-37053" height="160" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/01/09/Anshu-Bathla.jpg" width="120" /> 
  </div> 
  <h3 class="lb-h4">Anshu Bathla</h3> 
  <p>Anshu is a Lead Consultant – SRC at AWS, based in Gurugram, India. He works with customers across diverse verticals to help strengthen their security infrastructure and achieve their security goals. Outside of work, Anshu enjoys reading books and gardening at his home garden.</p> 
  <p></p>
 </div> 
</footer> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image"> 
   <img alt="Manoj Kumar" class="aligncenter size-full wp-image-37060" height="160" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/01/11/Manoj-Kumar.jpg" width="120" /> 
  </div> 
  <h3 class="lb-h4">Manoj Kumar</h3> 
  <p>Manoj is a Lead Consultant – SRC at AWS, based in Gurugram, India. He collaborates with diverse clients to design and implement comprehensive AWS Cloud security solutions. His expertise helps organizations fortify their cloud infrastructures, achieve compliance objectives, and provide robust data protection while using the advanced security features of AWS to support their business objectives.</p> 
  <p></p>
 </div> 
</footer>
