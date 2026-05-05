---
title: "Validate IAM policies by using IAM Policy Validator for AWS CloudFormation and GitHub Actions"
url: "https://aws.amazon.com/blogs/security/validate-iam-policies-by-using-iam-policy-validator-for-aws-cloudformation-and-github-actions/"
date: "Wed, 30 Aug 2023 13:04:28 +0000"
author: "Mitch Beaumont"
feed_url: "https://aws.amazon.com/blogs/security/tag/iam-access-analyzer/feed/"
---
<blockquote>
 <p><strong>April 15, 2024:</strong> AWS has launched two new GitHub Actions that can be used to simplify some of the steps covered in this blog post. Click here to learn more abbot the new GitHub actions for <a href="https://github.com/aws-actions/cloudformation-aws-iam-policy-validator" rel="noopener" target="_blank">AWS CloudFormation</a> and <a href="https://github.com/aws-actions/terraform-aws-iam-policy-validator" rel="noopener" target="_blank">HashiCorp’s Terraform</a>.</p>
</blockquote> 
<hr /> 
<p>In this blog post, I’ll show you how to automate the validation of <a href="https://aws.amazon.com/iam/" rel="noopener" target="_blank">AWS Identity and Access Management (IAM)</a> policies by using a combination of the <a href="https://github.com/awslabs/aws-cloudformation-iam-policy-validator" rel="noopener" target="_blank">IAM Policy Validator for AWS CloudFormation (cfn-policy-validator)</a> and <a href="https://github.com/features/actions" rel="noopener" target="_blank">GitHub Actions</a>. Policy validation is an approach that is designed to minimize the deployment of unwanted IAM identity-based and resource-based policies to your <a href="https://aws.amazon.com/" rel="noopener" target="_blank">Amazon Web Services (AWS)</a> environments. </p> 
<p>With GitHub Actions, you can automate, customize, and run software development workflows directly within a repository. Workflows are defined using YAML and are stored alongside your code. I’ll discuss the specifics of how you can set up and use GitHub actions within a repository in the sections that follow.</p> 
<p>The cfn-policy-validator tool is a command-line tool that takes an <a href="https://aws.amazon.com/cloudformation/" rel="noopener" target="_blank">AWS CloudFormation</a> template, finds and parses the IAM policies that are attached to IAM roles, users, groups, and resources, and then runs the policies through <a href="https://aws.amazon.com/iam/features/analyze-access/" rel="noopener" target="_blank">IAM Access Analyzer</a> <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-reference-policy-checks.html" rel="noopener" target="_blank">policy checks</a>. Implementing IAM policy validation checks at the time of code check-in helps shift security to the left (closer to the developer) and shortens the time between when developers commit code and when they get feedback on their work.</p> 
<p>Let’s walk through an example that checks the policies that are attached to an IAM role in a CloudFormation template. In this example, the cfn-policy-validator tool will find that the trust policy attached to the IAM role allows the role to be assumed by external principals. This configuration could lead to unintended access to your resources and data, which is a security risk.</p> 
<h2>Prerequisites</h2> 
<p>To complete this example, you will need the following:</p> 
<ol> 
 <li>A GitHub account</li> 
 <li>An AWS account, and an identity within that account that has permissions to create the IAM roles and resources used in this example</li> 
</ol> 
<h2 id="step_1">Step 1: Create a repository that will host the CloudFormation template to be validated</h2> 
<p>To begin with, you need to create a GitHub repository to host the CloudFormation template that is going to be validated by the cfn-policy-validator tool.</p> 
<h3>To create a repository:</h3> 
<ol> 
 <li>Open a browser and go to <a href="https://github.com/" rel="noopener" target="_blank">https://github.com</a>.</li> 
 <li>In the upper-right corner of the page, in the drop-down menu, choose <strong>New repository</strong>. For <strong>Repository name</strong>, enter a short, memorable name for your repository.</li> 
 <li>(Optional) Add a <strong>description</strong> of your repository.</li> 
 <li>Choose either the option <strong>Public (the repository is accessible to everyone on the internet) </strong>or <strong>Private (the repository is accessible only to people access is explicitly shared with).</strong></li> 
 <li>Choose <strong>Initialize this repository with: Add a README file</strong>.</li> 
 <li>Choose <strong>Create repository</strong>. Make a note of the repository’s name.</li> 
