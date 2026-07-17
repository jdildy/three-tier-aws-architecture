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
