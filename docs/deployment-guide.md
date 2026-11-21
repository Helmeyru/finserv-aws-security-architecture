# FinServ — CloudFormation Deployment and Verification Guide

**Project:** Manara Project 8 — Secure Multi-Tier Architecture with GuardDuty, KMS & Security Hub

**Region:** `us-east-1`

**Scope:** Educational deployment reference and review guide. This is **not** a verified, one-command, clean-account redeployment procedure.

**Last documentation review:** November 21, 2026

## 1. Important scope and safety notes

This repository preserves the CloudFormation files used throughout a multi-day AWS security lab. Some files are cumulative, some configure resources created by earlier stacks, and some are alternative or temporary experiments. **Do not deploy all templates together or into the original account without reviewing existing CloudFormation stacks first.** Named resources, security-group rules, account-level configuration, AWS Config recorders, and Organizations policies can conflict with existing resources.

The currently demonstrated application uses **CloudFront and a private S3 website, API Gateway `GET /health`, a VPC-attached Lambda function, and DynamoDB through a gateway endpoint**. The VPC includes public, private application, and isolated database subnets, but there is **no persistent RDS database, EC2 web/application fleet, or NAT gateway**. Web/app/DB security groups also illustrate a future conventional three-tier workload.

AWS Config remediation was exercised **manually**, WAF managed rules used **Count** mode, and the Organizations SCP is **reference-only**. Successful creation of individual stacks is not evidence that the repository was redeployed from scratch. See [requirements compliance](requirements-compliance.md) and [recorded security tests](../tests/security-test-results.md).

## 2. Prerequisites

- An AWS account with explicit approval to create and incur charges for infrastructure. Use a dedicated lab account wherever possible.
- AWS CLI v2 and PowerShell on Windows, or an equivalent shell with commands adapted accordingly.
- AWS credentials configured with permissions appropriate to the selected template. Certain IAM templates require `CAPABILITY_NAMED_IAM`; Organizations policies require an eligible Organizations management context.
- Permission to create VPC, S3, KMS, CloudFormation, Lambda, API Gateway, DynamoDB, GuardDuty, Security Hub, Config, CloudTrail, CloudWatch, SNS, and Systems Manager resources as needed.
- A budget and billing alerts. GuardDuty, Security Hub, Config, CloudTrail, CloudWatch, KMS, CloudFront, WAF, and test RDS instances can incur charges.
- A local copy of this repository and a clear list of *existing* stacks and resources in the target account.

### Read-only environment checks

Run these commands from the repository root; they do **not** create or delete AWS resources:

```powershell
aws sts get-caller-identity --output json --no-cli-pager
aws configure get region
aws cloudformation list-stacks --region us-east-1 --stack-status-filter CREATE_COMPLETE UPDATE_COMPLETE UPDATE_ROLLBACK_COMPLETE --query "StackSummaries[].{Name:StackName,Status:StackStatus}" --output table --no-cli-pager
```

Confirm the account and region before any deployment. Do not paste real credentials, private secrets, or unredacted account identifiers into the public repository.

## 3. Recommended dependency order — architectural reference

The following order is inferred from the templates' roles and known dependencies. **It is not a claim of successful fresh-account execution**, and some files must not be deployed if their resources already exist.

