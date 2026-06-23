# 🌐 Ultimate Production Network Load Balancer — Master Guide

> **Enterprise-grade GCP Network Load Balancer setup** with security hardening, monitoring, disaster recovery, and full operational documentation.

---

## 🎯 Executive Summary

This guide provides a production-grade, enterprise-level Network Load Balancer setup with best practices, monitoring, security, and disaster recovery. Each step follows a logical ascending order with comprehensive explanations.

---

## 📋 Pre-Setup Checklist

### Prerequisites

- Google Cloud Project with billing enabled
- IAM permissions: **Compute Admin**, **Network Admin**
- Your office IP address (for SSH restrictions)
- Domain name *(optional, for SSL)*
- Understanding of production requirements

### Naming Convention Used

**Format:** `{env}-{service}-{component}-{role}`

| Example | Description |
|---|---|
| `prod-web-app-01` | Production web application instance |
| `prod-nlb-ip` | Production network load balancer IP |
| `prod-health-check` | Production health check |

---

## Step 1: Environment Configuration & Planning

### 1.1 Set Default Region and Zone

> **Why this is first:** Establishes the foundation for all resources, prevents misconfiguration, and ensures data residency compliance.

**Console Steps:**

1. Navigate to **Compute Engine → Settings**
2. Configure:

```
Default Region: us-central1
Default Zone:   us-central1-a
```

3. Click **Save**

---

### 1.2 Enable Required APIs

1. Navigate to **APIs & Services → Library**
2. Enable the following APIs:
   - Compute Engine API
   - Cloud Monitoring API
   - Cloud Logging API
   - Cloud Pub/Sub API *(for alerts)*

---

### 1.3 Create Service Account *(Optional — for automation)*

1. Navigate to **IAM & Admin → Service Accounts**
2. Create a Service Account:
   - **Name:** `prod-nlb-sa`
   - **Roles:** Compute Admin, Monitoring Editor
3. Create and download key *(store securely)*

---

## Step 2: Network & Security Foundation

### 2.1 Create Production Network *(Optional — for isolated network)*

> **Why:** Provides network isolation for production workloads.

**Console Steps:**

1. Navigate to **VPC Network → VPC Networks**
2. Click **Create VPC Network**

```
Name:                   prod-network
Subnet creation mode:   Custom
Subnet:                 prod-subnet-us-central1
Region:                 us-central1
IP range:               10.0.0.0/24
```

---

### 2.2 Create Firewall Rules *(Security First Approach)*

#### 📌 HTTP Rule — Allow Web Traffic

```
Name:              prod-allow-http
Description:       Allow HTTP traffic to production web servers
Network:           default (or prod-network)
Priority:          1000
Direction:         Ingress
Action:            Allow
Target tags:       prod-http-tag
Source IP ranges:  0.0.0.0/0
Protocols/ports:   tcp:80
```

#### 📌 HTTPS Rule — For Future SSL

```
Name:              prod-allow-https
Description:       Allow HTTPS traffic to production web servers
Network:           default (or prod-network)
Priority:          1000
Direction:         Ingress
Action:            Allow
Target tags:       prod-http-tag
Source IP ranges:  0.0.0.0/0
Protocols/ports:   tcp:443
```

#### 📌 SSH Rule — Restricted Access (Production Security)

```
Name:              prod-allow-ssh
Description:       Allow SSH from authorized management IPs
Network:           default (or prod-network)
Priority:          900
Direction:         Ingress
Action:            Allow
Target tags:       prod-http-tag
Source IP ranges:  YOUR_OFFICE_IP/32, VPN_CIDR
Protocols/ports:   tcp:22
```

#### 📌 Health Check Rule — Critical for Load Balancer

```
Name:              prod-allow-health-check
Description:       Allow health check probes from Google
Network:           default (or prod-network)
Priority:          800
Direction:         Ingress
Action:            Allow
Target tags:       prod-http-tag
Source IP ranges:  130.211.0.0/22, 35.191.0.0/16
Protocols/ports:   tcp:80
```

> **Pro Tip:** Create all firewall rules before instances to ensure they can be accessed immediately.

---

