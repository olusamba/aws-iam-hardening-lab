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

---

## ⚙️ How to Reproduce

Follow these steps to replicate the AWS IAM Hardening Lab in your own AWS account.  
This guide assumes basic familiarity with IAM, AWS CLI, and AWS console.

---

### 1. Prerequisites
- ✅ An active AWS account (Free Tier is enough)  
- ✅ AWS CLI installed and configured  
- ✅ Git installed locally  
- ✅ (Optional) Terraform or CloudFormation for IaC automation  
- ✅ Basic knowledge of IAM concepts (users, groups, roles, policies)  

---

### 2. Clone the Repository
```bash
git clone git@github.com:olusamba/aws-iam-hardening-lab.git
cd aws-iam-hardening-lab
```

---

### 3. Create a Lab Admin User
- Log in with your **root account** (only for initial setup)
- Create a New IAM user `iam-lab-admin`.
- Attach **AdministratorAccess** policy (temporary).
- Generate access keys and store them securely.

---

### 4. Configure AWS CLI
```bash
aws configure
```
- Enter the access key/secret key for `iam-lab-admin`.
- Choose region (e.g., `eu-west-1`).
- Set output format to `json`.

Verify setup:
```bash
aws s3 ls
```

---

### 5. Baseline IAM Setup
- Create IAM groups: `Admins`, `Developers`, `Auditors`.
- Attach least-privilege policies from `/policies/`.
- Add test IAM users to these groups.

Example:
``` bash
aws iam create-group --group-name Admins
aws iam attach-group-policy \
  --group-name Admins \
  --policy-arn arn:aws:iam::aws:policy/AdministratorAccess
```

---

### 5.1 ABAC with Team Tags
- Create IAM users with tags (Payments, Analytics).
- Create S3 buckets tagged with the same team.
- Apply the ABAC policy from `/policies/abac-team-policy.json`.
```bash
aws iam create-user --user-name dev-payments --tags Key=team,Value=Payments
aws iam create-user --user-name dev-analytics --tags Key=team,Value=Analytics

aws s3api create-bucket --bucket payments-team-data --region eu-west-1
aws s3api put-bucket-tagging \
  --bucket payments-team-data \
  --tagging 'TagSet=[{Key=team,Value=Payments}]'
```

**Expected result:**
- `dev-payments` can access  `payments-team-data`.
- `dev-payments`is **denied** if they try to access `analytics-team-data`.

---

### 6. Enforce MFA
- Require MFA for all IAM users.
- Save screenshots of setup in `/screenshots/mfa/`.

---

### 7. Harden Root Account
- Remove any root access keys.
- Enable MFA on root account.
- Store recovery codes offline.

---

### 8. Apply IAM Best Practices
- Enable **IAM Access Analyzer**.
- Apply **Service Control Policies (SCPs)** if using AWS Organizations.
- Use custom policies from `/policies/`.

---

### 9. Enable Logging & Monitoring
- Enable **CloudTrail** in all regions.
- Enable **AWS Config** for compliance.
- Send logs to a dedicated S3 bucket with restricted access.
Example:
```bash
aws s3api create-bucket \
  --bucket iam-hardening-logs-$(date +%s) \
  --region eu-west-1 \
  --create-bucket-configuration LocationConstraint=eu-west-1
```

---

### 10. Test & Validate
- Try privilege escalation scenarios (see `/attacks/`).
- Example: attempt `iam:CreateAccessKey` from a restricted user → should be denied.
- Save logs and screenshots in `/screenshots/tests/`.

---

### 11. Cleanup
- Remove test users, groups, and temporary policies.
- Delete unused CloudTrail trails/buckets to avoid costs.
```bash
aws iam delete-user --user-name test-user
aws iam delete-group --group-name Developers
```

---

## 📑 Reports & Evidence

This lab includes multiple forms of evidence to demonstrate that the security controls were **implemented, tested, and validated**.  
The goal is to provide both **technical logs** and **visual proof** so that a reviewer can quickly verify the outcomes.

---

### 1. CloudTrail Reports (`/reports/cloudtrail/`)
- Contains raw CloudTrail logs in JSON format.  
- Examples include:
  - **Root login attempts**  
  - **Access key creation events**  
  - **Denied API calls** when policies block actions  
- **Expected proof**: shows “AccessDenied” or “MFA required” in the event logs.

---

### 2. AWS Config Compliance (`/reports/config/`)
- Contains compliance snapshots from AWS Config.  
- Demonstrates whether resources comply with security rules (e.g., MFA enabled, S3 not public).  
- **Expected proof**: non-compliant resources are flagged until fixed.

---

### 3. MFA Evidence (`/screenshots/mfa/`)
- Screenshots showing MFA setup for IAM users and the root account.  
- **Expected proof**: console screenshot showing “MFA device assigned”.

---

### 4. Test Scenarios (`/screenshots/tests/`)
- Visual proof of denied/allowed actions during test cases.  
- Examples:
  - A `Payments` user successfully accesses the `payments-team-data` bucket.  
  - The same user is denied when accessing `analytics-team-data`.  
- **Expected proof**: AWS console or CLI output showing “AccessDenied”.

---

### 5. Incident Drill Reports (`/reports/incidents/`)
- Written reports of simulated incidents (e.g., exposed access key).  
- Includes timeline of events, CloudTrail queries, and remediation steps.  
- **Expected proof**: documented analysis of what was done and lessons learned.

---

### 6. Attack Scenarios (`/attacks/`)
- Contains privilege escalation attempts.  
- Each scenario documents:
  - The attempted attack (e.g., creating new keys, escalating roles).  
  - The expected outcome (denied by policy).  
  - Logs or screenshots confirming the denial.  
- **Expected proof**: shows that the hardening measures are effective.
