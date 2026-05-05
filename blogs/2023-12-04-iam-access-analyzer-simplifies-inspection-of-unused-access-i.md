---
title: "IAM Access Analyzer simplifies inspection of unused access in your organization"
url: "https://aws.amazon.com/blogs/security/iam-access-analyzer-simplifies-inspection-of-unused-access-in-your-organization/"
date: "Mon, 04 Dec 2023 20:24:46 +0000"
author: "Achraf Moussadek-Kabdani"
feed_url: "https://aws.amazon.com/blogs/security/tag/iam-access-analyzer/feed/"
---
<p><a href="https://aws.amazon.com/iam/features/analyze-access/" rel="noopener" target="_blank">AWS Identity and Access Management (IAM) Access Analyzer</a> offers tools that help you set, verify, and refine permissions. You can use <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-findings.html" rel="noopener" target="_blank">IAM Access Analyzer external access findings</a> to continuously monitor your <a href="https://docs.aws.amazon.com/organizations/latest/userguide/orgs_introduction.html" rel="noopener" target="_blank">AWS Organizations</a> organization and Amazon Web Services (AWS) accounts for public and cross-account access to your resources, and verify that only intended external access is granted. Now, you can use <strong>IAM Access Analyzer unused access findings</strong> to identify unused access granted to IAM roles and users in your organization.</p> 
<p>If you lead a security team, your goal is to manage security for your organization at scale and make sure that your team follows best practices, such as the <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#grant-least-privilege" rel="noopener" target="_blank">principle of least privilege</a>. When your developers build on AWS, they create IAM roles for applications and team members to interact with AWS services and resources. They might start with broad permissions while they explore AWS services for their use cases. To identify unused access, you can review the <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_access-advisor-view-data.html" rel="noopener" target="_blank">IAM last accessed information</a> for a given IAM role or user and refine permissions gradually. If your company has a multi-account strategy, your roles and policies are created in multiple accounts. You then need visibility across your organization to make sure that teams are working with just the required access.</p> 
<p>Now, <strong>IAM Access Analyzer simplifies inspection of unused access</strong> by reporting unused access findings across your IAM roles and users. IAM Access Analyzer continuously analyzes the accounts in your organization to identify unused access and creates a centralized dashboard with findings. From a <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-settings.html#access-analyzer-delegated-administrator" rel="noopener" target="_blank">delegated administrator account</a> for IAM Access Analyzer, you can use the dashboard to review unused access findings across your organization and prioritize the accounts to inspect based on the volume and type of findings. The findings highlight unused roles, unused access keys for IAM users, and unused passwords for IAM users. For active IAM users and roles, the findings provide visibility into unused services and actions. With the IAM Access Analyzer integration with <a href="https://aws.amazon.com/eventbridge/" rel="noopener" target="_blank">Amazon EventBridge</a> and <a href="https://aws.amazon.com/security-hub/" rel="noopener" target="_blank">AWS Security Hub</a>, you can automate and scale rightsizing of permissions by using event-driven workflows.</p> 
<p>In this post, we’ll show you how to set up and use IAM Access Analyzer to identify and review unused access in your organization.</p> 
<h2>Generate unused access findings</h2> 
<p>To generate unused access findings, you need to create an analyzer. An <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-concepts.html" rel="noopener" target="_blank">analyzer</a> is an IAM Access Analyzer resource that continuously monitors your accounts or organization for a given finding type. You can create an analyzer for the following findings:</p> 
<ul> 
 <li><a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-concepts.html#access-analyzer-concepts-external" rel="noopener" target="_blank">External access findings</a></li> 
 <li><a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-concepts.html#access-analyzer-concepts-unused" rel="noopener" target="_blank">Unused access findings</a></li> 