## Step 3: Compute Resources (Web Server Instances)

### 3.1 Create Production-Ready Instances with Enhanced Configuration

#### Instance 1: `prod-web-app-01` (Primary Zone)

**Console Steps:**

1. Navigate to **Compute Engine → VM Instances**
2. Click **Create Instance**

**Basic Configuration:**

```
Name:           prod-web-app-01
Region:         us-central1
Zone:           us-central1-a
Machine Type:   e2-standard-2 (2 vCPU, 8GB RAM)
Boot Disk:      Debian 12 (64-bit)
Disk Size:      50 GB
Disk Type:      pd-standard
```

**Advanced Options — Networking:**

```
Network tags:          prod-http-tag, prod-web-server
Hostname:              prod-web-app-01.prod.internal
Network Service Tier:  Premium
```

**Labels (for cost tracking):**

```
environment:  production
application:  web-server
version:      v1.0.0
managed-by:   terraform
cost-center:  prod-ops
```

**Management:**

```
Startup script:      (see below)
Deletion protection: Enabled
Preemptibility:      None (production)
```

**Startup Script — Enhanced with Monitoring:**

```bash
#!/bin/bash
# Production Web Server Setup Script
# Version: 1.0.0

set -e

echo "========================================="
echo "Starting Production Web Server Setup"
echo "Instance: prod-web-app-01"
echo "Zone: us-central1-a"
echo "Timestamp: $(date)"
echo "========================================="

# Update system
apt-get update
apt-get upgrade -y

# Install web server
apt-get install apache2 -y
apt-get install curl wget htop nethogs -y

# Configure Apache
cat > /etc/apache2/conf-available/security.conf << EOF
ServerTokens Prod
ServerSignature Off
TraceEnable Off
EOF
a2enconf security

# Start and enable Apache
systemctl enable apache2
systemctl start apache2

# Create health check endpoint (for advanced monitoring)
mkdir -p /var/www/html/health
cat > /var/www/html/health/check << EOF
OK
EOF

# Create main web page
cat > /var/www/html/index.html << EOF
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Production Server 01</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            margin: 0;
            padding: 0;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
        }
        .container {
            background: rgba(255,255,255,0.1);
            padding: 40px;
            border-radius: 20px;
            backdrop-filter: blur(10px);
            box-shadow: 0 8px 32px 0 rgba(31,38,135,0.37);
            text-align: center;
        }
        h1 { font-size: 2.5em; margin-bottom: 10px; }
        .badge {
            display: inline-block;
            background: #4CAF50;
            padding: 5px 15px;
            border-radius: 20px;
            font-size: 0.9em;
            margin: 10px 0;
        }
        .detail {
            text-align: left;
            background: rgba(0,0,0,0.2);
            padding: 15px;
            border-radius: 10px;
            margin: 20px 0;
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="badge">✅ PRODUCTION</div>
        <h1>🚀 Web Server: prod-web-app-01</h1>
        <div class="detail">
            <p><strong>🔹 Environment:</strong> Production</p>
            <p><strong>🔹 Zone:</strong> us-central1-a</p>
            <p><strong>🔹 Hostname:</strong> $(hostname)</p>
            <p><strong>🔹 Server IP:</strong> $(hostname -I | cut -d' ' -f1)</p>
            <p><strong>🔹 Status:</strong> 🟢 Healthy</p>
            <p><strong>🔹 Version:</strong> v1.0.0</p>
        </div>
    </div>
</body>
</html>
EOF

# Set proper permissions
chown -R www-data:www-data /var/www/html/
chmod -R 755 /var/www/html/

# Configure log rotation
cat > /etc/logrotate.d/apache2-prod << EOF
/var/log/apache2/*.log {
    daily
    missingok
    rotate 7
    compress
    delaycompress
    notifempty
    create 640 www-data adm
    sharedscripts
    postrotate
        systemctl reload apache2
    endscript
}
EOF

echo "========================================="
echo "Production Web Server Setup Complete!"
echo "Instance: prod-web-app-01"
echo "========================================="
```

---

#### Instance 2: `prod-web-app-02`

