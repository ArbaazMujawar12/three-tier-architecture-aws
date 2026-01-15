# AWS 3-Tier Architecture Deployment Guide (2 Availability Zones)

## **1. Project Overview**

This document provides a complete step-by-step guide to deploy a highly available, scalable, and secure 3-tier architecture on AWS using two Availability Zones.

The architecture follows a standard enterprise design pattern:

- Presentation Tier (Frontend)
- Application Tier (Backend)
- Database Tier

All tiers are isolated using private subnets with controlled network access.

---

## **Pre-Deployment Requirements**

Before starting the deployment, ensure that all required files, scripts, and resources are available locally.

**Download Required Files**

All required application files, configuration files, and EC2 user data scripts can be downloaded from the following GitHub repository:

https://github.com/ArbaazMujawar12/three-tier-architecture-aws.git

Clone the repository using:

```bash
git clone https://github.com/ArbaazMujawar12/three-tier-architecture-aws.git
```

---

## **2. Architecture Diagram**

![System Architecture](/Screenshots/3-Tier-Archi.png)

```text
User → Frontend ALB → Frontend EC2 → Backend ALB → Backend EC2 → RDS (MySQL)
```

---

**Architecture Summary**

User → Internet-facing Frontend ALB → Frontend EC2 → Internal Backend ALB → Backend EC2 → RDS

**AWS Services Used**

- Amazon VPC
- EC2
- Application Load Balancer
- Auto Scaling Group
- Amazon RDS (MySQL)
- Internet Gateway
- NAT Gateway

---

## **3. Network Configuration**

### **Step 1: Create VPC**

Create a VPC with the following CIDR block:

```
10.0.0.0/16
```

This VPC will host all networking components.

**Screenshot:**
![VPC Created](/Screenshots/VPC.png)

---

### **Step 2: Create Subnets (8 Total)**

Create the following subnets across two Availability Zones:

**Public Web Subnets**

- Pub-web-sub-1a → 10.0.0.0/20
- Pub-web-sub-1b → 10.0.16.0/20

**Private Web Subnets (Frontend Tier**)

- Pvt-web-sub-1a → 10.0.32.0/20
- Pvt-web-sub-1b → 10.0.48.0/20

**Private App Subnets (Backend Tier)**

- Pvt-app-sub-1a → 10.0.64.0/20
- Pvt-app-sub-1b → 10.0.80.0/20

**Private DB Subnets (Database Tier)**

- Pvt-db-sub-1a → 10.0.96.0/20
- Pvt-db-sub-1b → 10.0.112.0/20

**Screenshot:**
![Subnets Created](/Screenshots/subnet.png)

---

### **Step 3: Create Route Tables (7 Total)**

Create the following route tables:

- **web-pub-RT**
- **web-pvt-RT-1a**
- **web-pvt-RT-1b**
- **app-pvt-RT-1a**
- **app-pvt-RT-1b**
- **db-pvt-RT-1a**
- **db-pvt-RT-1b**

**Screenshot:**
![Route Table Created](/Screenshots/RT.png)

---

### **Step 4: Associate Subnets with Route Tables**

The following table shows the association between route tables and their respective subnets across availability zones.

| Route Table Name | Associated Subnets             |
| ---------------- | ------------------------------ |
| web-pub-RT       | Pub-web-sub-1a, Pub-web-sub-1b |
| web-pvt-RT-1a    | Pvt-web-sub-1a                 |
| web-pvt-RT-1b    | Pvt-web-sub-1b                 |
| app-pvt-RT-1a    | Pvt-app-sub-1a                 |
| app-pvt-RT-1b    | Pvt-app-sub-1b                 |
| db-pvt-RT-1a     | Pvt-db-sub-1a                  |
| db-pvt-RT-1b     | Pvt-db-sub-1b                  |

---

### **Step 5: Create Internet Gateway and NAT Gateway**

1. Create an Internet Gateway and attach it to the VPC
2. Create a NAT Gateway in a public subnet

---

### **Step 6: Update Route Tables**

**Public Route Table:**

```
0.0.0.0/0 → Internet Gateway
```

**All Private Route Tables:**

```
0.0.0.0/0 → NAT Gateway
```

**Screenshot:**
![VPC Map](/Screenshots/vpcmap.png)

---

## **4. Security Configuration**

### **Step 7: Create Security Groups**

**frontend-alb-sg**

```
HTTP (80) from 0.0.0.0/0
```

**frontend-server-sg**

```
SSH (22) from your IP
HTTP (80) from frontend-alb-sg
```

**backend-alb-sg**

```
HTTP (80) from frontend-server-sg
```

**backend-server-sg**

```
SSH (22) from your IP
HTTP (80) from backend-alb-sg
```

**db-sg**

```
MySQL (3306) from backend-server-sg
```

**Screenshot:**
![Security Groups](/Screenshots/SG.png)

---

## **5. Database Layer**

### **Step 8: Create RDS Subnet Group**

1. Name: db-subnet-group

2. Subnets:
   - Pvt-db-sub-1a
   - Pvt-db-sub-1b

---

### **Step 9: Create RDS MySQL Database**

- Engine: MySQL
- Attach db-sg
- Use db-subnet-group

**Screenshot:**
![RDS instance details page](/Screenshots/RDS.png)

---

## **6. Load Balancers and Target Groups**

### **Step 10: Create Target Groups**

- frontend-alb-TG (Instance target)
- backend-alb-TG (Instance target)

---

### **Step 11: Create Application Load Balancers**

1. Frontend ALB

   - Internet-facing
   - Subnets: Public Web Subnets
   - Listener forwards to frontend-alb-TG

2. Backend ALB
   - Internal
   - Subnets: Private App Subnets
   - Listener forwards to backend-alb-TG

**Screenshot:**
![ALB Created](/Screenshots/ALB.png)

---

## **7. Compute Layer**

### **Step 12: Launch Temporary EC2 Instances**

- Launch one frontend EC2 instance
- Launch one backend EC2 instance
- Used only for configuration and AMI creation

---

### **Step 13: Server Configuration**

1. Frontend Server

   - Install NGINX
   - Install Git
   - Configure NGINX server block

2. Backend Server
   - Install LAMP stack
   - Install MariaDB 10.5
   - Enable Apache
   - Update /var/www permissions
   - Install Git

---

### **Step 14: Connect Backend to RDS**

- SSH into backend server
- Connect to RDS using MySQL client
- Insert provided database schema/data

---

### **Step 15: Create AMIs**

- Stop both EC2 instances
- Create AMI:
  - Frontend AMI
  - Backend AMI

**Screenshot:**
![AMI Created](/Screenshots/ami.png)

---

## **8. Auto Scaling Setup**

### **Step 16: Create Launch Templates**

1. Frontend Launch Template

   - Frontend AMI
   - User data script
   - Backend ALB DNS added in script

2. Backend Launch Template
   - Backend AMI
   - User data script
   - RDS endpoint, username, password configured

---

### **Step 17: Create Auto Scaling Groups**

1. Frontend ASG

   - Subnets: Pvt-web-sub-1a, Pvt-web-sub-1b
   - Attached to frontend-alb-TG

2. Backend ASG
   - Subnets: Pvt-app-sub-1a, Pvt-app-sub-1b
   - Attached to backend-alb-TG

---

## **9. Application Access**

### **Step 18: Access the Application**

1. Copy the Frontend ALB DNS name
2. Paste it into a web browser
3. Application should load successfully

**Screenshot:**
![App Running](/Screenshots/APP%201.png)

**Screenshot:**
![App Running](/Screenshots/APP%202.png)
