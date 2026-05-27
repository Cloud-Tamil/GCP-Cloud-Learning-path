# ☁️ CloudMart Production Network Setup

## Full CLI Automation Script (GCP)

```bash
#!/bin/bash

# =========================================================
# ☁️ CloudMart Production Network Setup - Full Automation
# =========================================================

# =========================================================
# PART 1 — Configure Region & Zone
# =========================================================

echo "Configuring default region and zone..."

gcloud config set compute/region us-central1
gcloud config set compute/zone us-central1-a

# =========================================================
# PART 2 — Create Custom VPC Network
# =========================================================

echo "Creating VPC network..."

gcloud compute networks create cloudmart-prod-network \
    --subnet-mode=custom

# =========================================================
# PART 3 — Create Subnets
# =========================================================

echo "Creating Web subnet..."

gcloud compute networks subnets create cloudmart-web-subnet \
    --network=cloudmart-prod-network \
    --region=us-central1 \
    --range=10.10.0.0/24

echo "Creating Application subnet..."

gcloud compute networks subnets create cloudmart-app-subnet \
    --network=cloudmart-prod-network \
    --region=us-central1 \
    --range=10.10.1.0/24 \
    --enable-private-ip-google-access

echo "Creating Database subnet..."

gcloud compute networks subnets create cloudmart-db-subnet \
    --network=cloudmart-prod-network \
    --region=us-central1 \
    --range=10.10.2.0/24 \
    --enable-private-ip-google-access

# =========================================================
# PART 4 — Create Firewall Rules
# =========================================================

echo "Creating HTTP firewall rule..."

gcloud compute firewall-rules create cloudmart-allow-http-web \
    --network=cloudmart-prod-network \
    --direction=INGRESS \
    --action=ALLOW \
    --rules=tcp:80 \
    --source-ranges=0.0.0.0/0 \
    --target-tags=cloudmart-web

echo "Creating HTTPS firewall rule..."

gcloud compute firewall-rules create cloudmart-allow-https-web \
    --network=cloudmart-prod-network \
    --direction=INGRESS \
    --action=ALLOW \
    --rules=tcp:443 \
    --source-ranges=0.0.0.0/0 \
    --target-tags=cloudmart-web

echo "Creating Web-to-App firewall rule..."

gcloud compute firewall-rules create cloudmart-allow-web-to-app \
    --network=cloudmart-prod-network \
    --direction=INGRESS \
    --action=ALLOW \
    --rules=tcp:8080,tcp:8443 \
    --source-ranges=10.10.0.0/24 \
    --target-tags=cloudmart-app

echo "Creating App-to-DB firewall rule..."

gcloud compute firewall-rules create cloudmart-allow-app-to-db \
    --network=cloudmart-prod-network \
    --direction=INGRESS \
    --action=ALLOW \
    --rules=tcp:3306,tcp:5432 \
    --source-ranges=10.10.1.0/24 \
    --target-tags=cloudmart-db

echo "Creating SSH firewall rule..."

# Replace YOUR_PUBLIC_IP with your actual public IP
gcloud compute firewall-rules create cloudmart-allow-ssh-corp \
    --network=cloudmart-prod-network \
    --direction=INGRESS \
    --action=ALLOW \
    --rules=tcp:22 \
    --source-ranges=YOUR_PUBLIC_IP/32 \
    --target-tags=cloudmart-ssh

echo "Creating Health Check firewall rule..."

gcloud compute firewall-rules create cloudmart-allow-health-checks \
    --network=cloudmart-prod-network \
    --direction=INGRESS \
    --action=ALLOW \
    --rules=tcp:80,tcp:8080 \
    --source-ranges=130.211.0.0/22,35.191.0.0/16 \
    --target-tags=cloudmart-web,cloudmart-app

echo "Creating Default Deny firewall rule..."

gcloud compute firewall-rules create cloudmart-deny-all \
    --network=cloudmart-prod-network \
    --direction=INGRESS \
    --action=DENY \
    --rules=all \
    --source-ranges=0.0.0.0/0 \
    --priority=65534

# =========================================================
# PART 5 — Create Web Server VM
# =========================================================

echo "Creating Web Server VM..."

gcloud compute instances create cloudmart-web-vm \
    --zone=us-central1-a \
    --machine-type=e2-micro \
    --subnet=cloudmart-web-subnet \
    --tags=cloudmart-web,cloudmart-ssh \
    --image-family=debian-12 \
    --image-project=debian-cloud \
    --metadata=startup-script='#!/bin/bash
apt-get update
apt-get install -y nginx
systemctl enable nginx
systemctl start nginx
echo "<h1>CloudMart Web Server - Production</h1>" > /var/www/html/index.html'

# =========================================================
# PART 6 — Create Application Server VM
# =========================================================

echo "Creating Application Server VM..."

gcloud compute instances create cloudmart-app-vm \
    --zone=us-central1-a \
    --machine-type=e2-micro \
    --subnet=cloudmart-app-subnet \
    --no-address \
    --tags=cloudmart-app \
    --image-family=debian-12 \
    --image-project=debian-cloud \
    --metadata=startup-script='#!/bin/bash
apt-get update
apt-get install -y python3'

# =========================================================
# PART 7 — Create Database Server VM
# =========================================================

echo "Creating Database Server VM..."

gcloud compute instances create cloudmart-db-vm \
    --zone=us-central1-a \
    --machine-type=e2-micro \
    --subnet=cloudmart-db-subnet \
    --no-address \
    --tags=cloudmart-db \
    --image-family=debian-12 \
    --image-project=debian-cloud \
    --metadata=startup-script='#!/bin/bash
apt-get update
apt-get install -y mysql-server'

# =========================================================
# PART 8 — Verify VM Instances
# =========================================================

echo "Listing all VM instances..."

gcloud compute instances list

# =========================================================
# PART 9 — SSH Access
# =========================================================

echo "To connect via SSH run:"
echo "gcloud compute ssh cloudmart-web-vm --zone=us-central1-a"

# =========================================================
# PART 10 — Connectivity Testing
# =========================================================

echo ""
echo "Inside Web VM:"
echo "Ping Application Server Internal IP"
echo "Example:"
echo "ping 10.10.1.X"

echo ""
echo "Ping Database Server Internal IP"
echo "Expected Result: Request timeout / failed"

# =========================================================
# ✅ CloudMart Three-Tier Architecture Completed
# =========================================================

echo ""
echo "CloudMart Production Environment Setup Completed Successfully."
```

---

## 🚀 Execute the Script

Save the script:

```bash
nano cloudmart-setup.sh
```

Paste the script and save.

Make executable:

```bash
chmod +x cloudmart-setup.sh
```

Run the script:

```bash
./cloudmart-setup.sh
```

---

## ✅ Expected Outcome

- Custom VPC created
- Three subnets configured
- Firewall rules configured
- Web VM created with NGINX
- Application VM created
- Database VM created
- Three-tier architecture deployed successfully
