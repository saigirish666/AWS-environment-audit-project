# AWS-environment-audit-project

This guided project is part of my cloud journey and focuses on **auditing** an AWS environment using core services like IAM, EC2, VPC, Security Groups, Network ACLs, CloudWatch, and CloudTrail.  
All tasks were performed on the latest AWS console, using an AWS Skill Builder lab environment.

---

## 🚀 Table of Contents

- [Lab Overview](#lab-overview)  
- [Lab Sections](#lab-sections)  
  - [1. Audit IAM User Permissions](#1-audit-iam-user-permissions)  
  - [2. Review EC2 Security Group Configuration](#2-review-ec2-security-group-configuration)  
  - [3. Review VPC and Network ACLs](#3-review-vpc-and-network-acls)  
  - [4. Audit CloudWatch Metrics and EBS Monitoring](#4-audit-cloudwatch-metrics-and-ebs-monitoring)  
  - [5. Review CloudTrail Logs in S3](#5-review-cloudtrail-logs-in-s3)  
- [Key Learnings & Best Practices](#key-learnings--best-practices)  
- [🌟 Observations & Console Evolution](#-observations--console-evolution)  
- [Evidence of Completion](#evidence-of-completion)  
- [📌 Related Projects](#-related-projects)  

---

## Lab Overview

**Objective:**  
Perform a basic security and compliance audit of an AWS environment by reviewing:

- IAM users, groups, and effective permissions  
- EC2 Security Groups and network isolation  
- VPC, subnets, and Network ACL rules  
- CloudWatch metrics and monitoring views  
- CloudTrail configuration and raw logs stored in S3  

Screenshots for each section are stored in the `screenshots/` folder (IAM, Security Groups, NACLs, CloudWatch, CloudTrail, S3 logs, and assessment result).

---

## Lab Sections

### 1. Audit IAM User Permissions

**Goal:**  
Review the permissions of an IAM user and validate effective access using the IAM Policy Simulator.

**Steps:**  
- Opened **IAM → Users** and inspected the `user-1` account.  
- Reviewed **Security credentials** (access keys, console password, MFA status).  
- Checked **Groups** and confirmed membership in `user1group`.  
- On the **Permissions** tab, verified that `ReadOnlyAccess` is attached via the group.  
- Opened **IAM Policy Simulator**, selected `user-1`, chose the IAM service, and simulated:
  - `DeleteGroup`  
  - `DeleteRolePolicy`  
- Both actions were **denied**, matching the read‑only policy expectations.

**Key Learnings:**  
- Group-based policies simplify permission management for multiple users.  
- Policy Simulator is essential to prove which actions are allowed or implicitly denied before granting access.

---

### 2. Review EC2 Security Group Configuration

**Goal:**  
Understand how Security Groups protect the Web, Bastion, and SQL instances and enforce tiered access.

**Steps:**  
- Opened **EC2 → Security Groups** and reviewed three groups: `WebServerSG`, `BastionSG`, and `SQLSG`.  
- For **WebServerSG**:
  - Inbound rules allow HTTP (80) and HTTPS (443) only from `10.10.10.0/24` (internal network).  
  - RDP (3389) allowed only from the **BastionSG** security group.  
- For **BastionSG**:
  - Inbound SSH (22) and RDP (3389) allowed from `10.10.10.0/24` (management network).  
  - Outbound rules allow traffic so the bastion can reach internal instances.  
- For **SQLSG**:
  - Inbound SQL ports restricted to traffic coming from specific Security Groups (Web/Bastion), not from public IP ranges.  
  - Outbound rules kept minimal to required access only.

**Key Learnings:**  
- Security Groups are **stateful** and operate at the **instance level**.  
- Bastion + Web + DB security group chaining creates a secure, layered architecture where the database is never directly exposed.

---

### 3. Review VPC and Network ACLs

**Goal:**  
Verify subnet-level controls using Network ACLs and confirm the VPC topology used by the EC2 instances.

**Steps:**  
- From the **Web Server** instance details, copied the **VPC ID**.  
- Opened **VPC → Your VPCs** and selected the matching **Lab VPC**.  
- From the VPC details, opened the **Main network ACL**.  
- On the **Inbound rules** tab:
  - Observed explicit allow rules for required traffic types.  
  - Saw a more general deny rule with a higher rule number.  
- On the **Outbound rules** tab:
  - Confirmed similar explicit allow + deny structure for outbound traffic.  

**Key Learnings:**  
- Network ACLs are **stateless** and apply at the **subnet boundary**.  
- Rules are evaluated in order of rule number; the **lowest number has highest priority**.  
- Combining Security Groups and NACLs gives defense-in-depth.

---

### 4. Audit CloudWatch Metrics and EBS Monitoring

**Goal:**  
Locate and interpret CloudWatch metrics for EC2 instances and EBS volumes used in the lab.

**Steps:**  
- Opened **CloudWatch → Metrics → EC2 → Per-Instance Metrics**.  
- Searched for `CPUUtilization` and graphed metrics for the **SQL Server** instance.  
- Observed CPU trends over the selected time period and experimented with different **Statistic** and **Period** values.  
- Switched to **EC2 → Volumes**, selected the volume attached to the Web Server, and opened the **Monitoring** tab.  
- Reviewed metrics like:
  - Average read/write latency  
  - Volume IOPS, throughput checks  
  - Queue length and read/write throughput  

**Key Learnings:**  
- CloudWatch provides near real-time metrics that can be used as **audit evidence** for performance and availability.  
- EBS volume metrics help detect storage bottlenecks or unusual patterns.

---

### 5. Review CloudTrail Logs in S3

**Goal:**  
Verify CloudTrail configuration and view raw API/console activity logs stored in S3.

**Steps:**  
- Opened **CloudTrail → Trails** and selected `LabCloudTrail`.  
- Checked:
  - Trail status (logging enabled).  
  - S3 destination bucket name starting with `spl73logs`.  
- Opened **S3**, navigated into the `spl73logs...` bucket, then:  
  - `AWSLogs / <account-id> / CloudTrail / <region> / <year> / <month> / <day> /`  
- Opened one of the `.json.gz` log files and viewed raw event records (user identity, event name, source IP, time, etc.).

**Key Learnings:**  
- CloudTrail + S3 gives a centralized, immutable audit trail for all supported AWS API calls.  
- Logs can be further analyzed using Athena, external SIEM tools, or JSON viewers.

---

## Key Learnings & Best Practices

- Use **IAM groups and managed policies** to standardize permissions and simplify audits.  
- Prefer **Security Groups + Bastion Hosts + SG chaining** instead of exposing RDP/SQL directly to the internet.  
- Combine **Security Groups (stateful)** with **NACLs (stateless)** to implement layered network security.  
- Leverage **CloudWatch metrics** as part of operational and compliance dashboards.  
- Store and retain **CloudTrail logs** in S3 for investigations, compliance, and change tracking.

---

## 🌟 Observations & Console Evolution

- Security Group and VPC pages in the latest console provide clearer tabs for **Inbound / Outbound Rules** and quick navigation between related resources (VPC, subnets, NACLs).  
- CloudWatch metrics UI now supports easy switching between time ranges and periods, making it faster to zoom into incidents.  
- CloudTrail trails show direct links to their S3 bucket destinations, speeding up the process of jumping from configuration to raw logs.  

---

## Evidence of Completion

- Successfully completed the AWS Skill Builder lab **“Performing a Basic Audit of Your AWS Environment”**.  
- Lab assessment score: **100%** (screenshot stored as `Screenshots/Screenshot 2026-01-05 135035.png`).  

---

