---
title: "Customize the scope of IAM Access Analyzer unused access analysis"
url: "https://aws.amazon.com/blogs/security/customize-the-scope-of-iam-access-analyzer-unused-access-analysis/"
date: "Wed, 08 Jan 2025 17:35:14 +0000"
author: "Stéphanie Mbappe"
feed_url: "https://aws.amazon.com/blogs/security/tag/iam-access-analyzer/feed/"
---
<p><a href="https://aws.amazon.com/iam/access-analyzer/" rel="noopener" target="_blank">AWS Identity and Access Management Access Analyzer</a> simplifies inspecting unused access to guide you towards least privilege.&nbsp;You can use unused access findings to identify over-permissive access&nbsp;granted to <a href="https://aws.amazon.com/iam" rel="noopener" target="_blank">AWS Identity and Access Management (IAM)</a> roles and users in your accounts or organization. From a <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-settings.html#access-analyzer-delegated-administrator" rel="noopener" target="_blank">delegated administrator account</a> for IAM Access Analyzer, you can use the dashboard to review unused access findings across your organization and prioritize the accounts to inspect based on the volume and type of findings. The findings highlight unused roles, unused access keys for IAM users, and unused passwords for IAM users. For active IAM users and roles, the findings provide visibility into unused services and actions. Recently, IAM Access Analyzer launched new configuration capabilities that you can use to customize the analysis. You can select accounts, roles, and users to exclude, and focus on the areas that matter the most to you. You can use identifiers such as account ID or scale configuration using tags. By scoping the&nbsp;IAM Access Analyzer&nbsp;to monitor a subset of accounts and roles, you can reduce noise from unwanted findings. You can update the configuration when needed to change the scope of analysis. With this new offering, IAM Access Analyzer provides enhanced controls to help you tailor the analysis more closely to your organization’s security needs.</p> 
<p>In this post, we walk you through an example scenario. Imagine that you’re a cloud administrator in a company that uses <a href="https://aws.amazon.com/" rel="noopener" target="_blank">Amazon Web Services (AWS)</a>. You use <a href="https://aws.amazon.com/organizations" rel="noopener" target="_blank">AWS Organizations</a> to organize your workload into several organizational units (OUs) and accounts. You have dedicated accounts for testing and experimenting with new AWS features called <em>sandbox</em> accounts across your organization. The sandbox accounts can be created by anyone in your company and are centrally recorded. You’re using tags on IAM resources and have followed <a href="https://docs.aws.amazon.com/tag-editor/latest/userguide/best-practices-and-strats.html" rel="noopener" target="_blank">AWS best practices and strategies when tagging your AWS resources</a>. Tags are applied to the IAM roles created by your teams.</p> 
<p>To make sure that your teams are following the principle of least privilege and are working with only the required permissions to access the AWS accounts, you use IAM Access Analyzer. You created an unused access analyzer at the organization level so it will monitor the AWS accounts in your organization. You&nbsp;noticed that you have multiple unused access findings. After analysis, your security team suggests the exclusion of some AWS accounts, IAM roles, and users so they can focus on the relevant findings. They want the sandbox accounts and the IAM roles they use for security purposes (such as auditing, incident response) to be excluded from the unused access analysis.</p> 
<p>You can select accounts and roles to exclude when you <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-create-unused.html" rel="noopener" target="_blank">create a new analyzer</a> or <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-manage-unused.html" rel="noopener" target="_blank">update the analyzer</a> later. In this post, we show you how to configure IAM Access Analyzer unused access finding to exclude specific accounts across your organization and specific principals (IAM roles and IAM users) once you have set up an analyzer. There is no additional pricing for using the prescriptive recommendations after you have enabled unused access findings.</p> 
<h2 id="prerequisites">Prerequisites</h2> 
<p>The following are the prerequisites to configure IAM Access Analyzer for unused access analysis:</p> 
<ul> 
 <li>An unused access analyzer created at the organization level</li> 
 <li>Administrative level access to the IAM Access Analyzer delegated administrator account</li> 
 <li>A list of account IDs that you want to exclude</li> 
 <li>IAM roles with tags</li> 
