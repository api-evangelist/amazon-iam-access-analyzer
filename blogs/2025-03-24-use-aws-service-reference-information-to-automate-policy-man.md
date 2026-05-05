---
title: "Use AWS service reference information to automate policy management workflows"
url: "https://aws.amazon.com/blogs/security/use-aws-service-reference-information-to-automate-policy-management-workflows/"
date: "Mon, 24 Mar 2025 16:03:17 +0000"
author: "Ramesh Rajan"
feed_url: "https://aws.amazon.com/blogs/security/tag/iam-access-analyzer/feed/"
---
<p><a href="https://aws.amazon.com/" rel="noopener" target="_blank">Amazon Web Services (AWS)</a> provides service reference information in JSON format to help you automate policy management workflows. With the service reference information, you can access available actions across AWS services from machine-readable files. The service reference information helps to address a key customer need: keeping up with the ever-growing list of services and actions in AWS. As new services launch and existing services expand their capabilities, you can now conveniently identify and incorporate available actions, resources, and condition keys for each AWS service into your policy authoring and validation workflows. As your business expands and your AWS footprint grows, you might decide to automate your policy management workflows. With the service authorization reference, you can build custom tools to make it easier to evaluate and use new actions, resources, and condition keys that AWS services introduce.</p> 
<h2>Getting started with service reference information</h2> 
<p>The service reference information is static information about the actions, resources, and condition keys available for each service in AWS. To obtain the list of AWS services for which reference information is available, go to the following URL:<br /> <a href="https://servicereference.us-east-1.amazonaws.com/v1/service-list.json" rel="noopener" target="_blank">https://servicereference.us-east-1.amazonaws.com/v1/service-list.json</a></p> 
<p>This URL endpoint provides a JSON file that contains an up-to-date catalog of AWS services with available reference information. By querying this endpoint, you can retrieve the most current list of services supported by the AWS Service Reference Information feature.</p> 
<p>To retrieve the list of actions, resources, and condition keys for a specific AWS service, go to the following URL:<br /> https://servicereference.us-east-1.amazonaws.com/v1/<span style="color: #ff0000;"><em>&lt;service-name&gt;</em></span>/<span style="color: #ff0000;"><em>&lt;service-name&gt;</em></span>.json</p> 
<p>Replace <span style="color: #ff0000;"><em>&lt;service-name&gt;</em></span> with the name of the desired AWS service (for example, “s3” for <a href="https://aws.amazon.com/s3/" rel="noopener" target="_blank">Amazon Simple Storage service (Amazon S3)</a> or “ec2” for <a href="https://aws.amazon.com/ec2/" rel="noopener" target="_blank">Amazon Elastic Compute Cloud (Amazon EC2)</a>). This URL endpoint provides a JSON file that contains the comprehensive list of actions, resources, and condition keys that are available for that particular service.</p> 
<p>The following example shows the format of the output from the service-list.json file, which contains the service names and URLs for each service’s reference information:</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-text">[ 
    {
"service": "s3", 
        "url": "https://servicereference.us-east-1.amazonaws.com/v1/s3/s3.json" 
    }, 
    {
"service": "dynamodb", 
        "url": "https://servicereference.us-east-1.amazonaws.com/v1/dynamodb/dynamodb.json" 
    }, 
    …
]</code></pre> 
</div> 
<p>You can navigate to the service information page by using the <code style="color: #000000;">url</code> field to view the list of permissions for the service. You can also download the JSON file to use in your policy authoring workflows. For example, you can download the permissions for Amazon S3 by following this URL:<br /> <a href="https://servicereference.us-east-1.amazonaws.com/v1/s3/s3.json" rel="noopener" target="_blank">https://servicereference.us-east-1.amazonaws.com/v1/s3/s3.json</a></p> 
<p>The following example shows a partial output of the permissions for Amazon S3. The <a href="https://aws.amazon.com/iam/" rel="noopener" target="_blank">AWS Identity and Access Management (IAM)</a> actions are available in JSON format, and each action is its own JSON object. The <code style="color: #000000;">Name</code> field for those objects provides the name of the IAM action, the <code style="color: #000000;">ActionConditionKeys</code> field provides the available condition keys for this action, and the <code style="color: #000000;">Resources</code> field provides the available resources for this action.</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-text">{
  "Name" : "s3",
  "Actions" : [ {
    "Name" : "AbortMultipartUpload",
    "ActionConditionKeys" : [ "s3:AccessGrantsInstanceArn", "s3:AccessPointNetworkOrigin", "s3:DataAccessPointAccount", "s3:DataAccessPointArn", "s3:ResourceAccount", "s3:TlsVersion", "s3:authType", "s3:signatureAge", "s3:signatureversion", "s3:x-amz-content-sha256" ],
    "Resources" : [ {
      "Name" : "object"
    } ]
  }, {
    "Name" : "AssociateAccessGrantsIdentityCenter",
    "ActionConditionKeys" : [ "aws:ResourceTag/${TagKey}", "s3:ResourceAccount", "s3:TlsVersion", "s3:authType", "s3:signatureAge", "s3:signatureversion", "s3:x-amz-content-sha256" ],
    "Resources" : [ {
      "Name" : "accessgrantsinstance"
    } ],
    "Version": "v1.1" 
}</code></pre> 
</div> 
<h2>What can you build with the service reference information?</h2> 
<p>Let’s explore how you can make use of the service reference information through practical examples. To help you get started, here are two custom tools that use the service reference information. You can find these tools in our GitHub repository, ready for you to use and adapt to your specific needs. You can download the source code for these tools by visiting the following links:</p> 
<ul> 
 <li><a href="https://github.com/aws-samples/service-control-policy-preprocessor" rel="noopener" target="_blank">Service control policy (SCP) pre-processor</a></li> 
 <li><a href="https://github.com/aws-samples/service-authorization-reference-diff" rel="noopener" target="_blank">Notification tool for new or removed IAM actions</a></li> 
</ul> 
<h3>SCP pre-processor</h3> 
<p>The SCP pre-processor provides a convenient way to write SCPs.&nbsp;You run the SCP pre-processor as a command-line tool. The tool takes a&nbsp;single, monolithic JSON file and&nbsp;runs a series of transformations and optimizations, then outputs a collection of valid service control policies that fit within policy size quotas. The tool uses AWS service&nbsp;reference&nbsp;information data in order to optimize lists of IAM actions.</p> 
<h3>Notification tool for new or removed IAM actions</h3> 
<p>You might find yourself needing to update various policies throughout your AWS environment when new IAM actions or services are released. You can use this tool to notify you when new services or new actions are added or removed. It works by downloading the service reference information and comparing it to the previous version of the file when the tool last ran. You can use these notifications to perform actions like automatically updating IAM policies when new actions are added or manually reviewing the notifications for new, sensitive actions.</p> 
<p>Visit the source code repositories for the <a href="https://github.com/aws-samples/service-control-policy-preprocessor" rel="noopener" target="_blank">SCP pre-processor</a> and the <a href="https://github.com/aws-samples/service-authorization-reference-diff" rel="noopener" target="_blank">daily notification tool</a> to learn more.</p> 
<h2>Conclusion</h2> 
<p>The AWS service&nbsp;reference&nbsp;information makes it easier for you to create automation for policy authoring and validation. By providing the AWS service actions reference in JSON format, this feature enables you to create custom tools for policy authoring and management.</p> 
<p>We’re excited to know what kind of policy authoring tools you can think up.</p> 
<p>&nbsp;<br />If you have feedback about this post, submit comments in the<strong> Comments</strong> section below. If you have questions about this post, <a href="https://console.aws.amazon.com/support/home" rel="noopener noreferrer" target="_blank">contact AWS Support</a>.</p> 
<footer> 
 <div class="blog-author-box">
  <img alt="Ramesh Rajan" class="alignleft size-full" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/03/13/thiagarr.jpg" style="margin-left: 12px; margin-right: 18px; margin-top: 12px; margin-bottom: 6px; width: 93.750px; height: 125px;" />
  <span class="lb-h4" style="line-height: 2.1em; padding-top: 12px; margin-top: 24px;">Ramesh Rajan</span>
  <br />Ramesh is a Senior Solutions Architect based out of San Francisco. He holds a Bachelor of Science in Applied Sciences and a Master’s in Cyber Security and Information Assurance. He specializes in cloud migration, cloud security, compliance, and risk management.
 </div> 
 <div class="blog-author-box">
  <img alt="Matt Luttrell" class="alignleft size-full" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2021/09/28/Matt-Luttrell-Author.jpg" style="margin-left: 12px; margin-right: 18px; margin-top: 12px; margin-bottom: 6px; width: 93.750px; height: 125px;" />
  <span class="lb-h4" style="line-height: 2.1em; padding-top: 12px; margin-top: 24px;">Matt Luttrell</span>
  <br />Matt is a Principal Solutions Architect on the AWS Identity Solutions team. When he’s not spending time chasing his kids around, he enjoys skiing, cycling, and the occasional video game.
 </div> 
</footer>
