# ☁️ CloudMart Production Network Setup

This guide walks through the manual creation of a **custom VPC**, **subnets**, **firewall rules**, and a **three‑tier VM architecture** (Web → App → DB) on Google Cloud Platform (GCP).

---

## 📌 Prerequisites

- A GCP project with **billing enabled**
- **Compute Engine API** enabled
- **IAM role**: `Compute Network Admin` + `Compute Instance Admin` (or `Owner`)
- A terminal / browser with access to [GCP Console](https://console.cloud.google.com)

---

## Part 1 - Create Custom VPC Network
- Go to **Navigation Menu** ☰ → **VPC network** → **VPC networks**
- Click **+ CREATE VPC NETWORK**

| Field | Value |
|-------|-------|
| Name | `cloudmart-prod-network` |
| Subnet creation mode | **Custom** |

> Do **not** click **Create** yet — proceed to Part 2.

---

## Part 2 - Create Subnets

- Inside the same VPC creation form, add **three subnets**:

### 2.1 Web Subnet

| Field | Value |
|-------|-------|
| Name | `cloudmart-web-subnet` |
| Region | `us-central1` |
| IP range | `10.10.0.0/24` |

- Click **Done**.

### 2.2 App Subnet

| Field | Value |
|-------|-------|
| Name | `cloudmart-app-subnet` |
| Region | `us-central1` |
| IP range | `10.10.1.0/24` |
| **Private Google Access** | **ON** |

- Click **Done**.

### 2.3 Database Subnet

| Field | Value |
|-------|-------|
| Name | `cloudmart-db-subnet` |
| Region | `us-central1` |
| IP range | `10.10.2.0/24` |
| **Private Google Access** | **ON** |

- Click **Done** → then click **CREATE** at the bottom.

✅ **VPC network `cloudmart-prod-network` is ready.**

---

## Part 3 - Firewall Rules

- Go to: **Navigation Menu** → **VPC network** → **Firewall**

### Rule 1 – Allow HTTP (to Web)

| Field | Value |
|-------|-------|
| Name | `cloudmart-allow-http-web` |
| Network | `cloudmart-prod-network` |
| Direction | Ingress |
| Action | Allow |
| Targets | Specified target tags |
| Target tags | `cloudmart-web` |
| Source IP ranges | `0.0.0.0/0` |
| Protocols/ports | `tcp:80` |

- Click **CREATE**.

### Rule 2 - Allow HTTPS (to Web)

| Field | Value |
|-------|-------|
| Name | `cloudmart-allow-https-web` |
| Target tags | `cloudmart-web` |
| Source IP ranges | `0.0.0.0/0` |
| Protocols/ports | `tcp:443` |

### Rule 3 - Web → App

| Field | Value |
|-------|-------|
| Name | `cloudmart-allow-web-to-app` |
| Target tags | `cloudmart-app` |
| Source IP ranges | `10.10.0.0/24` |
| Protocols/ports | `tcp:8080,tcp:8443` |

### Rule 4 - App → DB

| Field | Value |
|-------|-------|
| Name | `cloudmart-allow-app-to-db` |
| Target tags | `cloudmart-db` |
| Source IP ranges | `10.10.1.0/24` |
| Protocols/ports | `tcp:3306,tcp:5432` |

### Rule 5 - SSH (your corporate IP)

- First find your public IP:  
👉 Open [https://ifconfig.me](https://ifconfig.me) and copy the IP.

| Field | Value |
|-------|-------|
| Name | `cloudmart-allow-ssh-corp` |
| Target tags | `cloudmart-ssh` |
| Source IP ranges | `YOUR_IP/32` (e.g., `203.0.113.45/32`) |
| Protocols/ports | `tcp:22` |

### Rule 6 - Health Checks (GCP internal)

| Field | Value |
|-------|-------|
| Name | `cloudmart-allow-health-checks` |
| Target tags | `cloudmart-web,cloudmart-app` |
| Source IP ranges | `130.211.0.0/22,35.191.0.0/16` |
| Protocols/ports | `tcp:80,tcp:8080` |

### Rule 7 - Default Deny (lowest priority)

| Field | Value |
|-------|-------|
| Name | `cloudmart-deny-all` |
| Action | **Deny** |
| Source IP ranges | `0.0.0.0/0` |
| Priority | `65534` |
| Protocols | **All** |

✅ **Firewall rules complete.**

---

## Part 4 - Create Virtual Machines (Three‑Tier)

- Go to: **Navigation Menu** → **Compute Engine** → **VM instances**

### VM 1 - Web Server

Click **+ CREATE INSTANCE**

| Section | Field | Value |
|---------|-------|-------|
| Basics | Name | `cloudmart-web-vm` |
| | Region | `us-central1` |
| | Zone | `us-central1-a` |
| | Machine type | `e2-micro` |
| Networking (expand **Advanced options**) | Network | `cloudmart-prod-network` |
| | Subnetwork | `cloudmart-web-subnet` |
| | External IP | **Ephemeral** |
| | Network tags | `cloudmart-web,cloudmart-ssh` |
| Security (expand) | Startup script | (see below) |

**Startup Script:**
```bash
#!/bin/bash
apt-get update
apt-get install -y nginx
systemctl start nginx
echo '<h1>CloudMart Web Server - Production</h1>' > /var/www/html/index.html
