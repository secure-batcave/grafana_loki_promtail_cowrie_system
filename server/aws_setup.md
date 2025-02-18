# AWS Setup Guide

## 1. Create AWS Accounts

As always with AWS, things can quickly be updated, so this guide may be outdated. This guide works as of January 2025.

Amazon offers a free tier for new accounts, which includes Amazon Lightsail. Lightsail allows you to run a free virtual machine (VM) for 3 months, this VM is a little more powerful than the default EC2 free tier. You are going to create at least 2 AWS accounts, one for Grafana and Loki, and 1+ honeypot accounts. The idea is to set up the AWS accounts once such that the next time you login to the AWS console is to delete the account, since you can just SSH into the VM.

### Requirements
- **Email Address**: Use a unique email for each account. Gmail users can leverage the "+" trick (e.g., `emailaddress+aws1@gmail.com`) to manage multiple accounts with one email. Emails sent to these addresses will still arrive in your primary inbox, and you can create filters to organize them.
- **Bank Card**: Required for verification.
- **Phone Number**: Used for SMS verification.

### Steps
1. Go to AWS website and click **Create an AWS Account**.
2. Enter your email, choose a password, and input other required details.
3. Verify your email and phone number.
4. Add your payment method.
5. Complete the sign-up process by choosing the **Basic support Plan** (free).
6. Repeat steps 1-5 for each additional account, changing the email while all else can remain the same.

---

## 2. Basic AWS Account Configuration

### Secure Your Account with MFA
1. Navigate to **Security Credentials** in the AWS Management Console (top right drop down menu).
2. Enable **Multi-Factor Authentication (MFA)**.
   - Use Google Authenticator or a similar OTP (One-Time Password) app.

### Set Up Billing Alerts and Budgets
1. Navigate to **Billing and Cost Management** in the AWS Management Console (top right drop down menu).
2. Create a **Budget** to monitor free tier usage (eg. Zero spend budget).
   - Specify the email address to send cost threshold notifications to.
3. Navigate to **Billing Preferences**, enable PDF invoices and enable AWS Free Tier alerts.
   - Specify the email address to send Free Tier usage alerts to.

### Reminder for Temporary Accounts
- If you’re using the account temporarily for the free tier:
  - Mark your calendar to delete the account before the free period ends (3 months for Lightsail, 1 year for EC2).

---

## 3. Launching a Lightsail VM, Configuring SSH, and Updating Firewall Rules

### Steps
1. **Create a Lightsail VM**
   - Login to your AWS account and navigate to the Lightsail console.
   - Click **Create Instance**.
   - Select Region and OS (eg. Ubuntu 24.04 LTS).

2. **Set Up SSH Access**
   - Click Create custom key, download the key and save it to your local machine.
   - Secure the key with proper permissions:
     ```bash
     chmod 400 path/to/private-key.pem
     ```
   - Select the most expensive free tier instance size.
   - Click Create instance. Wait for the instance to be ready.
   - Use the following command to connect to the VM:
     ```bash
     ssh -i path/to/private-key.pem ubuntu@<instance-ip>
     ```

3. **Change the Default SSH Port (Cowrie honeypot servers only)**
   - Edit the SSH configuration file:
     ```bash
     sudo vi /etc/ssh/sshd_config
     ```
   - Update the `Port` directive (e.g., `Port 11222`) and save the file.
   - Restart the SSH service:
     ```bash
     sudo systemctl restart ssh
     ```
   - Verify the service is running:
     ```bash
     sudo systemctl status ssh
     ```

4.1 **Update AWS Firewall Rules (Cowrie honeypot servers only)**
   - Go to your Lightsail instance’s **Networking** tab.
   - In the IPv4 Firewall section, add a new rule:
     - **Application**: Custom
     - **Protocol**: TCP
     - **Port**: The new SSH port (e.g., 11222).
   - This will allow you to SSH into the VM using the new port.

4.2 **Update AWS Firewall Rules (Loki server only)**
   - Go to your Lightsail instance’s **Networking** tab.
   - In the IPv4 Firewall section, add a new rule:
     - **Application**: Custom
     - **Protocol**: TCP
     - **Port**: 3100
     - **Restrict to IP address**: <Cowrie Honeypot IP(s)>
   - This allows ONLY the Cowrie honeypot servers to send logs to the Loki server.

5. **Reconnect Using the New Port (Cowrie honeypot servers only)**
   - Test the updated configuration:
     ```bash
     ssh -i path/to/private-key.pem -p 11222 ubuntu@<instance-ip>
     ```

### Additional Notes
- If you plan to monitor TELNET traffic, add another firewall rule for port `23` (Cowrie honeypot servers only).
- Keep the private SSH key safe to avoid needing AWS account access for future SSH sessions.
- The idea is to set up the AWS account once such that the next time you login is to delete the account, since you can just SSH into the VM.

---