```
Name:   prod-web-app-02
Zone:   us-central1-b
Note:   Use same configuration — update server name in HTML
```

#### Instance 3: `prod-web-app-03`

```
Name:   prod-web-app-03
Zone:   us-central1-c
Note:   Use same configuration — update server name in HTML
```

---

### 3.2 Create Custom Image *(For Scalability)*

> **Why:** Creating a custom image ensures consistent deployments.

1. Stop one instance: `prod-web-app-01`
2. Navigate to **Compute Engine → Images**
3. Click **Create Image**:
   - **Name:** `prod-web-image-v1`
   - **Source:** `prod-web-app-01`
   - **Family:** `prod-web-server`
4. Click **Create**

---

## Step 4: Network Infrastructure (Load Balancer Components)

### 4.1 Reserve Static External IP

> **Why:** Production needs a fixed IP for DNS and firewall whitelisting.

**Console Steps:**

1. Navigate to **VPC Network → IP Addresses**
2. Click **Reserve External IP Address**

```
Name:        prod-nlb-ip
Description: Production NLB static IP
Region:      us-central1
Type:        Regional
Attached to: None
```

3. Click **Reserve**

> 📝 **Record the IP:** `34.123.45.67` *(example)*

---

### 4.2 Create Internal DNS Entry *(Optional)*

1. Navigate to **Cloud DNS → Zones**
2. Create a DNS entry:
   ```
   prod-app.example.com → prod-nlb-ip
   ```

---

### 4.3 Create Health Check (Advanced Configuration)

**Console Steps:**

1. Navigate to **Compute Engine → Health Checks**
2. Click **Create Health Check**

```
Name:                prod-health-check
Description:         Production health check with advanced monitoring
Protocol:            HTTP
Port:                80
Request path:        /health/check
Check interval:      30 seconds
Timeout:             5 seconds
Unhealthy threshold: 3
Healthy threshold:   2
Logging:             Enabled
```

**Why Advanced Health Check:**

- Custom path `/health/check` ensures application is fully functional
- Logging helps diagnose health issues
- Balanced thresholds prevent flapping

---

## Step 5: Load Balancer Backend Configuration

### 5.1 Create Target Pool

**Console Steps:**

1. Navigate to **Compute Engine → Target Pools**
2. Click **Create Target Pool**

```
Name:             prod-web-pool
Description:      Production web server target pool
Region:           us-central1
Session affinity: NONE
Protocol:         TCP
Load balancing:   External
Health Check:     prod-health-check
```

**Instances — Add all three:**

- `prod-web-app-01` (us-central1-a)
- `prod-web-app-02` (us-central1-b)
- `prod-web-app-03` (us-central1-c)

3. Click **Create**

---

### 5.2 Create Backup Pool *(Disaster Recovery)*

> **Why:** Ensures high availability even if primary pool fails.

1. Navigate to **Compute Engine → Target Pools**
2. Click **Create Target Pool**:
   - **Name:** `prod-web-backup-pool`
   - **Region:** `us-central1`
   - **Session affinity:** NONE
3. Add instances from different zones
4. This pool will be used as fallback

---

## Step 6: Forwarding Rule (Activate Load Balancer)

### 6.1 Create Primary Forwarding Rule

#### Method A: From IP Address

1. Navigate to **VPC Network → IP Addresses**
2. Click on `prod-nlb-ip`
3. Click **Create Forwarding Rule**

```
Name:        prod-nlb-forwarding-rule
Description: Production NLB forwarding rule
Region:      us-central1
IP:          prod-nlb-ip (selected)
Protocol:    TCP
Port:        80
Target:      Target pool
Target Pool: prod-web-pool
```

4. Click **Create**

#### Method B: From Load Balancing

1. Navigate to **Network Services → Load Balancing**
2. Click **Create Load Balancer**
3. Select **Network Load Balancer (TCP/SSL)**
4. Click **Start Configuration**
5. Select **From Internet → Continue**
6. Configure:

```
Name:      prod-nlb
IP Address: prod-nlb-ip
Port:       80
Backend:    prod-web-pool
```

7. Click **Create**

---

### 6.2 Create Backup Forwarding Rule *(Disaster Recovery)*

