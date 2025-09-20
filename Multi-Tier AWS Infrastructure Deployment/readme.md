# 🌐 Multi-Tier AWS Infrastructure Deployment

This project is a **hands-on guide and reference implementation** for deploying a production-grade **multi-tier AWS infrastructure**.  
It covers the setup of a **VPC with structured subnets**, **public and private routing**, **load balancing**, **auto scaling**, and a **highly available RDS database** layer.  

The architecture is designed for **high availability, scalability, and security**, making it suitable for hosting modern web applications with frontend, backend, and database tiers.  

---

## 🚀 Features

- **VPC & Subnets**
  - Structured **Web, App, and DB tiers**
  - Multi-AZ deployment for redundancy

- **Routing & Networking**
  - Public subnets connected via **Internet Gateway (IGW)**
  - Private subnets with **NAT Gateway** for outbound traffic
  - Route tables and subnet associations

- **Security Architecture**
  - Fine-grained **Security Groups** for each tier
  - Isolated DB tier with restricted inbound rules

- **Load Balancing & Scaling**
  - **Application Load Balancers (ALBs)** with target groups
  - **Launch Templates** for consistent server provisioning
  - **Auto Scaling Groups (ASGs)** for frontend and backend tiers

- **Custom AMIs**
  - NGINX, Apache, PHP, and MySQL pre-baked into **custom AMIs**

- **Database Layer**
  - **Multi-AZ RDS** for durability and high availability
  - RDS Subnet Group configured for DB isolation

---

## 📂 Project Structure

```bash
├── vpc/                 # VPC, subnets, route tables, gateways
├── security/            # Security group definitions
├── loadbalancers/       # ALB & target group setup
├── autoscaling/         # Launch templates & ASG configs
├── database/            # RDS subnet groups & DB instances
├── ami/                 # Custom AMI build scripts/configs
├── README.md            # Project documentation

```

## 🛠️ Prerequisites

- **AWS Account** with sufficient permissions  
- [**AWS CLI**](https://docs.aws.amazon.com/cli/) installed & configured  
- [**Terraform**](https://developer.hashicorp.com/terraform/downloads) or **AWS CloudFormation**  
- **SSH key pair** configured in AWS  

---

# 🖥️ Local Setup

## 📋 Prerequisites
- Web server with PHP support (**XAMPP**, **WAMP**, **MAMP**, etc.)
- **MySQL** database

---

# Architecture


<img width="3519" height="2684" alt="aws-three-tier-architecture" src="https://github.com/user-attachments/assets/311fb582-506d-434d-bbd3-fc2a8eccc94e" />


---
## ⚙️ Steps

### 1. Create VPC
- Create subnets:
  - Web Public: **1a, 1b, 1c**
  - Web Private: **1a, 1b, 1c**
  - App Private: **1a, 1b, 1c**
  - DB Private: **1a, 1b, 1c**

### 2. Create Route Tables
- Web Public
- Web Private (**1a, 1b, 1c**)
- App Private (**1a, 1b, 1c**)
- DB Private (**1a, 1b, 1c**)

👉 **Associate route tables with subnets**

### 3. Create Internet Gateway (IGW)
- Attach IGW to VPC  

### 4. Create NAT Gateway (NATGW)
- Place it in Web Public Subnet  
- Add routes in route tables:
  - **Public → IGW**
  - **Private → NATGW**

### 5. Create Security Groups
- Frontend ALB  
- Frontend Servers  
- Backend ALB  
- Backend Servers  
- DB Private Servers  

### 6. Database Layer
- Create **Database Subnet Group**  
- Create **Database Server**  

### 7. Load Balancers
- Create **Frontend ALB**  
- Create **Frontend ALB Target Group**  
- Create **Backend ALB**  
- Create **Backend ALB Target Group**  

### 8. AMIs
- **Frontend Server AMI**
  - Install Nginx
  - Install Git
- **Backend Server AMI**
  - Install PHP, MySQL, Apache
  - Install Git
  - Run the database script

### 9. Launch Templates & Auto Scaling
- Create Launch Template for **Frontend Server**
- Create Launch Template for **Backend Server**
- Create Auto Scaling Group for **Frontend Server**
- Create Auto Scaling Group for **Backend Server**

---

## 💻 Development

### Frontend Development
The frontend is built with **HTML, CSS, and JavaScript**.  
It uses the **Fetch API** to communicate with the backend.  

To make changes:
1. Modify files in the `frontend/` directory  
2. Test changes locally  

### Backend Development
The backend is built with **PHP** and provides simple API endpoints for retrieving and saving messages.  

To make changes:
1. Modify PHP files in the `backend/api/` directory  
2. Test changes locally  

---

## 🔐 Security Considerations

⚠️ This is a **demo application** and lacks several production-grade security features:  
- Input validation & sanitization  
- Authentication & authorization  
- HTTPS encryption  
- Protection against SQL injection (PDO with prepared statements is used)  
- CORS configuration  

---

## 📜 License
This project is released under the [MIT License](LICENSE).  


