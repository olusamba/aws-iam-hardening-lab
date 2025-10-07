# 🔐 AWS IAM Hardening Lab

## 📖 Introduction
This project is a hands-on security lab focused on **Identity and Access Management (IAM)** in AWS.  
The goal is to design, implement, and document security controls that follow **real-world best practices** while remaining easy to reproduce in a learning or demo environment.

By building this lab, I aimed to strengthen both my **technical skills** (IAM policies, CloudTrail monitoring, MFA enforcement) and my **consulting mindset** (explaining *why* these controls matter to non-technical stakeholders).

---

## 🎯 Objectives
- Apply the **Principle of Least Privilege (PoLP)** in IAM.  
- Explore **ABAC (Attribute-Based Access Control)** for scalable permissions.  
- Enforce strong security for **S3 buckets** (block public access, HTTPS-only, region restrictions).  
- Implement **Just-in-Time (JIT) admin access** with MFA.  
- Set up **CloudTrail and EventBridge** for monitoring and alerting.  
- Run an **incident response drill** simulating exposed access keys.  

---

## 📦 Deliverables
This repository contains:
- ✅ JSON IAM policies (`/policies/`)  
- ✅ Documentation of EventBridge and CloudTrail configuration (`/eventbridge/`, `/cloudtrail/`)  
- ✅ Security hardening examples for S3 (`/policies/s3-*`)  
- ✅ Incident response reports (`/reports/`)  
- ✅ Screenshots of test cases (`/screenshots/`)  

---

## 🧩 Architecture and Security Design

### 1. Just-in-Time Admin with MFA
- **Problem:** Permanent admin accounts increase attack exposure.  
- **Solution:** Create a temporary `AdminJIT` role assumed only after MFA.  
- **Why it matters:** Minimizes privileged access exposure.  
- **Expected proof:** Switching into `AdminJIT` fails without MFA.  

---

### 2. IAM PoLP & ABAC
- **PoLP:** Each user gets only what they need.  
- **ABAC:** Permissions are dynamically based on tags.  
- **Why it matters:** Prevents cross-team data exposure while keeping IAM policies scalable.  
- **Expected proof:** A user in `team=Payments` is denied access to Analytics resources.  

---

### 3. S3 Hardening
- **Problem:** Misconfigured S3 buckets are a common cause of breaches.  
- **Solution:** Enforce HTTPS-only, block public access, restrict by region.  
- **Expected proof:** Requests over HTTP or from other regions are denied.  

---

### 4. CloudTrail + EventBridge
- **CloudTrail:** Logs every API action (who did what, when).  
- **EventBridge:** Triggers SNS alerts on specific sensitive events.  
- **Why it matters:** Enables proactive detection and alerting.  
- **Expected proof:** SNS alert received on suspicious `GetCallerIdentity` event.  

---

### 5. Incident Drill (Exposed Access Key)
- **Scenario:** Simulate a leaked Access Key used by a developer.  
- **Response process:**  
  1. Detect the event via CloudTrail & SNS.  
  2. Investigate through CloudTrail logs.  
  3. Revoke the key and document findings.  
- **Why it matters:** Tests detection, investigation, and response capabilities.  
- **Expected proof:** Alert email, CloudTrail record, and key revocation evidence.  

---

## ⚙️ How to Reproduce

Follow these steps to replicate the AWS IAM Hardening Lab in your own AWS account.  
Basic familiarity with IAM, AWS CLI, and AWS Console is recommended.

---

### 1. Prerequisites
- ✅ An active AWS account (Free Tier is enough)  
- ✅ Git installed locally  
- ✅ AWS CLI installed and configured  
- ✅ (Optional) Terraform or CloudFormation for automation  
- ✅ Basic knowledge of IAM concepts (users, groups, roles, policies)  

---

### 2. Setup Steps
```bash
git clone git@github.com:olusamba/aws-iam-hardening-lab.git
cd aws-iam-hardening-lab
```

---

### 3. Create a Lab Admin User
- Log in with your **root account** (only for initial setup).  
- Create a new IAM user `iam-lab-admin`.  
- Attach the **AdministratorAccess** policy temporarily.  
- Generate access keys and store them securely.

---

### 4. Configure AWS CLI
```bash
aws configure
```
- Enter the access key/secret key for `iam-lab-admin`.
- Choose region (e.g., `eu-west-1`).
- Set output format to `json`.

Verify the connection:
```bash
aws s3 ls
```

---

### 5. Baseline IAM Setup
- Create IAM groups: `Admins`, `Developers`, `Auditors`.
- Attach least-privilege policies from `/policies/`.
- Add test users to these groups.

Example:
```bash
aws iam create-group --group-name Admins
aws iam attach-group-policy   --group-name Admins   --policy-arn arn:aws:iam::aws:policy/AdministratorAccess
```

---

### 6. ABAC with Team Tags
- Tag users and resources with `team=Payments`, `team=Analytics`, etc.
- Apply ABAC policy from `/policies/abac-team-policy.json`.

Example:
```bash
aws iam create-user --user-name dev-payments --tags Key=team,Value=Payments
aws s3api create-bucket --bucket payments-team-data --region eu-west-1
aws s3api put-bucket-tagging   --bucket payments-team-data   --tagging 'TagSet=[{Key=team,Value=Payments}]'
```

**Expected result:**  
`dev-payments` can access its team bucket but is denied when accessing other teams’ resources.

---

### 7. Enforce MFA
- Require MFA for all IAM users.  
- Save screenshots of setup in `/screenshots/mfa/`.

---

### 8. Monitoring with CloudTrail & EventBridge
Enable logging and create alerting rules.

```bash
aws cloudtrail lookup-events --max-results 5
```

- Confirm CloudTrail is recording API events.  
- Create EventBridge rules to trigger SNS notifications for sensitive actions:  
  - Root login (optional example)  
  - Access key creation/deletion  
  - `GetCallerIdentity` API from unusual sources  

👉 In this lab, the SNS alert pipeline was demonstrated for a **GetCallerIdentity** event during the incident drill.

---

### 9. Cleanup (Optional)
After completing the lab, clean up resources to avoid unnecessary costs:

```bash
aws iam delete-user --user-name dev-alice
aws iam delete-group --group-name Developers
aws cloudtrail delete-trail --name lab-trail
```

*(This is optional and only for teardown — do not run during the lab.)*

---

## 🧾 Reports & Evidence
| Folder | Description |
|---------|-------------|
| `/reports/` | Written reports documenting test scenarios and incident drills (Markdown format). |
| `/screenshots/` | Visual proof of configuration steps and validation results. |
| `/policies/` | JSON IAM policies implementing PoLP and ABAC. |
| `/eventbridge/` | Documentation of the alerting pipeline (EventBridge → SNS → Email). |
| `/cloudtrail/` | Documentation of CloudTrail setup for logging and auditing. |

---

## 👤 Author

**Olivier Lusamba**  
IAM & Cloud Security Analyst / Consultant | CompTIA Security+ | Multilingual (FR, EN, ES)  

I focus on Identity and Cloud Security, building hands-on labs that strengthen IAM governance, harden AWS environments, and provide practical demonstrations.  
With a background spanning professional sports, digital consulting, and technical projects, I bring resilience, communication skills, and a results-driven mindset to cybersecurity.  

- 🌐 [LinkedIn](https://www.linkedin.com/in/olivier-lusamba)  
- 💻 [GitHub](https://github.com/olusamba)  
