# 🔐 Cloud NGFW: Migrate VPC Firewall Rules (Network-based)

## 📌 Overview

We will demonstrates how to migrate traditional **VPC firewall rules** to **Cloud NGFW (Next-Generation Firewall) policies**.

The goal is to move from **decentralized, VPC-level rules** to a **centralized firewall policy model**, improving scalability, security, and manageability.\

---

## 🎯 Objectives

- Understand VPC firewall rule limitations  
- Create a Network Firewall Policy  
- Migrate existing firewall rules into the policy  
- Associate the policy with a VPC network  
- Validate traffic behavior after migration  

---

## 🧠 Architecture

### 🔹 Before Migration

VPC Network → Individual Firewall Rules

### 🔹 After Migration

Firewall Policy → Attached to VPC → Centralized Rule Management

---

## ⚙️ Prerequisites

- Google Cloud Project  
- Compute Engine API enabled  
- Basic knowledge of:
  - VPC Networks  
  - Firewall Rules  
  - gcloud CLI  

---
