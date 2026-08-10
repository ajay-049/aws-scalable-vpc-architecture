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

### Phase 3: Internet Connectivity & Routing
- **Internet Gateway (IGW):** Attached an IGW to the Bastion VPC to allow inbound and outbound internet traffic.
- **Route Tables:** Configured a custom Route Table (`Bastion-RT`) with a default route (`0.0.0.0/0`) pointing to the IGW, and explicitly associated it with the Bastion Public Subnet to make it truly "Public".

### Phase 4: Security & Bastion Host Deployment
- **Security Groups:** Created `Bastion-SG` acting as a stateful firewall, strictly allowing inbound SSH (Port 22) traffic from the public internet.
- **Bastion Host (Jump Server):** Deployed an Amazon Linux 2023 EC2 instance (`t3.micro`) within the Bastion Public Subnet. This server serves as the sole secure gateway for administrators to access private application servers via SSH.


### Phase 5: Application VPC & High Availability (HA) Architecture
- **App VPC (`172.32.0.0/16`):** Created a dedicated, isolated network for application workloads with a non-overlapping CIDR block to prevent routing conflicts.
- **Multi-AZ Subnet Design:** Designed for Fault Tolerance and High Availability by spanning subnets across two distinct Availability Zones (AZs):
  - **Public Subnets (`172.32.1.0/24`, `172.32.2.0/24`):** To host internet-facing resources like ALBs and NAT Gateways.
  - **Private Subnets (`172.32.3.0/24`, `172.32.4.0/24`):** Fully isolated networks to host backend application EC2 instances via Auto Scaling Groups.

  ### Phase 6: App VPC Internet Connectivity
- **Internet Gateway (IGW):** Created and attached `App-IGW` to the Application VPC.
- **Public Routing:** Created a custom Route Table (`App-Public-RT`) routing `0.0.0.0/0` traffic to the IGW. Explicitly associated only the two Public Subnets to this route table, leaving the Private Subnets fully isolated from direct internet access.

### Phase 7: Private Subnet Internet Access (NAT Gateway)
- **NAT Gateway:** Deployed a NAT Gateway in `App-Public-Subnet-1A` and associated an Elastic IP (EIP) with it. This acts as a proxy for outbound internet traffic.
- **Private Routing:** Created a dedicated Route Table (`App-Private-RT`) routing `0.0.0.0/0` traffic to the NAT Gateway. Associated both Private Subnets with this Route Table, enabling outbound internet access for private instances (e.g., for OS patching) while blocking all inbound internet connections.

### Phase 8: Cross-VPC Private Routing (Transit Gateway)
- **Transit Gateway:** Provisioned a Transit Gateway (`Main-Transit-Gateway`) to act as a centralized cloud router. This establishes a highly secure, private peering connection between the Bastion VPC and the Application VPC, ensuring SSH traffic never traverses the public internet.

- **VPC Attachments:** Created dedicated Transit Gateway Attachments for both the `Bastion-VPC` and `App-VPC`, effectively plugging both isolated networks into the centralized routing hub.