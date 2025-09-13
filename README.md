# EC2 Monitoring and Remediation System – ServiceNow Implementation

This document outlines the automated EC2 monitoring and remediation system implemented on the ServiceNow platform. The system addresses a key reliability gap: failed AWS EC2 instances previously went undetected or required manual remediation steps, increasing downtime and impacting business operations.  

By integrating AWS monitoring with ServiceNow, this solution enables near real-time detection, guided remediation, and full auditability for DevOps teams.  

---

## System Overview

When an EC2 instance fails, the AWS integration server updates the EC2 Instance table in ServiceNow. This triggers an automated remediation flow that:  

- Creates a ServiceNow incident documenting the failure.  
- Retrieves a knowledge article with step-by-step remediation instructions using AI Search.  
- Sends the article and context to the DevOps Slack channel.  
- Provides a one-click remediation option from the EC2 Instance record.  
- Logs all remediation attempts in a dedicated Remediation Log table.

This workflow reduces downtime, improves DevOps response speed, and creates visibility for both technical teams and auditors.  

---

## Implementation Steps

### 1. Core Application Setup
- Created a **scoped application**: `EC2 Monitoring and Remediation`.  
- Added two custom tables:  
  - **EC2 Instance** – stores health status and metadata for EC2 instances.  
  - **Remediation Log** – records every remediation attempt (API response, HTTP status, success flag).  
- Configured secure integration with AWS monitoring servers using:  
  - `Connection & Credential Alias`  
  - `HTTP Connection`  
  - `Basic Auth Credentials`  

### 2. Knowledge Integration
- Authored a knowledge article describing EC2 remediation steps.  
- Published the article in a Knowledge Base accessible to AI Search.  
- Configured AI Search to return this article when the flow is triggered.  

### 3. Automated Workflow (Flow Designer)
- **Trigger**: EC2 Instance status updates to `OFF`.  
- **Actions**:  
  - **AI Search** → retrieve EC2 remediation article.  
  - **Slack Action** → send article to DevOps Slack channel via webhook.  
  - **Incident Action** → create an incident in ServiceNow to document the failure.  

### 4. DevOps Workflow
- Implemented a **UI Action**: `Trigger EC2 Remediation`.  
- The button calls a **Script Include** (`EC2RemediationHelper`) via GlideAjax.  
- Script Include executes an API call to AWS to restart the failed instance.  
- Remediation attempt automatically logged in the Remediation Log table.  

---

## Architecture Diagram

See `Diagram.png` for a complete workflow view:  
- EC2 failure detection  
- ServiceNow integration (EC2 Instance + Remediation Log tables)  
- Flow Designer automation (AI Search → Slack → Incident)  
- DevOps manual remediation via UI Action

![System Overview](https://github.com/bcjumpman/ec2-remediation-system/blob/e768a35210159f7cdf58c0157c28dd79a110d44c/Diagram.png)

---

## Optimization

- Added ACLs for **tables**, **UI Action**, and **Script Include** to enforce security.  
- Optimized the Flow Designer workflow with a “for every update” trigger for reliability.  
- Enhanced Slack notifications with:  
  - Direct link to the incident record.  
  - Direct link to the knowledge article.  
- Standardized required fields on custom tables for consistency.  

---

## DevOps Usage

1. DevOps team receives a **Slack alert** when an EC2 instance fails.  
2. Alert includes a knowledge article with step-by-step remediation instructions.  
3. Engineer opens the associated EC2 Instance record in ServiceNow.  
4. Click **Trigger EC2 Remediation** to restart the instance.  
5. Verify remediation success in the **Remediation Log** table (e.g., HTTP 201 response).  

This process provides speed, visibility, and accountability across the full incident lifecycle.  

---
