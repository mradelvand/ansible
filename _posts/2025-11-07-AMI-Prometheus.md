---
title: "AMI & Prometheus — Custom AMI with the Prometheus Node Exporter"
date: 2025-11-07
categories: [AWS DevOps]
tags: [AMI, EC2, Prometheus, Automation, Monitoring]
---

# The Real-World Use Case: Open-Source Observability


**Push (CloudWatch Agent) vs. Pull (Prometheus Node Exporter) Monitoring**

The CloudWatch Agent is installed directly on your EC2 instance (the node).
It actively collects metrics (like memory, disk space, and custom application metrics) and logs, and then sends (pushes) that data across the network to the central AWS CloudWatch service
> use case : AWS Lambda Functions
> short-lived, unpredictable, or difficult-to-reach jobs because the target doesn't wait around for the central server.
> Requires the server to be configured in advance to receive data from specific agents. Data discovery is harder.

Prometheus Node Exporter uses a pull model, meaning the monitored node never initiates the connection; the central Prometheus server scrapes (pulls) the data from the node. 


> use cases: EC2 Instances & Kubernetes Nodes
> The pull model is ideal for monitoring long-lived, stable infrastructure that you can reliably find on a network.
> The server maintains a list of targets (nodes) it needs to monitor and actively checks them. Data discovery is easier.

---

##  What’s an AMI? 

An **AMI** is a pre-packaged snapshot of a server (create EBS snapshots behind the scenes) — the OS, tools, and configs all bundled together.  
 
No more installing Prometheus, security agents, or dependencies each time.

---
## Why “Bake” AMIs?

- **Automation First:**  Your servers boot fully configured in seconds — not minutes.  

- **Security & Consistency:** AMIs ensure every server meets your company’s standards from the start.  

- **Disaster Ready:** You can **copy your AMI across Availability Zones or Regions** for simple High Availability (HA) and Disaster Recovery (DR).

---
##  Hands-On: Baking a Prometheus Base AMI

### Step 1: Launch & Configure the Base Instance

Start with **Amazon Linux 2023** and add the setup script below in the **User Data** field.

 **Before You Begin**

| Config Detail | Action | Verification |
|----------------|---------|---------------|
| **Security Group** | Open TCP port **9100** to your Prometheus network (or `0.0.0.0/0` for testing). | Check inbound rules in EC2 console. |
| **User Data Script** | Copy the script below. | Check EC2 **System Log** after launch to confirm execution. |

```bash
#!/bin/bash
# Update system and install tools
yum update -y
yum install -y wget

# Download and extract Node Exporter
VERSION="1.8.1"
wget https://github.com/prometheus/node_exporter/releases/download/v${VERSION}/node_exporter-${VERSION}.linux-amd64.tar.gz
tar xvfz node_exporter-${VERSION}.linux-amd64.tar.gz
mv node_exporter-${VERSION}.linux-amd64/node_exporter /usr/local/bin/

# Setup user and service
useradd -rs /bin/false node_exporter
chown node_exporter:node_exporter /usr/local/bin/node_exporter

cat << 'EOF' > /etc/systemd/system/node_exporter.service
[Unit]
Description=Prometheus Node Exporter
After=network.target
[Service]
User=node_exporter
ExecStart=/usr/local/bin/node_exporter
Restart=always
[Install]
WantedBy=multi-user.target
EOF

# Enable and start service
systemctl daemon-reload
systemctl enable node_exporter
systemctl start node_exporter
```

---

### Step 2: Verify and Create the AMI

Before freezing your setup into an image, make sure the Node Exporter is running correctly:

| Step | Command | What You Should See |
|------|----------|---------------------|
| **Check Service** | `sudo systemctl status node_exporter` | Should show **Active: running** |
| **Check Metrics** | `curl http://localhost:9100/metrics` | Metrics data displayed |
| **Stop Instance** | Stop it from EC2 console | Instance state → **stopped** |
| **Create AMI** | Name it `prometheus-base-ami` | Wait until AMI status → **available** |

---

##  Launching a Differentiated Server

The AMI gives us the base setup. Now we’ll launch a new instance using that AMI and add a **unique identity** with a custom hostname.

---

### Step 3: Launch with a Custom Hostname

Launch a new instance using the `prometheus-base-ami` and reuse the same key pair and security group.

| Detail | Action | Use Case |
|---------|---------|----------|
| **New User Data** | Use the script below to set a custom hostname. | This separates the base setup (AMI) from the instance’s role identity. |

```bash
# User Data Script (Setting Custom Hostname)
#!/bin/bash
# The Prometheus Node Exporter is already baked in and running!
# This script runs AFTER the AMI's original configuration.

NEW_HOSTNAME="prometheus-web-01.local"
hostnamectl set-hostname $NEW_HOSTNAME
systemctl restart systemd-hostnamed
```

---
### Step 4: Verification & Why It Matters

| Step | Action | Expected Result |
|------|---------|-----------------|
| **Check Hostname** | SSH in and run: `hostname` | Should show `prometheus-web-01.local` |
| **Check Agent** | Run: `sudo systemctl status node_exporter` | Should still show **Active: running** |

By doing this, you achieve:

- **Monitoring & Auditing:** The hostname becomes a clear, readable label for Prometheus metrics.  
- **Configuration Management:** Tools like Ansible or Chef can use this hostname to automatically identify the server’s role.

---

## Critical Security Note: Why You Can't SSH as Root

If you try to SSH as `root`, you’ll see this message:

> Please login as the user "ec2-user" rather than the user "root".

###  Why This Happens

This is standard AWS security practice.  
Direct root access is disabled to reduce the risk of security breaches.  
The correct default login for these AMIs is **`ec2-user`**.

###  How to Log In Correctly

Use your key pair with the `ec2-user` account:

```bash
# CORRECT (Must use ec2-user)
ssh -i /path/to/key.pem ec2-user@<EC2-Public-IP>
```

---

##  Summary: The Two-Layer Deployment

| Layer | Tool | Purpose | Benefit |
|--------|------|----------|----------|
| **Layer 1: Baking** | AMI (`prometheus-base-ami`) | Base configuration — install core software (Node Exporter). | **Speed & Consistency:** Instances boot up fully monitored in seconds. |
| **Layer 2: Launch** | User Data (Custom Hostname) | Apply instance-specific details like hostname. | **Manageability:** Easier auditing and automated role configuration. |

---

## Very Important Point : AMIs are build for a specific AWS region
You can not launch an EC2 instance using an AMI in another AWS region, but you can copy the AMI to the target AWS region and then use it to create an EC2 instance.

---