| Phase | Files / stacks | Dependency and caution |
|---|---|---|
| 1 — VPC | `cloudformation/Recovered-Templates/manara-p8-network.yaml` | Creates VPC, two public, two private app, and two isolated DB subnets; note the VPC ID, subnet IDs and route table ID for later phases. |
| 2 — Network policies | `cloudformation/finserv-security-groups.yaml` | Needs `VpcId` and regional `DynamoDBPrefixListId`. Includes the application HTTPS/DynamoDB egress rule. |
| 3 — Encryption | `cloudformation/Recovered-Templates/finserv-encryption.yaml` | Creates a customer-managed KMS key and an S3 bucket with SSE-KMS. Avoid naming conflicts and account charges. |
| 4 — Data tier | `cloudformation/Day 14/finserv-data-tier.yaml` | Needs `VpcId` and `AppRouteTableId`; creates `finserv-transactions` DynamoDB table and gateway endpoint. |
| 5 — Private application | `cloudformation/Day 13/finserv-app-tier.yaml` | Needs the app security group, two private app subnets, and existing DynamoDB table. Creates named Lambda `finserv-private-app`. |
| 6 — API | `cloudformation/Day 15/finserv-api-tier.yaml` | Requires the named Lambda `finserv-private-app`. Exposes **only** `GET /health`; do not describe `/transaction` as an API route. |
| 7 — Web | `cloudformation/Day 12/finserv-web-tier.yaml` | Creates private S3 origin, CloudFront OAC and distribution; upload site content separately if required. |
| 8 — Audit, compliance, detection | Days 06–09, using the files listed below | Account-level recorders, trails and detectors are sensitive to duplicates and service activation charges; inspect existing deployments first. |
| 9 — Incident response, alerts | Days 10–11 | EventBridge, Lambda, SNS and related policies must have matching resource names/ARNs. A direct SNS publish test is not the same as an end-to-end GuardDuty-to-Lambda-to-SNS test. |
| 10 — Scoped remediation | Day 16 | Requires an approved disposable bucket, a functioning SSM document and correctly scoped IAM role; **automatic remediation is disabled**. |
| 11 — WAF and monitoring | Days 17 and 19 | WAF/CloudFront coupling can involve global `us-east-1` scope. Monitoring alarms require a valid SNS topic ARN and existing Lambda names. WAF Count does not block requests. |

**Potential duplication:** `finserv-security-groups.yaml` already defines an app-to-DynamoDB egress rule; `Day 14/finserv-dynamodb-egress.yaml` also defines this rule. Do not deploy both without first checking existing security-group egress permissions. They can conflict.

## 4. Repository template inventory (28 YAML files)

| Group | Template | Role / classification |
|---|---|---|
| Foundation | `cloudformation/Recovered-Templates/manara-p8-network.yaml` | VPC and subnet foundation |
| Foundation | `cloudformation/finserv-security-groups.yaml` | Tier security groups and egress policy |
| Foundation | `cloudformation/Recovered-Templates/finserv-encryption.yaml` | Customer-managed KMS key and private SSE-KMS S3 bucket |
| Day 04 | `cloudformation/Day 04/finserv-iam-role.yaml` | IAM access-control exercise; inspect references before deployment |
| Day 05 | `cloudformation/Day 05/finserv-secrets-manager.yaml` | Secrets Manager exercise; not proof of automatic RDS Lambda rotation |
| Day 06 | `cloudformation/Day 06/finserv-cloudtrail.yaml` | CloudTrail auditing |
| Day 07 | `cloudformation/Day 07/finserv-aws-config.yaml` | AWS Config storage bucket/policy; not a complete recorder in this file |
| Day 07 | `cloudformation/Day 07/finserv-config-rules.yaml` | Compliance rules; needs configured AWS Config service |
| Day 08 | `cloudformation/Day 08/finserv-guardduty.yaml` | GuardDuty configuration |
| Day 09 | `cloudformation/Day 09/finserv-securityhub.yaml` | Security Hub aggregation; standards status requires separate verification |
| Day 10 | `cloudformation/Day 10/finserv-incident-response.yaml` | Initial incident-response design |
| Day 11 | `cloudformation/Day 11/finserv-incident-response-day11.yaml` | Incident-response variant; review before reuse |
| Day 11 | `cloudformation/Day 11/finserv-sns-alerts.yaml` | SNS notification topic |
| Day 11 | `cloudformation/Day 11/finserv-sns-test.yaml` | SNS test exercise; review before reuse |
| Day 12 | `cloudformation/Day 12/finserv-web-tier.yaml` | Private S3/CloudFront web tier |
| Day 13 | `cloudformation/Day 13/finserv-app-tier.yaml` | VPC-attached Lambda and DynamoDB access |
| Day 14 | `cloudformation/Day 14/finserv-data-tier.yaml` | DynamoDB table and gateway endpoint |
| Day 14 | `cloudformation/Day 14/finserv-dynamodb-egress.yaml` | Additional egress rule; potential duplicate |
| Day 15 | `cloudformation/Day 15/finserv-api-tier.yaml` | API Gateway GET `/health` route |
| Day 16 | `cloudformation/Day 16/finserv-day16-safe-remediation.yaml` | Scoped SSM Automation document |
| Day 16 | `cloudformation/Day 16/finserv-day16-remediation-lab.yaml` | Lab resources for controlled remediation testing |
| Day 16 | `cloudformation/Day 16/finserv-day16-config-remediation.yaml` | Config rule/remediation settings; `Automatic: false` |
| Day 17 | `cloudformation/Day 17/finserv-waf.yaml` | Edge WAF reference and managed-rule configuration; verify deployed behavior |
| Day 18 | `cloudformation/Day 18/finserv-day18-ebs.yaml` | Temporary encrypted EBS demonstration; deleted after testing |
| Day 18 | `cloudformation/Day 18/finserv-day18-rds-subnet.yaml` | Temporary DB subnet group demonstration |
| Day 18 | `cloudformation/Day 18/finserv-day18-rds.yaml` | Temporary PostgreSQL RDS demonstration; deleted after testing |
| Day 18 | `cloudformation/Day 18/finserv-day18-scp.yaml` | Organizations SCP reference; **not deployed** |
| Day 19 | `cloudformation/Day 19/finserv-day19-monitoring.yaml` | Three CloudWatch alarms; requires SNS topic ARN |

