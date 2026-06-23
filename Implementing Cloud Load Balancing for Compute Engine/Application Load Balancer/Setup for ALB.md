# 🚀 Complete Production Application Load Balancer Setup Guide with VPC

A comprehensive, step-by-step guide to deploying a production-grade **Application Load Balancer (ALB)** on **Google Cloud Platform (GCP)** with a dedicated VPC network, managed instance groups, health checks, firewall rules, and full traffic verification.

---

## 📋 Complete Assenting Order (Logical Sequence)

| Order | Step | Component |
|-------|------|-----------|
| 1st | [Set Region/Zone](#step-1-set-default-region-and-zone) | Foundation |
| 2nd ⭐ | [Create VPC Network](#step-2-create-vpc-network) | Network Foundation |
| 3rd | [Create Firewall Rules](#step-3-create-firewall-rules) | Security Foundation |
| 4th | [Create Instance Template](#step-4-create-instance-template) | Infrastructure Blueprint |
| 5th | [Create Health Check](#step-5-create-health-check) | Monitoring Foundation |
| 6th | [Create Managed Instance Group](#step-6-create-managed-instance-group) | Compute Resources |
| 7th | [Reserve Static IP Address](#step-7-reserve-static-ip-address) | Network Entry Point |
| 8th | [Create Backend Service](#step-8-create-backend-service) | Load Balancer Core |
| 9th | [Create URL Map](#step-9-create-url-map) | Routing Rules |
| 10th | [Create Target Proxy](#step-10-create-target-http-proxy) | Connection Bridge |
| 11th | [Create Forwarding Rule](#step-11-create-forwarding-rule) | Traffic Entry Point |
| 12th | [Create Individual VMs (Optional)](#step-12-create-individual-vms-alternative-setup) | Alternative Setup |
| 13th | [Test and Verify](#step-13-test-and-verify) | Validation |

---

## Step 1: Set Default Region and Zone

> **Why First:** This establishes where all your resources will be created.

### Google Cloud Console Setup

Open **Cloud Shell** (top right corner icon) and run:

```bash
# Set production region
gcloud config set compute/region us-central1

# Set production zone
gcloud config set compute/zone us-central1-a

# Verify settings
gcloud config list
```

> **Production Note:**
> - `us-central1` (Iowa) is recommended for production
> - Multiple zones available: `a`, `b`, `c` for high availability
> - This ensures all resources are created in the correct location

✅ **Checkpoint:** Region and zone set successfully.

---

## Step 2: Create VPC Network

> **Why Second:** VPC network must exist before any resources can be created — it's the foundation for networking.

### Navigate to VPC Network

From **Navigation Menu (☰)**, select **VPC network > VPC networks**

### Create Production VPC Network

1. Click **Create VPC Network**

2. Configure **Basic Settings:**

   | Field | Value |
   |-------|-------|
   | Name | `prod-vpc-network` |
   | Description | Production VPC network for load balancer and backend instances |
   | Subnet creation mode | Custom |

3. **Create Subnet** — click **Add subnet:**

   | Field | Value |
   |-------|-------|
   | Name | `prod-subnet-us-central1` |
   | Region | `us-central1` |
   | IP address range | `10.10.0.0/24` |
   | Private Google access | Off (keep default) |
   | Flow logs | Off (keep default) |

4. **Firewall Rules Options:**
   - Uncheck ☐ "Allow HTTP traffic"
   - Uncheck ☐ "Allow HTTPS traffic"
   - *(We'll create specific firewall rules in Step 3)*

5. **Advanced Options:**

   | Field | Value |
   |-------|-------|
   | VPC peering | None |
   | Network MTU | 1460 (default) |
   | Routing mode | Regional |

6. Click **Create**

### Alternative: Create Custom VPC with Multiple Subnets (Optional Advanced)

If you need subnets in multiple zones:

| Subnet Name | Region | IP Range | Purpose |
|-------------|--------|----------|---------|
| `prod-subnet-a` | us-central1 | `10.10.1.0/24` | Zone A instances |
| `prod-subnet-b` | us-central1 | `10.10.2.0/24` | Zone B instances |
| `prod-subnet-c` | us-central1 | `10.10.3.0/24` | Zone C instances |
| `prod-subnet-lb` | us-central1 | `10.10.10.0/28` | Load balancer resources |

> **Production Note:**
> - Use private IP ranges (`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`)
> - Plan subnet sizes carefully — **they cannot be changed after creation**
> - For production, use `/24` subnets (256 addresses) or larger

✅ **Checkpoint:** VPC network `prod-vpc-network` should appear in the list.

---

## Step 3: Create Firewall Rules

> **Why Third:** Firewall rules must exist before any instances are created to ensure proper security.

### Navigate to Firewall Rules

From **Navigation Menu (☰)**, select **VPC network > Firewall**

---

### Rule 1 — `prod-allow-web` (Allow Web Traffic)

1. Click **Create Firewall Rule**
2. Configure:

   | Field | Value |
   |-------|-------|
   | Name | `prod-allow-web` |
   | Description | Allow HTTP and HTTPS traffic to production web servers |
   | Network | `prod-vpc-network` ⭐ |
   | Direction of traffic | Ingress |
   | Action on match | Allow |
   | Target tags | `prod-web-server` |
   | Source IP ranges | `0.0.0.0/0` |
   | Protocols and ports | ✅ Specified — `tcp:80,443` |
   | Enforcement | Enable (default) |

3. Click **Create**

---

### Rule 2 — `prod-allow-health-check` (Allow Health Checks)

1. Click **Create Firewall Rule**
2. Configure:

   | Field | Value |
   |-------|-------|
   | Name | `prod-allow-health-check` |
   | Description | Allow Google Cloud health checks to monitor instances |
   | Network | `prod-vpc-network` ⭐ |
   | Direction of traffic | Ingress |
   | Action on match | Allow |
   | Target tags | `prod-health-check` |
   | Source IP ranges | `130.211.0.0/22`, `35.191.0.0/16` |
   | Protocols and ports | ✅ Specified — `tcp:80` |
   | Enforcement | Enable (default) |

3. Click **Create**

---

### Rule 3 — `prod-allow-ssh` (SSH Access — Optional for Production)

For production, consider limiting SSH access:

1. Click **Create Firewall Rule**
2. Configure:

   | Field | Value |
   |-------|-------|
   | Name | `prod-allow-ssh` |
   | Description | Allow SSH access from secure networks only |
   | Network | `prod-vpc-network` |
   | Direction of traffic | Ingress |
   | Action on match | Allow |
   | Target tags | `prod-ssh-access` |
   | Source IP ranges | `YOUR_OFFICE_IP/32` ⚠️ *Replace with your IP* |
   | Protocols and ports | ✅ Specified — `tcp:22` |

3. Click **Create**

---

### Rule 4 — `prod-allow-internal` (Allow Internal Communication)

1. Click **Create Firewall Rule**
2. Configure:

   | Field | Value |
   |-------|-------|
   | Name | `prod-allow-internal` |
   | Description | Allow internal communication between instances |
   | Network | `prod-vpc-network` |
   | Direction of traffic | Ingress |
   | Action on match | Allow |
   | Target tags | `prod-internal` |
   | Source IP ranges | `10.10.0.0/24` (or your subnet range) |
   | Protocols and ports | ✅ Specified — `tcp:0-65535`, `udp:0-65535`, `icmp` |

3. Click **Create**

✅ **Checkpoint:** Firewall rules should appear in the list with the VPC network `prod-vpc-network`.

---

## Step 4: Create Instance Template

> **Why Fourth:** Instance template defines how VMs will be created — needed before creating the instance group.

### Navigate to Instance Templates

From **Navigation Menu**, select **Compute Engine > Instance templates**

Click **Create instance template**

### Configure Template

| Field | Value |
|-------|-------|
| Name | `prod-backend-template` |
| Description | Production backend instance template for load balancer |
| Region | `us-central1` |

**Machine type:**
- Click **Select** → Choose **E2** → Select `e2-standard-2` (2 vCPU, 8 GB memory) → Click **Select**

**Boot disk:**
- Click **Change** → OS: `Debian` → Version: `Debian GNU/Linux 12 (bookworm)` → Size: `20 GB` → Click **Select**

**Networking:**
- Network: `prod-vpc-network` ⭐
- Subnet: `prod-subnet-us-central1` ⭐
- Firewall: ✅ Allow HTTP traffic

**Advanced options > Networking > Network tags — add all of:**
- `prod-backend`
- `prod-health-check`
- `prod-web-server`
- `prod-ssh-access` *(if you created the SSH rule)*

**Advanced options > Metadata > Startup script:**

```bash
#!/bin/bash
apt-get update
apt-get install apache2 -y
a2enmod ssl
a2ensite default-ssl
systemctl restart apache2

vm_hostname="$(curl -H "Metadata-Flavor:Google" http://169.254.169.254/computeMetadata/v1/instance/name)"
vm_zone="$(curl -H "Metadata-Flavor:Google" http://169.254.169.254/computeMetadata/v1/instance/zone | cut -d/ -f4)"
vm_ip="$(curl -H "Metadata-Flavor:Google" http://169.254.169.254/computeMetadata/v1/instance/network-interfaces/0/ip)"

echo "
<!DOCTYPE html>
<html>
<head><title>Production Backend</title></head>
<body>
  <h1>Production Environment</h1>
  <p>Server: $vm_hostname</p>
  <p>Zone: $vm_zone</p>
  <p>Internal IP: $vm_ip</p>
  <p>VPC: prod-vpc-network</p>
  <p>Service: Production Web Application</p>
  <p>Time: $(date)</p>
</body>
</html>" | tee /var/www/html/index.html

systemctl restart apache2
```

Click **Create**

✅ **Checkpoint:** Instance template `prod-backend-template` should appear in the list.

---

## Step 5: Create Health Check

> **Why Fifth:** Health check must exist before creating the Managed Instance Group (for autohealing).

### Navigate to Health Checks

From **Navigation Menu**, select **Compute Engine > Health checks**

Click **Create health check**

### Configure Health Check

| Field | Value |
|-------|-------|
| Name | `prod-health-check` |
| Description | Production health check with stricter thresholds for high availability |
| Protocol | HTTP |
| Port | `80` |
| Check interval | `10` seconds (faster detection) |
| Timeout | `5` seconds |
| Healthy threshold | `2` (requires 2 successful checks) |
| Unhealthy threshold | `2` (requires 2 failed checks) |
| Request path | `/` |

Click **Create**

> **Production Note:** Stricter thresholds detect failures faster while avoiding false positives.

✅ **Checkpoint:** Health check `prod-health-check` should appear in the list.

---

## Step 6: Create Managed Instance Group

> **Why Sixth:** MIG creates and manages the VM instances that will serve traffic.

### Navigate to Instance Groups

From **Navigation Menu**, select **Compute Engine > Instance groups**

Click **Create instance group**

### Configure MIG

| Field | Value |
|-------|-------|
| Name | `prod-backend-group` |
| Description | Production managed instance group for load balancer with high availability |
| Location | Multiple zones |
| Region | `us-central1` |
| Zones | ✅ `us-central1-a` ✅ `us-central1-b` ✅ `us-central1-c` |
| Instance template | `prod-backend-template` |
| Distribution shape | Balanced |
| Autoscaling | Toggle **OFF** (configure after load balancer) |
| Number of instances | `3` |

**Advanced options > Autohealing:**
- Enable: ✅ Autohealing
- Health check: `prod-health-check`
- Initial delay: `60` seconds

**Advanced options > Network:**
- Network: `prod-vpc-network` ⭐
- Subnet: `prod-subnet-us-central1` ⭐

Click **Create** *(this may take 2–3 minutes)*

✅ **Checkpoint:** Instance group `prod-backend-group` should appear with 3 instances creating.

---

## Step 7: Reserve Static IP Address

> **Why Seventh:** Need a static IP before creating the load balancer frontend.

### Navigate to External IP Addresses

From **Navigation Menu**, select **VPC network > External IP addresses**

Click **Reserve Static Address**

### Configure Static IP

| Field | Value |
|-------|-------|
| Name | `prod-lb-ipv4-1` |
| Description | Production Load Balancer static IP address |
| Network Service Tier | Premium |
| IP version | IPv4 |
| Type | Global |
| Purpose | HTTP/S Load Balancing |
| Attached to | None |

Click **Reserve**

> ⚠️ **Copy the IP address displayed — you'll need it later.**

**Alternative Path:**
From Navigation Menu → **Networking > Load balancing** → Click **Create Load Balancer** → In frontend configuration, click **Create IP address**

✅ **Checkpoint:** Static IP `prod-lb-ipv4-1` should appear in the list with status **"Reserved"**.

---

## Step 8: Create Backend Service

> **Why Eighth:** Backend service connects the load balancer to your instance group.

### Navigate to Load Balancing

From **Navigation Menu**, select **Networking > Load balancing**

Click **Create Load Balancer**

### Start Configuration

| Field | Value |
|-------|-------|
| Type | Application Load Balancer (HTTP/S) |
| Facing | Public facing (external) |
| Region scope | Global |

Click **Continue**

### Configure Backend Service

1. Click **Backend configuration** in the left menu
2. Click **Create or select backend services > Create a backend service**

   | Field | Value |
   |-------|-------|
   | Name | `prod-backend-service` |
   | Description | Production web application backend service |
   | Backend type | Instance group |
   | Instance group | `prod-backend-group` |
   | Region | `us-central1` |
   | Port | `80` |
   | Balancing mode | Rate |
   | Max RPS | `100` |
   | Health check | `prod-health-check` |
   | Session affinity | None |
   | Protocol | HTTP |
   | Timeout | `30` seconds |
   | Logging | Optional |
   | CDN | Disabled (default) |

3. Click **Done**, then **Save**

✅ **Checkpoint:** Backend service `prod-backend-service` should show **3 healthy instances**.

---

## Step 9: Create URL Map

> **Why Ninth:** URL map defines routing rules for incoming requests.

In the **same Load Balancer configuration:**

1. Click **Host and path rules** in the left menu
2. Configure:

   | Field | Value |
   |-------|-------|
   | URL Map Name | `prod-web-map` |
   | Default service | `prod-backend-service` |

3. Click **Save**

> **Production Note:** For advanced routing, you can add rules such as:
> - `/api/*` → API backend service
> - `/images/*` → Cloud Storage bucket
> - `/video/*` → Video processing service

✅ **Checkpoint:** URL map `prod-web-map` should be configured with default backend.

---

## Step 10: Create Target HTTP Proxy

> **Why Tenth:** Proxy bridges the forwarding rule and URL map.

In the **same Load Balancer configuration:**

1. Click **Frontend configuration** in the left menu
2. Click **Add Frontend IP and Port**

   | Field | Value |
   |-------|-------|
   | Name | `prod-frontend` |
   | Protocol | HTTP |
   | Network | `prod-vpc-network` ⭐ |
   | IP address | `prod-lb-ipv4-1` |
   | Port | `80` |

3. Click **Done**

> **Note:** Creating the frontend automatically creates the target HTTP proxy.

**Verify Target Proxy (Optional):**
- Click **Review and finalize**
- Under **Summary**, note that `http-lb-proxy` is created automatically
- Click **Create**

✅ **Checkpoint:** Load balancer should be created with frontend, backend, and proxy.

---

## Step 11: Create Forwarding Rule

> **Why Eleventh:** Forwarding rule routes incoming traffic to the target proxy — created automatically when configuring frontend.

### Verification

The forwarding rule is already created. In the **Load Balancer details page**, look at the **Frontend** section. You should see:

| Field | Value |
|-------|-------|
| IP | `prod-lb-ipv4-1` |
| Port | `80` |
| Protocol | HTTP |

### To View the Forwarding Rule (Optional)

1. From **Navigation Menu**, select **VPC network > External IP addresses**
2. Click the IP address `prod-lb-ipv4-1`
3. Under **In use by**, it should show your forwarding rule

✅ **Checkpoint:** Load balancer should be fully configured and running.

---

## Step 12: Create Individual VMs (Alternative Setup)

> **Why This is Alternative:** This creates separate VMs without MIG — for testing or simple setups.

### Navigate to VM Instances

From **Navigation Menu**, select **Compute Engine > VM Instances**

---

### Create `prod-web-1`

1. Click **Create Instance**
2. Configure:

   | Field | Value |
   |-------|-------|
   | Name | `prod-web-1` |
   | Region | `us-central1` |
   | Zone | `us-central1-a` |
   | Machine type | `e2-standard-2` |
   | OS | Debian GNU/Linux 12 (bookworm) |
   | Boot disk size | 20 GB |
   | Network | `prod-vpc-network` ⭐ |
   | Subnet | `prod-subnet-us-central1` ⭐ |
   | Public IP | Ephemeral (or create static IP) |
   | Firewall | ✅ Allow HTTP traffic |

3. **Network tags — add:**
   - `prod-web-server`
   - `prod-health-check`
   - `prod-backend`
   - `prod-ssh-access` *(if you created SSH rule)*

4. **Advanced options > Startup script:**

```bash
#!/bin/bash
apt-get update
apt-get install apache2 -y
a2enmod ssl
a2ensite default-ssl
systemctl restart apache2

vm_hostname="$(curl -H "Metadata-Flavor:Google" http://169.254.169.254/computeMetadata/v1/instance/name)"
vm_zone="$(curl -H "Metadata-Flavor:Google" http://169.254.169.254/computeMetadata/v1/instance/zone | cut -d/ -f4)"
vm_ip="$(curl -H "Metadata-Flavor:Google" http://169.254.169.254/computeMetadata/v1/instance/network-interfaces/0/ip)"

echo "
<!DOCTYPE html>
<html>
<head><title>Production Server 1</title></head>
<body>
  <h1>Production Web Server: prod-web-1</h1>
  <p>Environment: Production</p>
  <p>Server: $vm_hostname</p>
  <p>Zone: $vm_zone</p>
  <p>Internal IP: $vm_ip</p>
  <p>VPC: prod-vpc-network</p>
</body>
</html>" | tee /var/www/html/index.html
```

5. Click **Create**

---

### Create `prod-web-2`

Repeat the steps above with:
- **Name:** `prod-web-2`
- **Zone:** `us-central1-b`
- Update the hostname in the startup script

---

### Create `prod-web-3`

Repeat the steps above with:
- **Name:** `prod-web-3`
- **Zone:** `us-central1-c`
- Update the hostname in the startup script

✅ **Checkpoint:** Three VMs should appear in the list with internal IPs on `prod-vpc-network`.

---

## Step 13: Test and Verify

### Verify Instance Health

1. Go to **Networking > Load balancing**
2. Click your load balancer `prod-web-map`
3. Click the backend name `prod-backend-service`
4. Under **Instance health**, verify:
   - All instances show **Healthy** ✅
   - Green checkmarks for each instance

---

### Test Load Balancer

1. From the **Load Balancer details page**, copy the IP Address from the **Frontend** section
2. Open a new browser tab and visit:

```
http://[IP_ADDRESS]
```

**Expected output:**

```
Production Environment
Server: prod-backend-group-xxxx
Zone: us-central1-a
Internal IP: 10.10.0.x
VPC: prod-vpc-network
Service: Production Web Application
Time: [Current Date/Time]
```

> 💡 Refresh multiple times — you should see **different server names and zones**, confirming load balancing is working.

---

### Test Individual VMs (Optional)

1. Go to **Compute Engine > VM Instances**
2. Copy the **External IP** of `prod-web-1`
3. Visit `http://[IP_ADDRESS]`
4. Should show: `Production Web Server: prod-web-1`
5. Repeat for `prod-web-2` and `prod-web-3`

---

### Test Internal Connectivity (Optional)

SSH into one instance and ping another using internal IP:

```bash
ping 10.10.0.x
```

---

### Test Health Check Endpoint

```bash
curl http://[INSTANCE_INTERNAL_IP]/health
```

✅ **Final Checkpoint:** Load balancer is fully functional with all instances serving traffic.

---

## 📊 Complete Sequential Order Summary

| Order | Component | Resource Name | Purpose | VPC Connection |
|-------|-----------|---------------|---------|----------------|
| 1st | Region/Zone | `us-central1/a` | Foundation | — |
| 2nd ⭐ | VPC Network | `prod-vpc-network` | Network Foundation | **NEW** |
| 3rd | Firewall Rules | `prod-allow-web`, `prod-allow-health-check` | Security | Uses `prod-vpc-network` |
| 4th | Instance Template | `prod-backend-template` | VM Blueprint | Uses `prod-vpc-network` |
| 5th | Health Check | `prod-health-check` | Monitoring | — |
| 6th | Managed Instance Group | `prod-backend-group` | Compute Resources | Uses `prod-vpc-network` |
| 7th | Static IP | `prod-lb-ipv4-1` | Network Entry | Attaches to `prod-vpc-network` |
| 8th | Backend Service | `prod-backend-service` | Connection | — |
| 9th | URL Map | `prod-web-map` | Routing | — |
| 10th | Target Proxy | `http-lb-proxy` (auto) | Bridge | — |
| 11th | Forwarding Rule | (auto created) | Traffic Entry | Uses `prod-vpc-network` |
| 12th | Individual VMs | `prod-web-1`, `prod-web-2`, `prod-web-3` | Alternative | Uses `prod-vpc-network` |
| 13th | Test & Verify | — | Validation | — |

---

## 🔍 Troubleshooting

### Instances Not Healthy

Check in this order:

1. **VPC Network** (Step 2) — Instance has network connectivity
2. **Firewall rules** (Step 3) — Allow health check traffic on `prod-vpc-network`
3. **Health check** (Step 5) — Configured correctly
4. **Instance group** (Step 6) — Autohealing enabled
5. **Startup script** (Step 4) — Apache installed correctly

---

### Load Balancer Not Working

Check in this order:

1. **VPC Network** (Step 2) — Load balancer frontend uses correct VPC
2. **Firewall rules** (Step 3) — Allow incoming traffic on `prod-vpc-network`
3. **Backend service** (Step 8) — Has healthy instances
4. **URL map** (Step 9) — Points to correct backend
5. **Frontend** (Step 10) — Correct IP and port
6. **Static IP** (Step 7) — Reserved properly

---

### IP Not Found

Check in this order:

1. **VPC Network** (Step 2) — Global IP access enabled
2. **Static IP** (Step 7) — Reserved properly on `prod-vpc-network`
3. **Forwarding rule** (Step 11) — Using correct IP
4. **Load balancer** — Created successfully

---

### SSH Fails

Check in this order:

1. **VPC Network** (Step 2) — Subnet has proper routing
2. **Firewall rule** (Step 3) — `prod-allow-ssh` has correct source IP
3. **Network tags** — Instance has `prod-ssh-access` tag
4. **SSH key** — Proper authentication configured

---

## ✅ Complete Production Readiness Checklist

### Network Foundation

- [ ] Region/Zone configured (Step 1)
- [ ] VPC Network created (Step 2) ⭐
- [ ] Subnet configured with IP range (Step 2) ⭐
- [ ] Subnet has available IPs (Step 2) ⭐

### Security

- [ ] Firewall rules created on `prod-vpc-network` (Step 3)
- [ ] Web traffic allowed (Step 3)
- [ ] Health check traffic allowed (Step 3)
- [ ] SSH access restricted (Step 3 — Optional)

### Infrastructure

- [ ] Instance template created (Step 4)
- [ ] Template uses `prod-vpc-network` (Step 4) ⭐
- [ ] Startup script configured (Step 4)
- [ ] Health check configured (Step 5)
- [ ] Managed instance group created (Step 6)
- [ ] MIG uses `prod-vpc-network` (Step 6) ⭐
- [ ] 3 instances provisioned (Step 6)

### Load Balancer

- [ ] Static IP reserved (Step 7)
- [ ] Backend service configured (Step 8)
- [ ] URL map created (Step 9)
- [ ] Target proxy created (Step 10)
- [ ] Forwarding rule active (Step 11)
- [ ] Frontend uses `prod-vpc-network` (Step 10) ⭐

### Verification

- [ ] All instances show **Healthy** status (Step 13)
- [ ] Load balancer IP responds with web page (Step 13)
- [ ] Internal IPs are on `prod-vpc-network` (Step 13) ⭐
- [ ] Multiple zones show up in response (Step 13)
- [ ] Health checks passing (Step 13)

---

## 📝 Important Production Notes

1. **VPC Network** — All resources must be in the same VPC network to communicate
2. **Subnet Sizing** — Plan subnet ranges carefully — they **cannot be changed after creation**
3. **Firewall Rules** — Rules are enforced at the VPC level
4. **Regional Resources** — All resources should be in the same region for optimal performance
5. **Health Checks** — Health checks must be allowed by firewall rules on the VPC
6. **Network Tags** — Instances need proper tags to be affected by firewall rules
7. **Static IP** — Global static IP is required for HTTP/S load balancers

---

## 🎯 Final Architecture Summary

Your complete production environment now includes:

| Component | Resource | Status |
|-----------|----------|--------|
| VPC Network | `prod-vpc-network` with subnet `prod-subnet-us-central1` | ✅ |
| Firewall Rules | Web, health check, and SSH access on the VPC | ✅ |
| Instance Template | Standardized VM configuration using the VPC | ✅ |
| Health Check | Autohealing monitoring | ✅ |
| Managed Instance Group | 3 instances across 3 zones on the VPC | ✅ |
| Static IP | Global IP for load balancer | ✅ |
| Backend Service | Connected to MIG on the VPC | ✅ |
| URL Map | Routing configuration | ✅ |
| Target Proxy | HTTP bridging | ✅ |
| Forwarding Rule | Traffic entry point | ✅ |

**Your production Application Load Balancer is now fully operational with a dedicated VPC network!** 🚀
