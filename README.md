flowchart LR
    %% Entry + DNS
    User[End User] -->|HTTPS| Route53[Route 53\nflyforward.aero]
    Route53 --> ALB[ALB\nprod ALB in public subnets]

    %% Workload VPC
    subgraph VPC_Workload[Production Workload VPC]
        ALB -->|HTTP/HTTPS| ASG[ASG\nEC2 instances in private subnets]
        ASG -->|App traffic| BastionSG[(Bastion SG)]
    end

    %% Bastion (Mgmt access)
    Internet[Admin over Internet] -->|SSH| Bastion[Bastion Host]
    Bastion -->|SSH| ASG

    %% DB VPC
    subgraph VPC_DB[Production DB VPC]
        RDS[RDS SQL Server\nMulti-AZ]
    end

    %% Network / VPN
    subgraph Network_VPC[Network VPC]
        ClientVPN[Client VPN Endpoint]
    end

    %% Management / AD
    subgraph Mgmt_VPC[Management VPC]
        AWSAD[AWS Managed AD]
    end

    %% Connectivity
    ClientVPN -->|Secure access| VPC_Workload
    ClientVPN -->|Optional| VPC_DB

    VPC_Workload <-->|Peering/TGW| VPC_DB
    VPC_Workload <-->|Peering/TGW| Mgmt_VPC

    %% RDS integrations
    ASG -->|DB connections| RDS
    AWSAD -->|Directory join| RDS

    %% Logging / S3
    ALB -->|Access logs| S3Logs[S3 Logs Bucket]
    RDS -->|Backups| S3Backups[S3 RDS Backup Bucket]
    ClientVPN -->|Connection logs| CWLogs[CloudWatch Log Group]