## 5. Parameters and outputs to look up

Do not copy account-specific IDs from someone else's deployment. Inspect the current account and CloudFormation stack outputs instead.

| Parameter | Source / expected value |
|---|---|
| `VpcId` | Network stack output `VpcId` |
| `PrivateSubnet1Id` / `PrivateSubnet2Id` | Two IDs in network output `PrivateAppSubnetIds` |
| `AppRouteTableId` | Existing private application route-table ID from EC2/VPC or network stack resources; **not** an output of the published network template |
| `AppSecurityGroupId` | Security-groups stack output `AppSecurityGroupId` |
| `DynamoDBPrefixListId` | AWS-managed DynamoDB prefix list for `us-east-1`; query it in the target account |
| `DynamoDBTableName` | `finserv-transactions` (created by data-tier template) |
| `SNSTopicArn` | SNS alerts stack output `SecurityAlertsTopicArn` |
| `TestBucketName` | Approved disposable S3 bucket **only**; never a production bucket |
| `RemediationRoleArn` | IAM role scoped to the SSM remediation exercise; must exist before remediation configuration |
| `RemediationDocumentName` | SSM Automation document `FinServ-SafeS3EncryptionRemediation` |

For example, the following are **read-only** lookup commands:

```powershell
aws cloudformation describe-stacks --stack-name manara-p8-network --region us-east-1 --query "Stacks[0].Outputs" --output table --no-cli-pager
aws cloudformation describe-stacks --stack-name finserv-security-groups --region us-east-1 --query "Stacks[0].Outputs" --output table --no-cli-pager
aws ec2 describe-managed-prefix-lists --region us-east-1 --filters Name=prefix-list-name,Values=com.amazonaws.us-east-1.dynamodb --query "PrefixLists[].{Id:PrefixListId,Name:PrefixListName}" --output table --no-cli-pager
```

A stack name here is an example from the recorded lab; use `list-stacks` to confirm its actual name in your account first.

## 6. Safe template inspection and validation

To validate syntax with AWS CloudFormation without creating resources, inspect each file individually. Example:

```powershell
aws cloudformation validate-template --region us-east-1 --template-body "file://cloudformation/Recovered-Templates/manara-p8-network.yaml" --no-cli-pager
aws cloudformation validate-template --region us-east-1 --template-body "file://cloudformation/Day 13/finserv-app-tier.yaml" --no-cli-pager
aws cloudformation validate-template --region us-east-1 --template-body "file://cloudformation/Day 14/finserv-data-tier.yaml" --no-cli-pager
```

Template validation does **not** test IAM authorization, referenced resources, runtime application behavior, pricing, nor cross-stack correctness. Review each template's `Parameters`, `Resources`, `Outputs`, and hardcoded names before deployment.

