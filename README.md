```mermaid
flowchart LR
  %% Entry + DNS
  User[End user] -->|HTTPS| Route53[Route 53 (flyforward.aero)]
  Route53 --> ALB[ALB]

  %% Workload VPC
  subgraph VPC_Workload[Production Workload VPC]
    ALB -->|HTTP/HTTPS| ASG[ASG EC2 instances]
    Bastion[Bastion host] -->|SSH| ASG
  end

  %% DB VPC
  subgraph VPC_DB[Production DB VPC]
    RDS[RDS SQL Server]
  end

  %% Network / VPN
  subgraph Network_VPC[Network VPC]
    ClientVPN[Client VPN endpoint]
  end

  %% Management / AD
  subgraph Mgmt_VPC[Management VPC]
    AWSAD[AWS Managed AD]
  end

  %% Connectivity
  ClientVPN -->|Secure access| VPC_Workload
  ClientVPN -->|DB access| VPC_DB

  VPC_Workload <-->|TGW/peering| VPC_DB
  VPC_Workload <-->|TGW/peering| Mgmt_VPC

  %% RDS integrations
  ASG -->|DB connections| RDS
  AWSAD -->|Directory integration| RDS

  %% Logging / S3
  ALB -->|Access logs| S3Logs[S3 logs bucket]
  RDS -->|Backups| S3Backups[S3 backups bucket]
  ClientVPN -->|Connection logs| CWLogs[CloudWatch log group]
```
