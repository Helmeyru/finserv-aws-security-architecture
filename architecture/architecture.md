flowchart TB
    U["Users and API Clients"]

    subgraph Edge["Public Entry Points"]
        CF["Amazon CloudFront"]
        WAF["AWS WAF - Managed Rules in COUNT Mode"]
        S3["Private Amazon S3 - Static Website"]
        API["Amazon API Gateway"]
    end

    U --> CF
    WAF -.-> CF
    CF --> S3
    U --> API

    subgraph VPC["FinServ VPC - Two Availability Zones"]
        PUB["Two Public Subnets"]
        APP["Two Private Application Subnets"]
        DB["Two Isolated Database Subnets"]
        L["AWS Lambda - Private Application"]
        EP["DynamoDB Gateway VPC Endpoint"]

        APP --- L
        APP --- EP
        L -->|"HTTPS via VPC routing"| EP
    end

    API --> L
    EP --> DDB["Amazon DynamoDB - Transactions"]

    subgraph Security["Security and Governance"]
        IAM["IAM Least Privilege"]
        KMS["AWS KMS Encryption"]
        CT["AWS CloudTrail"]
        CONFIG["AWS Config"]
        GD["Amazon GuardDuty"]
        SH["AWS Security Hub"]
        SSM["SSM Manual Remediation"]
    end

    subgraph Monitoring["Monitoring and Notifications"]
        CW["Amazon CloudWatch"]
        SNS["Amazon SNS Alerts"]
    end

    L -. "Logs and Metrics" .-> CW
    CW --> SNS
    CT -. "Audit Events" .-> Security
    GD -. "Security Findings" .-> SH
    CONFIG -. "Compliance" .-> SSM
    IAM -. "Permissions" .-> L
    KMS -. "Encryption" .-> S3