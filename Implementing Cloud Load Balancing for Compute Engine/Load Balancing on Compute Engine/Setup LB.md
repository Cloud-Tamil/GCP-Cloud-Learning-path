# 🚀 GCP Production Environment — Complete Console Implementation Guide

A step-by-step guide to deploying a production-grade web infrastructure on **Google Cloud Platform** using the GCP Console. This guide covers VPC networking, managed instance groups, autoscaling, health checks, and HTTP(S) load balancing.

---

## 📋 Table of Contents

- [Implementation Order](#implementation-order)
- [Step 1: Create VPC Network](#step-1-create-vpc-network)
- [Step 2: Create Subnets](#step-2-create-subnets)
- [Step 3: Create Firewall Rules](#step-3-create-firewall-rules)
- [Step 4: Create Instance Template](#step-4-create-instance-template)
- [Step 5: Create Health Check](#step-5-create-health-check)
- [Step 6: Create Managed Instance Group](#step-6-create-managed-instance-group)
- [Step 7: Create Backend Service](#step-7-create-backend-service)
- [Step 8: Create Load Balancer IP Address](#step-8-create-load-balancer-ip-address)
- [Step 9: Create URL Map](#step-9-create-url-map)
- [Step 10: Create HTTP/HTTPS Proxy](#step-10-create-httphttps-proxy)
- [Step 11: Create Forwarding Rules](#step-11-create-forwarding-rules)
- [Step 12: SSL Certificate (Optional)](#step-12-ssl-certificate-optional)
- [Step 13: Network Load Balancer (Optional)](#step-13-network-load-balancer-tcp---optional)
- [Step 14: Verify & Test](#step-14-verify--test)
- [Dependency Quick Reference](#dependency-quick-reference)
- [Common Mistakes to Avoid](#common-mistakes-to-avoid)
- [Troubleshooting Guide](#troubleshooting-guide)
- [Success Criteria Checklist](#success-criteria-checklist)

---

## ✅ Implementation Order (Verified)

```
1.  VPC Network          (Foundation)
         ↓
2.  Subnets              (Network Segmentation)
         ↓
3.  Firewall Rules       (Security)
         ↓
4.  Instance Template    (Blueprint)
         ↓
5.  Health Check         (Monitoring)
         ↓
6.  Managed Instance Group (Compute)
         ↓
7.  Backend Service      (Load Balancing Logic)
         ↓
8.  Load Balancer IP Address (Networking)
         ↓
9.  URL Map              (Routing Rules)
         ↓
10. HTTP/HTTPS Proxy     (Traffic Handler)
         ↓
11. Forwarding Rules     (Traffic Entry)
         ↓
12. SSL Certificate      (Security — Optional)
         ↓
13. Verify & Test        (Validation)
```

---

## Step 1: Create VPC Network

**Console Path:** `Navigation Menu → VPC Network → VPC Networks → CREATE VPC NETWORK`

| Field | Value |
|---|---|
| Name | `prod-vpc` |
| Subnet creation mode | Custom |
| Firewall rules | None *(created separately in Step 3)* |

<details>
<summary>📸 Console Screenshot Guide</summary>

```
[VPC Networks] → [CREATE VPC NETWORK]
┌─────────────────────────────────────────────────────┐
│ Name:              prod-vpc                         │
│ Subnet creation:   ○ Automatic    ● Custom          │
│                                                     │
│ [SUBNET CREATION]                                   │
│ (We'll add subnets below)                           │
└─────────────────────────────────────────────────────┘
```

</details>

---

## Step 2: Create Subnets

**Console Path:** In the same VPC creation screen → **Add Subnet**

| Subnet Name | Region | IP Range | Private Google Access |
|---|---|---|---|
| `prod-subnet-1` | us-central1 | 10.10.0.0/24 | On |
| `prod-subnet-2` | us-central1 | 10.10.1.0/24 | On |
| `prod-subnet-3` | us-central1 | 10.10.2.0/24 | On |

<details>
<summary>📸 Console Screenshot Guide</summary>

```
[VPC Networks] → [CREATE VPC NETWORK] → [SUBNET] section
┌─────────────────────────────────────────────────────┐
│ SUBNET 1:                                           │
│ Name:              prod-subnet-1                    │
│ Region:            us-central1                      │
│ IP range:          10.10.0.0/24                     │
│ Private Google:    ☑ On                             │
│                                                     │
│ [+ ADD SUBNET]                                      │
└─────────────────────────────────────────────────────┘
```

</details>

Click **CREATE** after adding all subnets.

---

## Step 3: Create Firewall Rules

**Console Path:** `Navigation Menu → VPC Network → Firewall → CREATE FIREWALL RULE`

Create all four rules below individually.

### Rule 1: HTTP Access

| Field | Value |
|---|---|
| Name | `prod-allow-http` |
| Network | `prod-vpc` |
| Direction | Ingress |
| Action | Allow |
| Target tags | `prod-http-server` |
| Source IP ranges | `0.0.0.0/0` |
| Protocols and ports | `tcp:80` |

### Rule 2: HTTPS Access

| Field | Value |
|---|---|
| Name | `prod-allow-https` |
| Network | `prod-vpc` |
| Direction | Ingress |
| Action | Allow |
| Target tags | `prod-http-server` |
| Source IP ranges | `0.0.0.0/0` |
| Protocols and ports | `tcp:443` |

### Rule 3: Health Checks

| Field | Value |
|---|---|
| Name | `prod-allow-health-checks` |
| Network | `prod-vpc` |
| Direction | Ingress |
| Action | Allow |
| Target tags | `prod-http-server` |
| Source IP ranges | `130.211.0.0/22`, `35.191.0.0/16` |
| Protocols and ports | `tcp:80` |

### Rule 4: Internal Communication

| Field | Value |
|---|---|
| Name | `prod-allow-internal` |
| Network | `prod-vpc` |
| Direction | Ingress |
| Action | Allow |
| Target tags | `prod-http-server` |
| Source IP ranges | `10.10.0.0/16` |
| Protocols and ports | `tcp:0-65535`, `udp:0-65535`, `icmp` |

<details>
<summary>📸 Console Screenshot Guide</summary>

```
[Firewall] → [CREATE FIREWALL RULE]
┌─────────────────────────────────────────────────────┐
│ Name:              prod-allow-http                  │
│ Network:           prod-vpc                         │
│ Direction:         ● Ingress    ○ Egress            │
│ Action:            ● Allow      ○ Deny              │
│ Target tags:       prod-http-server                 │
│ Source IP ranges:  0.0.0.0/0                        │
│                                                     │
│ Protocols:         ☑ TCP                            │
│ Ports:             80                               │
└─────────────────────────────────────────────────────┘
```

</details>

Click **CREATE** for each rule.

---

## Step 4: Create Instance Template

**Console Path:** `Navigation Menu → Compute Engine → Instance Templates → CREATE INSTANCE TEMPLATE`

### Basic Configuration

| Section | Field | Value |
|---|---|---|
| Identity | Name | `prod-web-template` |
| Identity | Region | `us-central1` |
| Machine Configuration | Machine type | `e2-standard-2` (2 vCPU, 8 GB) |
| Boot Disk | Operating System | Debian |
| Boot Disk | Version | Debian GNU/Linux 12 |
| Boot Disk | Size | 20 GB |
| Networking | Network | `prod-vpc` |
| Networking | Subnet | `prod-subnet-1` |
| Networking | Network tags | `prod-http-server` |

### Advanced — Startup Script

Navigate to the **Management** tab and paste the following startup script:

```bash
#!/bin/bash
# Update package index
apt-get update

# Install Apache
apt-get install -y apache2

# Create custom index page
cat <<EOF > /var/www/html/index.html
<!DOCTYPE html>
<html>
<head>
    <title>Production Environment</title>
    <style>
        body { font-family: Arial, sans-serif; text-align: center; padding: 50px; background: #f0f0f0; }
        h1 { color: #1a73e8; }
        .server-info { background: white; padding: 20px; border-radius: 8px; box-shadow: 0 2px 4px rgba(0,0,0,0.1); }
    </style>
</head>
<body>
    <div class="server-info">
        <h1>🚀 Production Web Server</h1>
        <p>Hostname: <strong>$(hostname)</strong></p>
        <p>Zone: <strong>$(curl -s http://metadata.google.internal/computeMetadata/v1/instance/zone -H 'Metadata-Flavor: Google' | cut -d'/' -f4)</strong></p>
        <p>Instance ID: <strong>$(curl -s http://metadata.google.internal/computeMetadata/v1/instance/id -H 'Metadata-Flavor: Google')</strong></p>
        <p>Environment: <strong>Production</strong></p>
        <p>Deployed: <strong>$(date)</strong></p>
    </div>
</body>
</html>
EOF

# Start and enable Apache
systemctl start apache2
systemctl enable apache2

echo "✅ Startup script completed successfully!"
```

<details>
<summary>📸 Console Screenshot Guide</summary>

```
[Instance Templates] → [CREATE INSTANCE TEMPLATE]
┌─────────────────────────────────────────────────────┐
│ Name:              prod-web-template                │
│ Region:            us-central1                      │
│ Machine type:      e2-standard-2                    │
│                                                     │
│ [BOOT DISK]                                         │
│ OS: Debian GNU/Linux 12                             │
│ Size: 20 GB                                         │
│                                                     │
│ [NETWORKING]                                        │
│ Network:           prod-vpc                         │
│ Subnet:            prod-subnet-1                    │
│ Network tags:      prod-http-server                 │
│                                                     │
│ [MANAGEMENT]                                        │
│ Startup script:    (paste above script)             │
└─────────────────────────────────────────────────────┘
```

</details>

Click **CREATE** and wait 1–2 minutes.

---

## Step 5: Create Health Check

**Console Path:** `Navigation Menu → Compute Engine → Health Checks → CREATE HEALTH CHECK`

| Field | Value |
|---|---|
| Name | `prod-web-health-check` |
| Protocol | HTTP |
| Port | 80 |
| Request path | `/` |
| Check interval | 10 seconds |
| Timeout | 5 seconds |
| Healthy threshold | 2 |
| Unhealthy threshold | 3 |

<details>
<summary>📸 Console Screenshot Guide</summary>

```
[Health Checks] → [CREATE HEALTH CHECK]
┌─────────────────────────────────────────────────────┐
│ Name:              prod-web-health-check            │
│ Protocol:          HTTP                             │
│ Port:              80                               │
│ Request path:      /                                │
│ Check interval:    10 seconds                       │
│ Timeout:           5 seconds                        │
│ Healthy threshold: 2                                │
│ Unhealthy threshold: 3                              │
└─────────────────────────────────────────────────────┘
```

</details>

Click **CREATE**.

---

## Step 6: Create Managed Instance Group

**Console Path:** `Navigation Menu → Compute Engine → Instance Groups → CREATE INSTANCE GROUP`

### Basic Configuration

| Field | Value |
|---|---|
| Name | `prod-web-mig` |
| Instance template | `prod-web-template` |
| Location | Multiple zones |
| Region | `us-central1` |

### Autoscaling Configuration

| Field | Value |
|---|---|
| Autoscaling policy | On |
| Minimum instances | 3 |
| Maximum instances | 10 |
| Target CPU utilization | 70% |
| Cool down period | 120 seconds |

### Health Check

| Field | Value |
|---|---|
| Health check | `prod-web-health-check` |

<details>
<summary>📸 Console Screenshot Guide</summary>

```
[Instance Groups] → [CREATE INSTANCE GROUP]
┌─────────────────────────────────────────────────────┐
│ Name:              prod-web-mig                     │
│ Template:          prod-web-template                │
│ Location:          Multiple zones                   │
│ Region:            us-central1                      │
│                                                     │
│ [AUTOSCALING]                                       │
│ Minimum: 3        Maximum: 10                       │
│ Target CPU: 70%   Cool down: 120s                   │
│                                                     │
│ [HEALTH CHECK]                                      │
│ Health check:      prod-web-health-check            │
└─────────────────────────────────────────────────────┘
```

</details>

Click **CREATE** and wait **3–5 minutes** for instances to start.

---

## Step 7: Create Backend Service

**Console Path:** `Navigation Menu → Network Services → Load Balancing → CREATE LOAD BALANCER`

### Load Balancer Type Selection

```
○ HTTP(S) Load Balancing → Internet facing → Continue
```

### Backend Configuration

| Field | Value |
|---|---|
| Name | `prod-web-backend` |
| Backend type | Instance group |
| Instance group | `prod-web-mig` |
| Port | 80 |
| Balancing mode | Utilization |
| Maximum utilization | 80% |
| Health check | `prod-web-health-check` |
| Timeout | 30 seconds |

<details>
<summary>📸 Console Screenshot Guide</summary>

```
[Load Balancing] → [CREATE LOAD BALANCER]
┌─────────────────────────────────────────────────────┐
│ BACKEND CONFIGURATION                               │
│ Name:              prod-web-backend                 │
│ Backend type:      Instance group                   │
│ Instance group:    prod-web-mig                     │
│ Port:              80                               │
│ Balancing mode:    Utilization                      │
│ Max utilization:   80%                              │
│ Health check:      prod-web-health-check            │
│ Timeout:           30 seconds                       │
└─────────────────────────────────────────────────────┘
```

</details>

---

## Step 8: Create Load Balancer IP Address

**Console Path:** In the same Load Balancer configuration → **Frontend Configuration**

| Field | Value |
|---|---|
| IP address | Create IP address |
| Name | `prod-lb-ip` |

<details>
<summary>📸 Console Screenshot Guide</summary>

```
[Load Balancer] → [Frontend Configuration]
┌─────────────────────────────────────────────────────┐
│ IP Address:        [Create IP Address]              │
│ Name:              prod-lb-ip                       │
│                                                     │
│ ⏳ Waiting for IP allocation...                     │
└─────────────────────────────────────────────────────┘
```

</details>

Wait for IP address creation **(1–2 minutes)**.

---

## Step 9: Create URL Map

**Console Path:** In the Load Balancer configuration → **Host and Path Rules**

| Field | Value |
|---|---|
| Select a service | `prod-web-backend` |

<details>
<summary>📸 Console Screenshot Guide</summary>

```
[Load Balancer] → [Host and Path Rules]
┌─────────────────────────────────────────────────────┐
│ HOST & PATH RULES                                   │
│ Host:              *                                │
│ Path:              *                                │
│ Service:           prod-web-backend                 │
└─────────────────────────────────────────────────────┘
```

</details>

---

## Step 10: Create HTTP/HTTPS Proxy

**Console Path:** In Load Balancer configuration → **Frontend Configuration (continued)**

| Field | Value |
|---|---|
| Protocol | HTTP |
| Port | 80 |

<details>
<summary>📸 Console Screenshot Guide</summary>

```
[Load Balancer] → [Frontend Configuration]
┌─────────────────────────────────────────────────────┐
│ PROXY & FRONTEND                                    │
│ Protocol:          HTTP                             │
│ Port:              80                               │
│ IP Address:        prod-lb-ip                       │
└─────────────────────────────────────────────────────┘
```

</details>

---

## Step 11: Create Forwarding Rules

**Console Path:** `Load Balancer → REVIEW AND FINALIZE → CREATE`

<details>
<summary>📸 Console Screenshot Guide</summary>

```
[Load Balancer] → [REVIEW AND FINALIZE]
┌─────────────────────────────────────────────────────┐
│ CONFIGURATION SUMMARY                               │
│ ✓ Backend: prod-web-backend                        │
│ ✓ Host and Path Rules: prod-web-backend            │
│ ✓ Frontend: HTTP:80 prod-lb-ip                     │
│                                                     │
│ [CREATE]                                            │
└─────────────────────────────────────────────────────┘
```

</details>

Click **CREATE** and wait **5–10 minutes** for provisioning.

---

## Step 12: SSL Certificate (Optional)

**Console Path:** `Navigation Menu → Network Services → Load Balancing → SSL Certificates → CREATE SSL CERTIFICATE`

| Field | Value |
|---|---|
| Name | `prod-web-ssl-cert` |
| Certificate type | Google-managed |
| Domains | `www.yourdomain.com` |

### Update Load Balancer for HTTPS

Navigate to `Load Balancing → Click on your load balancer → EDIT` and add an HTTPS frontend with the following configuration:

| Field | Value |
|---|---|
| Protocol | HTTPS |
| Port | 443 |
| Certificate | `prod-web-ssl-cert` |

---

## Step 13: Network Load Balancer (TCP — Optional)

**Console Path:** `Navigation Menu → Network Services → Load Balancing → CREATE LOAD BALANCER → TCP Load Balancing`

### Backend Configuration

| Field | Value |
|---|---|
| Name | `prod-network-pool` |
| Instance group | `prod-web-mig` |
| Port | 80 |

### Frontend Configuration

| Field | Value |
|---|---|
| IP address | Create IP address → `prod-network-lb-ip` |
| Port | 80 |

---

## Step 14: Verify & Test

### 1. Check Instance Status

**Console Path:** `Compute Engine → VM Instances`

```
Expected: 3 instances running with prod-http-server tag
Status: ✅ RUNNING
```

### 2. Check Instance Group Health

**Console Path:** `Compute Engine → Instance Groups → prod-web-mig`

```
Expected: All 3 instances show "Healthy"
Status: ✅ HEALTHY (3/3)
```

### 3. Test HTTP Load Balancer

**Console Path:** `Network Services → Load Balancing → Click your load balancer`

1. Copy the IP address from the **Frontend** section
2. Open your browser and navigate to `http://[LOAD-BALANCER-IP]`
3. **Expected result:** Production web page displaying hostname and zone info

### 4. Test Load Balancer Distribution

Run the following to verify traffic is distributed across instances:

```bash
# Send multiple requests to verify load balancing
for i in {1..10}; do
    echo "Request $i:"
    curl -s http://[LOAD-BALANCER-IP] | grep -E "Hostname|Zone"
    echo "---"
done
```

### 5. Test Network Load Balancer (if created)

```bash
# Test TCP load balancer
curl -s http://[NETWORK-LB-IP]
```

---

## Dependency Quick Reference

```
1.  VPC Network       ← Required by everything
2.  Subnets           ← Required for IP addressing
3.  Firewall Rules    ← Required for network access
4.  Instance Template ← Required for creating VMs
5.  Health Checks     ← Required for monitoring
6.  Instance Group    ← Uses template & health checks
7.  Backend Service   ← Uses instance group & health checks
8.  Load Balancer IP  ← Required for frontend
9.  URL Map           ← Required for routing
10. HTTP Proxy        ← Uses URL map
11. Forwarding Rules  ← Uses proxy & IP
12. SSL Certificate   ← Optional, uses load balancer
13. Verify & Test     ← Final validation
```

---

## ⚠️ Common Mistakes to Avoid

| ❌ Mistake | ✅ Solution |
|---|---|
| Skipping firewall rules | Create all 4 firewall rules before instances |
| Rushing health checks | Wait for all 3 instances to show "Healthy" |
| Forgetting network tags | Ensure `prod-http-server` tag is on template |
| Using wrong subnet | Use `prod-subnet-1` in template |
| Skipping verification | Always test HTTP load balancer URL |

---

## 🆘 Troubleshooting Guide

### Instances Not Starting

1. Go to `Compute Engine → VM Instances`
2. Verify the instance template has the correct startup script
3. Check startup script logs via **Serial port output**

### Health Check Failing

1. Ensure the firewall rule for health checks exists (`prod-allow-health-checks`)
2. Verify Apache is running on instances
3. Confirm the health check path is set to `/`

### Load Balancer Not Working

1. Verify the backend service has the instance group attached
2. Check that forwarding rules are created
3. Wait 5–10 minutes for provisioning to complete
4. Check that the firewall rule allows ingress from `0.0.0.0/0`

---

## ✅ Success Criteria Checklist

- [ ] VPC Network `prod-vpc` created with 3 subnets
- [ ] 4 Firewall rules created and active
- [ ] Instance template `prod-web-template` created
- [ ] Health check `prod-web-health-check` created
- [ ] Managed Instance Group has 3 instances running
- [ ] All instances show as "Healthy"
- [ ] Backend service created with MIG
- [ ] Load balancer has static IP address
- [ ] URL map routes to backend service
- [ ] HTTP proxy is active
- [ ] Forwarding rules exist
- [ ] HTTP load balancer returns web page
- [ ] Load balancer distributes traffic across zones
