# Hands-on Practice: AWS Auto Scaling **Without** a Load Balancer

**Goal:**  Demonstrate how to configure Auto Scaling in AWS using EC2, AMI, Launch Templates and an Auto Scaling Group (ASG) without attaching any Load Balancer. This is a hands-on lab to learn instance automation and self-healing behavior.

---

## Prerequisites
- An AWS account with permissions to manage EC2, AMIs, Launch Templates, and Auto Scaling.
- Basic knowledge of SSH and Linux terminal commands.
- A locally stored key pair (PEM) for SSH access.
- AWS Console access. Region selection matters stay consistent.

---

## Architecture Summary
1. Launch and configure a single EC2 instance (Ubuntu) with NGINX.
2. Create an AMI from the configured instance.
3. Create a Launch Template that references the AMI.
4. Create an Auto Scaling Group (ASG) from the Launch Template (no Load Balancer).
5. Test self-healing by terminating instances and observing ASG recreate instances.

> Note: Without a Load Balancer, each instance must be reachable via public IP (or you must implement some DNS strategy). This setup is intended for learning and internal demos not recommended for production-facing apps.

---

## Detailed Step-by-Step

### 1) Launch base EC2 instance (Ubuntu)
1. Sign in to AWS Console → EC2 → Launch Instance.
2. Choose an OS (e.g., **Ubuntu Server 22.04 LTS**).
3. Instance type: `t2.micro` (or as required).
4. Create or select a **Key pair**. Download and store the `.pem` file securely.
5. Network settings:
   - Edit network settings → **Auto-assign Public IP** → Enable.
   - Create a new security group (e.g., `launch-wizard-2`).
6. Security Group Inbound rule:
   - Type: **HTTP**, Port: 80, Source: **0.0.0.0/0** (Anywhere) for this lab.
   - (Optional) SSH: Type **SSH**, Port 22, Source: your IP.
7. Launch the instance and note its **Public IP**.

---

### 2) Connect and configure NGINX
From your machine (PowerShell, terminal):

```bash
# Example using PowerShell on Windows
ssh -i .\path\to\public-server.pem ubuntu@<PUBLIC_IP>
```

Once connected:

```bash
sudo apt-get update
sudo apt-get install nginx -y
sudo service nginx start
sudo systemctl enable nginx.service
sudo service nginx status
```

Configure the webroot:

```bash
cd /var/www/html
ls
# Remove default NGINX page
sudo rm index.nginx-debian.html
# Create a simple index.html
sudo nano index.html
# Paste:
# <h1>This is Auto-Scaling Without Using Load Balancer</h1>
# Save and exit (Ctrl+S, Ctrl+X)
```

Open `http://<PUBLIC_IP>` in your browser to verify the page displays.

---

### 3) Create AMI from configured instance
1. In the EC2 console: Select the instance → **Actions → Image and templates → Create Image**.
2. Give the image a descriptive name (e.g., `asg-ubuntu-nginx-ami`).
3. Click **Create Image**.
4. Go to **AMIs** and wait until the AMI `State` becomes **Available**.
5. Once AMI is available, you can optionally terminate the original instance to save costs.

---

### 4) Create Launch Template
1. EC2 console → **Launch Templates** → **Create launch template**.
2. Enter name and description.
3. For **AMI**, choose the AMI you created earlier (`asg-ubuntu-nginx-ami`).
4. Choose Instance Type (e.g., `t2.micro`).
5. Choose Key Pair (same PEM) and the Security Group `launch-wizard-2`.
6. Configure other options as needed (user data can be added here for bootstrapping).
7. Create the Launch Template.

> Tip: You can add a **User Data** script to install or update software at boot time if you want the instances to reconfigure themselves automatically.

---

### 5) Create Auto Scaling Group (ASG) WITHOUT Load Balancer
1. EC2 console → **Auto Scaling Groups** → **Create Auto Scaling group**.
2. Select the Launch Template you created.
3. Name the ASG (e.g., `asg-no-lb-demo`).
4. Choose **Network**: select 2 or more subnets across different Availability Zones for resilience.
   - Note: Instances must be able to reach the Internet (public IP or NAT gateway) if needed.
5. **Skip** attaching a load balancer when prompted.
6. Under **Health checks and scaling**:
   - Health check type: **EC2** (default).
   - Health check grace period: **30 seconds** (adjust as per boot time).
7. Desired capacity / Minimum / Maximum:
   - For testing, set Min = 1, Desired = 1, Max = 2 (or choose values matching your test).
   - You can keep defaults, then review.
8. Review and create the ASG.

---

### 6) Validate Auto Scaling Behavior
- After creation, ASG will launch instances based on the desired capacity.
- Note the new instance(s) public IP(s). Access via `http://<PUBLIC_IP>` to verify the page served by NGINX.
- **Test self-healing:** Terminate one of the ASG-managed instances from the EC2 console.
  - ASG will detect the terminated instance and automatically launch a replacement within ~30 seconds (depending on health-check settings).
- Confirm the replacement instance displays the same index page.

---

## Optional: Add CloudWatch alarm-based scaling (not required for this lab)
- If you want dynamic scaling (scale out/in based on CPU), create a CloudWatch alarm and attach a scaling policy to the ASG.
- For this hands-on, we keep the ASG static and just validate the self-healing feature.

---

## Troubleshooting
- **No HTTP response:** Verify Security Group inbound rules allow HTTP (port 80). Confirm the instance has a public IP and NGINX is running.
- **SSH timeout:** Check SSH inbound rule and ensure your client IP allowed or use 0.0.0.0/0 temporarily for testing (less secure).
- **AMI missing packages:** If you need additional packages at boot, add them via **User Data** in the Launch Template or bake them into the AMI.
- **Replacement instance not appearing:** Check ASG activity history, CloudWatch metrics, and health check configuration. Ensure IAM permissions are intact for ASG to launch instances.

---

## Cost and Clean-up
- Terminate instances and delete the ASG when done.
- Deregister and delete AMIs and snapshots if you want to avoid ongoing storage charges.
- Delete unused Launch Templates.
- Monitor your AWS bill keep an eye on EC2 and EBS snapshot storage costs.

---

## Flowchart for this Activity 
<img width="988" height="808" alt="Auto-Scaling without Load Balancing" src="https://github.com/user-attachments/assets/dd8bb35e-0c5e-4709-9d62-e7449fb510fe" />

---

## Author
**Rushikesh Dase**  
Cloud Computing Enthusiast | AWS Learner  
📧 *daserushikesh@gmail.com*
### 🔗 Connect with Me
If you’d like to check out more of my cloud-related hands-on practices and projects, connect with me on (https://www.linkedin.com/in/rushi-dase).

---

**Notes:** This is a hands-on practice lab intended to show how ASG performs instance replacement without a Load Balancer. It is not recommended for production unless additional networking, DNS, and traffic routing considerations are implemented.
