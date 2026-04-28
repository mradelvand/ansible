---
title: "Understanding AWS IAM — A Practical Guide"
date: 2025-10-26
categories: [AWS Security]
tags: [IAM, Security, Access Control]
---

#  Understanding AWS IAM — A Practical Guide

In the **IAM (Identity and Access Management)** world, there are **two core concepts** every AWS learner must understand:


<img width="967" height="491" alt="IAM user vs role excalidraw(1)" src="https://github.com/user-attachments/assets/9d85d81a-11c4-42d2-80b3-e1e209177791" />

## IAM Users vs  IAM Roles

| Concept | Description | Typical Use Case | Credentials Type |
|----------|--------------|------------------|------------------|
| **IAM User** | A **human** or **application** that requires *permanent credentials*. | Developers, admins, or apps needing console or CLI access. | Username/password or permanent access keys. |
| **IAM Role** | Used by **AWS services** (EC2, Lambda, CloudFormation) or **external accounts** to get *temporary credentials*. | Lambda accessing S3 or CloudWatch; EC2 accessing DynamoDB. | Temporary, time-limited access keys. |

---

##  IAM Permissions — Users, Groups, and Policies

> **Why create users and groups?**  
> To manage access efficiently through **policies** — JSON documents that define allowed or denied actions.

###  Concept Summary

- **Users** → Represent individual identities.  
- **Groups** → Organize users with similar permissions.  
- **Policies** → Define *what actions* can be performed *on which resources*.


<img width="997" height="608" alt="IAM Policy structure excalidraw" src="https://github.com/user-attachments/assets/fd77fa46-b18d-437f-911a-b85297ec2da2" />

---

###  Quick Hands-On: Create a User and Group

1. **Create IAM User** → e.g., `dev-user`  
2. **Create IAM Group** → e.g., `AdminGroup`  
3. **Attach Policy** → `AdministratorAccess`  
4. **Add User to Group`**  
5. ✅ `dev-user` inherits permissions from `AdminGroup`

---

## 🛡️ IAM Best Practices

| Best Practice | Why It Matters |
|----------------|----------------|
| 🚫 Don’t use the **root account** | Only for initial setup. |
|  One physical person = **One IAM User** | Avoid shared credentials. |
|  Manage access with **Groups** | Easier auditing and permission updates. |
|  Use **inline policies** only for exceptions | Keeps permissions centralized. |
|  Enforce **strong passwords** | Example: 12+ characters, uppercase, numbers, symbols. |
|  Use **MFA (Multi-Factor Authentication)** | Adds an extra security layer. |
|  Use **Access Keys** for CLI/SDK only | Never share access keys publicly. |

---

###  Quick Hands-On: Create a Strong Password Policy

**Steps:**

1. Go to **IAM → Account Settings → Password Policy**
2. Configure:
   - Min length: **12**
   - Require **uppercase**, **numbers**, and **symbols**
   - Expire after **90 days**
   - Prevent reuse of last **3 passwords**

 **Result:** A secure and consistent password policy.

---

##  Why Group Policies Are Better than Inline Policies

Imagine 10 developers each with their own admin-level inline policies.  
If one leaves, you'd need to remove or edit every individual policy.  

> **Better Approach:**  
> Use **Groups** with shared **Group Policies** — update once, and all users inherit the change.

**Security Benefit:**  
Easier to audit, update, and enforce least-privilege principles.

---

##  IAM Roles — Explained

IAM Roles are **identities with permission policies** but **no permanent credentials**.  
They are **assumed temporarily** by trusted AWS services or external users.

###  Example: Lambda Function Role

A **Lambda function** that needs to:
- Read messages from **SQS**
- Write logs to **CloudWatch**

 Attach a Role with:

```json
{
  "Effect": "Allow",
  "Action": ["sqs:ReceiveMessage", "logs:CreateLogStream", "logs:PutLogEvents"],
  "Resource": "*"
}
```


##  Auditing IAM Permissions

| Tool | Description | Description |
|------|-------------|-------------|
| Credentials Report	| Lists all users, password/MFA status, and key age. | Identify inactive or insecure accounts. |
| Access Advisor	| Shows last-used permissions for each role or user.	| Remove unused or stale permissions. |
| Service Last Accessed Data	| Tracks when each service was last accessed. |	Clean up old roles or policies. |

**Example:**
If a user hasn’t accessed EC2 in 90 days, remove EC2 permissions from their policy.

---

###  Quick Hands-On: Create a Strong Password Policy

**Steps:**

1. Go to **IAM → Policies → Create Policy**
2. Choose:
   - **Service**: IAM
   - **Actions**: ListUsers, GetUser
   - 
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": ["iam:ListUsers", "iam:GetUser"],
    "Resource": "*"
  }]
}
```

---

##  Quiz AWS IAM — Test Your Knowledge

### Question 1: IAM Role Definition

A company has an application running on an EC2 instance that needs to read objects from an S3 bucket.
What is the **most secure** way to grant access?

- [ ] A. A permanent identity for human users.
- [ ] B. A password management feature.
- [ ] C. A temporary identity assumed by AWS services or external accounts.
- [ ] D. A static policy document for users.

<details>
<summary>💡 Show Answer & Explanation</summary>

✅ Correct Answer: **C**  
IAM Roles provide **temporary credentials** that are assumed by trusted entities like EC2, Lambda, or external AWS accounts.
</details>


### Question 2: MFA Enforcement

You must enforce MFA for sensitive actions (e.g., deleting resources).
Which method achieves this?
- [ ] A. Enable MFA in the password policy.
- [ ] B. Use a policy condition requiring MFA authentication.
- [ ] C. Create a new root user with MFA.
- [ ] D. Use Access Advisor to enforce MFA.
<details>
<summary>💡 Show Answer & Explanation</summary>

✅ Correct Answer: **B**
Use IAM condition key:
```json
"Condition": { "Bool": { "aws:MultiFactorAuthPresent": "true" } }
```
This enforces MFA before executing certain actions.
</details>

### Question 3: Policy Evaluation Logic

A user belongs to two groups — one allows s3:GetObject, another explicitly denies it.
What happens?

- [ ] A. The user is allowed access.
- [ ] B. The deny is ignored.
- [ ] C. The user is denied access.
- [ ] D. Depends on the order of policies.

<details> 
<summary>💡 Show Answer & Explanation</summary>

✅ Correct Answer: **C**
In IAM policy evaluation, explicit denies always override any allows.

</details>
