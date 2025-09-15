# AWS VPC Peering — Step‑by‑Step Guide

A concise, high‑level but detailed README describing how to peer two AWS VPCs (each with a public subnet and one EC2 instance). The peering connection is configured to allow **SSH (TCP/22)** and **ICMP (ping)** between the EC2 instances.

---

## 📘 Overview

* **VPC A CIDR**: `10.0.0.0/16`
* **VPC B CIDR**: `10.1.0.0/16`
* Each VPC has **1 public subnet** with a single EC2 instance.
* **Internet Gateway** (IGW) attached to each VPC for initial public access.
* **VPC Peering** allows private communication between instances.
* **Security Groups** allow **SSH** and **ICMP** from the peer VPC CIDR.
* **Route tables** updated for cross‑VPC connectivity.

---

## 🏗️ Architecture

```
[ Internet ]
     |
  IGW VPC-A                     IGW VPC-B
  (10.0.0.0/16)                  (10.1.0.0/16)
     |                              |
  subnet-1 (10.0.1.0/24)      subnet-2 (10.1.1.0/24)
     |                              |
  EC2-A (public+private)     EC2-B (public+private)
     \_________pcx-xxxxx___________/
            VPC Peering
```

---

## ✅ Prerequisites

* AWS account with permissions for VPC, EC2, and networking.
* AWS CLI installed (`aws configure`) **or** AWS Console access.
* Non‑overlapping CIDR ranges (peering fails with overlaps).
* SSH key pair (`.pem`) for EC2 access.
* Region‑specific AMI ID (replace placeholder `ami-...`).

---

## 🚀 Steps

### 1. Create VPCs and Subnets

```bash
# VPC A
aws ec2 create-vpc --cidr-block 10.0.0.0/16 --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=VPC-A}]'

# VPC B
aws ec2 create-vpc --cidr-block 10.1.0.0/16 --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=VPC-B}]'

# Subnets
aws ec2 create-subnet --vpc-id <VPC_A_ID> --cidr-block 10.0.1.0/24 --availability-zone <AZ>
aws ec2 create-subnet --vpc-id <VPC_B_ID> --cidr-block 10.1.1.0/24 --availability-zone <AZ>

# Enable public IPs on subnets
aws ec2 modify-subnet-attribute --subnet-id <SUBNET_A_ID> --map-public-ip-on-launch
aws ec2 modify-subnet-attribute --subnet-id <SUBNET_B_ID> --map-public-ip-on-launch
```

### 2. Create and Attach Internet Gateways

```bash
# IGW for VPC A
aws ec2 create-internet-gateway --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=VPC-A-IGW}]'
aws ec2 attach-internet-gateway --vpc-id <VPC_A_ID> --internet-gateway-id <IGW_A_ID>

# Route Table for VPC A
aws ec2 create-route-table --vpc-id <VPC_A_ID>
aws ec2 create-route --route-table-id <RTB_A_ID> --destination-cidr-block 0.0.0.0/0 --gateway-id <IGW_A_ID>
aws ec2 associate-route-table --route-table-id <RTB_A_ID> --subnet-id <SUBNET_A_ID>

# Repeat for VPC B
```

### 3. Configure Security Groups

```bash
# SG in VPC A
aws ec2 create-security-group --group-name SG-A --description "Allow SSH & ICMP from VPC B" --vpc-id <VPC_A_ID>
aws ec2 authorize-security-group-ingress --group-id <SG_A_ID> --protocol tcp --port 22 --cidr 10.1.0.0/16
aws ec2 authorize-security-group-ingress --group-id <SG_A_ID> --protocol icmp --port -1 --cidr 10.1.0.0/16

# SG in VPC B
aws ec2 create-security-group --group-name SG-B --description "Allow SSH & ICMP from VPC A" --vpc-id <VPC_B_ID>
aws ec2 authorize-security-group-ingress --group-id <SG_B_ID> --protocol tcp --port 22 --cidr 10.0.0.0/16
aws ec2 authorize-security-group-ingress --group-id <SG_B_ID> --protocol icmp --port -1 --cidr 10.0.0.0/16
```

### 4. Launch EC2 Instances

```bash
aws ec2 run-instances \
  --image-id <AMI_ID> \
  --count 1 \
  --instance-type t3.micro \
  --key-name MyKey \
  --security-group-ids <SG_A_ID> \
  --subnet-id <SUBNET_A_ID> \
  --associate-public-ip-address
```

Repeat for VPC B.

### 5. Create VPC Peering Connection

```bash
# Create request
aws ec2 create-vpc-peering-connection --vpc-id <VPC_A_ID> --peer-vpc-id <VPC_B_ID>

# Accept request
aws ec2 accept-vpc-peering-connection --vpc-peering-connection-id <PCX_ID>
```

### 6. Update Route Tables

```bash
# VPC A route to VPC B
aws ec2 create-route --route-table-id <RTB_A_ID> --destination-cidr-block 10.1.0.0/16 --vpc-peering-connection-id <PCX_ID>

# VPC B route to VPC A
aws ec2 create-route --route-table-id <RTB_B_ID> --destination-cidr-block 10.0.0.0/16 --vpc-peering-connection-id <PCX_ID>
```

### 7. (Optional) Enable DNS Resolution

```bash
aws ec2 modify-vpc-peering-connection-options \
  --vpc-peering-connection-id <PCX_ID> \
  --requester-peering-connection-options '{"AllowDnsResolutionFromRemoteVpc":true}' \
  --accepter-peering-connection-options '{"AllowDnsResolutionFromRemoteVpc":true}'
```

### 8. Test Connectivity

```bash
# From EC2-A (private IP of EC2-B)
ping -c 3 10.1.1.10
ssh -i MyKey.pem ec2-user@10.1.1.10
```

### 9. Clean Up

```bash
aws ec2 delete-vpc-peering-connection --vpc-peering-connection-id <PCX_ID>
aws ec2 terminate-instances --instance-ids <INSTANCE_IDS>
# Then delete subnets, IGWs, route tables, and VPCs
```

---

## 🔧 Troubleshooting

* Peering stuck in **pending** → Accept request.
* No SSH → Check routes + SGs.
* Ping blocked → Ensure ICMP rule exists.
* Overlapping CIDRs → Peering won’t work.
* DNS issues → Enable DNS resolution in peering connection.

---

## 📌 Notes

* VPC Peering is **not transitive** (A ↔ B, B ↔ C ≠ A ↔ C).
* Security Groups cannot directly reference peer VPC SGs — use CIDRs.
* Inter‑region peering incurs transfer costs.

---

## ⚡ Example Variables

```bash
VPC_A_CIDR=10.0.0.0/16
VPC_B_CIDR=10.1.0.0/16
SUBNET_A_CIDR=10.0.1.0/24
SUBNET_B_CIDR=10.1.1.0/24
AMI_ID=ami-0abcdef1234567890
KEY_NAME=MyKey
```

---

