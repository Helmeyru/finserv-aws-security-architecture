# FinServ — Security Testing and Verification

**Project:** Manara AWS Project 8
**Region:** `us-east-1`
**Environment:** AWS security demonstration lab
**Verification date:** November 21, 2026
**Overall status:** Functional and configuration checks completed; identified limitations and remaining verification items documented.

## 1. Purpose and Scope

This report records security controls and application behavior actually verified during the FinServ project. A **PASS** indicates that the stated check succeeded; it does **not** certify production readiness, comprehensive security, or successful deployment of every CloudFormation template.

The application uses synthetic data. API Gateway currently exposes `GET /health`; the transaction workflow was tested by invoking Lambda directly, not through an API transaction route.

## 2. Verification Summary

| Area | Check performed | Result |
|---|---|---|
| VPC design | Inspected subnet layout, route tables, and security-group rules | PASS — configuration reviewed |
| Network isolation | Application SG inbound TCP 8080 from web SG; outbound TCP 5432 to DB SG and HTTPS to DynamoDB prefix list | PASS — configuration reviewed |
| DynamoDB gateway endpoint | `available`, associated with application route table | PASS |
| IAM | Application inline policy limited to `PutItem`, `GetItem`, `DescribeTable` on the transactions table | PASS — scoped policy inspected |
| S3 encryption | Default SSE-KMS with customer-managed KMS key; S3 Bucket Key enabled; SSE-C blocked | PASS |
| S3 public access | Four S3 Block Public Access settings verified earlier in the project | PASS |
| KMS | Customer-managed key enabled; encrypted S3 test object uploaded and retrieved | PASS |
| API Gateway health | `GET /health` returned HTTP 200 with `status: healthy` | PASS |
| Lambda and DynamoDB | Direct Lambda invocation successfully performed synthetic `PutItem` and `GetItem` | PASS |
| CloudWatch Logs | Confirmed health-check and transaction execution logs | PASS |
| CloudFront | Distribution deployment and HTTP 200 requests verified earlier | PASS |
| AWS WAF | Managed rules in **Count** mode; metrics and sampled requests inspected | PASS — observational mode only |
| AWS Config | Three project rules returned `COMPLIANT` | PASS — reported rule status |
| SSM remediation | Scoped, manual S3 encryption remediation completed earlier | PASS — manual execution |
| GuardDuty | Detector `ENABLED`; generated Tor-related sample finding confirmed with `sample: true` | PASS — sample finding only |
| Security Hub | Findings retrieved; five displayed compliance findings were `PASSED` / `RESOLVED` | PASS — retrieval and review |
| CloudTrail — FinServ | `finserv-security-trail` logging, with no reported delivery errors | PASS |
| CloudWatch alarms | Three project alarms `OK`, with actions enabled | PASS — configuration/state only |
| SNS subscription | Email subscription confirmed | PASS |
| SNS direct delivery | Test message published and received in Gmail | PASS — direct delivery |
| EBS encryption | Temporary encrypted EBS volume verified, then deleted | PASS — demonstration |
| RDS security | Temporary private, encrypted PostgreSQL instance and managed credentials verified, then deleted | PASS — demonstration |
| RDS secret configuration | Rotation configuration and refreshed current secret version verified during the demonstration | PASS — configuration/version; not a full scheduled-rotation lifecycle test |
| Resource cleanup | No RDS instances, EBS volumes, or manual RDS snapshots returned in `us-east-1` | PASS — scoped inventory |
| Cost review | Nov 1–21 account-level unblended cost: approximately **USD 1.65** | PASS — within USD 20 reference budget; billing may lag |

## 3. Day 21 — Application Integration Testing

**Date:** November 21, 2026
**Result:** PASS for the implemented application paths.

- **API Gateway:** `GET /health` returned HTTP `200` and the expected healthy response.
- **Routing:** `GET /transaction` returned `404` because **no such route is configured**. This is a routing limitation, not a failed DynamoDB transaction.
- **Lambda:** `finserv-private-app` was `Active`, running Python 3.12 in the expected VPC and two private application subnets.
- **DynamoDB:** `finserv-transactions` was `ACTIVE`.
- **Direct transaction test:** Lambda returned `statusCode: 200`, `verified: true`, and `operation: PutItem + GetItem`.
- **Synthetic transaction ID:** `c91949aa-52ba-4b69-a922-a58a918af4d6`.
- **Performance:** Execution duration 377.98 ms; maximum observed memory 90 MB.
- **Logs:** CloudWatch recorded the transaction being saved and retrieved.

**Scope note:** The health check was tested end to end through API Gateway. The DynamoDB write/read workflow was tested by direct Lambda invocation; it was **not** tested through an API Gateway transaction endpoint.