</ul> 
<p>In the following sections, you will learn how to customize your IAM Access Analyzer to better suit your organization’s needs. This includes the following:</p> 
<ol type="1"> 
 <li>Explore how to exclude specific AWS accounts from the analyzer’s unused access findings.</li> 
 <li>See how to exclude tagged IAM roles from the analysis, allowing you to focus on the most relevant security insights and you see how to review exclusions on your analyzer to modify them as needed.</li> 
 <li>By the end, you will have a tailored unused access analyzer that provides more meaningful and actionable results for your organization.</li> 
</ol> 
<h2 id="exclude-specific-accounts-across-your-organization">Exclude specific accounts across your organization</h2> 
<p>In this section, you will see how to update your existing unused access analyzer at the organization level through the AWS Management Console and <a href="https://aws.amazon.com/cli/" rel="noopener" target="_blank">AWS Command Line Interface (AWS CLI)</a> to exclude specific AWS account IDs from its analysis.</p> 
<p>If you don’t have an unused access analyzer in the organization, see <a href="https://aws.amazon.com/blogs/security/iam-access-analyzer-simplifies-inspection-of-unused-access-in-your-organization/" rel="noopener" target="_blank">this post</a> for instructions on how to create one.</p> 
<p><strong>Use the console to update your unused access analyzer:</strong></p> 
<ol type="1"> 
 <li>Connect to your IAM Access Analyzer delegated administrator account (by default, your organization management account).</li> 
 <li>Open the IAM Access Analyzer console in your management account.&nbsp;You will see the dashboard with your active finding by selecting the analyzer of your choice on the top right. In this example, the analyzer has 251 active findings.<br /> 
  <div class="wp-caption aligncenter" id="attachment_36732" style="width: 1297px;">
   <img alt="Figure 1: Unused access findings dashboard without exclusions" class="size-full wp-image-36732" height="604" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/12/02/Customize-scope-IAM-Access-Analyzer-1.jpeg" style="border: 1px solid #bebebe;" width="1287" />
   <p class="wp-caption-text" id="caption-attachment-36732">Figure 1: Unused access findings dashboard without exclusions</p>
  </div> </li> 
 <li>You can see the split of active findings per account. The example account has 57 active findings that you want to exclude from it.<br /> 
  <div class="wp-caption aligncenter" id="attachment_36733" style="width: 998px;">
   <img alt="Figure 2: Unused access findings per account" class="size-full wp-image-36733" height="292" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/12/02/Customize-scope-IAM-Access-Analyzer-2.png" style="border: 1px solid #bebebe;" width="988" />
   <p class="wp-caption-text" id="caption-attachment-36733">Figure 2: Unused access findings per account</p>
  </div></li> 
 <li>Select&nbsp;<strong>Analyzer settings</strong> under <strong>Access Analyzer</strong> in that navigation pane.</li> 
 <li>The analyzer settings page presents the analyzers in your AWS Region and their status.</li> 
 <li>Select your unused access analyzer in the list based on its name.<br /> 
  <div class="wp-caption aligncenter" id="attachment_36734" style="width: 1159px;">
   <img alt="Figure 3: Active access analyzers" class="size-full wp-image-36734" height="641" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/12/02/Customize-scope-IAM-Access-Analyzer-3.png" style="border: 1px solid #bebebe;" width="1149" />
   <p class="wp-caption-text" id="caption-attachment-36734">Figure 3: Active access analyzers</p>
  </div></li> 
 <li>On the <strong>Analyzer</strong> page, you can see the analyzer settings and a new tab called&nbsp;<strong>Exclusion</strong>. Because you have no excluded AWS accounts, the count of <strong>Excluded AWS accounts</strong> is <strong>0</strong> and there are no accounts displayed.<br /> 
  <div class="wp-caption aligncenter" id="attachment_36735" style="width: 1102px;">
   <img alt="Figure 4: Unused access analyzer exclusion tab" class="size-full wp-image-36735" height="736" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/12/02/Customize-scope-IAM-Access-Analyzer-4.png" style="border: 1px solid #bebebe;" width="1092" />
   <p class="wp-caption-text" id="caption-attachment-36735">Figure 4: Unused access analyzer exclusion tab</p>
  </div> </li> 
 <li>Choose&nbsp;<strong>Manage</strong> in the <strong>Excluded AWS accounts</strong> section.</li> 
 <li>Select <strong>Choose from organization</strong> and <strong>Hierarchy</strong> and choose <strong>Exclude</strong> next to the sandbox account that you want to exclude.<br /> 
  <div class="wp-caption aligncenter" id="attachment_36736" style="width: 1157px;">
   <img alt="Figure 5: Exclude sandbox account" class="size-full wp-image-36736" height="844" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/12/02/Customize-scope-IAM-Access-Analyzer-5.png" style="border: 1px solid #bebebe;" width="1147" />
   <p class="wp-caption-text" id="caption-attachment-36736">Figure 5: Exclude sandbox account</p>
  </div> </li> 
 <li>After you select&nbsp;<strong>Exclude</strong> for the sandbox account, the account will be deselected and will appear in <strong>AWS accounts to exclude</strong>. The count of accounts to exclude has changed from <strong>0</strong> to <strong>1</strong>. After you have finished, choose <strong>Save changes</strong>.<br /> 
  <div class="wp-caption aligncenter" id="attachment_36737" style="width: 883px;">
   <img alt="Figure 6: Verify that the account is excluded and save changes" class="size-full wp-image-36737" height="422" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/12/02/Customize-scope-IAM-Access-Analyzer-6.png" style="border: 1px solid #bebebe;" width="873" />
   <p class="wp-caption-text" id="caption-attachment-36737">Figure 6: Verify that the account is excluded and save changes</p>
  </div> </li> 
 <li>The page will be automatically updated with your changes. You can then review the <strong>Excluded AWS accounts</strong> and verify that your excluded account is correctly configured.<br /> 
  <div class="wp-caption aligncenter" id="attachment_36738" style="width: 1353px;">
   <img alt="Figure 7: Analyzer configuration updated with excluded account" class="size-full wp-image-36738" height="554" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/12/02/Customize-scope-IAM-Access-Analyzer-7.png" style="border: 1px solid #bebebe;" width="1343" />
   <p class="wp-caption-text" id="caption-attachment-36738">Figure 7: Analyzer configuration updated with excluded account</p>
  </div> </li> 
 <li>You can go back to the console dashboard and see the results. In this example, the exclusion of the sandbox account has caused the total number of active findings to go down from 251 to 194.<br /> 
  <div class="wp-caption aligncenter" id="attachment_36739" style="width: 1575px;">
   <img alt="Figure 8: Dashboard showing a reduction in active findings" class="size-full wp-image-36739" height="932" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/12/02/Customize-scope-IAM-Access-Analyzer-8.png" style="border: 1px solid #bebebe;" width="1565" />
   <p class="wp-caption-text" id="caption-attachment-36739">Figure 8: Dashboard showing a reduction in active findings</p>
  </div> </li> 
