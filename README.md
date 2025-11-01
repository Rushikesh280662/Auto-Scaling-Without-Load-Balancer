# Hands-on Practice of Auto Scaling Without a Load Balancer

## Overview
Demonstrates how to configure **Auto Scaling** in AWS **without using a Load Balancer**.  
The setup ensures high availability by automatically creating new EC2 instances when one fails, using a custom Amazon Machine Image (AMI) and an Auto Scaling Group (ASG).

---

## Steps Performed

### 1. Launch EC2 Instance
- OS: Ubuntu  
- Created or selected a key pair for SSH access.  
- Enabled **Auto-assign Public IP**.  
- Created a new Security Group named `launch-wizard-2`.  
- Edited **Inbound Rules**:  
  - Type: HTTP  
  - Source: Anywhere (IPv4)  

---

### 2. Configure EC2 and Deploy Nginx
- Connected via PowerShell:  
  ```bash
  ssh -i <path_to_keypair> ubuntu@<public_ip>
  ```
- Updated packages and installed Nginx:  
  ```bash
  sudo apt-get update
  sudo apt-get install nginx -y
  sudo service nginx start
  sudo systemctl enable nginx.service
  ```
- Replaced the default web file:  
  ```bash
  cd /var/www/html
  sudo rm index.nginx-debian.html
  sudo nano index.html
  ```
  Added:
  ```html
  <h1>This is Auto-Scaling Without Using Load Balancer</h1>
  ```
- Verified the web page by visiting `http://<public_ip>` in a browser.

---

### 3. Create an AMI
- Selected the EC2 instance → **Actions → Image and Templates → Create Image**.  
- Named and created the AMI.  
- Waited until status became **Available**, then terminated the original instance.

---

### 4. Create Launch Template
- Created a new **Launch Template**.  
- Go to Application and OS Images (Amazon Machine Image) and Select My AMIs
- Instance Type: `t2.micro` (or chosen type).  
- Selected the same Key Pair and Security Group (`launch-wizard-2`).

---

### 5. Configure Auto Scaling Group (ASG)
- Created an **Auto Scaling Group** linked to the Launch Template.  
- Selected 2 or more **Subnets** (preferably across different Availability Zones).  
- Skipped the Load Balancer configuration.  
- Updated **Health Check Grace Period** from `300s` to `30s`.  
- Accepted default scaling settings and launched the ASG.

---

### 6. Testing Auto Scaling
- Verified that the ASG automatically launched an instance.  
- Terminated the instance manually and confirmed a new one was created automatically within ~30 seconds.

---

## Key Takeaways
- Demonstrated EC2 setup and configuration.  
- Created and used a custom AMI for scaling.  
- Implemented Auto Scaling functionality **without a Load Balancer**.  
- Understood fault tolerance and automated recovery in AWS.

---

## Tools & Technologies
- **AWS EC2**
- **Amazon Machine Image (AMI)**
- **Launch Template**
- **Auto Scaling Group (ASG)**
- **Nginx (Ubuntu Server)**
- **PowerShell (SSH Access)**

---

                 **To understand it better I generate The Flowchart**
<img width="988" height="808" alt="Auto-Scaling without Load Balancing" src="https://github.com/user-attachments/assets/db4b77a8-9215-44d8-a7b1-18a2ffae764a" />

---

## Author
**Rushikesh Dase**  
Cloud Computing Enthusiast | AWS Learner  
📧 *daserushikesh@gmail.com*
### 🔗 Connect with Me
If you’d like to check out more of my cloud-related hands-on practices and projects, connect with me on (https://www.linkedin.com/in/rushi-dase).

