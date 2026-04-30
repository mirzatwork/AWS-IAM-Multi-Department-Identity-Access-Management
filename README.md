# AWS-IAM-Multi-Department-Identity-Access-Management

**Platform:** Amazon Web Services (IAM, S3, CloudTrail)  
**Cost:** 100% Free Tier  
**LinkedIn:** [linkedin.com/in/mirzarayyan](https://linkedin.com/in/mirzarayyan) · **GitHub:** [github.com/mirzatwork](https://github.com/mirzatwork?tab=repositories)

---

## Overview

A hands-on AWS IAM project simulating identity and access management for a four-department organisation. Built entirely on the AWS Free Tier, this project applies core cloud security principles including least-privilege access, group-based permissions, MFA enforcement, and audit logging via CloudTrail.

---

## Project Scope

| Metric | Detail |
|--------|--------|
| Departments | HR · Finance · Engineering · Security |
| IAM Groups | 4 |
| IAM Users | 09 |
| Custom Policies | 4 (3 S3 bucket policies + Force-MFA) |
| S3 Buckets | 3 (HR, Finance, Engineering) |
| Audit Logging | CloudTrail trail — org-audit-trail |

---

## Organisation Structure

| Department | IAM Group | Users | Policies Attached |
|------------|-----------|-------|-------------------|
| HR | HR-Team | hr-manager, hr-analyst, hr-intern | S3-HR-ReadWrite, IAMUserChangePassword, Force-MFA-Policy |
| Finance | Finance-Team | finance-manager, finance-analyst, billing-viewer | S3-Finance-ReadWrite, AWSBillingReadOnlyAccess, Force-MFA-Policy |
| Engineering | Engineering-Team | dev-lead, developer-1, developer-2 | PowerUserAccess, S3-Eng-FullAccess, Force-MFA-Policy |
| Security | Security-Team | security-admin, soc-analyst, compliance-officer | SecurityAudit, Force-MFA-Policy |

> **Design principle:** All policies are attached to Groups, never to individual users. This enforces clean auditability and makes permission changes scalable.

---

## Implementation Steps

### Step 1 — Secure the Root Account
- Enabled MFA on root using Google Authenticator
- Deleted all root access keys
- Root account locked away — all further work done via `iam-admin`
---

### Step 2 — Create Admin IAM User
- Created `iam-admin` with `AdministratorAccess` policy
- Enabled MFA on `iam-admin`
- All subsequent AWS work performed as `iam-admin` — root unused
---

### Step 3 — Create Four Department Groups
Created the following IAM User Groups (no policies attached yet):
- `HR-Team`
- `Finance-Team`
- `Engineering-Team`
- `Security-Team`

<!-- SCREENSHOT: IAM User Groups list showing all 4 groups -->
<img width="975" height="305" alt="image" src="https://github.com/user-attachments/assets/11a94d32-0c93-497d-8aa9-d1791286bc54" />


---

### Step 4 — Create S3 Buckets
Created three private S3 buckets — one per department that handles files:

| Bucket | Department |
|--------|------------|
| `company-hr-bucket-1` | HR |
| `company-finance-bucket-2` | Finance |
| `company-eng-bucket-3` | Engineering |

All buckets: Block all public access ON · Default encryption · No versioning (lab scope)

<!-- SCREENSHOT: S3 bucket list showing all 3 buckets -->
<img width="975" height="647" alt="image" src="https://github.com/user-attachments/assets/5de8e198-c2c6-438c-915b-c59898241d3b" />


---

### Step 5 — Create Custom IAM Policies

Four custom policies created via the JSON editor:

**S3-HR-ReadWrite** — Allows HR group to read, write, list, and delete objects in the HR bucket only.

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": ["s3:GetObject","s3:PutObject","s3:ListBucket","s3:DeleteObject"],
    "Resource": [
      "arn:aws:s3:::company-hr-bucket-1",
      "arn:aws:s3:::company-hr-bucket-1/*"
    ]
  }]
}
```

**S3-Finance-ReadWrite** — Read/write/list access to Finance bucket only. No delete (financial records protection).

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": ["s3:GetObject","s3:PutObject","s3:ListBucket"],
    "Resource": [
      "arn:aws:s3:::company-finance-bucket-2",
      "arn:aws:s3:::company-finance-bucket-2/*"
    ]
  }]
}
```