</ul> 
<p>An analyzer for unused access findings is a new analyzer that continuously monitors roles and users, looking for permissions that are granted but not actually used. This analyzer is different from an analyzer for external access findings; you need to create a new analyzer for unused access findings even if you already have an analyzer for external access findings.</p> 
<p>You can centrally view unused access findings across your accounts by creating an analyzer at the organization level. If you operate a standalone account, you can get unused access findings by creating an analyzer at the account level. This post focuses on the organization-level analyzer setup and management by a central team.</p> 
<h3>Pricing</h3> 
<p>IAM Access Analyzer charges for unused access findings based on the number of IAM roles and users analyzed per analyzer per month. You can still use IAM Access Analyzer external access findings at no additional cost. For more details on pricing, see <a href="https://aws.amazon.com/iam/access-analyzer/pricing/" rel="noopener" target="_blank">IAM Access Analyzer pricing</a>.</p> 
<h3>Create an analyzer for unused access findings </h3> 
<p>To enable unused access findings for your organization, you need to create your analyzer by using the IAM Access Analyzer console or APIs in your management account or a delegated administrator account. A delegated administrator is a member account of the organization that you can delegate with administrator access for IAM Access Analyzer. A best practice is to use your management account only for <a href="https://docs.aws.amazon.com/organizations/latest/userguide/orgs_best-practices_mgmt-acct.html#bp_mgmt-acct_use-mgmt" rel="noopener" target="_blank">tasks that require the management account</a> and use a delegated administrator for other tasks. For steps on how to add a delegated administrator for IAM Access Analyzer, see <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-settings.html#access-analyzer-delegated-administrator" rel="noopener" target="_blank">Delegated administrator for IAM Access Analyzer</a>.</p> 
<h4>To create an analyzer for unused access findings (console)</h4> 
<ol> 
 <li>From the delegated administrator account, open the <a href="https://console.aws.amazon.com/access-analyzer/home" rel="noopener" target="_blank">IAM Access Analyzer console</a>, and in the left navigation pane, select <strong>Analyzer settings</strong>.</li> 
 <li>Choose <strong>Create analyzer</strong>.</li> 
 <li>On the <strong>Create analyzer</strong> page, do the following, as shown in Figure 1: 
  <ol> 
   <li>For <strong>Findings type</strong>, select <strong>Unused access analysis</strong>.</li> 
   <li>Provide a <strong>Name</strong> for the analyzer.</li> 
   <li>Select a <strong>Tracking period</strong>. The tracking period is the threshold beyond which IAM Access Analyzer considers access to be unused. For example, if you select a tracking period of 90 days, IAM Access Analyzer highlights the roles that haven’t been used in the last 90 days.</li> 
   <li>Set your <strong>Selected accounts</strong>. For this example, we select <strong>Current organization</strong> to review unused access across the organization.</li> 
   <li>Select <strong>Create</strong>.<br />&nbsp;</li> 
  </ol> 
  <div class="wp-caption aligncenter" id="attachment_32638" style="width: 750px;">
   <img alt="Figure 1: Create analyzer page" class="size-full wp-image-32638" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/12/04/img1.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-32638">Figure 1: Create analyzer page</p>
  </div> </li> 
</ol> 
<p>Now that you’ve created the analyzer, IAM Access Analyzer starts reporting findings for unused access across the IAM users and roles in your organization. IAM Access Analyzer will periodically scan your IAM roles and users to update unused access findings. Additionally, if one of your roles, users or policies is updated or deleted, IAM Access Analyzer automatically updates existing findings or creates new ones. IAM Access Analyzer uses a <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/using-service-linked-roles.html" rel="noopener" target="_blank">service-linked role</a> to review <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_access-advisor-view-data.html" rel="noopener" target="_blank">last accessed information</a> for all roles, user access keys, and user passwords in your organization. For active IAM roles and users, IAM Access Analyzer uses <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_access-advisor-view-data.html" rel="noopener" target="_blank">IAM service and action last accessed information</a> to identify unused permissions.</p> 
<blockquote>
 <p><strong>Note</strong>: Although IAM Access Analyzer is a regional service (that is, you enable it for a specific <a href="https://aws.amazon.com/about-aws/global-infrastructure/regions_az/?nc1=h_ls" rel="noopener" target="_blank">AWS Region</a>), unused access findings are linked to IAM resources that are global (that is, not tied to a Region). To avoid duplicate findings and costs, enable your analyzer for unused access in the single Region where you want to review and operate findings.</p>