## 4. Day 22 — Security Monitoring and Detection

**Date:** November 21, 2026
**Result:** PASS for service configuration, finding retrieval, and direct SNS delivery.

- **GuardDuty:** Detector enabled, with CloudTrail, DNS, and VPC Flow Logs data sources enabled. Other optional protection features remained disabled.
- **GuardDuty finding:** The Tor exit-node finding was an explicitly generated sample (`sample: true`), **not evidence of a confirmed attack**.
- **Security Hub:** Findings retrieved successfully. The displayed compliance findings were marked `PASSED` and `RESOLVED`.
- **CloudTrail:** The main FinServ trail was logging and reported no delivery errors.
- **AWS Config:** All three reported project rules were `COMPLIANT`.
- **CloudWatch:** Three project alarms were `OK`, with notification actions enabled.
- **SNS:** Confirmed email subscription and successful **direct** email delivery to Gmail.
- **SNS test message ID:** `44207327-55e0-546d-b5f7-105ce2dbec62`.

**Not yet verified:** Automatic delivery triggered by a real CloudWatch alarm state transition or a GuardDuty-to-SNS integration. The direct SNS message does **not** prove those paths.

## 5. Day 23 — IAM, Encryption, Networking, and Cost Audit

**Date:** November 21, 2026
**Result:** PASS for reviewed controls, with one CloudTrail follow-up.

- **IAM:** Lambda application role policy permits `dynamodb:PutItem`, `GetItem`, and `DescribeTable` only on the project's DynamoDB table. This review does not cover every effective permission from other attached policies or organizational controls.
- **S3:** Default customer-managed SSE-KMS encryption is enabled; Bucket Key is enabled; SSE-C is blocked.
- **VPC endpoint:** DynamoDB gateway endpoint is `available` and associated with the application route table.
- **Security group:** Restricted inbound and outbound rules confirmed; no unrestricted `0.0.0.0/0` outbound rule reported for the application security group.
- **Temporary resources:** No RDS instances, EBS volumes, or manual RDS snapshots returned in the checked Region.
- **Reported cost:** USD 1.6473 account-level unblended cost for November 1–21, 2026. KMS contributed approximately USD 1.13. Cost Explorer data may be delayed and is not necessarily project-only.
- **Main CloudTrail:** `finserv-security-trail` logging successfully.
- **Follow-up:** An additional `security-trail` with home Region `us-east-2` returned `IsLogging: false` and `LatestDeliveryError: NoSuchBucket`. Its ownership and configuration need separate review; it was **not** deleted.

## 6. Remaining Checks and Limitations

| Item | Status | Explanation |
|---|---|---|
| CloudWatch alarm-to-SNS delivery | NOT TESTED | Alarm configuration was verified; no alarm transition was deliberately triggered. |
| GuardDuty-to-automated-response delivery | NOT TESTED | A sample finding was inspected; automated response was not demonstrated end to end. |
| Transaction API endpoint | NOT IMPLEMENTED | API Gateway only exposes `GET /health`; DynamoDB transaction was tested directly through Lambda. |
| Complete IaC redeployment | NOT TESTED | Selected modified templates passed CloudFormation validation, which does not guarantee a full stack deployment. |
| AWS WAF blocking | NOT IMPLEMENTED | Managed rule groups were in Count mode; the attempted custom blocking rule did not deploy. |
| AWS Organizations SCP | NOT DEPLOYED | Reference template was validated only. |
| Day 18 EBS and RDS resources | REMOVED | Temporary security demonstrations were deleted after verification. |
| Additional CloudTrail in `us-east-2` | REVIEW REQUIRED | Trail not logging; recorded `NoSuchBucket` error. |
| Public GitHub documentation | REVIEW REQUIRED | Confirm Markdown tables, diagrams, visibility, and absence of sensitive information before submission. |

## 7. Evidence and Testing Principles

Evidence consists of AWS CLI results, Lambda execution logs and response, SNS delivery confirmation, repository CloudFormation templates, and architecture diagrams. Raw terminal output is not automatically published in this repository; sensitive identifiers or credentials must not be included in public evidence.

Tests used synthetic data. Security checks were primarily read-only, and temporary billable resources were cleaned up after their demonstrations. The project is an **educational lab**, not a formally audited or production-ready financial-services platform.

## 8. Submission Readiness

**Overall assessment:** Core project implementation and the Day 21–23 checks are completed and documented. Submission remains conditional on the final GitHub presentation/visibility review and acknowledgement of the limitations above.

This report deliberately keeps incomplete or untested integration paths visible rather than labeling all checks as passed.
