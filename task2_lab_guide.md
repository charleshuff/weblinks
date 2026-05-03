# Task 2 — AWS Lab Step-by-Step Command Guide

Follow these steps in order in the AWS lab environment. Take screenshots at each marked point.

---

## Prerequisites

- Log into the AWS Management Console via the lab environment link
- Make sure your system clock is visible or run `date` in the terminal before each screenshot

---

## Step 1: Launch the EC2 Instance (Covers A1 + A2 + A3)

### 1.1 Navigate to EC2
1. In the AWS Console, search for **EC2** and open the EC2 Dashboard.
2. Click **Launch Instance**.

### 1.2 Configure the Instance

| Setting | Value |
|---------|-------|
| **Name** | `CompanyY-WebServer` |
| **AMI** | Amazon Linux 2023 AMI (HVM), SSD Volume Type |
| **Instance type** | t3.medium (2 vCPU, 4 GiB) |
| **Key pair** | Create a new key pair or select existing |
| **Network settings** | |
| - VPC | Default VPC (or create a new one) |
| - Subnet | Select a public subnet |
| - Auto-assign public IP | **Enable** |
| - Security group | Create new: `CompanyY-SG` |
| -- Rule 1 | SSH (port 22) from **your IP only** |
| -- Rule 2 | HTTP (port 80) from 0.0.0.0/0 |
| -- Rule 3 | HTTPS (port 443) from 0.0.0.0/0 |
| **Storage** | |
| - Root volume | 20 GiB, gp3 |
| - **Encryption** | **Enabled** — select `(default) aws/ebs` KMS key |

### 📸 SCREENSHOT A1c
Before clicking "Launch instance", take a screenshot of this page showing:
- Amazon Linux 2023 AMI selected
- t3.medium instance type
- Make sure date/time is visible (system clock in taskbar)

### 📸 SCREENSHOT A2c
Expand the **Storage** section and take a screenshot showing:
- Volume size: 20 GiB
- Volume type: gp3
- Encrypted: Yes
- KMS key: aws/ebs

3. Click **Launch Instance**.
4. Wait for the instance to enter the "Running" state.

---

## Step 2: Allocate and Associate an Elastic IP (Covers A3)

### 2.1 In the AWS Console:
1. Go to **EC2 > Elastic IPs** (left sidebar under Network & Security).
2. Click **Allocate Elastic IP address**.
3. Click **Allocate**.
4. Select the new Elastic IP, click **Actions > Associate Elastic IP address**.
5. Choose the `CompanyY-WebServer` instance.
6. Click **Associate**.

### 📸 SCREENSHOT A3b
Take a screenshot showing:
- The Elastic IP details page (public IP, associated instance)
- Or the instance's **Networking** tab showing both the Elastic IP (public) and the private IP
- Security group rules visible
- Date/timestamp visible

---

## Step 3: Connect to the Instance via SSH

```bash
# Replace with your key file path and Elastic IP
chmod 400 your-key.pem
ssh -i your-key.pem ec2-user@<YOUR_ELASTIC_IP>
```

Once connected, run `date` to display the current timestamp in your terminal. Keep this visible in screenshots.

---

## Step 4: Package Repository Setup and System Services (Covers A4)

Run these commands in order:

```bash
# Show timestamp
date

# Verify repositories
sudo dnf repolist

# Update all packages
sudo dnf update -y

# Install Apache web server
sudo dnf install httpd -y

# Enable and start Apache
sudo systemctl enable --now httpd

# Verify all required services are running
sudo systemctl status httpd --no-pager
sudo systemctl status sshd --no-pager
sudo systemctl status chronyd --no-pager
sudo systemctl status amazon-ssm-agent --no-pager
```

### 📸 SCREENSHOT A4b
Take a screenshot (or multiple screenshots) showing:
- Output of `dnf repolist`
- Output of `systemctl status httpd` (showing active/running)
- Output of `systemctl status sshd` (showing active/running)
- Output of `systemctl status chronyd` (showing active/running)
- Date/timestamp visible (the `date` output at top, or system clock)

**Tip:** You can combine the status checks into one screenshot-friendly view:
```bash
echo "=== $(date) ===" && echo "" && \
echo "--- REPOS ---" && sudo dnf repolist && echo "" && \
echo "--- HTTPD ---" && sudo systemctl is-active httpd && \
echo "--- SSHD ---" && sudo systemctl is-active sshd && \
echo "--- CHRONYD ---" && sudo systemctl is-active chronyd && \
echo "--- SSM AGENT ---" && sudo systemctl is-active amazon-ssm-agent
```

---

## Step 5: System Logging and Monitoring (Covers A5)

### 5.1 Install and Configure CloudWatch Agent

```bash
# Install the CloudWatch Agent
sudo dnf install amazon-cloudwatch-agent -y

# Run the configuration wizard (interactive)
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard
```