```
Name:    prod-nlb-backup-rule
IP:      Same static IP
Port:    8080
Target:  prod-web-backup-pool
```

---

## Step 7: Monitoring & Observability

### 7.1 Create Monitoring Dashboard

**Console Steps:**

1. Navigate to **Monitoring → Dashboards**
2. Click **Create Dashboard**
3. **Name:** `Production NLB Dashboard`
4. Add the following widgets:

#### Widget 1: Load Balancer Traffic

```
Metric:     loadbalancing.googleapis.com/https/request_count
Filters:    forwarding_rule_name="prod-nlb-forwarding-rule"
Chart type: Line chart
```

#### Widget 2: Instance CPU

```
Metric:     compute.googleapis.com/instance/cpu/utilization
Filters:    instance_name=~"prod-web-app-.*"
Chart type: Line chart
```

#### Widget 3: Instance Health

```
Metric:     compute.googleapis.com/instance/health_check/status
Filters:    instance_name=~"prod-web-app-.*"
Chart type: Table
```

#### Widget 4: HTTP Error Rates

```
Metric:     loadbalancing.googleapis.com/https/response_code_count
Filters:    response_code=~"5.*"
Chart type: Pie chart
```

---

### 7.2 Set Up Alerting Policies

#### Alert 1: High CPU Usage

```
Condition:     CPU > 80% for 5 minutes
Severity:      Critical
Notification:  Email, Slack, PagerDuty
```

#### Alert 2: Instance Unhealthy

```
Condition:     Health check status = UNHEALTHY
Severity:      Critical
Notification:  Email, Slack, PagerDuty
```

#### Alert 3: High Error Rate

```
Condition:     5xx errors > 5% for 5 minutes
Severity:      Warning
Notification:  Email, Slack
```

**Console Steps to Create Alerts:**

1. Navigate to **Monitoring → Alerting**
2. Click **Create Policy**
3. Configure conditions and notification channels

---

## Step 8: Logging & Audit

### 8.1 Enable Cloud Logging

**Console Steps:**

1. Navigate to **Logging → Logs Explorer**
2. Create the following saved queries:

#### Query 1: Apache Access Logs

```sql
resource.type="gce_instance"
resource.labels.instance_id=~"prod-web-app-.*"
logName="projects/PROJECT_ID/logs/apache2-access.log"
```

#### Query 2: Error Logs

```sql
resource.type="gce_instance"
resource.labels.instance_id=~"prod-web-app-.*"
severity="ERROR"
```

#### Query 3: Load Balancer Logs

```sql
resource.type="http_load_balancer"
resource.labels.forwarding_rule_name="prod-nlb-forwarding-rule"
```

---

### 8.2 Create Log Sinks *(For Compliance)*

1. Navigate to **Logging → Logs Router**
2. Create Sink:
   - **Name:** `prod-log-sink`
   - **Destination:** Cloud Storage bucket *(30-day retention)*
   - **Filter:** `resource.type="gce_instance" OR resource.type="http_load_balancer"`

---

## Step 9: Verification & Testing

### 9.1 Health Check Verification

**Console:**

1. Navigate to **Compute Engine → Health Checks**
2. Click `prod-health-check`
3. Verify all instances show ✅ **Healthy** status

---

### 9.2 Instance Accessibility Test

From **Cloud Shell:**

```bash
# Test individual instances
for i in 01 02 03; do
  IP=$(gcloud compute instances describe prod-web-app-$i \
    --zone=us-central1-a --format="get(networkInterfaces[0].accessConfigs[0].natIP)")
  echo "Testing prod-web-app-$i: $IP"
  curl -s http://$IP | grep -i "web server"
done
```

---

### 9.3 Load Balancer Test

```bash
# Get Load Balancer IP
LB_IP=$(gcloud compute addresses describe prod-nlb-ip \
  --region=us-central1 --format="get(address)")

# Test load balancing (20 requests)
echo "Testing Load Balancer at $LB_IP"
for i in {1..20}; do
  echo "Request $i:"
  curl -s http://$LB_IP | grep "Web Server:"
  sleep 0.5
done
```

