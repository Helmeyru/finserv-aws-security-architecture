# FinServ AWS Security Architecture — Verified Lab Design

The diagram distinguishes deployed components, tested integrations, and features whose end-to-end operation remains unverified. It is a logical view; placement does not imply every component is inside the VPC.

```mermaid
flowchart LR
    U["Users / API clients"]

    subgraph Edge["Public access and static website"]
        direction TB
        WAF["AWS WAF<br/>Managed rules: COUNT"]
        CF["Amazon CloudFront"]
        S3["Private S3 website bucket"]
        API["Amazon API Gateway<br/>GET /health verified"]
        WAF -. "Web ACL attached" .-> CF
        CF -->|"Origin Access Control"| S3
    end

    subgraph VPC["FinServ VPC — two Availability Zones"]
        direction TB
        PUB["Two public subnets"]
        APP["Two private app subnets"]
        DB["Two isolated DB subnets<br/>No persistent RDS instance"]
        L["Application Lambda<br/>VPC-attached"]
        EP["DynamoDB gateway endpoint"]
        APP --- L
        APP --- EP
        L -->|"HTTPS via endpoint"| EP
    end

    DDB["Amazon DynamoDB<br/>Regional managed service"]

    subgraph Govern["Security, compliance and audit"]
        direction TB
        IAM["IAM scoped policies"]
        KMS["Customer-managed KMS key<br/>S3 SSE-KMS"]
        CT["CloudTrail<br/>Multi-Region management events<br/>Integrity validation"]
        AUDIT["Private CloudTrail audit S3 bucket"]
        CT -->|"Audit records"| AUDIT
        CONFIG["AWS Config<br/>3 FinServ rules compliant"]
        SSM["SSM Automation<br/>Manual remediation verified"]
        GD["GuardDuty<br/>Synthetic findings"]
        SH["Security Hub<br/>CIS v5 assessment: 38%"]
        CONFIG -->|"Manual remediation"| SSM
        GD -. "Findings aggregated" .-> SH
    end

    subgraph Response["Verified detection-to-response path"]
        direction TB
        EB["Amazon EventBridge<br/>High-severity rule"]
        IR["Incident-response Lambda<br/>ESCALATE tested"]
        EB -->|"Matched event"| IR
    end

    subgraph Observe["Monitoring and notification"]
        direction TB
        FLOW["VPC Flow Logs<br/>ALL traffic"]
        LOGS["CloudWatch Logs<br/>Flow Logs: 7-day retention"]
        ALARM["CloudWatch alarms"]
        SNS["SNS alert topic<br/>Direct email tested"]
        FLOW -->|"Ingestion verified: NODATA records"| LOGS
        ALARM -. "Alarm-to-SNS untested" .-> SNS
    end

    U --> CF
    U --> API
    API -.->|"Application integration not fully verified"| L
    EP --> DDB
    GD -->|"Synthetic severity-8 finding"| EB
    IR -->|"ESCALATE log event"| LOGS
    IR -. "Automatic SNS skipped in test" .-> SNS
    VPC -. "VPC-wide network metadata" .-> FLOW
    L -. "Application logs and metrics" .-> LOGS
    IAM -. "Execution permissions" .-> L
    KMS -. "SSE-KMS" .-> S3

    classDef limited fill:#fff0d6,stroke:#b7791f,color:#543b16
    classDef verified fill:#e9f7ef,stroke:#39875e,color:#153f2b
    class WAF,DB,SNS,ALARM limited
    class EB,IR,FLOW,LOGS,CONFIG verified
```

**Evidence boundaries:** The GuardDuty finding was synthetically generated. Lambda logged `ESCALATE`, but its automatic SNS notification was skipped. CloudWatch received VPC Flow Log `NODATA` records; specific accepted/rejected traffic records were not demonstrated. RDS test resources were removed after testing. CIS 38% was the observed account score on November 22, 2026, not a certification or Manara project grade.

**Application caveat:** API Gateway `GET /health` was verified; this diagram does not imply that the CloudFront website successfully called a transaction API or that the entire application flow was end-to-end tested.
