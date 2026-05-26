# ☁️ Cloud Networking in Google Cloud Platform (GCP)

## 📘 Overview

This project demonstrates the fundamentals of networking in Google Cloud Platform (GCP) using both default and custom Virtual Private Cloud (VPC) networks.

The lab covers:

- Understanding default and custom VPC networks
- Creating subnet-based custom networks
- Configuring firewall rules
- Applying network tags to virtual machines
- Securing communication between instances

---

# 🚀 What You'll Learn

- Basic concepts and constructs of Google Cloud networking
- Difference between default and custom VPC networks
- How subnet-based networks work in GCP
- How firewall rules control traffic flow
- How to use instance tags for firewall policies
- How to create secure cloud network architectures

---

# 🏗️ Architecture Overview

```text
                    INTERNET
                        │
                        ▼
              ┌─────────────────┐
              │ Firewall Rules  │
              └─────────────────┘
                        │
                        ▼
         ┌───────────────────────────┐
         │   Custom VPC Network      │
         │  cloudmart-prod-network   │
         └───────────────────────────┘
                │           │
                ▼           ▼

      ┌────────────────┐   ┌────────────────┐
      │ Web Subnet     │   │ App Subnet     │
      │ 10.10.0.0/24   │   │ 10.10.1.0/24   │
      └────────────────┘   └────────────────┘
                │
                ▼
      ┌────────────────┐
      │ DB Subnet      │
      │ 10.10.2.0/24   │
      └────────────────┘
```

---

# 📋 Prerequisites

Before starting this lab, ensure you have:

- A Google Cloud account
- Billing enabled
- Access to Google Cloud Console
- Basic understanding of networking concepts
- Compute Engine API enabled

---

### CloudMart GCP Networking Lab

Built for learning and hands-on practice in Google Cloud Networking and Infrastructure.