---

### 9.4 Load Testing *(Optional — for production validation)*

```bash
# Install Apache Bench
sudo apt-get install apache2-utils -y

# Run load test (1000 requests, 100 concurrent)
ab -n 1000 -c 100 http://$LB_IP/
```

---

### 9.5 Browser-Based Verification

1. Open browser: `http://[LB_IP]`
2. Press **F12 → Network Tab**
3. Refresh **10 times**
4. Verify requests are distributed across instances
5. Check response headers

---

## Step 10: Production Security Hardening

### 10.1 Enable VPC Service Controls

1. Navigate to **VPC Service Controls**
2. Create Perimeter: `prod-perimeter`
3. Add services: Compute Engine, Cloud Storage
4. Add resources: All production resources

---

### 10.2 Set Up IAM Roles

1. Navigate to **IAM & Admin → IAM**
2. Grant minimum required permissions:

| Role | Permission Level |
|---|---|
| Developers | Compute Viewer only |
| Ops Team | Compute Admin, Monitoring Editor |
| Auditors | Logging Viewer only |

---

### 10.3 Enable Audit Logging

1. Navigate to **IAM & Admin → Audit Logs**
2. Enable audit logs for:
   - Compute Engine
   - Cloud Storage
   - Cloud Monitoring
   - Cloud Logging

---

## Step 11: Disaster Recovery Setup

### 11.1 Create Instance Templates

1. Navigate to **Compute Engine → Instance Templates**
2. Create Template:

```
Name:       prod-web-template-v1
Machine:    e2-standard-2
Boot Disk:  Debian 12 (custom image)
Labels:     environment=production
```

---

### 11.2 Create Managed Instance Group *(For Auto-healing)*

1. Navigate to **Compute Engine → Instance Groups**
2. Create Instance Group:

```
Name:         prod-web-mig
Type:         Managed
Template:     prod-web-template-v1
Zones:        us-central1-a, us-central1-b, us-central1-c
Autoscaling:  Enabled
Health Check: prod-health-check
```

---

### 11.3 Set Up Backup Script

```bash
#!/bin/bash
# prod-backup-script.sh

# Backup instance configurations
gcloud compute instances list --filter="tags.items=prod-http-tag" \
  --format="json" > prod-instances-backup-$(date +%Y%m%d).json

# Backup firewall rules
gcloud compute firewall-rules list --format="json" > prod-firewall-backup-$(date +%Y%m%d).json

# Backup target pool
gcloud compute target-pools describe prod-web-pool \
  --region=us-central1 --format="json" > prod-targetpool-backup-$(date +%Y%m%d).json
```

---

## Step 12: Documentation & Handover

### 12.1 Create Architecture Diagram

**Components to Document:**

```
Internet
  └── Static IP (prod-nlb-ip)
        └── Forwarding Rule (prod-nlb-forwarding-rule)
              └── Target Pool (prod-web-pool)
                    ├── prod-web-app-01 (us-central1-a)
                    ├── prod-web-app-02 (us-central1-b)
                    └── prod-web-app-03 (us-central1-c)

Monitoring Stack
  ├── Cloud Monitoring Dashboard
  ├── Alerting Policies
  └── Cloud Logging + Log Sinks
```

---

### 12.2 Create Runbooks

#### Runbook: Server Failure

```
1. Instance goes down
2. Monitoring alerts Ops team
3. Ops checks health check
4. If unhealthy:
   a. SSH into instance
   b. Check service: systemctl status apache2
   c. Restart:       systemctl restart apache2
   d. Verify health: curl localhost/health/check
5. If persistent:
   a. Create new instance from template
   b. Add to target pool
   c. Remove failed instance
```

---

### 12.3 Create Handover Document