</ol> 
<h2 id="step_2">Step 2: Clone the repository locally</h2> 
<p>Now that the repository has been created, clone it locally and add a CloudFormation template.</p> 
<h3>To clone the repository locally and add a CloudFormation template:</h3> 
<ol> 
 <li>Open the command-line tool of your choice.</li> 
 <li>Use the following command to clone the new repository locally. Make sure to replace <span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;GitHubOrg&gt;</span> and <span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;RepositoryName&gt;</span> with your own values. 
  <div class="hide-language"> 
   <pre><code class="lang-text">git clone git@github.com:<span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;GitHubOrg&gt;</span>/<span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;RepositoryName&gt;</span>.git</code></pre> 
  </div> </li> 
 <li>Change in to the directory that contains the locally-cloned repository. 
  <div class="hide-language"> 
   <pre><code class="lang-text">cd <span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;RepositoryName&gt;</span></code></pre> 
  </div> <p>Now that the repository is locally cloned, populate the locally-cloned repository with the following sample CloudFormation template. This template creates a single IAM role that allows a principal to assume the role to perform the <span style="font-family: courier;">S3:GetObject</span> action.</p> </li> 
 <li>Use the following command to create the sample CloudFormation template file.<br /> 
  <blockquote>
   <p><strong>WARNING:</strong> This sample role and policy <strong>should not</strong> be used in production. Using a wildcard in the principal element of a role’s trust policy would allow any IAM principal in any account to assume the role.</p>
  </blockquote> 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">cat &lt;&lt; EOF &gt; sample-role.yaml

AWSTemplateFormatVersion: "2010-09-09"
Description: Base stack to create a simple role
Resources:
  SampleIamRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Statement:
          - Effect: Allow
            Principal:
              AWS: "*"
            Action: ["sts:AssumeRole"]
      Path: /      
      Policies:
        - PolicyName: root
          PolicyDocument:
            Version: 2012-10-17
            Statement:
              - Resource: "*"
                Effect: Allow
                Action:
                  - s3:GetObject
EOF</code></pre> 
  </div> </li> 
</ol> 
<p>Notice that <a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-iam-role.html#cfn-iam-role-assumerolepolicydocument" rel="noopener" target="_blank">AssumeRolePolicyDocument</a> refers to a trust policy that includes a wildcard value in the principal element. This means that the role could potentially be assumed by an external identity, and that’s a risk you want to know about.</p> 
<h2>Step 3: Vend temporary AWS credentials for GitHub Actions workflows</h2> 
<p>In order for the cfn-policy-validator tool that’s running in the GitHub Actions workflow to use the IAM Access Analyzer API, the GitHub Actions workflow needs a set of temporary AWS credentials. The <a href="https://github.com/marketplace/actions/configure-aws-credentials-for-github-actions" rel="noopener" target="_blank">AWS Credentials for GitHub Actions</a> action helps address this requirement. This action implements the AWS SDK credential resolution chain and exports environment variables for other actions to use in a workflow. Environment variable exports are detected by the cfn-policy-validator tool.</p> 
<p>AWS Credentials for GitHub Actions supports four methods for fetching credentials from AWS, but the recommended approach is to use GitHub’s OpenID Connect (OIDC) provider in conjunction with a configured <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers.html" rel="noopener" target="_blank">IAM identity provider</a> endpoint.</p> 
<h3>To configure an IAM identity provider endpoint for use in conjunction with GitHub’s OIDC provider:</h3> 
<ol> 
 <li>Open the <a href="https://console.aws.amazon.com/console/home" rel="noopener" target="_blank">AWS Management Console</a> and navigate to IAM.</li> 
 <li>In the left-hand menu, choose <strong>Identity providers</strong>, and then choose <strong>Add provider</strong>.</li> 
 <li>For <strong>Provider type</strong>, choose <strong>OpenID Connect</strong>.</li> 
 <li>For <strong>Provider URL</strong>, enter<br /><span style="font-family: courier;">https://token.actions.githubusercontent.com</span></li> 
 <li>Choose <strong>Get thumbprint</strong>.</li> 
 <li>For <strong>Audiences</strong>, enter <span style="font-family: courier;">sts.amazonaws.com</span></li> 
 <li>Choose <strong>Add provider</strong> to complete the setup.</li> 