</ol> 
<p><strong>Use AWS CLI to update your unused access analyzer:</strong></p> 
<p>You can update your existing analyzer using the AWS CLI command&nbsp;<code style="color: #000000;">aws accessanalyzer update-analyzer</code>. Use the following command, replacing <code style="color: #ff0000;"><em>&lt;YOUR-ANALYZER-NAME&gt;</em></code> with the name of your analyzer.</p> 
<div class="hide-language"> 
 <pre><code class="lang-text">aws accessanalyzer update-analyzer 
--analyzer-name <span style="color: #ff0000;"><em>&lt;YOUR-ANALYZER-NAME&gt;</em></span>
--configuration '{
  "unusedAccess": {
    "analysisRule": {
      "exclusions": [
        {
          "accountIds": [
            "222222222222"
          ]
        }
      ]
    }
  }
}'
</code></pre> 
</div> 
<p>You will obtain a result similar to the following:</p> 
<div class="hide-language"> 
 <pre><code class="lang-text">{
    "revisionId": "<span style="color: #ff0000;"><em>&lt;UNIQUE-REVISION-NUMBER&gt;</em></span>", 
    "configuration": {
        "unusedAccess": {
            "analysisRule": {
                "exclusions": [
                    {
                        "accountIds": [
                            "222222222222"
                        ]
                    }
                ]
            }, 
            "unusedAccessAge": 90
        }
    }
</code></pre> 
</div> 
<p>You have successfully excluded a sandbox account from the unused access analysis. Now you will exclude the IAM roles used by the security team to audit your accounts based on tags.</p> 
<h2 id="excluding-specific-principals-in-your-organization-using-tags">Excluding specific principals in your organization using tags</h2> 
<p>In this section, you will see how to update an existing unused access analyzer by excluding tagged IAM roles in your organization using the console and then AWS CLI.</p> 
<p><strong>Use the console to update your unused access analyzer:</strong></p> 
<ol type="1"> 
 <li>Open the IAM Access Analyzer console.</li> 
 <li>Review the summary dashboard containing your unused findings. Choose <strong>Analyzer settings</strong> at the top of the screen.<br /> 
  <div class="wp-caption aligncenter" id="attachment_36742" style="width: 1021px;">
   <img alt="Figure 9: IAM Access Analyzer summary dashboard" class="size-full wp-image-36742" height="340" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/12/02/Customize-scope-IAM-Access-Analyzer-9.png" style="border: 1px solid #bebebe;" width="1011" />
   <p class="wp-caption-text" id="caption-attachment-36742">Figure 9: IAM Access Analyzer summary dashboard</p>
  </div> </li> 
 <li>You will see a list of analyzers created in your account in that Region. Select the analyzer that you want to update.</li> 
 <li>Review the analyzer page. On the <strong>Exclusion</strong>&nbsp;tab, you will see <strong>Exclude IAM users and roles with tags</strong> with a count of <strong>0</strong>.<br /> 
  <div class="wp-caption aligncenter" id="attachment_36743" style="width: 1095px;">
   <img alt="Figure 10: Configure exclusion of IAM roles using tag" class="size-full wp-image-36743" height="702" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/12/02/Customize-scope-IAM-Access-Analyzer-10.png" style="border: 1px solid #bebebe;" width="1085" />
   <p class="wp-caption-text" id="caption-attachment-36743">Figure 10: Configure exclusion of IAM roles using tag</p>
  </div> </li> 
 <li>Choose&nbsp;<strong>Manage</strong> in the <strong>Excluded IAM users and roles with tags</strong> section.</li> 
 <li>Add the tags attached to the roles that you want to exclude from the analysis and choose <strong>Save changes</strong>.<br /> 
  <div class="wp-caption aligncenter" id="attachment_36756" style="width: 1296px;">
   <img alt="Figure 11: Add tag to exclude" class="size-full wp-image-36756" height="246" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/12/02/Customize-scope-IAM-Access-Analyzer-11.png" style="border: 1px solid #bebebe;" width="1286" />
   <p class="wp-caption-text" id="caption-attachment-36756">Figure 11: Add tag to exclude</p>
  </div> </li> 
 <li>You can now see that <strong>Excluded IAM users and roles with tags</strong> now has a count of <strong>1</strong>, and you can see the tags in the list.<br /> 
  <div class="wp-caption aligncenter" id="attachment_36745" style="width: 1297px;">
   <img alt="Figure 12: List of exclusion tags" class="size-full wp-image-36745" height="163" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/12/02/Customize-scope-IAM-Access-Analyzer-12.png" style="border: 1px solid #bebebe;" width="1287" />
   <p class="wp-caption-text" id="caption-attachment-36745">Figure 12: List of exclusion tags</p>
  </div> </li> 
</ol> 
<p><strong>Use AWS CLI to update your unused access analyzer:</strong></p> 
<p>You can also update your existing analyzer using the AWS CLI command <code style="color: #000000;">aws accessanalyzer update-analyzer</code>. Using the following command, replace <code style="color: #ff0000;"><em>&lt;YOUR-ANALYZER-NAME&gt;</em></code> with the name of your analyzer.</p> 
<div class="hide-language"> 
 <pre><code class="lang-text">aws accessanalyzer update-analyzer 
--analyzer-name <span style="color: #ff0000;"><em>&lt;YOUR-ANALYZER-NAME&gt;</em></span> 
--configuration '{
  "unusedAccess": {
    "analysisRule": {
      "exclusions": [
        {
          "accountIds": [
            "222222222222"
          ]
        },
        {
          "resourceTags": [
            {
              <strong>"team": "security"</strong>
            }
          ]
        }
      ]
    }
  }
}'

