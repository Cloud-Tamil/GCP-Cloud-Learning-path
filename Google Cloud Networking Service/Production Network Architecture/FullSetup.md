# CloudMart GCP Production Network Learning

This project demonstrates how to build a secure three-tier production network in Google Cloud Platform manually and using gcloud scripts.

---

# Architecture

- Custom VPC
- Web Tier
- Application Tier
- Database Tier
- Firewall Security
- Private Networking
- Public/Private VM Design

---

# Network Design

| Tier | Subnet | CIDR | Access |
|------|------|------|------|
| Web | 10.10.0.0/24 | Public | Internet-facing |
| App | 10.10.1.0/24 | Private | Internal |
| Database | 10.10.2.0/24 | Restricted | Most Secure |

---

# Project Structure

| Folder | Purpose |
|---|---|
| manual-guide | Step-by-step console setup |
| scripts | Automation scripts |
| screenshots | Console screenshots |
| architecture | Network diagrams |

---

# Manual Setup Guide

Follow these guides:

1. manual-guide/01-create-vpc.md
2. manual-guide/02-create-firewall-rules.md
3. manual-guide/03-create-vms.md
4. manual-guide/04-testing.md

---

# Automated Deployment

## Create Network

```bash
./scripts/create-network.sh
```

## Create Firewalls

```bash
./scripts/create-firewalls.sh
```

## Create VMs

```bash
./scripts/create-vms.sh
```

---

# Testing

```bash
./scripts/test-network.sh
```

Expected Output:

```text
✅ Web Server Accessible
✅ SSH Working
✅ App VM Private
✅ DB VM Private
```

---

# Cleanup

```bash
./scripts/cleanup.sh
```

---

# Learning Objectives

- Understand VPC networking
- Create secure firewall rules
- Build subnet isolation
- Understand public vs private VMs
- Learn production network architecture
- Practice Google Cloud networking

---
