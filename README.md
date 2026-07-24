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

**NOTE** 
NAT Gateway was initially configured to demonstrate private subnet outbound internet access. 
It was removed after validation to avoid unnecessary costs. In production, NAT Gateways 
would typically remain deployed for private workloads requiring outbound access.
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


## Auto Scaling Group (ASG)

### Purpose
The Auto Scaling Group (ASG) manages the lifecycle and availability of EC2 instances in the application tier. It ensures the desired number of instances are running and automatically replaces unhealthy instances.

### Configuration

**Launch Template**
- `three-tier-launch-template`

**Subnets**
- Private-App-A (Availability Zone A)
- Private-App-B (Availability Zone B)

**Capacity**
- Minimum: 2 instances
- Desired: 2 instances
- Maximum: 2 instances

### Architecture

The ASG distributes EC2 instances across multiple Availability Zones to improve availability.

```
Availability Zone A          Availability Zone B

Private-App-A                Private-App-B
      |                            |
      v                            v
   EC2 Instance              EC2 Instance
```

### Responsibilities

The ASG is responsible for:

- Launching EC2 instances using the Launch Template.
- Maintaining the desired number of running instances.
- Replacing failed or unhealthy instances.
- Supporting horizontal scaling when scaling policies are configured.

### Design Decisions

**Why use an ASG instead of manually creating EC2 instances?**

Auto Scaling Groups provide self-healing capabilities. If an instance fails, the ASG automatically launches a replacement instance, reducing manual intervention and improving application availability.

**Why deploy across multiple Availability Zones?**

Deploying instances across multiple Availability Zones prevents a single AZ failure from taking down the application tier.

**Why are instances deployed in private subnets?**

The application tier should not be directly accessible from the internet. Traffic should flow through the Application Load Balancer, which provides centralized traffic management and reduces the attack surface.