For any future rebuild, first plan each stack's parameters and create a CloudFormation **Change Set**. Review its proposed resource additions, replacements and deletions. Obtain approval before executing, especially for IAM, encryption, AWS Config, and resources with stored data.

## 7. Verification and evidence

Existing Day 21–23 evidence is recorded in [Security Test Results](../tests/security-test-results.md). It reports:

- API Gateway `GET /health` HTTP 200, while `/transaction` was not configured as an API route.
- Direct Lambda synthetic transaction successfully writing to and reading from DynamoDB through the private application design.
- Enabled GuardDuty and Security Hub with a **sample** GuardDuty finding (not a real compromise).
- AWS Config compliant checks; a controlled SSM runbook successfully executed **manually**.
- Three CloudWatch alarms configured with SNS actions; a **direct** SNS email test was received. Alarm-triggered delivery was not proven by that direct test.
- KMS-encrypted S3 demonstration, AWS service-access controls, cost review and cleanup evidence.

Additional **read-only** sample verification commands:

```powershell
aws cloudformation describe-stacks --region us-east-1 --stack-name finserv-app-tier --query "Stacks[0].StackStatus" --output text --no-cli-pager
aws lambda get-function-configuration --region us-east-1 --function-name finserv-private-app --query "{State:State,VpcId:VpcConfig.VpcId,Subnets:VpcConfig.SubnetIds}" --output json --no-cli-pager
aws dynamodb describe-table --region us-east-1 --table-name finserv-transactions --query "Table.{Name:TableName,Status:TableStatus}" --output json --no-cli-pager
aws cloudwatch describe-alarms --region us-east-1 --alarm-name-prefix finserv- --query "MetricAlarms[].{Name:AlarmName,State:StateValue,Actions:ActionsEnabled}" --output table --no-cli-pager
```

These checks assume the corresponding resources still exist. A missing or renamed resource is not evidence that the original test never occurred.

## 8. Cleanup and cost control

The Day 18 test RDS instance and EBS volume were intentionally deleted after verification. The live lab may still incur usage charges (for example KMS, CloudTrail, Config, CloudWatch, Security Hub, GuardDuty or WAF). Use Cost Explorer and current service consoles to verify charges; past billing figures are not a forecast.

Before deleting **any** stack, inventory its resources with `describe-stack-resources`, determine whether it is referenced by another stack, and confirm that retained S3 buckets/objects, KMS keys and logs have appropriate retention. Emptying buckets, scheduling KMS key deletion, removing a Config recorder, or deleting CloudTrail logs can destroy evidence. **This guide deliberately provides no blanket delete command.**

### Cleanup checklist

1. Confirm the exact AWS account and region and obtain permission for cleanup.
2. Identify live versus demonstration stacks, and export any evidence needed for grading.
3. Check for dependencies and S3/KMS retained resources; take backups where required.
4. Delete only explicitly approved temporary resources, in reverse dependency order.
5. Verify no unintended RDS instances, EBS volumes, snapshots or billable services remain.
6. Review Cost Explorer after billing data catches up.

## 9. Known limitations and security decisions

- The implemented application is a small **serverless** lab, not a production financial-services application. The isolated DB subnet tier is present as architecture; no RDS instance remains running.
- AWS WAF managed rules in Count mode are observational; the proposed blocking rule was not deployed.
- The SSM/AWS Config remediation control was intentionally manual, not automatic.
- SCP deployment and IAM permission boundaries are not established as enforced controls by the recorded evidence.
- Secrets Manager exercises are not evidence of automatic RDS credential rotation through a Lambda rotation function.
- Security Hub aggregation is demonstrated, but formal CIS/PCI standards coverage and scores were not established by the verification report.
- A secondary CloudTrail in `us-east-2` had a delivery error during auditing. Investigate its ownership and purpose before making changes.
- The repository has not been validated as a fully automated, end-to-end fresh-account deployment.

**Reviewers:** For the status of every learning objective, start with [Requirements Compliance Matrix](requirements-compliance.md). For actual command outputs and test boundaries, see [Security Test Results](../tests/security-test-results.md).
