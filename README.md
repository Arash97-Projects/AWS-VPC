# AWS VPC Project — 1 VPC, 4 Subnets (2 Public, 2 Private)

## 📌 Project Overview

* **Goal:** Create a single VPC with 4 subnets (2 public, 2 private).
* **Resources:** VPC, Subnets, Internet Gateway, NAT Gateway, Route Tables, Security Groups, EC2 Instances.
* **Connectivity:** Private instances access the internet via a NAT Gateway in the public subnet.

---

## 📐 Architecture Diagram

![tree](https://github.com/user-attachments/assets/693d7d6b-b5cc-4f7b-8a42-b5edf466c80b)

---

## 🌐 CIDR Block Plan

| Component        | CIDR Block  |
| ---------------- | ----------- |
| VPC              | 10.0.0.0/16 |
| Public Subnet A  | 10.0.1.0/24 |
| Public Subnet B  | 10.0.2.0/24 |
| Private Subnet A | 10.0.3.0/24 |
| Private Subnet B | 10.0.4.0/24 |

---

## 🛠️ Prerequisites

* [x] AWS Account with VPC/EC2 permissions.
* [x] AWS CLI installed (`aws configure`).
* [x] SSH Key Pair or Session Manager permissions.

---

## 🚀 Step-by-Step Setup (Console)

### 1. Create VPC

```markdown
Go to **VPC Service** → Create VPC → CIDR `10.0.0.0/16`
```

### 2. Create Subnets

* **2 Public:** Enable auto-assign public IPv4.
* **2 Private:** No public IPs.

### 3. Create Internet Gateway (IGW)

```markdown
VPC → Internet Gateways → Create → Attach to VPC
```

### 4. Configure Route Tables

* **Public RT:** `0.0.0.0/0` → IGW.
* **Private RT:** Later, `0.0.0.0/0` → NATGW.

### 5. Create NAT Gateway

* Allocate Elastic IP.
* Place NATGW in **Public Subnet**.
* Update Private RT → `0.0.0.0/0` → NATGW.

### 6. Configure Security Groups

* **Public SG:** Allow SSH (22) from your IP.
* **Private SG:** Allow SSH only from Public SG.

### 7. Launch EC2 Instances

* Launch **1 EC2** in each subnet.
* Public → with public IP.
* Private → without public IP.

### 8. Test Connectivity

> Steps:

* [x] SSH to Public EC2.
* [x] From Public EC2 → SSH into Private EC2.
* [x] From Private EC2 → Run `curl https://ifconfig.co` → should show NAT EIP.

### 9. Cleanup

> Important: Always delete resources to avoid charges.

* Terminate EC2s.
* Delete NATGW + release EIP.
* Delete IGW.
* Delete Subnets, Route Tables, then VPC.

---

## 💻 AWS CLI Example

```bash
# Create VPC
VPC_ID=$(aws ec2 create-vpc --cidr-block 10.0.0.0/16 --query 'Vpc.VpcId' --output text)
aws ec2 create-tags --resources $VPC_ID --tags Key=Name,Value=MyVPC

# Create Subnet
PUB_A=$(aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block 10.0.1.0/24 --availability-zone us-east-1a --query 'Subnet.SubnetId' --output text)
PRIV_A=$(aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block 10.0.3.0/24 --availability-zone us-east-1a --query 'Subnet.SubnetId' --output text)

# Enable Public IP Auto-assign
aws ec2 modify-subnet-attribute --subnet-id $PUB_A --map-public-ip-on-launch

# Create IGW
IGW_ID=$(aws ec2 create-internet-gateway --query 'InternetGateway.InternetGatewayId' --output text)
aws ec2 attach-internet-gateway --internet-gateway-id $IGW_ID --vpc-id $VPC_ID

# Create Public Route Table
PUB_RTB=$(aws ec2 create-route-table --vpc-id $VPC_ID --query 'RouteTable.RouteTableId' --output text)
aws ec2 create-route --route-table-id $PUB_RTB --destination-cidr-block 0.0.0.0/0 --gateway-id $IGW_ID

# Create NAT Gateway
ALLOCATION_ID=$(aws ec2 allocate-address --domain vpc --query 'AllocationId' --output text)
NATGW_ID=$(aws ec2 create-nat-gateway --subnet-id $PUB_A --allocation-id $ALLOCATION_ID --query 'NatGateway.NatGatewayId' --output text)
aws ec2 wait nat-gateway-available --nat-gateway-ids $NATGW_ID

# Private Route Table
PRIV_RTB=$(aws ec2 create-route-table --vpc-id $VPC_ID --query 'RouteTable.RouteTableId' --output text)
aws ec2 create-route --route-table-id $PRIV_RTB --destination-cidr-block 0.0.0.0/0 --nat-gateway-id $NATGW_ID

# Security Groups
PUB_SG=$(aws ec2 create-security-group --group-name public-sg --description "Public SG" --vpc-id $VPC_ID --query 'GroupId' --output text)
aws ec2 authorize-security-group-ingress --group-id $PUB_SG --protocol tcp --port 22 --cidr YOUR.IP.ADDRESS/32

PRIV_SG=$(aws ec2 create-security-group --group-name private-sg --description "Private SG" --vpc-id $VPC_ID --query 'GroupId' --output text)
aws ec2 authorize-security-group-ingress --group-id $PRIV_SG --protocol tcp --port 22 --source-group $PUB_SG
```

---

## ✅ Testing Checklist

* [x] SSH into Public EC2 via public IP.
* [x] From Public EC2 → SSH into Private EC2.
* [x] From Private EC2 → Run `curl https://ifconfig.co`.
* [x] Verify Private EC2s cannot be reached directly from the internet.

---

## ⚠️ Cleanup

* [x] Terminate EC2s.
* [x] Delete NATGW + release EIP.
* [x] Delete Route Tables, Subnets, IGW.
* [x] Delete VPC.


---

## 💡 Notes

* NAT Gateway charges per hour + traffic — **delete after testing**.
* Use NATGW for production, NAT instance only for demos.
* Consider **AWS Systems Manager Session Manager** for private EC2 access instead of a bastion host.