</ol> 
<p>At this point, make a note of the OIDC provider name. You’ll need this information in the next step.</p> 
<p>After it’s configured, the IAM identity provider endpoint should look similar to the following:</p> 
<div class="wp-caption aligncenter" id="attachment_30741" style="width: 790px;">
 <img alt="Figure 1: IAM Identity provider details" class="size-full wp-image-30741" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/08/28/img1-8.png" style="border: 1px solid #bebebe;" width="780" />
 <p class="wp-caption-text" id="caption-attachment-30741">Figure 1: IAM Identity provider details</p>
</div> 
<h2>Step 4: Create an IAM role with permissions to call the IAM Access Analyzer API</h2> 
<p>In this step, you will create an IAM role that can be assumed by the GitHub Actions workflow and that provides the necessary permissions to run the cfn-policy-validator tool.</p> 
<h3>To create the IAM role:</h3> 
<ol> 
 <li>In the IAM console, in the left-hand menu, choose<strong> Roles</strong>, and then choose <strong>Create role</strong>. </li> 
 <li>For <strong>Trust entity type</strong>, choose <strong>Web identity</strong>.</li> 
 <li>In the <strong>Provider</strong> list, choose the new GitHub OIDC provider that you created in the earlier step. For <strong>Audience</strong>, select <strong>sts.amazonaws.com</strong> from the list.</li> 
 <li>Choose <strong>Next</strong>.</li> 
 <li>On the <strong>Add permission</strong> page, choose <strong>Create policy</strong>.</li> 
 <li>Choose <strong>JSON</strong>, and enter the following policy: 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
              "iam:GetPolicy",
              "iam:GetPolicyVersion",
              "access-analyzer:ListAnalyzers",
              "access-analyzer:ValidatePolicy",
              "access-analyzer:CreateAccessPreview",
              "access-analyzer:GetAccessPreview",
              "access-analyzer:ListAccessPreviewFindings",
              "access-analyzer:CreateAnalyzer",
              "s3:ListAllMyBuckets",
              "cloudformation:ListExports",
              "ssm:GetParameter"
            ],
            "Resource": "*"
        },
        {
          "Effect": "Allow",
          "Action": "iam:CreateServiceLinkedRole",
          "Resource": "*",
          "Condition": {
            "StringEquals": {
              "iam:AWSServiceName": "access-analyzer.amazonaws.com"
            }
          }
        } 
    ]
}</code></pre> 
  </div> </li> 
 <li>After you’ve attached the new policy, choose <strong>Next</strong>.<br /> 
  <blockquote>
   <p><strong>Note:</strong> For a full explanation of each of these actions and a CloudFormation template example that you can use to create this role, see the <a href="https://github.com/awslabs/aws-cloudformation-iam-policy-validator#iam-policy-required-to-run-the-iam-policy-validator-for-aws-cloudformation" rel="noopener" target="_blank">IAM Policy Validator for AWS CloudFormation</a> GitHub project.</p>
  </blockquote> </li> 
 <li>Give the role a name, and scroll down to look at <strong><a href="#step_1">Step 1: Select trusted entities</a>.</strong> <p>The default policy you just created allows GitHub Actions from organizations or repositories outside of your control to assume the role. To align with the IAM best practice of <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#grant-least-privilege" rel="noopener" target="_blank">granting least privilege</a>, let’s scope it down further to only allow a specific GitHub organization and the repository that you created earlier to assume it.</p> </li> 
 <li>Replace the policy to look like the following, <strong>but</strong> don’t forget to replace {AWSAccountID}, {GitHubOrg} and {RepositoryName} with your own values. 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Federated": "arn:aws:iam::{AWSAccountID}:oidc-provider/token.actions.githubusercontent.com"
            },
            "Action": "sts:AssumeRoleWithWebIdentity",
            "Condition": {
                "StringEquals": {
                    "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
                },
                "StringLike": {
                    "token.actions.githubusercontent.com:sub": "repo:${GitHubOrg}/${RepositoryName}:*"
                }
            }
        }
    ]
}</code></pre> 
  </div> </li> 
