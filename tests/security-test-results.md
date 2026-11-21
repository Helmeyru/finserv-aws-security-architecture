\# FinServ — Security Testing and Verification



\*\*Project:\*\* Manara AWS Project 8

\*\*Region:\*\* us-east-1

\*\*Environment:\*\* AWS Security Lab

\*\*Status:\*\* Initial verification completed; final integration testing pending



\## 1. Purpose



This document records the security controls implemented and tested during the project. It distinguishes confirmed test results from work still requiring final verification.



\## 2. Completed Tests



| Security control | Verification | Result |

|---|---|---|

| VPC architecture | Verified public, private application, and isolated database subnets across two AZs | PASS |

| Network segmentation | Reviewed route tables and security-group restrictions | PASS |

| S3 encryption | Uploaded and downloaded SSE-KMS encrypted test object | PASS |

| S3 public access | Verified all four S3 Block Public Access settings | PASS |

| KMS | Verified customer-managed key enabled | PASS |

| DynamoDB connectivity | Lambda successfully performed synthetic PutItem and GetItem operations | PASS |

| CloudFront | Verified distribution deployment and successful HTTP 200 responses | PASS |

| AWS WAF | Inspected Count metrics and sampled requests | PASS — Count mode |

| AWS Config | Verified scoped S3 encryption compliance | PASS |

| SSM remediation | Executed manual remediation successfully | PASS |

| EBS encryption | Verified temporary encrypted EBS volume | PASS |

| RDS security | Verified private, encrypted PostgreSQL database and RDS-managed credentials | PASS |

| RDS secret rotation | Verified rotation configuration and current secret version | PASS |

| CloudWatch alarms | Verified three alarms existed and reported OK | PASS — Configuration |

| SNS | Verified email subscription confirmation | PASS — Subscription |



\## 3. Final Tests — Pending



These checks are reserved for Days 21–24.



| Test | Status |

|---|---|

| End-to-end application health and transaction tests | PENDING |

| CloudWatch alarm notification delivery | PENDING |

| GuardDuty and Security Hub final findings verification | PENDING |

| CloudTrail logging verification | PENDING |

| Network and IAM security review | PENDING |

| CloudFormation repository quality and deployment dependency review | PENDING |

| Final cost and resource cleanup review | PENDING |



\## 4. Known Limitations



\- AWS WAF managed rule groups operate in Count mode, not full blocking mode.

\- The attempted custom WAF blocking configuration was not successfully deployed.

\- The AWS Organizations SCP was prepared as a reference example but was not deployed.

\- Day 18 EBS and RDS resources were temporary and deleted after testing.

\- CloudWatch alarm configuration was verified, but SNS alarm-delivery testing remains pending.

\- CloudFormation validation does not establish successful deployment or full security compliance.

\- Some infrastructure templates represent alternative implementations and should not be deployed together without reviewing dependencies.

\- This project is a learning environment, not a production financial-services system.



\## 5. Testing Principles



\- Use synthetic application data only.

\- Avoid unapproved modifications to production systems.

\- Prefer read-only validation commands.

\- Remove temporary billable resources after testing.

\- Record evidence without exposing passwords, keys, tokens, or customer information.



\## 6. Final Acceptance Criteria



The final submission is ready when the application, security monitoring, and logging checks have been completed, important findings are documented, temporary resources are accounted for, and the Git repository has passed its final security review.



\*\*Final testing status:\*\* In progress.

## Day 21 - Application Integration Testing (2026-11-21)
- PASS: API Gateway GET /health returned HTTP 200 and healthy status.
- PASS: Private Lambda was Active in the expected VPC and two private subnets.
- PASS: DynamoDB finserv-transactions was ACTIVE.
- PASS: Direct Lambda invocation returned HTTP statusCode 200 with verified=true.
- PASS: Synthetic DynamoDB PutItem and GetItem succeeded.
- PASS: CloudWatch Logs confirmed transaction saved and retrieved.
- Verified transaction ID: c91949aa-52ba-4b69-a922-a58a918af4d6
- Lambda duration: 377.98 ms; maximum memory used: 90 MB.
- NOTE: API Gateway exposes GET /health only; /transaction is not configured.
