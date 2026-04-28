---
title: "AWS Storage Cheat Sheet — EBS vs. EFS"
date: 2025-11-10
categories: [AWS Storage]
tags: [EFS, EC2, Storage, EBS, AWS]
---

# AWS Storage Cheat Sheet: EBS vs. EFS

Choosing between **EBS** and **EFS** is one of the first storage decisions in AWS.  
This quick guide shows when to use a **dedicated disk** vs. a **shared file system** — fast.

---

##  Quick Comparison & Scenarios

| Service | Best For | Access Scope | Performance | Pricing | Notes |
|----------|-----------|---------------|--------------|----------|--------|
| **EBS (Elastic Block Store)** | High-performance, single-instance databases | Single AZ | High IOPS, low latency | Pay per GB/month | Attach to one EC2. Use gp3/io2 for better control. |
| **EFS (Elastic File System)** | Shared web files across a multi-AZ cluster (Linux) | Multi-AZ | Elastic throughput | Pay per usage or IA tier | Scales automatically; supports many EC2s. |
| **FSx for Windows** | Windows shared file systems | Multi-AZ | High IOPS + Windows ACLs | Pay per capacity | Ideal for Active Directory and SMB apps. |
| **Instance Store** | Temporary scratch space or caching | Local to instance | Ultra-fast (NVMe) | Included with instance | Data lost on stop/terminate. Great for caching. |

---

##  EBS Essentials

| Feature | Summary |
|----------|----------|
| **AZ-Bound** | Volumes stay within one AZ. To migrate: create a snapshot → restore in a new AZ. |
| **Performance Types** | - **gp3:** Cost-effective, flexible IOPS. <br> - **io1/io2:** High-performance, provisioned IOPS. <br> - **gp2:** Legacy type (IOPS tied to volume size). |
| **IOPS Behavior** | gp2 = size-dependent; gp3/io2 = size-independent. |
| **Backups** | Snapshots consume IOPS → run during off-peak hours. |
| **Termination Rules** | Root volume deletes on termination (default). Data volumes persist unless delete-on-terminate is enabled. |

 *Think of EBS as your EC2’s personal SSD.*

---

##  EFS Essentials

| Feature | Summary |
|----------|----------|
| **Shared Access** | Mount across multiple AZs and EC2s — perfect for scaling web apps. |
| **Linux Only** | POSIX-compliant (NFS). Use FSx for Windows for SMB workloads. |
| **Performance Modes** | - **General Purpose:** Low latency (default). <br> - **Max I/O:** Higher throughput for parallel workloads. |
| **Throughput Modes** | - **Elastic:** Auto scales with workload (recommended). <br> - **Bursting:** Based on storage size. <br> - **Provisioned:** Fixed throughput. |
| **Cost Optimization** | Lifecycle Policies → Move idle files to **EFS-IA** or **Archive**. |
| **Availability** | - **Regional:** Multi-AZ for production. <br> - **One Zone:** Lower cost for dev/test. |

 *Think of EFS as your shared drive — scalable and multi-instance.*

---

- **EBS →** Fast, single-instance storage (like an SSD).  
- **EFS →** Shared, elastic storage across AZs.  
- **FSx →** Windows-native shared filesystem.  
- **Instance Store →** Ephemeral, high-speed scratch space.  

 **Pro Tips:**
- Use **EBS gp3** for production — great balance of cost and performance.  
- Choose **EFS with Elastic Throughput** for unpredictable workloads.  
- Enable **EFS Lifecycle Management** for automatic cost control.  
- Avoid storing critical data in **Instance Store** (it’s temporary).  

---

 **Summary:**  
EBS gives you *speed and control*.  
EFS gives you *scale and collaboration*.  
Pick based on your workload’s **scope** (single vs. shared) and **persistence needs**.  
