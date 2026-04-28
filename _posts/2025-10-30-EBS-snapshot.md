---
title: "Architectural Deep Dive: Amazon EBS and snapshot"
date: 2025-10-30
categories: [AWS Storage]
tags: [EBS, Snapshots]
---

# Architectural Deep Dive: Amazon EBS and snapshot
When you launch an EC2 instance (a virtual server in the cloud), you need a place for your data to live. That place is the **Amazon Elastic Block Store (EBS)** Volume. 
Think of an EBS volume as a high-performance, network-attached hard drive specifically designed to work with your EC2 instance.

---

## The Foundation of EBS Volumes

EBS is a **network drive** you attach to your EC2 instance while it runs, using the network for communication (which introduces a tiny bit of latency, unlike a physical drive).

###  Data Persistence is Key

The primary purpose of an EBS volume is **data persistence**. Your data lives on the EBS volume, which is completely independent of the running life of your EC2 instance.

**Scenario:**
If your EC2 instance is terminated (shut down and deleted), the EBS volume can still be preserved.  
You can then launch a new EC2 instance and **mount the original EBS volume** to it — all your data, configurations, and files will be instantly available on the new server.

---

## The Availability Zone (AZ) Rule

EBS volumes are designed for **high availability within a specific physical location**.

### AZ Binding

When you create an EBS volume, it is **bound to a specific Availability Zone (AZ)** (e.g., `us-east-1a`).

**Restriction:**  
You cannot directly attach an EBS volume to an instance in a different AZ  
(e.g., an EBS in `us-east-1a` cannot attach to an instance in `us-east-1b`).

**Default Behavior:**  
A standard EBS volume can be mounted to only **one EC2 instance at a time**.  


###  Exception — EBS Multi-Attach

A special feature called **EBS Multi-Attach** allows a single volume to be attached to **up to 16 Nitro-based EC2 instances** within the same AZ.  
This feature is available for **Provisioned IOPS SSD (io1/io2)** volumes and requires a **clustered file system** to prevent data corruption.

**In practice:**  
Most workloads use the single-instance-per-volume model for simplicity and safety.

---

##  Provisioning for Performance and Cost

EBS requires you to **predefine your storage and performance** needs (provisioning).

- **Pre-Allocation:** You specify both the capacity (GB) and performance metric (**IOPS**).  
- **Performance Tiers:** By setting size and IOPS, you define your performance tier.  
- **Billing:** You’re billed for the capacity you provision — not what you actually use.  
- **Scalability:** You can increase capacity and performance (IOPS) anytime without downtime.  
- **Flexibility:** You can create **unattached volumes**, keeping them ready to attach on-demand.

---

## The Critical “Delete on Termination” Attribute

This simple checkbox has **massive implications** for data management.  
It determines what happens to an EBS volume when its associated EC2 instance is terminated.

### Root Volume (Default: Enabled)
By default, **“Delete on Termination”** is enabled for the **root volume** — the one containing the OS.  
When you terminate the instance, this volume is automatically deleted to save cost and avoid clutter.

### Data Volumes (Default: Disabled)
For **additional EBS volumes**, the attribute is **disabled** by default.  
This ensures your **application data or logs are preserved** even if the instance is deleted.

---

##  Preserving the Root Volume — A Debugging Scenario

The most common reason to **disable "Delete on Termination"** for the root volume is for **debugging, troubleshooting, or post-mortem analysis**.

**Example:**
You deploy a new application on EC2, but it keeps failing to start.  
After several troubleshooting attempts, you terminate the instance to rebuild it.  
If the root volume is deleted (default behavior), you lose all logs and configurations.

**The Fix:**
Before termination, disable **Delete on Termination** for the root volume.  
Then, when the instance is deleted, the root volume remains.  
You can **attach it to another healthy EC2 instance** (a temporary “forensic” instance) and mount it to review logs, retrieve files, or debug the issue.

---

##  EBS Snapshots — The Power of Point-in-Time Backups

An **EBS Snapshot** is an **incremental, point-in-time backup** of your EBS volume stored in **Amazon S3** (behind the scenes).

### Snapshot Best Practices

- You can take snapshots of a running instance, but for consistency, it’s recommended to **stop the instance or pause I/O operations first**.  
- This ensures the snapshot captures a **complete, application-consistent state**.

###  Using Snapshots Across AZs or Regions

Snapshots are the **official method** to transfer data across Availability Zones or Regions:

1. Take a snapshot of the source volume (e.g., `us-east-1a`).
2. Create a new EBS volume from that snapshot in the destination AZ (e.g., `us-east-1b`).
3. Attach the new volume to your target EC2 instance.

---

##  Advanced Snapshot Management Features

AWS offers advanced tools to optimize snapshot cost, performance, and recovery speed.

###  EBS Snapshot Archive
Move older snapshots to the **Archive Tier**, which is **up to 75% cheaper**.  
However, restoring from the archive takes **24–72 hours**.

**Use Case:** Long-term retention for compliance where immediate restore is not needed.

---

###  Recycle Bin for EBS Snapshots
Acts as a **safety net** for accidental deletions.  
Instead of permanent deletion, snapshots are moved to a **Recycle Bin** for a retention period (1 day to 1 year).  
You can restore them anytime before the retention expires.

---

###  Fast Snapshot Restore (FSR)

Normally, restoring a volume from a snapshot involves **lazy loading**, meaning data is loaded from S3 only when accessed — resulting in slower initial reads.

**FSR eliminates this delay.**

- **Goal:** Create fully-initialized EBS volumes instantly.  
- **How It Works:** FSR pre-initializes data for a snapshot in a specific AZ.  
- **Use Cases:**  
  - Disaster recovery  
  - Auto Scaling with large AMIs  
  - Rapid test environment creation  
- **Cost Consideration:** You pay for the time FSR is enabled per AZ, so use it strategically.

---

##  Summary

Amazon EBS is the **foundation of persistent storage** in AWS.  
It provides flexibility, reliability, and control over how your data is stored and protected.

**Key Takeaways:**

- EBS volumes persist independently of EC2 instances.  
- Bound to a single AZ by default.  
- Snapshots enable cross-AZ and cross-region transfers.  
- “Delete on Termination” determines lifecycle behavior.  
- Advanced snapshot features balance cost, safety, and speed.


