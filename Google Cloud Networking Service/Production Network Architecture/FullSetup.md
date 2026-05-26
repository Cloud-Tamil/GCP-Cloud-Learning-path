# CloudMart Production Network on Google Cloud

A complete three-tier production-grade networking setup in Google Cloud Platform (GCP) for an e-commerce platform.

---

# Architecture Overview

This project demonstrates:

- Custom VPC Network
- Three-tier architecture
- Web subnet
- Application subnet
- Database subnet
- Secure firewall segmentation
- Private VM architecture
- SSH restricted access
- Validation and testing scripts

---

# Architecture Diagram

![Architecture](architecture/architecture-diagram.png)

---

# Network Design

| Tier | Subnet | CIDR | Purpose |
|------|---------|------|----------|
| Web Tier | cloudmart-web-subnet | 10.10.0.0/24 | Internet-facing servers |
| App Tier | cloudmart-app-subnet | 10.10.1.0/24 | Internal business logic |
| DB Tier | cloudmart-db-subnet | 10.10.2.0/24 | Secure database layer |

---

# Security Design

## Firewall Rules

| Rule | Purpose |
|------|----------|
| HTTP | Allow website access |
| HTTPS | Secure traffic |
| Web → App | Internal API communication |
| App → DB | Database communication |
| SSH | Restricted admin access |
| Health Checks | Google load balancer probes |
| Default Deny | Block everything else |

---

# Deployment Methods

## Option 1: Manual Deployment (Console UI)

Follow:
- manual-guide/01-create-vpc.md
- manual-guide/02-create-subnets.md
- manual-guide/03-firewall-rules.md

---

## Option 2: CLI Deployment

Run:

```bash
chmod +x scripts/*.sh
./scripts/create-network.sh
./scripts/create-firewalls.sh
./scripts/create-vms.sh
```

---

# Validation

Run:

```bash
./scripts/validation.sh
```

Expected result:

```text
✅ HTTP Access: Working
✅ SSH Access: Working
✅ App VM: No public IP
✅ Database VM: No public IP
🎉 PRODUCTION NETWORK FULLY VERIFIED!
```

---

# Security Best Practices Used

- Least privilege access
- Private application/database tiers
- Restricted SSH access
- Default deny firewall model
- Network segmentation
- Controlled ingress traffic

---

# Cost Optimization

- e2-micro instances
- Free-tier eligible resources
- Minimal architecture footprint

---

# Future Improvements

- Load Balancer
- Managed Instance Groups
- Cloud SQL
- Cloud Armor
- Cloud NAT
- VPC Flow Logs
- Cloud Monitoring

---

# Cleanup

To remove all resources:

```bash
./scripts/cleanup.sh
```

---

# Author

Your Name

---

# License

MIT License
