# Manara Project 8 — Requirements Compliance Matrix

**Project:** FinServ — AWS Secure Multi-Tier Architecture
**Program:** Manara AWS Solutions Architect Learning Path
**AWS Region:** `us-east-1`
**Review date:** November 22, 2026
**Evidence basis:** Repository source templates and recorded Day 21–24 tests. Statuses are not claims of fresh live-resource verification.

## Status definitions

- **Verified:** The project records a successful implementation and relevant test.
- **Implemented / not end-to-end tested:** Configuration or code exists, but full operation was not demonstrated.
- **Partial / demonstration:** Some parts were implemented, or a temporary lab resource was created and deleted.
- **Not demonstrated:** No sufficient evidence of the full requested capability.

## Mandatory deliverables

| Requirement | Evidence | Status |
|---|---|---|
| Architecture diagram | [Architecture overview](../architecture/architecture.md), editable `.drawio` and a PNG preview if present | Verified in repository |
| Public GitHub repository and README | [Project README](../README.md), CloudFormation templates, and test report | Repository publication completed; confirm anonymous browser access before submission |
| Live demonstration or video | CloudFront and API Gateway endpoints were used during testing | Optional; recording or live demo not established as a submission artifact |

## Architecture and security objectives

| Manara requirement | Project evidence / limitation | Status |
|---|---|---|
| Defense-in-depth three-tier architecture | Public, private application, and isolated database subnet design across two AZs; actual functional app uses Lambda and DynamoDB, not a persistent RDS back end | Verified architectural design; serverless implementation |
| Encrypt data at rest with KMS | S3 SSE-KMS using a customer-managed key verified; EBS/RDS encryption was demonstrated on temporary resources | Verified for S3; partial across all named services |
| Encrypt data in transit | HTTPS API Gateway / CloudFront paths and HTTPS access to DynamoDB documented | Verified for tested paths |
| KMS policies and grant-based access | Customer-managed key and key use checked; separate grant-based workflow not proven | Partial |
| SSM Parameter Store SecureString with customer-managed KMS key | No complete deployed-and-tested example established | Not demonstrated |
| Secrets Manager centralized secret storage | Secret and RDS-managed credential demonstrations documented | Partial / demonstration |
| RDS automatic credential rotation via Lambda | Temporary RDS test completed and removed; no scheduled Lambda rotation lifecycle verified | Not demonstrated end-to-end |
| GuardDuty threat detection | Detector enabled; synthetic Tor and Backdoor EC2/C&C findings inspected. Severity-8 Backdoor finding successfully triggered EventBridge and Lambda; not a real incident | Verified sample workflow |
| GuardDuty finding categories (Recon, Backdoor, CryptoCurrency) | Synthetic Tor and Backdoor C&C findings examined; Recon and CryptoCurrency categories not demonstrated | Partial |
| Security Hub findings aggregation | Findings reviewed with other AWS security controls | Verified |
| CIS / PCI DSS compliance frameworks and scoring | CIS AWS Foundations Benchmark v5.0 enabled; 40 controls enabled; PASS/FAIL findings retrieved. CIS score: 38% (15 passed, 11 failed) on November 22, 2026; PCI DSS not enabled. | VERIFIED - CIS assessment operational; full compliance not claimed |
| AWS Config compliance rules | Three compliance rules returned COMPLIANT | Verified |
| AWS Config automatic remediation with SSM | SSM runbook manually executed successfully; automatic remediation disabled | Verified manual remediation; automatic mode not demonstrated |
| CloudTrail multi-Region API auditing | Multi-Region logging, KMS encryption, management read/write events, and log-file validation verified live. S3 object-level data events not enabled. | VERIFIED - management-event auditing |
| WAF + Shield Standard protection | CloudFront and WAF managed rules deployed in Count mode; no verified active WAF block rule | Partial; monitoring rather than active WAF blocking |
| IAM least privilege | Scoped Lambda permissions for DynamoDB and restricted security groups reviewed | Verified for reviewed policies; not a complete IAM audit |
| IAM permission boundaries | No successfully applied boundary established | Not demonstrated |
| Organizations SCP deny-root policy | SCP reference/validation only; not applied to an AWS Organization | Reference only |
| EventBridge + Lambda incident response | November 22: synthetic high-severity GuardDuty finding matched EventBridge, invoked Lambda, and produced ESCALATE; 1 invocation, 0 errors. SNS notifications remained disabled. | VERIFIED — incident processing; SNS notification not tested |
| CloudWatch and SNS alerting | Three alarms configured, direct SNS test email received | Verified direct SNS; alarm-triggered notification untested |
| Compliance/incident evidence | [Security test report](../tests/security-test-results.md) and CloudFormation YAML | Verified documentation |

## Important project boundaries

1. The live demonstration uses a serverless Lambda/DynamoDB transaction path. The isolated database subnets and temporary RDS deployment demonstrate a separate tier concept; they do not mean the current app uses RDS.
2. A direct SNS test email does **not** prove that a CloudWatch alarm or a GuardDuty/EventBridge event automatically sent a notification.
3. WAF **Count** rules measure matches; they do not block requests on their own. Shield Standard's baseline protection is distinct from a demonstrated application-layer blocking test.
4. An AWS Config manual SSM remediation run does **not** prove automatic remediation was enabled.
5. A CloudFormation template is implementation evidence, not proof that all resources can be redeployed together without additional configuration.
6. Temporary RDS and EBS demonstration resources were deleted. Do not describe them as persistent live application components.
7. Do not claim CIS or PCI DSS compliance certification based solely on Security Hub being enabled.

## Recommended next validation work (optional, not completed claims)

- VERIFIED: CloudTrail log-file validation and multi-Region trail logging were confirmed using AWS CLI. S3 object-level data events remain outside the verified configuration.
- VERIFIED: Synthetic GuardDuty finding triggered EventBridge and Lambda (ESCALATE; 1 invocation; 0 errors). PENDING: Automatic SNS delivery from Lambda and CloudWatch alarm notifications.
- VERIFIED: CIS AWS Foundations Benchmark v5.0 was enabled and evaluated on November 22, 2026, with a 38% score (15 passed, 11 failed, out of 40 enabled controls). FOLLOW-UP: Review remaining failed controls and confirm post-remediation evaluations of S3.1, EC2.7, and FinServ EC2.6. The other 14 enabled controls were not shown as passed or failed in the recorded summary.
- Demonstrate automatic Config remediation only against a disposable, non-production test resource after reviewing permissions and side effects.
- Add a separately scoped Secrets Manager scheduled-rotation proof of concept only if time and cost permit.

## Supporting repository references

- [README](../README.md)
- [Security testing and verification report](../tests/security-test-results.md)
- [Architecture overview](../architecture/architecture.md)
- [CloudFormation templates](../cloudformation/)
- [CIS v5.0 Remediation Plan](cis-remediation-plan.md)


**Assessment:** Core project architecture, encryption, detection, audit, and testing are substantially demonstrated. Advanced automation, governance, rotation, and framework-scoring outcomes remain partial or unverified. A final passing decision belongs to Manara reviewers.
