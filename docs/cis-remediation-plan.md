# FinServ — CIS AWS Foundations Benchmark v5.0 Remediation Plan

**Project:** Manara Project 8 — FinServ AWS Secure Multi-Tier Architecture
**Region:** `us-east-1`
**Evidence date:** November 22, 2026 (UTC timestamps in Security Hub findings)
**Scope:** Educational AWS security lab; remediation planning, not certification or a claim that issues are fixed.

## 1. Observed posture and evidence

The recorded Security Hub dashboard showed **38%** with **15 passed and 11 failed controls** among **40 enabled controls**. A subsequent read-only AWS CLI query returned **26 ACTIVE/FAILED findings across 12 distinct Security Hub control IDs**, and `Compliance.AssociatedStandards[].StandardsId` identified **`standards/cis-aws-foundations-benchmark/v/5.0.0` for every returned finding**.

These are different observations: dashboard control counts and per-resource active finding counts need not coincide because of refresh timing, aggregation, or status changes. Do not replace the 38% historical score with an invented new percentage. Query the current standard dashboard and findings again before claiming post-remediation improvement. The earlier 40-control breakdown does not establish the states of the other 14 controls.

**Status for every item below: OPEN / REMEDIATION NOT VERIFIED.** Prioritize safe read-only investigation, and obtain approval before any resource modifications. Do not apply bulk S3 policy, MFA Delete, IAM, CloudTrail, or AWS Config changes automatically.

## 2. Failed finding inventory and remediation actions

| Control | Active findings | Risk / finding | Priority | Proposed remediation | Verification evidence | Status |
|---|---:|---|---|---|---|---|
| **IAM.6** | 1 | Root account does not meet hardware MFA requirement | High | Review root recovery access; enroll an approved phishing-resistant hardware authenticator according to account policy; ensure a second recovery method where supported | Review root MFA configuration and await Security Hub reevaluation | Open |
| **IAM.15** | 1 | IAM password minimum length less than 14 | High | Set account IAM password policy minimum length to **14 or greater**, after checking effects on IAM users | `aws iam get-account-password-policy`; check control reevaluation | Open |
| **IAM.16** | 1 | Password reuse restriction not enforced | High | Configure an appropriate IAM password reuse-prevention history (e.g., 24 previous passwords, if permitted by account policy) | `aws iam get-account-password-policy`; check control reevaluation | Open |
| **Account.1** | 1 | Missing security alternate contact | Medium | Add and validate a monitored security alternate contact at account level | `aws account get-alternate-contact --alternate-contact-type SECURITY` (where authorized); reevaluate | Open |
| **IAM.18** | 1 | AWS Support incident role absent | Medium | Review control wording and AWS Support access design; create an appropriately scoped support-access role with approved trust and permissions if needed | IAM role/policy review and Security Hub reevaluation | Open |
| **CloudTrail.7** | 1 | CloudTrail log bucket has no S3 server access logging | High | Identify the exact CloudTrail destination bucket; configure server access logging to a separate dedicated, protected log bucket and prevent recursive self-logging; test delivery | S3 bucket logging configuration; delivery evidence; control reevaluation | Open |
| **Config.1** | 1 | AWS Config recorder/service-linked role configuration does not meet the control | High | Read the active configuration recorder and IAM role; plan service-linked-role migration only if justified, without disrupting recording | `aws configservice describe-configuration-recorders`; recorder status and finding reevaluation | Open |
| **S3.22** | 1 | S3 object-level write events not audited | Medium | Scope CloudTrail **write data events** to approved buckets/prefixes; estimate data-event charges first | CloudTrail selectors, sample test write event, and finding reevaluation | Open |
| **S3.23** | 1 | S3 object-level read events not audited | Medium | Scope CloudTrail **read data events** to approved buckets/prefixes; estimate data-event charges first | CloudTrail selectors, sample test read event, and finding reevaluation | Open |
| **S3.5** | **7** | Multiple S3 buckets lack an explicit TLS-only policy | High | Identify all seven affected resources; add safe bucket-policy Deny for `aws:SecureTransport=false`, merging with existing CloudFront OAC/CloudTrail/Config permissions rather than overwriting them | Inspect `get-bucket-policy` for each bucket; harmless HTTPS access test, policy review, reevaluation | Open |
| **S3.20** | **8** | MFA Delete not enabled on affected buckets | Medium / operationally sensitive | Identify each affected bucket and its versioning status; assess whether MFA Delete is suitable for the lab. It requires special account-root/API handling, can complicate CloudFormation operations, and must not be enabled in bulk | Per-bucket `get-bucket-versioning`, documented risk decision, reevaluation or accepted lab exception | Open |
| **EC2.2** | **2** | Default VPC security groups permit traffic | High | Identify which VPCs/default SGs are affected; inventory any ENIs that use those groups; remove unnecessary inbound and outbound rules only after checking dependencies | `describe-security-groups` and usage inventory; reevaluation | Open |

