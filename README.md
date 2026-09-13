# AWS Cloud Infrastructure Project — Web Hosting, Monitoring & Automated Backup

## Overview
End-to-end deployment of a web application on AWS, including web server setup, automated backup to S3, CloudWatch monitoring with email alerts, and incident troubleshooting — designed to simulate real-world cloud support engineering tasks.

## Architecture
- EC2 instance (Ubuntu 22.04 LTS, t2.micro) running Apache web server
- Default VPC with Security Group restricting SSH to a specific IP, HTTP/HTTPS open
- UFW firewall configured on the instance for an additional layer of security
- S3 bucket for storing backups
- Automated daily backups using a Bash script + cron job
- CloudWatch alarm monitoring CPU utilization, with SNS email notifications
- Manual incident simulation and root cause analysis (RCA)

## Steps Performed

### 1. EC2 Instance Setup
- Launched an Ubuntu 22.04 EC2 instance (t2.micro, free tier)
- Created a Security Group (`web-server-sg`) restricting SSH (port 22) to my IP only, and allowing HTTP (80) / HTTPS (443) from anywhere
- Connected via SSH using a downloaded key pair

### 2. Web Server Deployment
- Installed Apache2 on the instance
- **Troubleshooting encountered:** Apache failed to start with error `Address already in use: could not bind to address 0.0.0.0:80`. Diagnosed using `sudo ss -tulpn | grep :80`, which revealed nginx was already running on port 80. Resolved by stopping and disabling nginx, then successfully starting Apache.
- Configured UFW firewall (`allow OpenSSH`, `allow Apache Full`)
- Deployed a custom HTML landing page and verified it was publicly accessible via the instance's public IP

### 3. Automated Backup (S3 + Cron)
- Created a private S3 bucket (`support-project-backup-huzaif`) to store backups
- Installed AWS CLI v2 on the instance
- **Troubleshooting encountered:** Initial AWS CLI installation via `apt` failed (`Package awscli is not available`), and a subsequent installer run reported a corrupted/incomplete install. Resolved by removing all previous install artifacts (`/usr/local/aws-cli`, related symlinks) and performing a clean install from the official AWS CLI v2 installer.
- Configured AWS CLI credentials using `aws configure`
- Wrote a `backup.sh` script that compresses the web root directory and uploads it to S3, logging the result
- Verified the script manually (`./backup.sh`) — confirmed successful upload in the S3 console
- Automated the script using a cron job (`0 2 * * * /home/ubuntu/backup.sh`) to run daily at 2 AM
- Verified the cron entry using `crontab -l`

### 4. Monitoring & Alerting (CloudWatch + SNS)
- Created a CloudWatch alarm (`high-cpu-alarm`) on the `CPUUtilization` metric for the instance, triggering when CPU exceeds 70% for 5 minutes
- Created an SNS topic (`cpu-alerts`) with an email subscription
- Confirmed the subscription via the confirmation email
- Tested the alarm by generating CPU load using the `stress` tool (`stress --cpu 2 --timeout 120`)
- Verified the alarm transitioned to "In Alarm" state (CPU spiked to ~95%) and received the SNS email notification with full alarm details

### 5. Troubleshooting Simulation (RCA)
Simulated a service outage to practice incident response:

**Issue:** Website became inaccessible (connection refused)
**Root Cause:** Apache2 service was intentionally stopped (`sudo systemctl stop apache2`)
**Diagnosis:** Checked service status (`systemctl status apache2`) and reviewed error logs (`tail /var/log/apache2/error.log`)
**Resolution:** Restarted the service (`sudo systemctl restart apache2`)
**Verification:** Confirmed the service was active and the website was reachable again via `curl http://<public-ip>` and browser
**Prevention:** Consider configuring `Restart=on-failure` in the systemd unit file for automatic recovery

## Tools Used
AWS (EC2, S3, IAM, CloudWatch, SNS), Ubuntu Linux, Apache2, UFW, AWS CLI v2, Bash Scripting, Cron

## Screenshots
See the `screenshots/` folder for:
- Website running in browser
- CloudWatch alarm graph showing CPU spike
- SNS email alert notification
- S3 backup upload confirmation
- backup.sh script and crontab configuration
- Terminal log of the troubleshooting simulation (stop → diagnose → restart → verify)

## Key Learnings
- Diagnosing port conflicts between services (nginx vs Apache) using `ss`
- Recovering from a corrupted CLI installation by fully cleaning and reinstalling
- Writing and scheduling a backup automation script with cron
- Setting up metric-based alerting and verifying it end-to-end with a real load test
- Following a structured incident response process: detect → diagnose → resolve → verify → document
