# AWS 3-Tier Architecture: Cost-Optimized Bootcamp Edition

This repository contains a modular **AWS CloudFormation** suite designed to deploy a scalable, secure, and highly available 3-tier application.

## 🎯 Project Overview

The goal of this project is to respect architectural norms (separating Web, App, and Data layers) while minimizing AWS costs.

### The "No-NAT" Workaround

In a standard production environment, private instances use a **NAT Gateway** to download updates. To save costs, this project uses a 5-step deployment flow:

1. We launch a temporary instance in a **Public Subnet**.
2. We install all dependencies (Node.js, Python, Java, etc.).
3. We "bake" that instance into a **Custom AMI**.
4. We use that AMI to launch our **Backend** in a **Private Subnet**.
5. The backend runs securely without needing a constant outbound internet connection.

---

## 🏗️ Deployment Sequence

Each template must be launched in order, as they use `Export` and `ImportValue` to share physical IDs (like VPC IDs and Subnet IDs).

### Step 1: Network Foundation (`vpc-network.yml`)

* **Purpose:** Creates the VPC, 2 Public Subnets, and 2 Private Subnets across two Availability Zones.
* **Key Feature:** Sets up the Internet Gateway and Public Route Tables.

### Step 2: Security Layer (`securitygroups.yml`)

* **Purpose:** Defines the "Firewalls" for the Load Balancer, Web Servers, and Database.
* **Rule:** Strictly follows the principle of least privilege (e.g., RDS only accepts traffic from the Backend SG).

### Step 3: Data Tier (`rds.yml`)

* **Purpose:** Deploys a Multi-AZ (optional) RDS MySQL instance.
* **Security:** Uses **AWS Secrets Manager** to handle credentials instead of hardcoded passwords.

### Step 4: The Build Layer (`ami.yml`)

* **Purpose:** Launches a temporary EC2 instance in the Public Subnet.
* **Action:** You must SSH into this instance, install your app dependencies, and create an Image (AMI) from the AWS Console once finished.

### Step 5: High Availability (`alb-tg-asg.yml`)

* **Purpose:** Deploys the Application Load Balancer (ALB) and the Auto Scaling Group (ASG).
* **Requirement:** This step requires the **AMI ID** generated in Step 4.

---

## 🛠 Prerequisites

* An AWS Account.
* AWS CLI configured (optional).
* A KeyPair created in your preferred region (e.g., `eu-north-1`).

---

## 🚀 How to Launch

1. **VPC:** Upload `vpc-network.yml` to CloudFormation. Name it `network-stack`.
2. **Security:** Upload `securitygroups.yml`. Name it `security-stack`.
3. **Database:** Upload `rds.yml`. Wait for the status `CREATE_COMPLETE`.
4. **AMI Build:** Upload `ami.yml`. Once the instance is up:
* Connect via SSH/Session Manager.
* Run your setup scripts.
* **Crucial:** Create an AMI from this instance and note the `ami-xxxxxxx` ID.


5. **App Tier:** Upload `alb-tg-asg.yml`. Provide the AMI ID from the previous step as a parameter.

---

## ⚠️ Security Note

While this architecture is cost-effective, in a **Production Enterprise** environment, it is highly recommended to use a **NAT Gateway** or **VPC Endpoints** to allow private instances to receive security patches without manual AMI rebuilding.

---

**Next Step:** Would you like me to help you write a small `UserData` script for your `ami.yml` to automate the dependency installation?