</blockquote> 
<h2>IAM Access Analyzer findings dashboard</h2> 
<p>Your analyzer aggregates findings from across your organization and presents them on a dashboard. The dashboard aggregates, in the selected Region, findings for both external access and unused access—although this post focuses on unused access findings only. You can use the dashboard for unused access findings to centrally review the breakdown of findings by account or finding types to identify areas to prioritize for your inspection (for example, sensitive accounts, type of findings, type of environment, or confidence in refinement).</p> 
<h3>Unused access findings dashboard – Findings overview</h3> 
<p>Review the findings overview to identify the total findings for your organization and the breakdown by finding type. Figure 2 shows an example of an organization with 100 active findings. The finding type <strong>Unused access keys</strong> is present in each of the accounts, with the most findings for unused access. To move toward least privilege and to <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#rotate-credentials" rel="noopener" target="_blank">avoid long-term credentials</a>, the security team should clean up the unused access keys.</p> 
<div class="wp-caption aligncenter" id="attachment_32639" style="width: 790px;">
 <img alt="Figure 2: Unused access finding dashboard" class="size-full wp-image-32639" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/12/04/img2.png" style="border: 1px solid #bebebe;" width="780" />
 <p class="wp-caption-text" id="caption-attachment-32639">Figure 2: Unused access finding dashboard</p>
</div> 
<h3>Unused access findings dashboard – Accounts with most findings</h3> 
<p>Review the dashboard to identify the accounts with the highest number of findings and the distribution per finding type. In Figure 2, the <strong>Audit</strong> account has the highest number of findings and might need attention. The account has five unused access keys and six roles with unused permissions. The security team should prioritize this account based on volume of findings and review the findings associated with the account.</p> 
<h2>Review unused access findings</h2> 
<p>In this section, we’ll show you how to review findings. We’ll share two examples of unused access findings, including unused access key findings and unused permissions findings.</p> 
<h3>Finding example: unused access keys</h3> 
<p>As shown previously in Figure 2, the IAM Access Analyzer dashboard showed that accounts with the most findings were primarily associated with unused access keys. Let’s review a finding linked to unused access keys.</p> 
<h4>To review the finding for unused access keys</h4> 
<ol> 
 <li>Open the <a href="https://console.aws.amazon.com/access-analyzer/home?" rel="noopener" target="_blank">IAM Access Analyzer console</a>, and in the left navigation pane, select <strong>Unused access</strong>.</li> 
 <li>Select your analyzer to view the unused access findings.</li> 
 <li>In the search dropdown list, select the property <strong>Findings type</strong>, the <strong>Equals</strong> operator, and the value <strong>Unused access key</strong> to get only <strong>Findings type = Unused access key</strong>, as shown in Figure 3.<br />&nbsp; 
  <div class="wp-caption aligncenter" id="attachment_32640" style="width: 750px;">
   <img alt="Figure 3: List of unused access findings" class="size-full wp-image-32640" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/12/04/img3.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-32640">Figure 3: List of unused access findings</p>
  </div> </li> 
 <li>Select one of the findings to get a view of the available access keys for an IAM user, their status, creation date, and last used date. Figure 4 shows an example in which one of the access keys has never been used, and the other was used 137 days ago.<br />&nbsp; 
  <div class="wp-caption aligncenter" id="attachment_32641" style="width: 750px;">
   <img alt="Figure 4: Finding example - Unused IAM user access keys" class="size-full wp-image-32641" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/12/04/img4.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-32641">Figure 4: Finding example – Unused IAM user access keys</p>
  </div> </li> 
</ol> 
<p>From here, you can investigate further with the development teams to identify whether the access keys are still needed. If they aren’t needed, you should <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html#Using_CreateAccessKey" rel="noopener" target="_blank">delete the access keys</a>.</p> 
<h3>Finding example: unused permissions</h3> 
<p>Another goal that your security team might have is to make sure that the IAM roles and users across your organization are following the principle of least privilege. Let’s walk through an example with findings associated with unused permissions.</p> 
<h4>To review findings for unused permissions</h4> 
<ol> 
 <li>On the list of unused access findings, apply the filter on <strong>Findings type = Unused permissions</strong>.</li> 
 <li>Select a finding, as shown in Figure 5. In this example, the IAM role has 148 unused actions on <a href="https://aws.amazon.com/rds/" rel="noopener" target="_blank">Amazon Relational Database Service (Amazon RDS)</a> and has not used a service action for 200 days. Similarly, the role has unused actions for other services, including <a href="https://aws.amazon.com/ec2/" rel="noopener" target="_blank">Amazon Elastic Compute Cloud (Amazon EC2)</a>, <a href="https://aws.amazon.com/s3/" rel="noopener" target="_blank">Amazon Simple Storage Service (Amazon S3)</a>, and <a href="https://aws.amazon.com/dynamodb/" rel="noopener" target="_blank">Amazon DynamoDB</a>.<br />&nbsp; 
  <div class="wp-caption aligncenter" id="attachment_32642" style="width: 750px;">
   <img alt="Figure 5: Finding example - Unused permissions" class="size-full wp-image-32642" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/12/04/img5.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-32642">Figure 5: Finding example – Unused permissions</p>
  </div> </li> 
