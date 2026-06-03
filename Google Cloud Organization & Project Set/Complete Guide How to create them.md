# ☁️ Google Cloud Platform (GCP) Project & Organization Guide

A beginner-friendly guide to understanding Google Cloud Projects, Project IDs, Billing Accounts, Organizations, APIs, and project management using both the Google Cloud Console and gcloud CLI.

---

### 📖 Overview

Think of a Google Cloud Project as a digital container that holds all your cloud resources—such as virtual machines, databases, storage buckets, APIs, and networking components.

A project provides:

* Resource organization
* Billing management
* Identity and Access Management (IAM)
* API enablement
* Security boundaries
* Lifecycle management

Without a project, Google Cloud services cannot be used.

---

### 📚 Table of Contents

1. Introduction
2. What is a Google Cloud Project?
3. Why Do We Need a Project?
4. Understanding Project Identifiers
5. Creating a Project Using the Console
6. Setting Up Billing
7. Enabling APIs
8. Understanding Organizations
9. Organization vs No Organization
10. Creating an Organization
11. Managing Projects in an Organization
12. Creating Projects Using gcloud CLI
13. Billing Management Using CLI
14. API Management Using CLI
15. Project Cleanup
16. Best Practices
17. Summary

---

## 1. Introduction

Google Cloud Platform (GCP) organizes all resources inside Projects.

Examples of resources:

* Compute Engine Virtual Machines
* Cloud Storage Buckets
* Cloud SQL Databases
* Cloud Run Services
* Kubernetes Clusters
* BigQuery Datasets
* VPC Networks

Every resource created in GCP belongs to a specific project.

---

## 2. What is a Google Cloud Project?

A Google Cloud Project is the fundamental building block of Google Cloud.

Example:

```text
Project: ecommerce-dev

├── Compute Engine VM
├── Cloud SQL Database
├── Cloud Storage Bucket
├── Cloud Run Service
└── Enabled APIs
```

Projects help separate environments and workloads.

Examples:

```text
ecommerce-dev
ecommerce-test
ecommerce-prod
```

---

## 3. Why Do We Need a Project?

| Purpose               | Description                       |
| --------------------- | --------------------------------- |
| Resource Organization | Groups related resources together |
| Billing               | Tracks cloud spending             |
| IAM                   | Controls user access              |
| API Management        | Enables Google Cloud services     |
| Security Boundary     | Isolates resources                |
| Lifecycle Management  | Easy creation and deletion        |

---

## 4. Understanding Project Identifiers

Every Google Cloud project contains three identifiers.

### 4.1 Project Name

Example:

```text
Learning Project
```

Characteristics:

* Human-readable
* Can be modified
* Does not need to be unique

---

### 4.2 Project ID

Example:

```text
learning-gcp-2026
```

Characteristics:

* Globally unique
* Permanent
* Cannot be changed
* Used by APIs and CLI tools

Example:

```bash
gcloud config set project learning-gcp-2026
```

---

### 4.3 Project Number

Example:

```text
58646390136
```

Characteristics:

* Generated automatically
* Used internally by Google
* Rarely modified or referenced

---

## 5. Creating a Project Using the Console

### Step 1: Open Google Cloud Console

```text
https://console.cloud.google.com
```

Sign in using:

* Gmail Account
* Google Workspace Account

---

### Step 2: Create a New Project

1. Click Project Selector
2. Select **New Project**
3. Enter Project Name
4. Modify Project ID (optional)
5. Select Location
6. Click **Create**

Example:

```text
Project Name: My Learning Project
Project ID: my-learning-project
Location: No Organization
```

---

## 6. Setting Up Billing

Most Google Cloud services require billing.

Even when using:

* Free Trial Credits
* Always Free Tier Services

A billing account must be linked.

### Create Billing Account

Navigation:

```text
Billing → Create Billing Account
```

Required Information:

* Country
* Tax Details
* Payment Method

---

### Budget Alerts

Recommended budget:

```text
$10
```

Recommended alert thresholds:

| Alert   | Threshold |
| ------- | --------- |
| Alert 1 | 50%       |
| Alert 2 | 90%       |
| Alert 3 | 100%      |

Navigation:

```text
Billing → Budgets & Alerts
```

---

## 7. Enabling APIs

Google Cloud services operate through APIs.

Navigation:

```text
APIs & Services → Library
```

Common APIs:

* Compute Engine API
* Cloud Run API
* Cloud Storage API
* Cloud Build API
* Vertex AI API
* BigQuery API

