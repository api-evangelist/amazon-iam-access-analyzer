# Amazon IAM Access Analyzer (amazon-iam-access-analyzer)

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

AWS IAM Access Analyzer helps you set, verify, and refine your IAM policies by providing a suite of capabilities including findings for external, internal, and unused access, basic and custom policy checks for validating policies, and policy generation to generate fine-grained policies. It uses automated reasoning to identify resources shared with external entities and helps implement least privilege access across your AWS environment.

**URL:** [Visit APIs.json URL](https://raw.githubusercontent.com/api-evangelist/amazon-iam-access-analyzer/refs/heads/main/apis.yml)

**Run:** [Capabilities Using Naftiko](https://github.com/naftiko/fleet?utm_source=api-evangelist&utm_medium=readme&utm_campaign=company-api-evangelist&utm_content=repo)

## Tags:

 - Access Control, AWS, Compliance, IAM, Policy Management, Security

## Timestamps

- **Created:** 2026-03-16
- **Modified:** 2026-04-19

## APIs

### AWS IAM Access Analyzer API
The AWS IAM Access Analyzer API provides programmatic access to create and manage analyzers, findings, archive rules, and policy validations to identify and remediate unintended resource access across AWS accounts and organizations.

**Human URL:** [https://aws.amazon.com/iam/features/analyze-access/](https://aws.amazon.com/iam/features/analyze-access/)

#### Tags:

 - Access Control, IAM, Policy Management, Security

#### Properties

- [Documentation](https://docs.aws.amazon.com/access-analyzer/latest/APIReference/Welcome.html)
- [OpenAPI](openapi/amazon-iam-access-analyzer-openapi-original.yml)
- [GettingStarted](https://docs.aws.amazon.com/IAM/latest/UserGuide/what-is-access-analyzer.html)
- [Pricing](https://aws.amazon.com/iam/pricing/)
- [FAQ](https://aws.amazon.com/iam/faqs/)
- [JSONSchema](json-schema/iam-access-analyzer-analyzer-schema.json)
- [JSONStructure](json-structure/iam-access-analyzer-analyzer-structure.json)
- [Example](examples/iam-access-analyzer-analyzer-example.json)

## Common Properties

- [Portal](https://aws.amazon.com/iam/features/analyze-access/)
- [Website](https://aws.amazon.com/iam/)
- [Documentation](https://docs.aws.amazon.com/IAM/latest/UserGuide/what-is-access-analyzer.html)
- [TermsOfService](https://aws.amazon.com/service-terms/)
- [PrivacyPolicy](https://aws.amazon.com/privacy/)
- [Support](https://aws.amazon.com/premiumsupport/)
- [Blog](https://aws.amazon.com/blogs/security/tag/iam-access-analyzer/)
- [GitHubOrganization](https://github.com/aws)
- [Console](https://console.aws.amazon.com/access-analyzer/)
- [SignUp](https://portal.aws.amazon.com/billing/signup)
- [Login](https://signin.aws.amazon.com/)
- [StatusPage](https://health.aws.amazon.com/health/status)
- [Contact](https://aws.amazon.com/contact-us/)

## Features

| Name | Description |
|------|-------------|
| External Access Analysis | Identifies resources shared with external entities outside your AWS organization using automated reasoning. |
| Internal Access Analysis | Identifies which principals within your organization have access to selected resources. |
| Unused Access Analysis | Identifies unused IAM roles, access keys, console passwords, and unused service permissions. |
| Policy Validation | Validates IAM policies against best practices and custom security standards before deployment. |
| Policy Generation | Generates fine-grained IAM policies based on actual access activity logged in AWS CloudTrail. |
| Access Preview | Preview public and cross-account access to resources before deploying permission changes. |
| Archive Rules | Automatically archive findings that match specified criteria to reduce noise. |

## Use Cases

| Name | Description |
|------|-------------|
| Least Privilege Enforcement | Analyze actual API activity to generate minimal permission policies that implement least privilege access. |
| Security Compliance Auditing | Continuously monitor for unintended external access to sensitive resources like S3 buckets and IAM roles. |
| CI/CD Policy Validation | Integrate policy checks into deployment pipelines to catch overpermissive policies before they reach production. |
| Access Governance | Identify and remediate unused access across IAM users, roles, and service accounts organization-wide. |
| Cross-Account Access Review | Identify all resources shared across AWS accounts and validate the intent of each cross-account permission. |

## Integrations

| Name | Description |
|------|-------------|
| AWS CloudTrail | Uses CloudTrail activity logs to generate least-privilege IAM policies based on actual usage. |
| AWS Security Hub | Publishes Access Analyzer findings to Security Hub for centralized security monitoring. |
| AWS Organizations | Analyzes access across all accounts in an AWS Organization for comprehensive governance. |
| AWS Config | Triggers re-scanning of resources when configuration changes are detected. |
| Amazon EventBridge | Publishes finding events to EventBridge for automated security workflow responses. |

## Artifacts

Machine-readable API specifications organized by format.

### OpenAPI

- [AWS IAM Access Analyzer API](openapi/amazon-iam-access-analyzer-openapi-original.yml)

### JSON Schema

162 schema files covering analyzers, findings, archive rules, policies, and access previews.

### JSON Structure

162 JSON Structure files converted from JSON Schema.

### JSON-LD

- [Amazon IAM Access Analyzer Context](json-ld/amazon-iam-access-analyzer-context.jsonld)

### Examples

162 example JSON files generated from schemas.

## Capabilities

Naftiko capabilities organized as shared per-API definitions composed into customer-facing workflows.

### Shared Per-API Definitions

- [AWS IAM Access Analyzer API](capabilities/shared/iam-access-analyzer.yaml) — 28 operations for access analysis and policy management

### Workflow Capabilities

| Workflow | APIs Combined | Tools | Persona |
|----------|--------------|-------|---------|
| [Access Security Management](capabilities/access-security-management.yaml) | IAM Access Analyzer | 9 | Security Engineer, IAM Administrator |

## Vocabulary

- [Amazon IAM Access Analyzer Vocabulary](vocabulary/amazon-iam-access-analyzer-vocabulary.yaml) — Unified taxonomy mapping 7 resources, 11 actions, 1 workflow, and 2 personas across operational (OpenAPI) and capability (Naftiko) dimensions

## Rules

- [Amazon IAM Access Analyzer Spectral Rules](rules/amazon-iam-access-analyzer-spectral-rules.yml) — 22 rules across 8 categories enforcing Amazon IAM Access Analyzer API conventions

## Maintainers

**FN:** Kin Lane

**Email:** kin@apievangelist.com
