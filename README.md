AWS Cloud Infrastructure - Phase 1: Custom VPC & Web Server
📌 Project Overview
This project demonstrates the deployment of a secure, scalable cloud environment on AWS. Moving away from the default AWS setup, I designed and implemented a Custom Virtual Private Cloud (VPC) to host a Linux-based web server.

🏗 Architecture Components
VPC: Custom 10.0.0.0/16 network.

Subnet: Public Subnet (10.0.1.0/24) located in a specific Availability Zone.

Internet Gateway (IGW): Attached to the VPC to enable communication with the internet.

Route Table: Custom routing rules directing outbound traffic (0.0.0.0/0) through the IGW.

Compute: EC2 Instance (Ubuntu 24.04 LTS, t2.micro) configured with a Public IP.

🚀 Implementation Steps
Step 1: Networking (The Foundation)
Create VPC: * Navigate to VPC Dashboard > Your VPCs > Create VPC.

Choose VPC only.

Name tag: MyProject-VPC

IPv4 CIDR: 10.0.0.0/16

Create Subnet:

Go to Subnets > Create Subnet.

Select your MyProject-VPC.

Subnet name: Public-Subnet-01

IPv4 CIDR block: 10.0.1.0/24

Internet Gateway (IGW):

Go to Internet Gateways > Create Internet Gateway.

Name it MyProject-IGW.

Once created, click Actions > Attach to VPC and select your VPC.

Configure Routing:

Go to Route Tables. Find the one associated with your VPC (it’s created by default).

Edit Routes > Add Route:

Destination: 0.0.0.0/0

Target: Internet Gateway (select MyProject-IGW).

Subnet Associations: Click "Edit subnet associations" and check the box for Public-Subnet-01.

Step 2: Compute (The Server)
Launch Instance:

Go to EC2 Dashboard > Launch Instance.

AMI: Ubuntu Server 24.04 LTS (HVM).

Instance Type: t2.micro (Free Tier).

Network Settings: * VPC: Select MyProject-VPC.

Subnet: Public-Subnet-01.

Auto-assign public IP: Enable (Crucial: If you miss this, you can't SSH via the internet).

Security Group:

Create a new group: Web-Server-SG.

Inbound Rule 1: Type: SSH | Port: 22 | Source: My IP (Best practice) or 0.0.0.0/0.

Inbound Rule 2: Type: HTTP | Port: 80 | Source: 0.0.0.0/0.

Step 3: Configuration (The Software)
Connect: Open your terminal and run:
ssh -i "your-key.pem" ubuntu@<your-public-ip>

Installation Script:
Run these commands one by one:

Bash
# Update the package index
sudo apt update

# Install Nginx
sudo apt install nginx -y

# Ensure Nginx starts on boot
sudo systemctl enable nginx
sudo systemctl start nginx
Verification:

Copy your Public IPv4 address from the EC2 console.

Paste it into your browser.

Success: You should see the "Welcome to nginx!" page.

create an IAM role ec2 can access s3 ready only role , then go ec2 and modify the IAM role in the instance.

Phase 3: S3 Integration & Data Transfer
✅ STEP 1 – Create the S3 Bucket
S3 Dashboard: Click Create bucket.

Naming (Crucial): S3 names must be globally unique.

Suggested: megachandran-project-2026-v1

If it says "Bucket name already exists," just add a few random numbers at the end.

Region: Choose the same region where your EC2 is running (e.g., us-east-1). This keeps data transfer free and fast.

Settings: Keep "Block all public access" ON. Since we are using an IAM Role, the bucket should stay private from the rest of the world.

✅ STEP 2 – Create & Upload your Test File
On your laptop, create a file named secret-data.txt.

Type inside it: "Phase 3 Success: Verified access via IAM Role."

In the S3 Console, click your bucket > Upload > Add files > Select secret-data.txt.

✅ STEP 3 – Access from Ubuntu (The Proof)
Re-open your PuTTY session. We are going to use the AWS CLI you installed in Phase 2.

1. See the Bucket:

Bash
aws s3 ls if it not works install aws cli (must)
 sudo apt update
sudo apt install unzip -y
 # Download the installer
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"

# Unzip the package
unzip awscliv2.zip

# Run the official install script
sudo ./aws/install
now run : aws s3 ls
You should see your bucket name in the list.

2. Look inside the Bucket:

Bash
aws s3 ls s3://your-bucket-name
You should see secret-data.txt listed.
3. Download the file to your EC2:
The cp command works like the Linux "copy" command: aws s3 cp <source> <destination>. The . at the end means "download to the folder I am currently in."

Bash
aws s3 cp s3://your-bucket-name/secret-data.txt .
4. Read the file:

Bash
cat secret-data.txt
Bash
sudo cp secret-data.txt /var/www/html/index.html
Now, go to your browser and refresh your Public IP. Your website will now display the "secret" text you downloaded from the S3 bucket.

Phase 4: Monitoring & Alerting with CloudWatch
🔵 STEP 1: The "Why" (Interview Context)
If an interviewer asks, "How do you know if your server is crashing?"
Your Answer: "I use Amazon CloudWatch. It collects metrics like CPU and RAM. I specifically set up a CloudWatch Alarm to trigger if CPU utilization exceeds 70%, ensuring I can scale up or troubleshoot before the users experience downtime."

🔵 STEP 2: Exploring the Monitoring Tab
Go to the EC2 Console > Instances.

Select your running Ubuntu server.

Look at the bottom pane and click the Monitoring tab.

Notice the CPU Utilization graph. It’s likely very low (1–2%) right now.

Note: By default, AWS checks these metrics every 5 minutes. This is called "Basic Monitoring" and it's free.

🔵 STEP 3: Creating the High-CPU Alarm
Search for CloudWatch in the top AWS search bar.

On the left, go to Alarms > All alarms > Create alarm.

Select Metric: Click Select metric > EC2 > Per-Instance Metrics. Find your Instance ID and check the box for CPUUtilization. Click Select metric.

Conditions:

Threshold type: Static.

Whenever CPUUtilization is... Greater than 70.

Datapoints to alarm: 1 out of 1 (this means if it stays high for one 5-minute period, it triggers).

Actions (Notification): * Select In alarm.

Select Create new topic.

Topic name: HighCPUAlert.

Email: Enter your actual email address.

Click Create topic. (Important: Check your email inbox and click "Confirm Subscription" or the alarm won't be able to email you).

Name: EC2_High_CPU_Alert_Project.

Review and Create.

🔵 STEP 4: The Stress Test (The "Show-Off" Step)
To prove the alarm works, we will force the CPU to work hard. Go back to your PuTTY terminal and run:

Bash
# Install the stress tool
sudo apt update && sudo apt install stress -y

# Run the stress test for 10 minutes (600 seconds)
# This uses 1 CPU core at 100% capacity
stress --cpu 1 --timeout 600
What will happen?

In 5-10 minutes, look at your Monitoring tab in EC2. You will see the CPU graph spike to nearly 100%.

Go to the CloudWatch Alarms page. The state will change from OK (green) to In alarm (red).

If you confirmed your email, you will receive an automated alert from AWS!

🎯 Final Result of Phase 4
You have now proven that your infrastructure is Self-Aware.

Phase 1-3: You built the body (Server/Network/Storage).

Phase 4: You gave it a nervous system (Monitoring/Alerts).