**S3-Eng-FullAccess** — Full S3 access scoped to Engineering bucket only.

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": "s3:*",
    "Resource": [
      "arn:aws:s3:::company-eng-bucket-3",
      "arn:aws:s3:::company-eng-bucket-3/*"
    ]
  }]
}
```

**Force-MFA-Policy** — Denies ALL AWS actions for any user who has not authenticated with MFA. Applied to every group.

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "DenyAllWithoutMFA",
    "Effect": "Deny",
    "NotAction": [
      "iam:CreateVirtualMFADevice",
      "iam:EnableMFADevice",
      "iam:GetUser",
      "iam:ListMFADevices",
      "iam:ListVirtualMFADevices",
      "iam:ResyncMFADevice",
      "sts:GetSessionToken"
    ],
    "Resource": "*",
    "Condition": {
      "BoolIfExists": {
        "aws:MultiFactorAuthPresent": "false"
      }
    }
  }]
}
```

### Step 6 — Attach Policies to Groups

| Group | Policies Attached |
|-------|-------------------|
| HR-Team | S3-HR-ReadWrite · IAMUserChangePassword · Force-MFA-Policy |
| Finance-Team | S3-Finance-ReadWrite · AWSBillingReadOnlyAccess · Force-MFA-Policy |
| Engineering-Team | PowerUserAccess · S3-Eng-FullAccess · Force-MFA-Policy |
| Security-Team | SecurityAudit · Force-MFA-Policy |

<!-- SCREENSHOT: One group's Permissions tab showing attached policies (e.g. Security-Team) -->
<img width="975" height="493" alt="image" src="https://github.com/user-attachments/assets/182d68a0-2364-4eb9-81e7-76abca0a06cf" />


---

### Step 7 — Create IAM Users

All users created with:
- Console access enabled
- Auto-generated password + forced reset at first login
- Added to department group at creation (inherits all group policies)
- Credentials downloaded as `.csv` per user

<!-- SCREENSHOT: IAM Users list showing all users -->

<img width="975" height="406" alt="image" src="https://github.com/user-attachments/assets/dc4f281c-4f97-4017-8241-5a4cca3b85cf" />


> **Security note:** Because `Force-MFA-Policy` is attached to all groups, every user is locked out of all AWS actions until they set up their own MFA device at first login.

---

### Step 8 — Enable CloudTrail Audit Logging

- Trail name: `org-audit-trail`
- Logs stored in auto-generated S3 bucket
- Management events enabled (Read + Write)
- Security Team (`soc-analyst`) has `SecurityAudit` policy — read access to all CloudTrail logs

<!-- SCREENSHOT: CloudTrail Event History showing sample logged events -->
<img width="975" height="367" alt="image" src="https://github.com/user-attachments/assets/c1fecabd-831d-4cb3-9ef6-96e29bbe2570" />


---

## Security Design Highlights

**Least Privilege** — Every group is scoped to only the resources and actions it needs. Finance cannot touch the HR bucket. HR cannot access billing. Engineering's `PowerUserAccess` is still scoped to their S3 bucket for storage.

**No Direct User Policies** — All permissions flow through groups. Adding or removing a user from a group instantly adjusts their access — no per-user policy audit needed.

**MFA Everywhere** — `Force-MFA-Policy` creates a hard block: non-MFA sessions cannot perform any action except setting up MFA itself. This applies to all 12 department users.

**Separation of Duties** — The Security Team has `SecurityAudit` (read-only across all services) but no write access. They can monitor everything without being able to modify resources.

**Root Account Hardened** — MFA enabled, access keys deleted, root never used after initial setup.

---

## Repository Structure

```
aws-iam-project/
├── README.md
├── policies/
│   ├── s3-hr-readwrite.json
│   ├── s3-finance-readwrite.json
│   ├── s3-eng-fullaccess.json
│   └── force-mfa-policy.json
```

---

## Tools & Skills Demonstrated

`AWS IAM` `S3` `CloudTrail` `Least Privilege` `MFA Enforcement` `Group-Based Access Control` `Custom IAM Policies` `JSON Policy Language` `Cloud Security` `Identity & Access Management`