</ol> 
<p>The security team now has a view of the unused actions for this role and can investigate with the development teams to check if those permissions are still required.</p> 
<p>The development team can then refine the permissions granted to the role to remove the unused permissions.</p> 
<p>Unused access findings notify you about unused permissions for all service-level permissions and for 200 services at the action-level. For the list of supported actions, see&nbsp;<a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_access-advisor-action-last-accessed.html" rel="noopener" target="_blank">IAM action last accessed information services and actions</a>.</p> 
<h2>Take actions on findings</h2> 
<p>IAM Access Analyzer categorizes findings as active, resolved, and archived. In this section, we’ll show you how you can act on your findings.</p> 
<h3>Resolve findings</h3> 
<p>You can resolve unused access findings by deleting unused IAM roles, IAM users, IAM user credentials, or permissions. After you’ve completed this, IAM Access Analyzer automatically resolves the findings on your behalf.</p> 
<p>To speed up the process of removing unused permissions, you can use <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-policy-generation.html" rel="noopener" target="_blank">IAM Access Analyzer policy generation</a> to generate a fine-grained IAM policy based on your access analysis. For more information, see the blog post <a href="https://aws.amazon.com/blogs/security/use-iam-access-analyzer-to-generate-iam-policies-based-on-access-activity-found-in-your-organization-trail/" rel="noopener" target="_blank">Use IAM Access Analyzer to generate IAM policies based on access activity found in your organization trail</a>.</p> 
<h3>Archive findings</h3> 
<p>You can suppress a finding by archiving it, which moves the finding from the <strong>Active</strong> tab to the <strong>Archived </strong>tab in the <a href="https://console.aws.amazon.com/access-analyzer/home#/unused-access-findings" rel="noopener" target="_blank">IAM Access Analyzer console</a>. To archive a finding, open the IAM Access Analyzer console, select a <strong>Finding ID</strong>, and in the <strong>Next steps</strong> section, select <strong>Archive</strong>, as shown in Figure 6.</p> 
<div class="wp-caption aligncenter" id="attachment_32643" style="width: 790px;">
 <img alt="Figure 6: Archive finding in the AWS management console" class="size-full wp-image-32643" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/12/04/img6.png" style="border: 1px solid #bebebe;" width="780" />
 <p class="wp-caption-text" id="caption-attachment-32643">Figure 6: Archive finding in the AWS management console</p>
</div> 
<p>You can automate this process by creating <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-archive-rules.html" rel="noopener" target="_blank">archive rules</a> that archive findings based on their attributes. An archive rule is linked to an analyzer, which means that you can have archive rules exclusively for unused access findings.</p> 
<p>To illustrate this point, imagine that you have a subset of IAM roles that you don’t expect to use in your tracking period. For example, you might have an IAM role that is used exclusively for break glass access during your disaster recovery processes—you shouldn’t need to use this role frequently, so you can expect some unused access findings. For this example, let’s call the role DisasterRecoveryRole. You can create an archive rule to automatically archive unused access findings associated with roles named DisasterRecoveryRole, as shown in Figure 7.</p> 
<div class="wp-caption aligncenter" id="attachment_32644" style="width: 790px;">
 <img alt="Figure 7: Example of an archive rule" class="size-full wp-image-32644" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/12/04/img7.png" style="border: 1px solid #bebebe;" width="780" />
 <p class="wp-caption-text" id="caption-attachment-32644">Figure 7: Example of an archive rule</p>
