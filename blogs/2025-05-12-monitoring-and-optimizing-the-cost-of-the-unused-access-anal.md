---
title: "Monitoring and optimizing the cost of the unused access analyzer in IAM Access Analyzer"
url: "https://aws.amazon.com/blogs/security/monitoring-and-optimizing-the-cost-of-the-unused-access-analyzer-in-iam-access-analyzer/"
date: "Mon, 12 May 2025 19:02:46 +0000"
author: "Oscar Diaz"
feed_url: "https://aws.amazon.com/blogs/security/tag/iam-access-analyzer/feed/"
---
<p><a href="https://aws.amazon.com/iam/access-analyzer" rel="noopener" target="_blank">AWS Identity and Access Management (IAM) Access Analyzer</a> is a feature that you can use to identify resources in your AWS organization and accounts that are shared with external entities and to identify unused access. In this post, we explore how the unused access analyzer in IAM Access Analyzer works, dive into the cost implications, and share practical approaches to manage and optimize how you use it with a primary focus on cost optimization.</p> 
<blockquote>
 <p><strong>Note:</strong> While security best practices for managing <a href="https://aws.amazon.com/iam" rel="noopener" target="_blank">AWS Identity and Access Management (IAM)</a> resources are critical, this post emphasizes cost-saving strategies rather than detailed security guidance. We don’t cover step-by-step implementation details for the recommendations here; instead, we provide links to resources that you can use as guides for the process.</p>
</blockquote> 
<h2 id="understanding-the-unused-access-analyzer-in-iam-access-analyzer">Understanding the unused access analyzer in IAM Access Analyzer</h2> 
<p>IAM Access Analyzer has two capabilities to generate findings:</p> 
<ul> 
 <li><strong>External access analysis</strong> (no additional charge): Identifies resources shared with external entities. It requires one analyzer per AWS Region where you have resources.</li> 
 <li><strong>Unused access analysis</strong> (paid): Detects unused roles, access keys, and permissions. It requires only one analyzer per AWS account and analyzes IAM roles and users across Regions from a single analyzer.</li> 
</ul> 
<p>Both external access analysis and unused access analysis support <a href="https://aws.amazon.com/organizations" rel="noopener" target="_blank">AWS Organizations</a> and you can create a single analyzer per organization (in the case of external access analysis, per organization per Region).</p> 
<p>IAM Access Analyzer unused access analysis costs $0.20 per IAM role or user analyzed each month. The charges for existing roles and users happen at the beginning of the month. As new roles and users are added throughout the month, they are analyzed and charged at a rate of $0.20 per role or user. To help avoid duplicate charges, create only one unused access analyzer per account if using an account-level analyzer, or one unused access analyzer for the entire organization if using an organizational-level analyzer. You should avoid deleting and recreating an analyzer. If you recreate an analyzer, you will be charged again for the analysis.</p> 
<h2 id="reviewing-and-optimizing-your-usage">Reviewing and optimizing your usage</h2> 
<p>Before taking any actions to reduce costs, it’s crucial to understand your current usage. You can use the AWS Cost and Usage Report (AWS CUR) to identify how many unused access analyzers you have in your environment. To learn more, see <a href="https://docs.aws.amazon.com/cur/latest/userguide/cur-query-athena.html" rel="noopener" target="_blank">Querying Cost and Usage Reports using Amazon Athena</a>.</p> 
<p>Use the following Athena query on your CUR data to identify the unused access analyzers within your organization. Replace <code style="color: #ff0000;"><em>&lt;CUR_TABLE&gt;</em></code> with the name of your CUR table.</p> 
<div class="hide-language"> 
 <pre><code class="lang-text">SELECT
