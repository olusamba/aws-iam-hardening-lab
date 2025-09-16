# aws-iam-hardening-lab
AWS IAM Hardening Lab — JIT Admin (MFA), ABAC, PoLP, S3 hardening, CloudTrail &amp; EventBridge alerts.

# 🔐 AWS IAM Hardening Lab

## 📖 Introduction
This project is a hands-on security lab focused on **Identity and Access Management (IAM)** in AWS.  
The goal is to design, implement, and document security controls that follow **real-world best practices** while remaining easy to reproduce in a learning or demo environment.

By building this lab, I wanted to strengthen both my **technical skills** (writing and applying IAM policies, monitoring with CloudTrail, enforcing MFA) and my **consulting mindset** (explaining *why* these controls matter to non-technical stakeholders).

---

## 🎯 Objectives
- Apply the **Principle of Least Privilege (PoLP)** in IAM.  
- Explore **ABAC (Attribute-Based Access Control)** for scalable permissions.  
- Enforce strong security for **S3 buckets** (blocking public access, HTTPS-only, region restrictions).  
- Implement **Just-in-Time (JIT) admin access** with MFA.  
- Set up **CloudTrail and EventBridge** for monitoring and alerting.  
- Run an **incident response drill** simulating exposed access keys.  

---

## 📦 Deliverables
This repository contains:
- ✅ JSON IAM policies (`/policies/`)  
- ✅ EventBridge and CloudTrail configurations (`/eventbridge/`, `/cloudtrail/`)  
- ✅ Security hardening examples for S3 (`/policies/s3-*`)  
- ✅ Incident response reports (`/reports/`)  
- ✅ Screenshots of test cases (`/screenshots/`)  

---

## 🧩 Architecture and Security Design

### 1. Just-in-Time Admin with MFA
- **Problem**: Many companies keep permanent admin accounts. If an attacker steals one, they have full control 24/7.  
- **Solution**: Instead of standing admins, we create a role `AdminJIT` (Just-in-Time). A user must switch into this role, and only if they pass MFA.  
- **Concrete example**: A developer cannot act as admin all the time. If they need to create a new EC2 instance, they log in, pass MFA, and temporarily assume the admin role.  
- **Why it matters**: Minimizes exposure to elevated privileges — attackers have a much smaller window of opportunity.  
- **Expected proof**: Switching into `AdminJIT` fails if MFA is not provided.  

---

### 2. IAM PoLP & ABAC
- **PoLP (Principle of Least Privilege)**: Give each user only the rights they need.  
  Example: A developer working on front-end code only needs S3 read access, not admin rights.  
- **ABAC (Attribute-Based Access Control)**: Use tags to control access dynamically.  
  Example: A user tagged `team=Payments` can only access S3 buckets and EC2 instances also tagged `team=Payments`.  
- **Concrete example**: Instead of writing one policy for Payments, one for Analytics, one for Mobile — you write a single policy that enforces access by matching tags.  
- **Why it matters**: Prevents cross-team mistakes (a Payments engineer cannot touch Analytics data), while keeping policies scalable.  
- **Expected proof**: A Payments team developer is denied when trying to open an Analytics bucket.  

---

### 3. S3 Hardening
- **Problem**: Public S3 buckets are one of the most common data leak vectors. Companies have lost millions due to “misconfigured S3”.  
- **Solution**: Apply a strict S3 bucket policy that:  
  - blocks public access,  
  - forces HTTPS traffic only,  
  - restricts access to a defined AWS region.  
- **Concrete example**: If someone tries to download files using HTTP, or from another region like `us-east-1`, the request is denied.  
- **Why it matters**: Prevents accidental leaks of sensitive customer data.  
- **Expected proof**: Access attempt over HTTP fails, while HTTPS in the right region works.  

---

### 4. CloudTrail + EventBridge
- **CloudTrail**: Think of it as the security camera of AWS. It records every API call (who did what, and when).  
- **EventBridge**: Acts like an alarm system. When CloudTrail sees something sensitive (root login, Access Key created), EventBridge triggers an alert.  
- **Concrete example**: If someone logs in with the root account at 3AM, EventBridge sends an SNS email alert to security.  
- **Why it matters**: Allows **early detection** of suspicious or risky activity before damage is done.  
- **Expected proof**: An alert email is received whenever a root login happens.  

---

### 5. Incident Drill (Exposed Access Key)
- **Scenario**: Imagine a developer mistakenly uploads their AWS Access Key to GitHub. Attackers could use it immediately.  
- **Response process**:  
  1. Revoke or rotate the compromised key.  
  2. Investigate what actions were taken with it using CloudTrail logs.  
  3. Document findings and lessons learned in a report under `reports/`.  
- **Concrete example**: CloudTrail shows that the leaked key was used to list S3 buckets. The report documents this and the corrective steps (revoking key, enabling key monitoring).  
- **Why it matters**: Demonstrates not just prevention, but also **incident response capability** — essential for real-world security.  
- **Expected proof**: A written incident report with logs and the mitigation actions taken.  