Click:

```text
ENABLE
```

---

## 8. Understanding Organizations

Organizations provide centralized management for projects.

Resource Hierarchy:

```text
Organization
│
├── Folder
│   ├── Project A
│   └── Project B
│
└── Folder
    ├── Project C
    └── Project D
```

Organizations are commonly used by businesses and enterprises.

---

## 9. Organization vs No Organization

### No Organization

Recommended for:

* Students
* Personal Learning
* Training Labs

Advantages:

* Simple setup
* Fewer restrictions
* Faster onboarding

---

### Organization

Recommended for:

* Startups
* Businesses
* Enterprises

Advantages:

* Centralized IAM
* Security Policies
* Folder Structure
* Compliance Controls

---

## 10. Creating an Organization

Requirements:

✅ Custom Domain

Example:

```text
mycompany.com
```

Not Supported:

```text
gmail.com
```

---

### Using Cloud Identity Free

Steps:

1. Purchase a domain
2. Register for Cloud Identity Free
3. Verify domain ownership
4. Create administrator account

Example:

```text
admin@mycompany.com
```

5. Sign in to Google Cloud Console

The organization is created automatically.

---

## 11. Managing Projects in an Organization

Create projects under:

```text
IAM & Admin
    ↓
Manage Resources
    ↓
Create Project
```

Project Location:

```text
Organization Name
```

Example:

```text
mycompany.com
```

---

## 12. Creating Projects Using gcloud CLI

### Authenticate

```bash
gcloud auth login
```

Verify:

```bash
gcloud auth list
```

---

### Create Project

```bash
gcloud projects create my-first-project-2026
```

Advanced Example:

```bash
gcloud projects create my-sample-project-123 \
    --name="My First CLI Project" \
    --set-as-default \
    --labels=environment=development
```

---

### Project ID Rules

| Rule            | Requirement                 |
| --------------- | --------------------------- |
| Length          | 6–30 Characters             |
| Characters      | Lowercase, Numbers, Hyphens |
| Start Character | Letter                      |
| Unique          | Globally Unique             |

Valid:

```text
cloud-demo-project-01
```

Invalid:

```text
Cloud_Project
my project
123project
```

---

## 13. Billing Management Using CLI

List Billing Accounts:

```bash
gcloud billing accounts list
```

Link Billing Account:

```bash
gcloud billing projects link PROJECT_ID \
    --billing-account=BILLING_ACCOUNT_ID
```

Verify:

```bash
gcloud beta billing projects describe PROJECT_ID
```

---

## 14. API Management Using CLI

Enable Compute Engine:

```bash
gcloud services enable compute.googleapis.com
```

Enable Cloud Run:

```bash
gcloud services enable run.googleapis.com
```

Enable Multiple APIs:

```bash
gcloud services enable \
run.googleapis.com \
cloudbuild.googleapis.com \
artifactregistry.googleapis.com
```

List Enabled APIs:

```bash
gcloud services list
```

---

## 15. Project Cleanup

Delete Project:

```bash
gcloud projects delete PROJECT_ID
```

Example:

```bash
gcloud projects delete my-test-project-456
```

Deleting a project removes:

* Compute Engine Instances
* Cloud Storage Buckets
* Cloud SQL Databases
* Cloud Run Services
* Kubernetes Clusters

Always remove unused resources to prevent unexpected charges.

---

## 16. Best Practices

#### Separate Environments

```text
ecommerce-dev
ecommerce-test
ecommerce-prod
```

#### Enable Budget Alerts

Monitor spending proactively.

#### Use Labels

Example:

```bash
--labels=environment=production,team=platform
```

#### Enable Only Required APIs

Reduce security risks and simplify management.

#### Delete Unused Resources

Avoid unnecessary costs.

#### Follow Least Privilege Access

Grant only the permissions users require.

---

## 17. Summary

Before using Google Cloud:

✅ Create a Project

✅ Understand Project IDs

✅ Configure Billing

✅ Set Budget Alerts

✅ Enable Required APIs

✅ Learn IAM Basics

✅ Use "No Organization" for learning

✅ Use Organizations for business workloads

✅ Clean up resources after testing

> - By mastering Projects, Billing, Organizations, and the gcloud CLI, you build the foundation required for working with any Google Cloud service, including Compute Engine, Cloud Run, GKE, Cloud Storage, BigQuery, and Vertex AI.