</code></pre> 
</div> 
<p>A successful response will look like the following:</p> 
<div class="hide-language"> 
 <pre><code class="lang-text">{
    "revisionId": "<span style="color: #ff0000;"><em>&lt;UNIQUE-REVISION-NUMBER&gt;</em></span>", 
    "configuration": {
        "unusedAccess": {
            "analysisRule": {
                "exclusions": [
                    {
                        "accountIds": [
                            "222222222222"
                        ]
                    }, 
                    {
                        "resourceTags": [
                            {
                                <strong>"team": "security"</strong>
                            }
                        ]
                    }
                ]
            }, 
            "unusedAccessAge": 90
        }
    }
}
</code></pre> 
</div> 
<h2 id="review-the-exclusion-on-your-analyzer">Review the exclusion on your analyzer</h2> 
<p>You can review, remove, or update the exclusions configured on your analyzer by using the console or AWS CLI. For example, as a security administrator managing multiple accounts, you might initially exclude IAM roles that have the tag <code style="color: #000000;">security</code> from analysis. However, you might need to review these exclusions if your policies change, requiring analysis of certain security roles or removing the exclusion entirely. By adjusting your exclusions, you can make sure that your analyzer’s results remain relevant to your organization’s needs and account structure.</p> 
<p><strong>Review the exclusion on unused access analyzer using the console:</strong></p> 
<p>In this section, review the tags that have been excluded from an analyzer.</p> 
<ol type="1"> 
 <li>Open the IAM console.</li> 
 <li>Select <strong>Access Analyzer</strong>, under <strong>Access reports</strong>, you will see a summary dashboard of findings from an analyzer. 
  <ol type="a"> 
   <li>The <strong>Active findings</strong>&nbsp;section shows the number of active findings for unused roles, the number of active findings for unused credentials and the number of active findings for unused permissions.</li> 
   <li>The <strong>Findings overview</strong>&nbsp;section includes a breakdown of the active findings.</li> 
   <li>The <strong>Findings status</strong> section shows the status of findings (whether active, archived or resolved).<br /> 
    <div class="wp-caption aligncenter" id="attachment_36739" style="width: 1575px;">
     <img alt="Figure 13: Unused access analyzer dashboard " class="size-full wp-image-36739" height="932" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/12/02/Customize-scope-IAM-Access-Analyzer-8.png" style="border: 1px solid #bebebe;" width="1565" />
     <p class="wp-caption-text" id="caption-attachment-36739">Figure 13: Unused access analyzer dashboard</p>
    </div> </li> 
  </ol> </li> 
 <li>Select the <strong>Analyzer settings</strong> at the top of the screen.</li> 
 <li>Select the analyzer that you want to review to see the exclusion tags.<br /> 
  <div class="wp-caption aligncenter" id="attachment_36758" style="width: 1340px;">
   <img alt="Figure 14: Review unused access analyzer exclusions" class="size-full wp-image-36758" height="763" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/12/02/Customize-scope-IAM-Access-Analyzer-14-1.png" style="border: 1px solid #bebebe;" width="1330" />
   <p class="wp-caption-text" id="caption-attachment-36758">Figure 14: Review unused access analyzer exclusions</p>
  </div> </li> 