### Counts

- **26 findings** = **8 × S3.20 + 7 × S3.5 + 2 × EC2.2 + 9 × single-finding controls**.
- **12 distinct control IDs**; all were returned as ACTIVE and FAILED, workflow `NEW` in the recorded query.
- These values are **not** a new compliance score and do **not** prove all 12 controls are presently counted as failed in the latest dashboard score.

## 3. Read-only investigation sequence

1. Save the current Security Hub finding export and note the observation time. Group findings by `Compliance.SecurityControlId` and `Resources[].Id` so remediation targets are not guessed.
2. Identify the seven affected S3.5 buckets, eight S3.20 buckets, and two EC2.2 default security groups. Avoid broad permission changes.
3. Inspect the account IAM password policy, security alternate contact, root authentication options, and current support role.
4. Inspect the AWS Config recorder/recording role and the exact CloudTrail destination bucket before any configuration changes.
5. Estimate incremental CloudTrail data-event and S3 server-access-logging storage charges against the lab budget.
6. Obtain explicit approval for each planned change; apply one control at a time to non-production resources, verify functionality and collect evidence.
7. Recheck Security Hub after the control evaluation has refreshed. Record before/after results **per control and resource**, not simply the headline percentage.

## 4. Scope boundaries and follow-up

- The main FinServ CloudTrail multi-Region **management-event** logging and log-file integrity checks were verified previously; **S3 object-level data events** remain separate requirements.
- FinServ VPC Flow Logs delivered `NODATA` records to CloudWatch; accepted/rejected packet metadata has not been demonstrated, and default VPC Flow Logs remain outside the verified scope.
- AWS Config's **manual** SSM remediation demonstration passed; automatic remediation was intentionally disabled. Do not change that as part of this audit without a scoped test plan.
- S3.5 failures do not prove that HTTPS access was impossible or that CloudFront OAC failed. The control tests for explicit bucket policy enforcement.
- The reported CIS assessment is neither a formal certification nor a production readiness statement.

## 5. Evidence commands (read-only)

```powershell
aws securityhub get-findings --region us-east-1 --query "Findings[?Compliance.Status=='FAILED' && RecordState=='ACTIVE'].[Compliance.SecurityControlId,Resources[].Id]" --output json --no-cli-pager
aws iam get-account-password-policy --output json --no-cli-pager
aws configservice describe-configuration-recorders --region us-east-1 --output json --no-cli-pager
aws cloudtrail describe-trails --region us-east-1 --query "trailList[].{Name:Name,S3BucketName:S3BucketName,IsMultiRegionTrail:IsMultiRegionTrail}" --output table --no-cli-pager
```

If a command fails due to account permissions, document the limitation; do not broaden permissions merely to force a passing audit.

## 6. Repository references

- [Project README](../README.md)
- [Requirements compliance matrix](requirements-compliance.md)
- [Security test results](../tests/security-test-results.md)
- [CloudFormation deployment guide](deployment-guide.md)