line_item_usage_type,
product_region,
line_item_resource_id,
bill_payer_account_id,
line_item_usage_account_id,
SUM(line_item_unblended_cost)
FROM <span style="color: #ff0000;"><em>&lt;CUR_TABLE&gt;</em></span>
WHERE line_item_product_code = 'AWSIAMAccessAnalyzer'
AND line_item_line_item_type = 'Usage'
GROUP BY
line_item_usage_type,
product_region,
line_item_resource_id,
bill_payer_account_id,
line_item_usage_account_id
</code></pre> 
</div> 
<p>This query will give you a comprehensive view of your IAM Access Analyzer usage across your organization, including the cost per analyzer.</p> 
<p>Now, let’s walk through four things that you can do today to optimize your IAM Access Analyzer unused access analysis costs.</p> 
<h3 id="consolidate-unused-analyzers">Consolidate unused analyzers</h3> 
<p>Review your AWS CUR analysis results to identify opportunities for consolidation. If you’re using an organizational unused access analyzer, you should use a single analyzer. If you’re using an unused access analyzer per account, make sure a single account doesn’t have more than one analyzer.</p> 
<h3 id="use-tags-to-exclude-some-roles-or-users">Use tags to exclude some roles or users</h3> 
<p>Consider using tags to exclude certain roles or users from analysis. This approach can help scope your analysis and reduce costs by avoiding roles and users that you don’t want to analyze. To do this, you’ll need to implement a <a href="https://docs.aws.amazon.com/tag-editor/latest/userguide/best-practices-and-strats.html" rel="noopener" target="_blank">tagging strategy</a> for your IAM roles and users, identifying principals that might not require regular access analysis. Then, when creating or modifying an analyzer, use exclusion to skip analysis of tagged IAM roles and users. Regularly review your exclusion strategy to validate that it aligns with your organization’s security policies and compliance requirements.</p> 
<p>For a deeper dive into this process, including step-by-step guidance and practical examples, see <a href="https://aws.amazon.com/blogs/security/customize-the-scope-of-iam-access-analyzer-unused-access-analysis/" rel="noopener" target="_blank">Customize the scope of IAM Access Analyzer unused access analysis</a>.</p> 
<h3 id="regular-clean-up-of-iam-roles-and-users">Regular clean-up of IAM roles and users</h3> 
<p>Periodically review and remove unnecessary IAM roles and users. Because IAM Access Analyzer unused access analysis charges are based on the number of roles and users analyzed, removing unused roles and users will help reduce unused access findings cost. This is also a <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#remove-credentials" rel="noopener" target="_blank">security best practice for IAM</a>.</p> 
<h3 id="monitor-and-adjust">Monitor and adjust</h3> 
<p>Set up <a href="https://catalog.workshops.aws/awscff/en-US/foundations/1-budgets" rel="noopener" target="_blank">AWS Budgets</a> or <a href="https://catalog.workshops.aws/awscff/en-US/foundations/3-cost-anomaly" rel="noopener" target="_blank">AWS Cost Anomaly Detection</a> to track your IAM Access Analyzer unused access analysis costs. Create alerts for when costs exceed expected thresholds. By using the proactive approach, you can quickly identify and address unexpected cost increases.</p> 
<h2 id="conclusion">Conclusion</h2> 
<p>IAM Access Analyzer is a valuable tool for improving your organization’s security posture by detecting unused IAM roles, unused access keys for IAM users, unused passwords for IAM users, and unused services and actions for active IAM roles and users. You can then act based on those findings and support your effort to achieve least privilege access. By understanding the billing model and implementing these cost optimization strategies, you can maximize benefits while keeping costs under control. Remember, cost optimization is an ongoing process. Regularly review your usage and adjust your strategy as your needs evolve.</p> 
<p>To learn more about IAM Access Analyzer and its pricing, see <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-getting-started.html" rel="noopener" target="_blank">Getting started with AWS Identity and Access Management Access Analyzer</a>. We’re here to help you optimize your AWS environment, so reach out to AWS Support and your AWS account team if you need further assistance.</p> 
<p>If you have feedback about this post, submit comments in the <strong>Comments</strong> section below. </p> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image"> 
   <img alt="Oscar Diaz" class="aligncenter size-full wp-image-38262" height="160" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/05/08/Oscar-Diaz.jpg" width="120" /> 
  </div> 
  <h3 class="lb-h4">Oscar Diaz Cordovez</h3> 
  <p>Oscar is a Senior Technical Account Manager specializing in cloud operations and security. His passion for technology and innovation drives his expertise in cloud-native architectures, DevOps practices, and automation.</p> 
  <p></p>
 </div> 
</footer> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image"> 
   <img alt="Avi Harari" class="aligncenter size-full wp-image-38260" height="160" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/05/08/Avi-Harari.jpg" width="120" /> 
  </div> 
  <h3 class="lb-h4">Avi Harari</h3> 
  <p>Avi is a Senior Technical Account Manager at AWS supporting Enterprise customers with the adoption and use of AWS services. He is part of the AWS Cloud Operations technical community, focusing on Cloud governance and compliance on AWS.</p> 
  <p></p>
 </div> 
</footer>
