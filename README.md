# three-tier-aws-architecture

## Goal

Design and deploy a highly available 3-tier web application on AWS utilizing the AWS Well-Architected Framework

## Architecture

VPC 
- CIDR: 10.0.0.0/16

Availability Zones
- us-east-1a
- us-east-1b

Subnets

| Name | CIDR | Purpose |
|------|------|---------|
| Public A | 10.0.1.0/24 | ALB, NAT Gateway |
| Public B | 10.0.2.0/24 | ALB |
| Private App A | 10.0.3.0/24 | EC2 |
| Private App B | 10.0.4.0/24 | EC2 |
| Private DB A | 10.0.5.0/24 | RDS |
| Private DB B | 10.0.6.0/24 | RDS |


Internet Gateway
- Successfully attached to the VPC housing the three-tier weba application

NAT Gateway
Configured in the correct Subnet (Public AZ-A), utilizing the private-app-rt to sucessfully communicate between ec2 and nat
- Original route table rulings keeps traffic local (10.0.0.0/16)
- New route table rule routes all other traffic to the NAT (0.0.0.0)

## Security Group 
### 1. Application Load Balancer Security Group (ALB-SG)
**Purpose:** Accept incoming web traffic from users.

**Inbound Rules**
| Protocol | Port | Source |
|----------|------|--------|
| HTTP | 80 | 0.0.0.0/0 |

---


### 2. EC2 Security Group (EC2-SG)
**Purpose:** Allow only the Application Load Balancer to communicate with the application servers.

**Inbound Rules**
| Protocol | Port | Source |
|----------|------|--------|
| HTTP | 80 | ALB-SG |

---

### 3. RDS Security Group (RDS-SG)
**Purpose:** Allow only the application servers to connect to the database.

**Inbound Rules**
| Protocol | Port | Source |
|----------|------|--------|
| MySQL/Aurora | 3306 | EC2-SG |


## IAM Role

### EC2 IAM Role ("three-tier-ec2-role")

**Purpose:** Grants EC2 instances temporary AWS credentials without storing keys on the instance

**Policy Attached**
- AmazonSSManagedInstanceCore

**Benefits**
- Enables secure management through AWS Systems Manager Sessions Manager
- Eliminates the need to store AWS access keys on EC2 instances
- Follows AWS Security best practices by using temporary credentials

---

## Launch Template

### Launch Template (`three-tier-launch-template`)

**Purpose:** Serves as a reusable blueprint for EC2 instances launched by the Auto Scaling Group.

**Configuration**
- Amazon Linux 2023
- Instance Type: `t2.micro` (or `t3.micro`)
- Security Group: `EC2-SG`
- IAM Role: `three-tier-ec2-role`
- Root Volume: `8 GiB gp3`
- Public IP: Disabled
- Subnet: Configured by the Auto Scaling Group
