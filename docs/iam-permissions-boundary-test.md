# IAM Permissions Boundary — Policy Simulation

**Project:** Manara AWS Project 8 — FinServ
**Test type:** AWS IAM Policy Simulator
**Result:** PASSED — policy simulation

## Objective

Verify that an IAM permissions boundary restricts permissions even when an identity-based policy allows an action.

## Test Configuration

A simulated identity policy allowed:

- s3:ListBucket
- s3:DeleteBucket

The simulated permissions boundary allowed s3:ListBucket and explicitly denied s3:DeleteBucket.

The target was a fictional S3 bucket named finserv-boundary-demo-example.

## Results

| Action | Final decision | Allowed by boundary |
|---|---|---|
| s3:ListBucket | allowed | true |
| s3:DeleteBucket | explicitDeny | false |

## Evidence

[View IAM Policy Simulator Results](evidence/iam-permissions-boundary-test.json)

## Conclusion

The AWS IAM Policy Simulator confirmed that the permissions boundary prevented s3:DeleteBucket despite an Allow statement in the identity policy.

This demonstrates permissions-boundary policy evaluation, not enforcement on an actual deployed IAM role. No IAM resources or S3 buckets were created for this test.