</ol> 
<p>For information on best practices for configuring a role for the GitHub OIDC provider, see <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-idp_oidc.html" rel="noopener" target="_blank">Creating a role for web identity or OpenID Connect Federation (console)</a>.</p> 
<h2>Checkpoint</h2> 
<p>At this point, you’ve created and configured the following resources:</p> 
<ul> 
 <li>A GitHub repository that has been locally cloned and filled with a sample CloudFormation template.</li> 
 <li>An IAM identity provider endpoint for use in conjunction with GitHub’s OIDC provider.</li> 
 <li>A role that can be assumed by GitHub actions, and a set of associated permissions that allow the role to make requests to IAM Access Analyzer to validate policies.</li> 
</ul> 
<h2>Step 5: Create a definition for the GitHub Actions workflow</h2> 
<p>The workflow runs steps on <a href="https://docs.github.com/en/actions/using-github-hosted-runners/about-github-hosted-runners" rel="noopener" target="_blank">hosted runners</a>. For this example, we are going to use Ubuntu as the operating system for the hosted runners. The workflow runs the following steps on the runner:</p> 
<ol> 
 <li>The workflow checks out the CloudFormation template by using the community <span style="font-family: courier;">actions/checkout</span> action.</li> 
 <li>The workflow then uses the <span style="font-family: courier;">aws-actions/configure-aws-credentials</span> GitHub action to request a set of credentials through the IAM identity provider endpoint and the IAM role that you created earlier.</li> 
 <li>The workflow installs the cfn-policy-validator tool by using the <a href="https://pypi.org/project/cfn-policy-validator/" rel="noopener" target="_blank">python package manager, PIP</a>.</li> 
 <li>The workflow runs a validation against the CloudFormation template by using the cfn-policy-validator tool.</li> 
</ol> 
<p>The workflow is defined in a YAML document. In order for GitHub Actions to pick up the workflow, you need to place the definition file in a specific location within the repository: <span style="font-family: courier;">.github/workflows/main.yml</span>. Note the “.” prefix in the directory name, indicating that this is a hidden directory.</p> 
<h3>To create the workflow:</h3> 
<ol> 
 <li>Use the following command to create the folder structure within the locally cloned repository: 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-html">mkdir -p .github/workflows</code></pre> 
  </div> </li> 
 <li>Create the sample workflow definition file in the .github/workflows directory. Make sure to replace <span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;AWSAccountID&gt;</span> and <span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;AWSRegion&gt;</span> with your own information. 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">cat &lt;&lt; EOF &gt; .github/workflows/main.yml
name: cfn-policy-validator-workflow

on: push

permissions:
  id-token: write
  contents: read

jobs: 
  cfn-iam-policy-validation: 
    name: iam-policy-validation
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          role-to-assume: arn:aws:iam::<span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;AWSAccountID&gt;</span>:role/github-actions-access-analyzer-role
          aws-region: <span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;AWSRegion&gt;</span>
          role-session-name: GitHubSessionName
        
      - name: Install cfn-policy-validator
        run: pip install cfn-policy-validator

      - name: Validate templates
        run: cfn-policy-validator validate --template-path ./sample-role-test.yaml --region <span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;AWSRegion&gt;</span>
EOF
</code></pre> 
  </div> </li> 
</ol> 
<h2>Step 6: Test the setup</h2> 
<p>Now that everything has been set up and configured, it’s time to test.</p> 
<h3>To test the workflow and validate the IAM policy:</h3> 
<ol> 
 <li>Add and commit the changes to the local repository. 
  <div class="hide-language"> 
   <pre><code class="lang-text">git add .
