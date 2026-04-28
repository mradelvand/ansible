---
title: "Architectural Deep Dive: Elastic IPs"
date: 2025-10-25
categories: [AWS Architecture]
tags: [Elastic IP, Networking, Public IP]
---

#  Architectural Deep Dive:  Elastic IPs 

##  1. Elastic IP (EIP): Your Static Public Address

When you launch an **EC2 instance**, it typically gets:
- a **Public IP address** — routable over the internet  
- a **Private IP address** — routable only within your AWS internal network (e.g., your VPC)

###  The Core Issue
When you **stop and then start** an EC2 instance, its **Public IP address changes**.

###  The Solution
A **Static, Fixed Public IP address** that you can associate with your instance — this is an **Elastic IP (EIP)**.

---

###  Feature Comparison

| Feature | Dynamic Public IP | Elastic IP (EIP) |
|----------|-------------------|------------------|
| **Persistence** | Changes on stop/start | Fixed and permanent |
| **Ownership** | AWS-managed | You own it until you release it |
| **Charge** | No charge when in use | Free when associated and in use, charged when unassociated *(to prevent IPv4 hoarding)* |

 *You are **charged** when an EIP is **allocated but not associated** with any running instance.*

---

##  EIP: The Failover Mask

An **Elastic IP’s primary advantage** is its **stability**.

> An EIP allows you to quickly mask the failure of an instance or software by instantly re-associating the EIP from a failed instance to a healthy, running instance.

This rapid reassignment acts as a **basic manual failover** mechanism.

---

###  Important Consideration

This pattern, while effective, is **uncommon in production** due to a major limitation:

> You can only have **five Elastic IPs per AWS region per account**.  
> *(This is a soft limit and can be increased upon request.)*

---
##  Quick EIP Hands-On: Attaching an EIP to an EC2 Instance

Follow these simple steps to allocate and associate an Elastic IP:

1. Navigate to the **EC2 Dashboard** → **Elastic IPs** in the left navigation pane.  
2. Click **Allocate Elastic IP address** to get a new EIP.  
3. Select the newly created EIP → click **Actions** → **Associate Elastic IP address**.  
4. Under **Resource Type**, select **Instance**.  
5. Choose your **running EC2 instance** and one of its **Private IPs**.  
6. Click **Associate**.  
7. **Verify:** Stop and start the instance — the **Public IP remains unchanged**.  
8. **Crucial Step:** When finished, **release the Elastic IP**:  
   - Select it → **Actions** → **Release Elastic IP addresses**  
   - This prevents unwanted **billing charges**.

---

##  The Better Way

| The Better Way | Why It's Better |  Budget Evaluation |
|----------------|----------------|----------------------|
| **Use a Load Balancer (ELB)** | ELBs inherently provide a **static DNS name** and handle **failover**, **health checks**, and **traffic distribution** automatically. | **Cost-Effective:** A basic **Application Load Balancer (ALB)** is generally **cheaper** than managing dedicated EIPs and scales automatically. |
| **Use a Random Public IP and assign a DNS name (e.g., using Route 53)** | If direct access is needed, **Route 53** can map a hostname (like `api.example.com`) to the instance’s current public IP. If the IP changes, an **automation script** can update the DNS record. | **Free:** The public IP itself is free. Route 53 hosted zone and query costs are **minimal**. |

---

###  In Short

Use **EIPs only** for specific non-web-service needs, such as:
- A **NAT Gateway**  
- A **Jump Box** *(a secure instance used to access private network resources)*  

For **applications**, always use an **Elastic Load Balancer (ELB)**.

---
