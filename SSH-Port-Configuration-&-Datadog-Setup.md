# 🔐 SSH Port Configuration & Datadog Log Monitoring on AWS EC2

## Project Overview

This project documents how I configured a custom SSH port on an AWS EC2 Ubuntu instance and set up Datadog for log monitoring. I also walked through a real-world lockout scenario and how I recovered access — an invaluable lesson in cloud security and server management.

---

## 🧠 What is a Port?

Think of your server like an office building. The building has many doors (ports), each numbered. When you connect to a server using SSH, you are knocking on a specific door — by default, **Port 22**.

```
Your Computer  ──────────────────►  Server (EC2)
                SSH Connection          🚪 Port 22 (default)
```

Hackers and automated bots constantly scan the internet knocking on Port 22 because they know it's the default SSH door. **Changing it to a custom port** makes your server harder to find and reduces brute-force attacks.

---

## 🛠️ Tools & Technologies Used

| Tool | Purpose |
|------|---------|
| AWS EC2 | Cloud virtual machine (Ubuntu) |
| MobaXterm | SSH client to connect to the server |
| OpenSSH | SSH server running on the VM |
| Datadog | Log monitoring and observability |
| nano | Terminal text editor |

---

## 📋 Prerequisites

- An AWS account with an EC2 instance running Ubuntu
- MobaXterm installed on your local machine
- SSH key pair (.pem file) for your EC2 instance
- Basic knowledge of Linux terminal commands

---

## 🔧 Step-by-Step: Changing the SSH Port

### Step 1 — Connect to Your EC2 Instance

Open MobaXterm and connect to your EC2 instance using your existing SSH key:

```bash
ssh -i "your-key.pem" ubuntu@your-ec2-public-ip
```

### Step 2 — Check the Current SSH Status

Before making changes, verify SSH is running:

```bash
sudo systemctl status ssh
```

Expected output:
```
● ssh.service - OpenBSD Secure Shell server
     Active: active (running)
```

### Step 3 — Edit the SSH Configuration File

Open the SSH config file using nano:

```bash
sudo nano /etc/ssh/sshd_config
```

Look for the line that says:
```
#Port 22
```

Uncomment it (remove the `#`) and change the port number:
```
Port 2002
```

> ⚠️ **Important:** Make sure there are no typos — e.g., `2002q` is invalid. Double-check before saving!

### Step 4 — Save and Exit nano

- Press `Ctrl + O` to write/save the file
- Press `Enter` to confirm the filename
- Press `Ctrl + X` to exit nano

### Step 5 — Restart SSH Service

Apply the changes by restarting SSH:

```bash
sudo systemctl restart ssh
```

### Step 6 — Verify SSH is Listening on the New Port

```bash
sudo ss -tlnp | grep sshd
```

Expected output:
```
LISTEN  0  128  0.0.0.0:2002  ...
LISTEN  0  128     [::]:2002  ...
```

This confirms SSH is now listening on your new port. ✅

---

## ☁️ Step-by-Step: Updating AWS Security Group

Your AWS Security Group acts like a firewall. You **must** open the new port or you will be locked out.

1. Go to **AWS Console → EC2 → Instances**
2. Click your instance → **Security** tab
3. Click on your **Security Group**
4. Click **Edit Inbound Rules**
5. Add a new rule:

| Type | Protocol | Port Range | Source |
|------|----------|------------|--------|
| Custom TCP | TCP | 2002 | 0.0.0.0/0 |

6. Click **Save rules**

> 💡 **Tip:** Keep port 22 open until you confirm the new port works!

---

## 💻 Step-by-Step: Updating MobaXterm Session

1. Right-click your saved session in MobaXterm
2. Select **Edit session**
3. Change the **Port** field from `22` to `2002`
4. Click **OK** and reconnect

---

## 🚨 Real-World Scenario: Getting Locked Out & Recovery

During this project, I accidentally got locked out of my server. Here's what happened and how I recovered:

**What went wrong:**
- Changed the port in sshd_config
- Did not update the AWS Security Group
- Both Port 22 and new port were inaccessible

**Error received in MobaXterm:**
```
Network error: Connection refused
Session stopped
```

**Recovery Steps:**

1. Went to **AWS Console → EC2 → Instances**
2. Clicked **Connect → EC2 Instance Connect** (browser-based terminal)
3. Once inside, fixed the sshd_config and restarted SSH
4. Updated the Security Group inbound rules
5. Reconnected successfully via MobaXterm

> 🔑 **Key Lesson:** Always update your Security Group BEFORE disconnecting after a port change. And always keep a backup access method like EC2 Instance Connect available.

---

## 📊 Setting Up Datadog for Log Monitoring

Datadog allows you to monitor your server logs, track SSH logins, and set up alerts for suspicious activity.

### Install Datadog Agent

Run the following command (replace `YOUR_API_KEY` with your actual Datadog API key):

```bash
DD_API_KEY=YOUR_API_KEY bash -c "$(curl -L https://install.datadoghq.com/scripts/install_script_agent7.sh)"
```

### Verify Datadog Agent is Running

```bash
sudo systemctl status datadog-agent
```

### Enable SSH Log Collection

Edit the Datadog SSH integration config:

```bash
sudo nano /etc/datadog-agent/conf.d/ssh_check.d/conf.yaml
```

### Restart Datadog Agent

```bash
sudo systemctl restart datadog-agent
```

You can now monitor SSH logins, port activity, and system logs from the **Datadog dashboard**.

---

## ✅ Summary

| Task | Status |
|------|--------|
| Changed SSH port from 22 to 2002 | ✅ Done |
| Updated AWS Security Group | ✅ Done |
| Updated MobaXterm session | ✅ Done |
| Recovered from lockout using EC2 Instance Connect | ✅ Done |
| Installed and configured Datadog | ✅ Done |

---

## 📚 Key Commands Reference

```bash
# Check SSH status
sudo systemctl status ssh

# Edit SSH config
sudo nano /etc/ssh/sshd_config

# Restart SSH
sudo systemctl restart ssh

# Check what port SSH is listening on
sudo ss -tlnp | grep sshd

# Check Datadog agent status
sudo systemctl status datadog-agent
```

---

## 🔗 Resources

- [AWS EC2 Documentation](https://docs.aws.amazon.com/ec2/)
- [OpenSSH Manual](https://www.openssh.com/manual.html)
- [Datadog Agent Installation](https://docs.datadoghq.com/agent/)

---

*Documentation by [Your Name] | February 2026*