</ol> 
<ol start="5" type="1"> 
 <li>After applying the tags, the updated dashboard is shown after the next scan.<br /> 
  <div class="wp-caption aligncenter" id="attachment_36759" style="width: 1645px;">
   <img alt="Figure 15: Dashboard showing reduction of findings after exclusions" class="size-full wp-image-36759" height="948" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/12/02/Customize-scope-IAM-Access-Analyzer-15.png" style="border: 1px solid #bebebe;" width="1635" />
   <p class="wp-caption-text" id="caption-attachment-36759">Figure 15: Dashboard showing reduction of findings after exclusions</p>
  </div> </li> 
</ol> 
<p><strong>Review the exclusion on an unused access analyzer using AWS CLI:</strong></p> 
<p>Using the name of your analyzer, you can run the command <code style="color: #000000;">get-analyzer</code> to see the configured exclusion. Using the following command, replace <code style="color: #ff0000;"><em>&lt;YOUR-ANALYZER-NAME&gt;</em></code> with the name of your analyzer:</p> 
<p><code style="color: #000000;">aws accessanalyzer get-analyzer --analyzer-name <span style="color: #ff0000;"><em>&lt;YOUR-ANALYZER-NAME&gt;</em></span></code></p> 
<p>You will get a response similar to the following:</p> 
<div class="hide-language"> 
 <pre><code class="lang-text">{
  "analyzer": {
    "status": "ACTIVE",
    "name": "<span style="color: #ff0000;"><em>&lt;YOUR-ANALYZER-NAME&gt;</em></span>",
    "tags": {},
    "revisionId": "<span style="color: #ff0000;"><em>&lt;UNIQUE-REVISION-NUMBER&gt;</em></span>",
    "arn": "arn:aws:access-analyzer:<span style="color: #ff0000;"><em>&lt;REGION&gt;</em></span>:111111111111:analyzer/<span style="color: #ff0000;"><em>&lt;YOUR-ANALYZER-NAME&gt;</em></span>",
    "configuration": {
      "unusedAccess": {
        "analysisRule": {
          "exclusions": [
            {
              "accountIds": [
                "222222222222"
              ]
            },
            {
              "resourceTags": [
                {
                  "team": "security"
                }
              ]
            }
          ]
        },
        "unusedAccessAge": 90
      }
    },
    "type": "ORGANIZATION_UNUSED_ACCESS",
    "createdAt": "2024-10-11T22:26:57Z"
  }
}
</code></pre> 
</div> 
<h2 id="conclusion">Conclusion</h2> 
<p>In this post, you learned how to tailor your unused access analyzer to&nbsp;your needs by excluding specific&nbsp;accounts and IAM roles. To exclude the accounts in your organization from being monitored by IAM Access Analyzer, you can use a list of account IDs or select them from a hierarchical view of your organization structure. You can exclude IAM roles and IAM users based on tags. By customizing the exclusion on the unused access analyzer, you saw that the number of active findings went down, helping you focus on the findings that matter most.&nbsp;With this new offering, IAM Access Analyzer provides enhanced controls to help you tailor the analysis more closely to your organization’s security needs.</p> 
<ul> 
 <li>To learn more about IAM Access Analyzer Unused Access, see <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-findings.html" rel="noopener" target="_blank">Findings for external and unused access</a>.</li> 
 <li>To learn about the creation of unused access findings, see <a href="https://aws.amazon.com/blogs/security/iam-access-analyzer-simplifies-inspection-of-unused-access-in-your-organization/" rel="noopener" target="_blank">IAM Access Analyzer simplifies inspection of unused access in your organization</a>.</li> 
 <li>To learn about how to&nbsp;<a href="https://aws.amazon.com/blogs/security/refine-unused-access-using-iam-access-analyzer-recommendations/" rel="noopener" target="_blank">Refine unused access using IAM Access Analyzer recommendations</a>.</li> 
 <li>To learn about the strategies for achieving least privilege at scale, see Strategies for achieving least privilege at scale <a href="https://aws.amazon.com/blogs/security/strategies-for-achieving-least-privilege-at-scale-part-1/" rel="noopener" target="_blank">Part 1</a> and <a href="https://aws.amazon.com/blogs/security/strategies-for-achieving-least-privilege-at-scale-part-2/" rel="noopener" target="_blank">Part 2</a>.</li> 