```markdown
# Production NLB — Handover Document

## Infrastructure Overview
- Environment:          Production
- Region:               us-central1
- Load Balancer Type:   Network Load Balancer (L4)
- Number of Backends:   3

## Access Points
- Production URL:  http://34.123.45.67
- DNS Name:        prod-app.example.com

## Credentials
- Cloud Console:  [Provide IAM access]
- SSH Access:     Restricted to VPN IPs

## Monitoring
- Dashboard:  Production NLB Dashboard
- Alerts:     Email to ops@example.com

## Maintenance Windows
- Weekly:     Sunday 2AM – 4AM EST
- Emergency:  24/7 on-call rotation

## Support Contacts
- Primary:    John Doe (Ops Lead)
- Secondary:  Jane Smith (Backup)
- PagerDuty:  prod-nlb-oncall

## Links
- Console:     https://console.cloud.google.com
- Monitoring:  https://console.cloud.google.com/monitoring
- Logs:        https://console.cloud.google.com/logs
```

---

## ✅ Ultimate Production Checklist

### Phase 1: Foundation *(Steps 1–2)*

- [ ] Region/Zone configured
- [ ] APIs enabled
- [ ] Firewall rules created (HTTP, HTTPS, SSH, Health Check)
- [ ] Network isolation configured

### Phase 2: Compute *(Step 3)*

- [ ] All 3 instances created with enhanced startup scripts
- [ ] Custom image created
- [ ] Deletion protection enabled
- [ ] Labels and tags applied

### Phase 3: Network *(Steps 4–5)*

- [ ] Static IP reserved and documented
- [ ] Health check configured
- [ ] Target pool created
- [ ] All instances added to pool

### Phase 4: Activation *(Step 6)*

- [ ] Forwarding rule created
- [ ] Load balancer tested
- [ ] DNS configured *(if applicable)*

### Phase 5: Observability *(Steps 7–8)*

- [ ] Monitoring dashboard created
- [ ] Alerting policies configured
- [ ] Logging enabled
- [ ] Audit logs active

### Phase 6: Security *(Step 10)*

- [ ] VPC Service Controls enabled
- [ ] IAM roles restricted
- [ ] SSH access restricted
- [ ] Audit logging enabled

### Phase 7: Resilience *(Step 11)*

- [ ] Backup pool configured
- [ ] Instance templates created
- [ ] Disaster recovery plan documented
- [ ] Backup scripts in place

### Phase 8: Documentation *(Step 12)*

- [ ] Architecture diagram created
- [ ] Runbooks written
- [ ] Handover document prepared
- [ ] Knowledge base updated

---

## 💰 Cost Optimization Tips

### 1. Right-Size Instances

- Monitor CPU/Memory usage for 30 days
- Adjust machine types if over-provisioned
- Use committed use discounts for 1–3 year commitments

### 2. Network Optimization

- Use standard tier where latency isn't critical
- Enable CDN for static content
- Compress responses

### 3. Monitoring Cost Management

- Set log retention to 30 days (not indefinite)
- Limit metrics sampling rate
- Use metrics filters to reduce ingestion

---

## 🔧 Advanced Production Features *(Optional)*

### SSL/TLS Termination

```bash
# Create SSL certificate
gcloud compute ssl-certificates create prod-ssl-cert \
  --domains=prod-app.example.com \
  --global

# Update load balancer to use HTTPS
gcloud compute target-pools create prod-https-pool \
  --region=us-central1 \
  --http-health-check=prod-health-check
```

### WAF Integration (Cloud Armor)

```bash
# Create security policy
gcloud compute security-policies create prod-waf-policy

# Attach to load balancer
gcloud compute backend-services update prod-web-pool \
  --security-policy=prod-waf-policy
```

### Auto-scaling Configuration

```bash
# Create autoscaler
gcloud compute instance-groups managed set-autoscaling \
  prod-web-mig \
  --max-num-replicas=10 \
  --min-num-replicas=3 \
  --target-cpu-utilization=0.75 \
  --cool-down-period=60
```

---

## 🎉 Congratulations!

You've successfully set up a production-grade Network Load Balancer with:

- ✅ Enterprise-level security
- ✅ Comprehensive monitoring
- ✅ Disaster recovery capabilities
- ✅ Detailed documentation
- ✅ Production best practices

Your production environment is now **robust, scalable, and ready to handle real-world traffic!**

---

> ⚠️ *This guide is AI-generated, for reference only. Always validate configurations against your organization's specific requirements and GCP's latest documentation.*