git commit -m ‘added sample cloudformation template and workflow definition’</code></pre> 
  </div> </li> 
 <li>Push the local changes to the remote GitHub repository. 
  <div class="hide-language"> 
   <pre><code class="lang-text">git push</code></pre> 
  </div> <p>After the changes are pushed to the remote repository, go back to <a href="https://github.com" rel="noopener" target="_blank">https://github.com</a> and open the repository that you created earlier. In the top-right corner of the repository window, there is a small orange indicator, as shown in Figure 2. This shows that your GitHub Actions workflow is running.</p> 
  <div class="wp-caption aligncenter" id="attachment_30747" style="width: 750px;">
   <img alt="Figure 2: GitHub repository window with the orange workflow indicator" class="size-full wp-image-30747" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/08/29/img2-7.png" width="740" />
   <p class="wp-caption-text" id="caption-attachment-30747">Figure 2: GitHub repository window with the orange workflow indicator</p>
  </div> <p>Because the sample CloudFormation template used a wildcard value “*” in the principal element of the policy as described in the section <strong><a href="#step_2">Step 2: Clone the repository locally</a></strong>, the orange indicator turns to a red x (shown in Figure 3), which signals that something failed in the workflow.</p> 
  <div class="wp-caption aligncenter" id="attachment_30748" style="width: 750px;">
   <img alt="Figure 3: GitHub repository window with the red cross workflow indicator" class="size-full wp-image-30748" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/08/29/img3-9.png" width="740" />
   <p class="wp-caption-text" id="caption-attachment-30748">Figure 3: GitHub repository window with the red cross workflow indicator</p>
  </div> </li> 
 <li>Choose the red x to see more information about the workflow’s status, as shown in Figure 4. 
  <div class="wp-caption aligncenter" id="attachment_30749" style="width: 750px;">
   <img alt="Figure 4: Pop-up displayed after choosing the workflow indicator" class="size-full wp-image-30749" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/08/29/img4-5.png" width="740" />
   <p class="wp-caption-text" id="caption-attachment-30749">Figure 4: Pop-up displayed after choosing the workflow indicator</p>
  </div> </li> 
 <li>Choose <strong>Details</strong> to review the workflow logs. <p>In this example, the <strong>Validate templates</strong> step in the workflow has failed. A closer inspection shows that there is a blocking finding with the CloudFormation template. As shown in Figure 5, the finding is labelled as <span style="font-family: courier;">EXTERNAL_PRINCIPAL</span> and has a description of <span style="font-family: courier;">Trust policy allows access from external principals</span>.</p> 
  <div class="wp-caption aligncenter" id="attachment_30750" style="width: 750px;">
   <img alt="Figure 5: Details logs from the workflow showing the blocking finding" class="size-full wp-image-30750" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/08/29/img5-4.png" width="740" />
   <p class="wp-caption-text" id="caption-attachment-30750">Figure 5: Details logs from the workflow showing the blocking finding</p>
  </div> <p>To remediate this blocking finding, you need to update the principal element of the trust policy to include a principal from your AWS account (considered a zone of trust). The resources and principals within your account comprises of the zone of trust for the cfn-policy-validator tool. In the initial version of <span style="font-family: courier;">sample-role.yaml</span>, the IAM roles trust policy used a wildcard in the Principal element. This allowed principals outside of your control to assume the associated role, which caused the cfn-policy-validator tool to generate a blocking finding.</p> <p>In this case, the intent is that principals within the current AWS account (zone of trust) should be able to assume this role. To achieve this result, replace the wildcard value with the <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_principal.html#principal-accounts" rel="noopener" target="_blank">account principal</a> by following the remaining steps.</p> </li> 
 <li>Open <span style="font-family: courier;">sample-role.yaml</span> by using your preferred text editor, such as nano. 
  <div class="hide-language"> 
   <pre><code class="lang-text">nano sample-role.yaml</code></pre> 
  </div> <p>Replace the wildcard value in the principal element with the account principal <span style="font-family: courier;">arn:aws:iam::<span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;AccountID&gt;</span>:root</span>. Make sure to replace <span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;AWSAccountID&gt;</span> with your own AWS account ID.</p> 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">AWSTemplateFormatVersion: "2010-09-09"