</ul> 
<p>If you have feedback about this post, submit comments in the <strong>Comments</strong> section below. </p> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image"> 
   <img alt="Stéphanie Mbappe" class="aligncenter size-full wp-image-30064" height="160" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/07/07/StephMbappe.png" width="120" /> 
  </div> 
  <h3 class="lb-h4">P. Stéphanie Mbappe</h3> 
  <p>Stéphanie is a Security Consultant with Amazon Web Services. She delights in assisting her customers at any step of their security journey. Stéphanie enjoys learning, designing new solutions, and sharing her knowledge with others.</p> 
  <p></p>
 </div> 
</footer> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image"> 
   <img alt="Mathangi Ramesh" class="aligncenter size-full wp-image-31906" height="160" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/11/01/mathangi_ramesh-120x160-1.jpg" width="120" /> 
  </div> 
  <h3 class="lb-h4">Mathangi Ramesh</h3> 
  <p>Mathangi is the product manager for AWS Identity and Access Management. She enjoys talking to customers and working with data to solve problems. Outside of work, Mathangi is a fitness enthusiast and a Bharatanatyam dancer. She holds an MBA degree from Carnegie Mellon University.</p> 
  <p></p>
 </div> 
</footer> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image"> 
   <img alt="Reke Jarikre" class="aligncenter size-full wp-image-36750" height="160" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/12/02/Reke-Jarikre.png" width="120" /> 
  </div> 
  <h3 class="lb-h4">Reke Jarikre</h3> 
  <p>Reke is an Associate Security Consultant at Amazon Web Services. She is passionate about safeguarding client infrastructures and crafting robust security measures. Outside work, she enjoys exploring new technologies, public speaking and contributing to open-source projects.</p> 
  <p></p>
 </div> 
</footer>