**Wizard answers (suggested):**
- OS: Linux
- Are you using EC2 or on-premise: EC2
- Run as: root
- Do you want to turn on StatsD daemon: No
- Do you want to monitor metrics from CollectD: No
- Do you want to monitor CPU metrics per core: Yes
- Do you want to add EC2 dimensions: Yes
- Aggregation period: 60 seconds
- Default metrics config: Standard
- Are you satisfied: Yes
- Do you want to monitor log files: Yes
  - Log file path: `/var/log/messages`
  - Log group name: `CompanyY-system-logs`
  - Log stream name: `{instance_id}`
  - Add another log: Yes
  - Log file path: `/var/log/httpd/access_log`
  - Log group name: `CompanyY-httpd-access`
  - Log stream name: `{instance_id}`
  - Add another log: Yes
  - Log file path: `/var/log/httpd/error_log`
  - Log group name: `CompanyY-httpd-error`
  - Log stream name: `{instance_id}`
  - Add another log: No
- Store config in SSM Parameter Store: No

```bash
# Start the CloudWatch Agent
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
  -a fetch-config \
  -m ec2 \
  -s \
  -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json

# Verify it's running
sudo systemctl status amazon-cloudwatch-agent --no-pager
```

**Note:** The EC2 instance needs an IAM role with the `CloudWatchAgentServerPolicy` policy attached. If the lab doesn't have this pre-configured:
1. Go to **IAM > Roles > Create Role**
2. Select **EC2** as the trusted entity
3. Attach the `CloudWatchAgentServerPolicy` and `CloudWatchAgentAdminPolicy` managed policies
4. Name it `CompanyY-CloudWatch-Role`
5. Go to **EC2 > Instances > CompanyY-WebServer > Actions > Security > Modify IAM role**
6. Select the new role and save

### 5.2 Create a CloudWatch Alarm

1. Go to **CloudWatch > Alarms > Create Alarm**.
2. Click **Select metric > EC2 > Per-Instance Metrics**.
3. Search for your instance ID and select **CPUUtilization**.
4. Configure:
   - Statistic: Average
   - Period: 5 minutes
   - Threshold type: Static
   - Condition: Greater than 80
   - Datapoints to alarm: 3 out of 3
5. Notification: Create an SNS topic (e.g., `CompanyY-Alerts`) with your email.
6. Alarm name: `CompanyY-HighCPU`
7. Click **Create alarm**.

### 5.3 Create a CloudWatch Dashboard

1. Go to **CloudWatch > Dashboards > Create dashboard**.
2. Name: `CompanyY-WebServer-Dashboard`
3. Add widgets:
   - **Line graph** — CPUUtilization for the instance
   - **Line graph** — mem_used_percent (from CWAgent namespace, appears after agent runs for a few minutes)
   - **Number** — disk_used_percent
   - **Line graph** — NetworkIn and NetworkOut
4. Save the dashboard.

### 📸 SCREENSHOT A5b
Take a screenshot showing:
- The CloudWatch Dashboard with active metrics (CPU, memory, etc.)
- The CloudWatch Alarm configuration page or list showing the alarm
- Date/timestamp visible

**Tip:** Wait 5–10 minutes after starting the CloudWatch Agent before taking the screenshot, so metrics populate on the dashboard.

---

## Step 6: Network Analysis Tools (Covers A6 — for the video)

Install the tools you'll use in the recorded video:

```bash
# Install network analysis tools
sudo dnf install traceroute nmap iperf3 -y

# These are used during the video recording (see video_script.md)
```

---

## Step 7: Verify EBS Encryption (Additional A2c verification)

After the instance is running, you can verify encryption:

### In the Console:
1. Go to **EC2 > Volumes**.
2. Click on the volume attached to your instance.
3. Verify the **Encryption** field shows "Encrypted" and the KMS key ID is displayed.

### From the CLI (on the instance):
```bash
# If AWS CLI is available in the lab:
aws ec2 describe-volumes \
  --filters "Name=attachment.instance-id,Values=$(curl -s http://169.254.169.254/latest/meta-data/instance-id)" \
  --query "Volumes[*].{ID:VolumeId,Encrypted:Encrypted,KmsKeyId:KmsKeyId,Size:Size}" \
  --output table
```

---

## Summary of Screenshots Needed

| Screenshot | What to Capture | Section |
|-----------|-----------------|---------|
| A1c | EC2 Launch page: AMI (Amazon Linux 2023), instance type (t3.medium), with timestamp | A.1 |
| A2c | EBS volume encryption enabled, KMS key shown, with timestamp | A.2 |
| A3b | Elastic IP association, private IP, security group rules, with timestamp | A.3 |
| A4b | `dnf repolist` output + `systemctl status` for httpd, sshd, chronyd, with timestamp | A.4 |
| A5b | CloudWatch Dashboard with active metrics + alarm config, with timestamp | A.5 |

Collect all screenshots into a single PDF for submission alongside the main report.