</div> 
<h3>Automation</h3> 
<p>IAM Access Analyzer exports findings to both <a href="https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-what-is.html" rel="noopener" target="_blank">Amazon EventBridge</a> and <a href="https://docs.aws.amazon.com/securityhub/latest/userguide/what-is-securityhub.html" rel="noopener" target="_blank">AWS Security Hub</a>. Security Hub also forwards events to EventBridge.</p> 
<p>Using an <a href="https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-rules.html" rel="noopener" target="_blank">EventBridge rule</a>, you can match the incoming events associated with IAM Access Analyzer unused access findings and send them to targets for processing. For example, you can notify the account owners so that they can investigate and remediate unused IAM roles, user credentials, or permissions.</p> 
<p>For more information, see <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-eventbridge.html" rel="noopener" target="_blank">Monitoring AWS Identity and Access Management Access Analyzer with Amazon EventBridge</a>.</p> 
<h2>Conclusion</h2> 
<p>With IAM Access Analyzer, you can <strong>centrally identify, review, and refine unused access across your organization</strong>. As summarized in Figure 8, you can use the dashboard to review findings and prioritize which accounts to review based on the volume of findings. The findings highlight unused roles, unused access keys for IAM users, and unused passwords for IAM users. For active IAM roles and users, the findings provide visibility into unused services and actions. By reviewing and refining unused access, you can improve your security posture and get closer to the principle of least privilege at scale.</p> 
<div class="wp-caption aligncenter" id="attachment_32654" style="width: 790px;">
 <img alt="Figure 8: Process to address unused access findings" class="size-full wp-image-32654" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/12/04/img8_v2.png" style="border: 1px solid #bebebe;" width="780" />
 <p class="wp-caption-text" id="caption-attachment-32654">Figure 8: Process to address unused access findings</p>
</div> 
<p>The new IAM Access Analyzer unused access findings and dashboard are available in AWS Regions, excluding the AWS GovCloud (US) Regions and AWS China Regions. To learn more about how to use IAM Access Analyzer to detect unused accesses, see <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-findings.html" rel="noopener" target="_blank">the IAM Access Analyzer documentation</a>.</p> 
<p>If you have feedback about this post, submit comments in the<strong> Comments</strong> section below. If you have questions about this post, <a href="https://console.aws.amazon.com/support/home" rel="noopener" target="_blank">contact AWS Support</a>.</p> 
<p><strong>Want more AWS Security news? Follow us on <a href="https://twitter.com/AWSsecurityinfo" rel="noopener noreferrer" target="_blank" title="Twitter">Twitter</a>.</strong></p> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image"> 
   <img alt="Achraf Moussadek-Kabdani" class="aligncenter size-full wp-image-32646" height="160" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/12/04/akabdani.jpg" width="120" /> 
  </div> 
  <h3 class="lb-h4">Achraf Moussadek-Kabdani</h3> 
  <p>Achraf is a Senior Security Specialist at AWS. He works with global financial services customers to assess and improve their security posture. He is both a builder and advisor, supporting his customers to meet their security objectives while making security a business enabler.</p> 
  <p></p>
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image"> 
   <img alt="Author" class="alignnone size-full wp-image-19714" height="160" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2021/03/27/Yevgeniy-Ilyin-Author.jpg" width="120" /> 
  </div> 
  <h3 class="lb-h4">Yevgeniy Ilyin</h3> 
  <p>Yevgeniy is a Solutions Architect at AWS. He has over 20 years of experience working at all levels of software development and solutions architecture and has used programming languages from COBOL and Assembler to .NET, Java, and Python. He develops and code clouds native solutions with a focus on big data, analytics, and data engineering.</p> 
  <p></p>
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image"> 
   <img alt="Mathangi Ramesh" class="aligncenter size-full wp-image-11868" height="160" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2019/11/14/Mathangi-Ramesh.jpg" width="119" /> 
  </div> 
  <h3 class="lb-h4">Mathangi Ramesh</h3> 
  <p>Mathangi is the product manager for IAM. She enjoys talking to customers and working with data to solve problems. Outside of work, Mathangi is a fitness enthusiast and a Bharatanatyam dancer. She holds an MBA degree from Carnegie Mellon University.</p> 
  <p></p>
 </div> 
</footer>