Description: Base stack to create a simple role
Resources:
  SampleIamRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Statement:
          - Effect: Allow
            Principal:
              AWS: "arn:aws:iam::<span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;AccountID&gt;</span>:root"
            Action: ["sts:AssumeRole"]
      Path: /      
      Policies:
        - PolicyName: root
          PolicyDocument:
            Version: 2012-10-17
            Statement:
              - Resource: "*"
                Effect: Allow
                Action:
                  - s3:GetObject</code></pre> 
  </div> </li> 
 <li>Add the updated file, commit the changes, and push the updates to the remote GitHub repository. 
  <div class="hide-language"> 
   <pre><code class="lang-text">git add sample-role.yaml
git commit -m ‘replacing wildcard principal with account principal’
git push</code></pre> 
  </div> </li> 
</ol> 
<p>After the changes have been pushed to the remote repository, go back to <a href="https://github.com" rel="noopener" target="_blank">https://github.com</a> and open the repository. The orange indicator in the top right of the window should change to a green tick (check mark), as shown in Figure 6.</p> 
<div class="wp-caption aligncenter" id="attachment_30751" style="width: 790px;">
 <img alt="Figure 6: GitHub repository window with the green tick workflow indicator" class="size-full wp-image-30751" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/08/29/img6-4.png" width="780" />
 <p class="wp-caption-text" id="caption-attachment-30751">Figure 6: GitHub repository window with the green tick workflow indicator</p>
</div> 
<p>This indicates that no blocking findings were identified, as shown in Figure 7.</p> 
<div class="wp-caption aligncenter" id="attachment_30752" style="width: 790px;">
 <img alt="Figure 7: Detailed logs from the workflow showing no more blocking findings" class="size-full wp-image-30752" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/08/29/img7-4.png" width="780" />
 <p class="wp-caption-text" id="caption-attachment-30752">Figure 7: Detailed logs from the workflow showing no more blocking findings</p>
</div> 
<h2>Conclusion</h2> 
<p>In this post, I showed you how to automate IAM policy validation by using GitHub Actions and the IAM Policy Validator for CloudFormation. Although the example was a simple one, it demonstrates the benefits of automating security testing at the start of the development lifecycle. This is often referred to as <em>shifting security left</em>. Identifying misconfigurations early and automatically supports an iterative, fail-fast model of continuous development and testing. Ultimately, this enables teams to make security an inherent part of a system’s design and architecture and can speed up product development workflows.</p> 
<p>In addition to the example I covered today, IAM Policy Validator for CloudFormation can validate IAM policies by using a range of IAM Access Analyzer policy checks. For more information about these policy checks, see <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-reference-policy-checks.html" rel="noopener" target="_blank">Access Analyzer reference policy checks</a>.</p> 
<p>&nbsp;<br />If you have feedback about this post, submit comments in the<strong> Comments</strong> section below. If you have questions about this post, <a href="https://console.aws.amazon.com/support/home" rel="noopener noreferrer" target="_blank">contact AWS Support</a>.</p> 
<p><strong>Want more AWS Security news? Follow us on <a href="https://twitter.com/AWSsecurityinfo" rel="noopener noreferrer" target="_blank" title="Twitter">Twitter</a>.</strong></p> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image"> 
   <img alt="Mitch Beaumont" class="alignnone size-full wp-image-30551" height="160" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/08/16/beaumonm.jpg" width="120" /> 
  </div> 
  <h3 class="lb-h4">Mitch Beaumont</h3> 
  <p>Mitch is a Principal Solutions Architect for Amazon Web Services, based in Sydney, Australia. Mitch works with some of Australia’s largest financial services customers, helping them to continually raise the security bar for the products and features that they build and ship. Outside of work, Mitch enjoys spending time with his family, photography, and surfing.</p> 
 </div> 
</footer>
