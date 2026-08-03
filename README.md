# Highly Available & Scalable AWS VPC Architecture

## 🎯 Project Goal
To design and deploy a modular, secure, and scalable Virtual Private Cloud (VPC) network architecture on AWS, ensuring high availability of applications using Auto Scaling, Application Load Balancing, and Transit Gateway for secure cross-VPC communication.

## 🛠️ Tech Stack & Services Used
- **Compute:** EC2, Auto Scaling Groups (ASG), Launch Templates, Golden AMI
- **Networking:** VPC, Transit Gateway, NAT Gateway, Internet Gateway, ALB, Route 53
- **Security:** Bastion Host, IAM Roles, Security Groups, AWS Systems Manager (SSM)
- **Monitoring & Storage:** CloudWatch (Flow Logs, Custom Memory Metrics), S3

## 🏗️ Architecture Implementation Steps

### Phase 1: Network Foundation (VPCs)
To isolate environments and maintain high security, the architecture is split into two distinct VPCs:
1. **Bastion VPC (`192.168.0.0/16`):** Created as a dedicated management network. This acts as the secure entry point (jump host environment) for administrators to access the application infrastructure.

### Phase 2: Subnets Configuration
- **Bastion Public Subnet (`192.168.1.0/24`):** Created inside the Bastion VPC in a specific Availability Zone. This subnet is configured to auto-assign public IPs to house the Bastion Host, acting as our public-facing secure gateway.