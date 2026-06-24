# 🌏 AWS Multi-Region VPC Peering & Network Setup

> **Learning Journey** | AWS Cloud Networking | DevOps

![AWS](https://img.shields.io/badge/AWS-Cloud-orange?logo=amazon-aws&logoColor=white)
![VPC](https://img.shields.io/badge/VPC-Peering-blue)
![EC2](https://img.shields.io/badge/EC2-Instances-green)
![LAMP](https://img.shields.io/badge/LAMP-Stack-purple)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)

---

## 📌 Overview

This project demonstrates how to build a **secure, scalable multi-region AWS architecture** using Amazon VPC, subnets, EC2 instances, and VPC Peering across two AWS regions — **Mumbai (ap-south-1)** and **Tokyo (ap-northeast-1)**.

This was built as part of my AWS DevOps learning journey to understand real-world cloud networking, security, and connectivity design.

---

## 🏗️ What We Build

| Component | Details |
|-----------|---------|
| **VPC A (Mumbai)** | Private-only workload |
| **VPC B (Tokyo)** | Public + Private segmented architecture |
| **VPC Peering** | Cross-region private communication |
| **NAT Gateway** | Controlled outbound internet for private subnets |
| **LAMP Stack** | Apache + MySQL + PHP on public EC2 |

---

## 🗺️ Architecture Diagram

```
Mumbai Region                          Tokyo Region
┌─────────────────────┐               ┌──────────────────────────────────────┐
│  VPCA 192.168.0.0/16│               │  VPCB 10.0.0.0/16                    │
│                     │               │                                      │
│  ┌───────────────┐  │               │  ┌──────────────────────────────────┐ │
│  │subnet1A-Priv  │  │  VPC Peering  │  │ subnet1B-Public 10.0.1.0/24      │ │
│  │192.168.1.0/24 │◄─┼───────────────┼─►│ LAMP-EC2 + EC2A1 + NAT-GW1 + IGW│ │
│  │   EC2A1       │  │  (PING/SSH)   │  └──────────────────────────────────┘ │
│  └───────────────┘  │               │                                      │
└─────────────────────┘               │  ┌──────────────────────────────────┐ │
                                      │  │ subnet2B-Private 10.0.2.0/24     │ │
                                      │  │ EC2B2                            │ │
                                      │  └──────────────────────────────────┘ │
                                      │                                      │
                                      │  ┌──────────────────────────────────┐ │
                                      │  │ subnet3B-Private 10.0.3.0/24     │ │
                                      │  │ EC2B3 (NAT → Internet)           │ │
                                      │  └──────────────────────────────────┘ │
                                      └──────────────────────────────────────┘
```

> 📸 **Screenshot:** Add your architecture diagram from the AWS VPC dashboard here.

---

## 📋 Prerequisites

- AWS Account with access to **Mumbai** and **Tokyo** regions
- Basic understanding of VPC, subnets, and routing
- SSH client (Terminal / PuTTY)
- Key pairs created in both regions

---

## 🚀 Step-by-Step Setup

### Step 1: Create VPCs

#### 1.1 — VPC A (Mumbai Region)

1. Log in to AWS Console → Switch region to **Asia Pacific (Mumbai) `ap-south-1`**
2. Navigate to **VPC** → **Your VPCs** → **Create VPC**
3. Configure:
   - **Name:** `VPCA`
   - **IPv4 CIDR:** `192.168.0.0/16`
4. Create subnet:
   - **Name:** `subnet1A-Private`
   - **AZ:** `ap-south-1a`
   - **CIDR:** `192.168.1.0/24`
5. Create route table:
   - **Name:** `RTA`
   - Associate with `subnet1A-Private`

> 📸 **Screenshot:** VPC creation form showing VPCA with CIDR `192.168.0.0/16` | Subnet creation showing `subnet1A-Private`

---

#### 1.2 — VPC B (Tokyo Region)

1. Switch region to **Asia Pacific (Tokyo) `ap-northeast-1`**
2. Create VPC:
   - **Name:** `VPCB`
   - **IPv4 CIDR:** `10.0.0.0/16`
3. Create three subnets:

   | Subnet Name | CIDR | Type |
   |-------------|------|------|
   | `subnet1B-Public` | `10.0.1.0/24` | Public |
   | `subnet2B-Private` | `10.0.2.0/24` | Private |
   | `subnet3B-Private` | `10.0.3.0/24` | Private |

4. Create route tables `RTB1`, `RTB2`, `RTB3` and associate with respective subnets
5. Create **Internet Gateway** named `IGW1`
6. Attach `IGW1` to `VPCB`
7. Edit route table for `subnet1B-Public` → Add route:
   - **Destination:** `0.0.0.0/0`
   - **Target:** `IGW1`

> 📸 **Screenshot:** VPCB creation | All three subnets listed | IGW1 attached to VPCB | Route table for subnet1B-Public with IGW route

---

### Step 2: Launch EC2 Instances

#### Prerequisites

- Create **key pairs** in both Mumbai and Tokyo regions
- Create **Security Groups** in each region allowing:
  - SSH (port 22)
  - ICMP (All — for ping)
  - HTTP (port 80)

#### 2.1 — In VPCA (Mumbai)

| Instance Name | Subnet | Type |
|---------------|--------|------|
| `EC2A1` | `subnet1A-Private (192.168.1.0/24)` | Private |

> 📸 **Screenshot:** EC2 launch wizard showing `EC2A1` placed in `subnet1A-Private` with no public IP

---

#### 2.2 — In VPCB (Tokyo)

| Instance Name | Subnet | Notes |
|---------------|--------|-------|
| `EC2B1 (LAMP-EC2)` | `subnet1B-Public 10.0.1.0/24` | Ubuntu 26, **Auto-assign public IP: Enabled** |
| `EC2B2` | `subnet2B-Private 10.0.2.0/24` | Amazon Linux 2023 |
| `EC2B3` | `subnet3B-Private 10.0.3.0/24` | Amazon Linux 2023 |

> ⚠️ **Important:** Enable **Auto-assign Public IP** for `EC2B1` (LAMP server) only.

> 📸 **Screenshot:** EC2B1 network settings with public IP enabled | All three Tokyo instances in running state

---

### Step 3: Create NAT Gateway

1. Navigate to **VPC** → **NAT Gateways** → **Create NAT Gateway**
2. Configure:
   - **Name:** `NAT-GW1`
   - **Subnet:** `subnet1B-Public (10.0.1.0/24)`
   - **Connectivity type:** Public
   - **Elastic IP:** Allocate a new one
3. After `NAT-GW1` is active, edit route table **RTB3** (associated with `subnet3B-Private`):
   - **Destination:** `0.0.0.0/0`
   - **Target:** `NAT-GW1`

> ✅ This allows EC2B3 to reach the internet (outbound only) — inbound internet blocked.

> 📸 **Screenshot:** NAT Gateway creation with Elastic IP assigned | RTB3 route table showing NAT-GW1 as target

---

### Step 4: VPC Peering (Mumbai ↔ Tokyo)

#### 4.1 — Create Peering Connection

1. In **Mumbai region**, go to **VPC** → **Peering Connections** → **Create Peering Connection**
2. Configure:
   - **Local VPC (Requester):** `VPCA` (Mumbai)
   - **Account:** My account
   - **Region:** Another Region → `Asia Pacific (Tokyo) ap-northeast-1`
   - **VPC ID (Accepter):** ID of `VPCB`
3. Status will show **Pending Acceptance**

#### 4.2 — Accept the Peering Request

1. Switch to **Tokyo region**
2. Go to **VPC** → **Peering Connections**
3. Select the pending request → **Actions** → **Accept Request**

> 📸 **Screenshot:** Peering connection creation form (requester side) | Pending acceptance status | Accepting the request from Tokyo

---

#### 4.3 — Update Route Tables

**In Tokyo (RTB2 → subnet2B-Private):**

| Destination | Target |
|-------------|--------|
| `10.0.0.0/16` | local |
| `192.168.1.0/24` | Peering Connection (`pcx-xxxxx`) |

**In Mumbai (RTA → subnet1A-Private):**

| Destination | Target |
|-------------|--------|
| `192.168.0.0/16` | local |
| `10.0.2.0/24` | Peering Connection (`pcx-xxxxx`) |

> 📸 **Screenshot:** RTB2 edit routes showing peering connection as target | RTA edit routes showing peering connection as target

---

### Step 5: Test the Setup

#### 5.1 — Test VPC Peering (Cross-Region SSH + Ping)

```bash
# From your local machine, SSH into EC2B1 (public IP)
ssh -i key.pem ubuntu@<EC2B1-Public-IP>

# Copy Tokyo key to EC2B1
vi key.pem   # paste private key content
chmod 400 key.pem

# SSH from EC2B1 → EC2B2 (private subnet)
ssh -i key.pem ec2-user@<EC2B2-Private-IP>

# From EC2B2, ping EC2A1 in Mumbai (cross-region peering test)
ping <EC2A1-Private-IP>
# Expected: 64 bytes from 192.168.1.x: icmp_seq=1 ttl=127 time=127 ms ✅
```

> 📸 **Screenshot:** Successful SSH from EC2B1 → EC2B2 | Successful ping from EC2B2 → EC2A1 in Mumbai

---

#### 5.2 — Test NAT Gateway

```bash
# SSH from EC2B1 → EC2B3 (private subnet via NAT)
ssh -i key.pem ec2-user@<EC2B3-Private-IP>

# Test outbound internet via NAT Gateway
ping google.com
# Expected: replies from 142.250.x.x ✅
```

> 📸 **Screenshot:** Ping to google.com succeeding from EC2B3 (private subnet via NAT)

---

### Step 6: LAMP Stack Setup (EC2B1 — Public Subnet)

#### Install LAMP on Ubuntu

```bash
# SSH into LAMP-EC2 (EC2B1)
ssh -i key.pem ubuntu@<EC2B1-Public-IP>

# Update packages
sudo apt update && sudo apt upgrade -y

# Install Apache
sudo apt install apache2 -y
sudo systemctl enable apache2

# Install MySQL
sudo apt install mysql-server -y
sudo mysql_secure_installation

# Install PHP
sudo apt install php libapache2-mod-php php-mysql -y

# Restart Apache
sudo systemctl restart apache2
```

#### Create a Database Test Script

```bash
sudo nano /var/www/html/database_test.php
```

```php
<?php
$host = "localhost";
$user = "root";
$password = "your_password";

$conn = new mysqli($host, $user, $password);

if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}
echo "Database connection was successful";
?>
```

#### Verify in Browser

Visit: `http://<EC2B1-Public-IP>/database_test.php`

Expected output:
```
Database connection was successful
```

> 📸 **Screenshot:** Browser showing "Database connection was successful" at your EC2B1 public IP

---

## 📊 Network Summary

| Resource | Name | CIDR / Config |
|----------|------|---------------|
| VPC (Mumbai) | VPCA | 192.168.0.0/16 |
| VPC (Tokyo) | VPCB | 10.0.0.0/16 |
| Subnet (Mumbai) | subnet1A-Private | 192.168.1.0/24 |
| Subnet (Tokyo) | subnet1B-Public | 10.0.1.0/24 |
| Subnet (Tokyo) | subnet2B-Private | 10.0.2.0/24 |
| Subnet (Tokyo) | subnet3B-Private | 10.0.3.0/24 |
| Internet Gateway | IGW1 | Attached to VPCB |
| NAT Gateway | NAT-GW1 | In subnet1B-Public |
| VPC Peering | pcx-xxxxx | Mumbai ↔ Tokyo |

---

## 🔐 Security Group Rules

| Rule | Protocol | Port | Source/Destination |
|------|----------|------|--------------------|
| SSH | TCP | 22 | Your IP / 0.0.0.0/0 |
| HTTP | TCP | 80 | 0.0.0.0/0 |
| ICMP (Ping) | ICMP | All | VPC CIDRs |

---

## ✅ What I Learned

- Designing **multi-region AWS VPC architectures** from scratch
- Difference between **public** and **private** subnets and when to use each
- Setting up **cross-region VPC Peering** and updating route tables on both sides
- Using **NAT Gateway** to allow private instances to reach the internet (outbound only)
- Installing a **LAMP stack** on Ubuntu EC2 and verifying DB connectivity
- Understanding **security group** rules for SSH, ICMP, and HTTP

---

## 📁 Project Structure

```
aws-multi-region-vpc-peering/
├── README.md
└── screenshots/
    ├── 01-vpca-creation.png
    ├── 02-subnet1a-private.png
    ├── 03-vpcb-creation.png
    ├── 04-tokyo-subnets.png
    ├── 05-igw1-attached.png
    ├── 06-ec2-instances.png
    ├── 07-nat-gateway.png
    ├── 08-rtb3-nat-route.png
    ├── 09-peering-creation.png
    ├── 10-peering-accepted.png
    ├── 11-rtb2-peering-route.png
    ├── 12-rta-peering-route.png
    ├── 13-ping-success-cross-region.png
    ├── 14-nat-ping-google.png
    └── 15-lamp-db-success.png
```

---

## 🔗 References

- [AWS VPC Peering Documentation](https://docs.aws.amazon.com/vpc/latest/peering/what-is-vpc-peering.html)
- [AWS NAT Gateway Guide](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html)
- [LAMP Stack on Ubuntu](https://docs.vultr.com/install-linux-apache-mysql-and-php-lamp-on-ubuntu-20-04-lts)

---

## 👩‍💻 Author

**Sinsha C**  
AWS & DevOps Learner | Cloud Enthusiast

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-blue?logo=linkedin)](https://linkedin.com)
[![GitHub](https://img.shields.io/badge/GitHub-Follow-black?logo=github)](https://github.com)

---

> 💡 *Built this project as part of my AWS DevOps learning journey. Feel free to fork, star ⭐, and use it as a reference for your own cloud networking labs!*
