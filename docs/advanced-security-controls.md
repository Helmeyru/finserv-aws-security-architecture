# FinServ — Advanced Security Controls and Remaining Validation

**Project:** Manara AWS Project 8
**Region:** us-east-1
**Status:** Documentation and validation plan. The incomplete controls below must not be represented as deployed or tested.

## 1. Secrets Manager Automatic Rotation

**Status: NOT VERIFIED**

Secret storage and temporary RDS-managed credentials were demonstrated. A custom Lambda rotation function and complete scheduled rotation lifecycle were not verified.

A complete Lambda rotation demonstration would require:

1. A disposable database and test secret.
2. A Secrets Manager rotation Lambda with the required database and secret permissions.
3. Network access from Lambda to the database and Secrets Manager.
4. Rotation configuration with an appropriate schedule.
5. Verification of the createSecret, setSecret, testSecret and finishSecret steps.
6. Evidence that the new credential successfully authenticates.

The temporary project RDS database was deleted. No active custom Lambda rotation is claimed.

## 2. AWS Config Automatic Remediation

**Status: MANUAL REMEDIATION VERIFIED; AUTOMATIC NOT VERIFIED**

The project deployed an AWS Config rule, an SSM Automation runbook and a scoped remediation role.

Manual remediation completed successfully.

Automatic remediation remains disabled. To demonstrate it safely:

1. Create a disposable test resource.
2. Scope the Config rule and SSM remediation to that resource.
3. Review the Automation permissions and expected changes.
4. Enable automatic remediation for the test.
5. Introduce a controlled noncompliant configuration.
6. Verify that AWS Config triggers SSM Automation.
7. Capture the Automation execution result and final compliance status.
8. Clean up the test resources and disable test remediation.

Manual execution is not evidence of automatic triggering.

## 3. IAM Permissions Boundaries

**Status: NOT DEMONSTRATED**

A permissions boundary sets the maximum permissions available through identity-based policies on an IAM user or role.

A proper demonstration would:

1. Create a narrowly scoped managed IAM policy.
2. Apply it as a permissions boundary to a disposable test role.
3. Attach an identity policy requesting an action outside the boundary.
4. Test or simulate the effective permissions.
5. Record that the action remains unavailable despite the identity policy.

A boundary does not independently grant permissions and does not replace an SCP.

No enforced permission boundary test is claimed for FinServ.

## 4. AWS Organizations Service Control Policies

**Status: REFERENCE ONLY; NOT DEPLOYED**

Service Control Policies (SCPs) set organizational permission guardrails for member accounts. They do not independently grant permissions.

An example deny-root strategy requires careful review of AWS Organizations scope, principal conditions, recovery procedures and management-account limitations.

The project contains an SCP reference template, but no organization-wide enforcement test was performed.

**Safety:** Do not attach an account-wide deny policy merely to complete this educational project. Test organizational policies in a separate disposable member account.

## 5. AWS WAF Blocking

**Status: MANAGED RULES IN COUNT MODE; BLOCKING NOT VERIFIED**

AWS WAF was associated with CloudFront. Managed rules were configured in Count mode for observation.

Count mode records matching traffic but does not actively block those requests.

A safe validation would:

1. Create a temporary rule matching a unique test path through CloudFront.
2. Configure that specific test rule to Block.
3. Request the matching CloudFront path.
4. Verify HTTP 403 and the corresponding WAF metric or sampled request.
5. Verify an ordinary path remains accessible.
6. Remove the temporary test rule when finished.

A direct API Gateway request may bypass the CloudFront Web ACL and is not sufficient proof of CloudFront WAF enforcement.

## 6. Submission Evidence

Already demonstrated:

- Segmented VPC architecture and restricted security groups.
- S3 encryption with customer-managed KMS.
- API Gateway health check and direct Lambda/DynamoDB operations.
- GuardDuty synthetic findings and EventBridge/Lambda incident processing.
- Multi-Region FinServ CloudTrail auditing.
- AWS Config compliance and manual SSM remediation.
- Security Hub CIS v5.0 assessment.
- VPC Flow Logs ingestion and monitoring configuration.

Not demonstrated end to end:

- Custom Lambda secret rotation.
- Automatic AWS Config remediation.
- IAM permissions boundary enforcement.
- AWS Organizations SCP enforcement.
- WAF request blocking.
- Automatic incident-response SNS notification delivery.

This document describes completed controls and remaining implementation work. It is not evidence that the outstanding controls passed testing.