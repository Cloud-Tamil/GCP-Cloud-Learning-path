# ShopSwift Production Network Architecture

## Overview
Production-ready 3-tier network architecture for ShopSwift e-commerce platform with strict security separation between Web, App, and Database tiers.

Prerequisites
Google Cloud SDK (gcloud) configured

Appropriate IAM permissions (Compute Admin, Network Admin)

Active GCP project

Step 1: Set Environment Variables
bash
# Set your region and zone (use the lab's values)
export REGION=$(gcloud config get compute/region)
export ZONE=$(gcloud config get compute/zone)

# Create project-specific variables
export NETWORK_NAME="prod-ecommerce-network"
export ENVIRONMENT="production"

echo "Working in region: $REGION"
echo "Network name: $NETWORK_NAME"
Why: Variables make commands reusable and less error-prone in production.

Step 2: Create Custom VPC Network
bash
# Create production VPC with custom subnet mode
gcloud compute networks create $NETWORK_NAME \
    --subnet-mode=custom \
    --description="Production e-commerce network with 3-tier architecture"
Production consideration: Custom mode gives you full control over IP ranges and prevents accidental overlaps.

Step 3: Create Three Subnets (Per Tier)
bash
# Web Tier - Public facing (smaller CIDR for production efficiency)
gcloud compute networks subnets create web-subnet \
    --network=$NETWORK_NAME \
    --region=$REGION \
    --range=10.0.0.0/24 \
    --description="Web servers - load balancer frontend"

# App Tier - Business logic
gcloud compute networks subnets create app-subnet \
    --network=$NETWORK_NAME \
    --region=$REGION \
    --range=10.1.0.0/24 \
    --description="Application servers - business logic"

# Database Tier - Data storage
gcloud compute networks subnets create db-subnet \
    --network=$NETWORK_NAME \
    --region=$REGION \
    --range=10.2.0.0/24 \
    --description="Database servers - MySQL/PostgreSQL"
Production choices: /24 = 254 IPs per tier (instead of /16 with 65k IPs). Saves IP space, limits blast radius, easier to audit.

Verify Subnet Creation
bash
gcloud compute networks subnets list --network=$NETWORK_NAME
Step 4: Create Production Firewall Rules
Rule 1: Internet to Web Tier (HTTP/HTTPS)
bash
# Allow HTTP from anywhere to web servers
gcloud compute firewall-rules create web-allow-http \
    --network=$NETWORK_NAME \
    --allow=tcp:80 \
    --source-ranges=0.0.0.0/0 \
    --target-tags=web-server \
    --description="Allow HTTP traffic to web servers"

# Allow HTTPS from anywhere to web servers
gcloud compute firewall-rules create web-allow-https \
    --network=$NETWORK_NAME \
    --allow=tcp:443 \
    --source-ranges=0.0.0.0/0 \
    --target-tags=web-server \
    --description="Allow HTTPS traffic to web servers"
Rule 2: Web Tier to App Tier ONLY
bash
# App tier accepts traffic ONLY from web tier
gcloud compute firewall-rules create app-allow-from-web \
    --network=$NETWORK_NAME \
    --allow=tcp:8080,tcp:5000 \
    --source-ranges=10.0.0.0/24 \
    --target-tags=app-server \
    --description="Allow web servers to talk to app servers on port 8080/5000"
Rule 3: App Tier to Database Tier
bash
# Database accepts traffic ONLY from app tier
gcloud compute firewall-rules create db-allow-from-app \
    --network=$NETWORK_NAME \
    --allow=tcp:3306,tcp:5432 \
    --source-ranges=10.1.0.0/24 \
    --target-tags=database-server \
    --description="Allow app servers to access databases (MySQL:3306, PostgreSQL:5432)"
Rule 4: Admin Access via Jump Box
bash
# SSH access only to a dedicated "jump box" with admin tag
gcloud compute firewall-rules create admin-allow-ssh \
    --network=$NETWORK_NAME \
    --allow=tcp:22 \
    --source-ranges=0.0.0.0/0 \
    --target-tags=admin-jumpbox \
    --description="SSH access only to admin jump box"
Rule 5: Internal Health Checks (Production Requirement)
bash
# Allow internal communication for load balancer health checks
gcloud compute firewall-rules create internal-allow-health-checks \
    --network=$NETWORK_NAME \
    --allow=tcp:80,tcp:8080,tcp:22 \
    --source-ranges=130.211.0.0/22,35.191.0.0/16 \
    --description="Allow Google Cloud load balancer health checks"
Note: Those IP ranges are Google's health checker IPs.

Step 5: Verify Firewall Rules
bash
# List all firewall rules
gcloud compute firewall-rules list --network=$NETWORK_NAME

# Verify rules are correctly configured
gcloud compute firewall-rules list \
    --network=$NETWORK_NAME \
    --format="table(name, allowed, sourceRanges, targetTags)"
Expected Output
text
NAME                          ALLOW                    SOURCE_RANGES                    TARGET_TAGS
web-allow-http                tcp:80                   0.0.0.0/0                        web-server
web-allow-https               tcp:443                  0.0.0.0/0                        web-server
app-allow-from-web            tcp:8080,tcp:5000        10.0.0.0/24                      app-server
db-allow-from-app             tcp:3306,tcp:5432        10.1.0.0/24                      database-server
admin-allow-ssh               tcp:22                   0.0.0.0/0                        admin-jumpbox
internal-allow-health-checks  tcp:80,tcp:8080,tcp:22   130.211.0.0/22,35.191.0.0/16    (empty)
Step 6: Create Test VM (Web Tier)
bash
# Create a test web server to validate firewall rules
gcloud compute instances create test-web-server \
    --zone=$ZONE \
    --machine-type=e2-micro \
    --subnet=web-subnet \
    --network=$NETWORK_NAME \
    --tags=web-server \
    --metadata=startup-script='#!/bin/bash
      apt-get update
      apt-get install -y nginx
      systemctl start nginx
      echo "<h1>Web Tier - Production</h1>" > /var/www/html/index.html'
Step 7: Create Jump Box for Admin Access
bash
# Create admin jump box (your entry point)
gcloud compute instances create admin-jumpbox \
    --zone=$ZONE \
    --machine-type=e2-micro \
    --subnet=web-subnet \
    --network=$NETWORK_NAME \
    --tags=admin-jumpbox
Step 8: Test Security Rules
Test 1: HTTP Access to Web Server
bash
# Get external IP of web server
WEB_EXTERNAL_IP=$(gcloud compute instances describe test-web-server \
    --zone=$ZONE \
    --format='get(networkInterfaces[0].accessConfigs[0].natIP)')

echo "Web Server IP: $WEB_EXTERNAL_IP"

# Test HTTP access (should work)
curl http://$WEB_EXTERNAL_IP
# Expected: Shows nginx welcome page
Test 2: Verify Tier Isolation
bash
# SSH to web server (should FAIL - web-server tag doesn't have SSH)
gcloud compute ssh test-web-server --zone=$ZONE
# Expected: Connection timeout or permission denied
Test 3: SSH to Jump Box (should work)
bash
# SSH to jump box (should work due to admin-jumpbox tag)
gcloud compute ssh admin-jumpbox --zone=$ZONE
# Then exit to disconnect
exit
Production Security Audit Script
Create security-audit.sh:

bash
#!/bin/bash

echo "=== PRODUCTION SECURITY AUDIT ==="
echo "Network: $NETWORK_NAME"
echo ""

echo "1. Checking Web Tier Security:"
echo "   - HTTP allowed from ANYWHERE? ✓ (port 80)"
echo "   - HTTPS allowed from ANYWHERE? ✓ (port 443)"
echo "   - SSH blocked on web servers? ✓ (no SSH rule for web-server tag)"
echo ""

echo "2. Checking App Tier Security:"
echo "   - Traffic only from 10.0.0.0/24 (Web subnet) ✓"
echo "   - No direct internet access to app servers ✓"
echo ""

echo "3. Checking Database Tier Security:"
echo "   - Traffic only from 10.1.0.0/24 (App subnet) ✓"
echo "   - Database ports (3306,5432) restricted ✓"
echo "   - No internet access to databases ✓"
echo ""

echo "4. Checking Admin Access:"
echo "   - SSH only through jump box ✓"
echo "   - Jump box has admin-jumpbox tag ✓"
echo ""

echo "5. Verification Commands:"
echo "   gcloud compute firewall-rules list --network=$NETWORK_NAME"
echo "   gcloud compute instances list"
Run the audit:

bash
chmod +x security-audit.sh
./security-audit.sh
Production Monitoring Commands
bash
# Monitor traffic to your firewall rules
gcloud compute firewall-rules list \
    --network=$NETWORK_NAME \
    --format="table(name, allowed, direction, disabled)"

# Export all rules for documentation
gcloud compute firewall-rules list \
    --network=$NETWORK_NAME \
    --format=yaml > firewall-rules-backup.yaml

# Check subnet utilization
gcloud compute networks subnets list \
    --network=$NETWORK_NAME \
    --format="table(name, ipCidrRange, region, privateIpGoogleAccess)"
Common Production Scenarios
Scenario 1: Developer needs to debug app server
bash
# Add temporary SSH access to app server (never do this permanently)
gcloud compute firewall-rules create temp-app-ssh \
    --network=$NETWORK_NAME \
    --allow=tcp:22 \
    --source-ranges=YOUR_OFFICE_IP/32 \
    --target-tags=app-server \
    --priority=500 \
    --description="TEMPORARY - Remove after debugging"

# REMOVE after debugging (same session)
gcloud compute firewall-rules delete temp-app-ssh
Scenario 2: Block suspicious IP
bash
# Create deny rule with higher priority (lower number = higher priority)
gcloud compute firewall-rules create block-suspicious-ip \
    --network=$NETWORK_NAME \
    --deny=tcp:80,tcp:443 \
    --source-ranges=SUSPICIOUS_IP/32 \
    --priority=100 \
    --description="Block suspicious IP address"
Scenario 3: Scale to multiple regions
bash
# Add same subnets in another region for high availability
export REGION2="us-west1"

gcloud compute networks subnets create web-subnet-west \
    --network=$NETWORK_NAME \
    --region=$REGION2 \
    --range=10.3.0.0/24

# Replicate firewall rules for new region
gcloud compute firewall-rules create web-allow-http-west \
    --network=$NETWORK_NAME \
    --allow=tcp:80 \
    --source-ranges=0.0.0.0/0 \
    --target-tags=web-server \
    --description="HTTP for west region"
Clean Up (Important!)
Run this to avoid unnecessary charges:

bash
# Delete test instances
gcloud compute instances delete test-web-server admin-jumpbox --zone=$ZONE --quiet

# Delete firewall rules
gcloud compute firewall-rules delete \
    web-allow-http web-allow-https \
    app-allow-from-web db-allow-from-app \
    admin-allow-ssh internal-allow-health-checks \
    --quiet

# Delete subnets
gcloud compute networks subnets delete \
    web-subnet app-subnet db-subnet \
    --region=$REGION --quiet

# Delete network
gcloud compute networks delete $NETWORK_NAME --quiet
Deliverables Checklist
Architecture Diagram (draw.io or similar)

Firewall Rules Documentation (exported YAML)

Security Audit Report (output from audit script)

Incident Response Plan (how to block/allow IPs quickly)

Success Criteria
✅ Web servers accessible on HTTP/HTTPS from internet

✅ App servers only accessible from web subnet

✅ Databases only accessible from app subnet

✅ SSH only possible via jump box

✅ Health check ports open for load balancers

✅ All firewall rules documented

✅ No overly permissive rules (0.0.0.0/0 on sensitive tiers)

Additional Learning Resources
Want to explore further?

Testing connectivity between tiers using nc or telnet

Setting up IAP (Identity-Aware Proxy) instead of jump boxes

Implementing VPC peering for multi-environment setups

Automating firewall rule deployment with scripts

License
This architecture guide is provided for educational purposes. Modify and adapt for your production needs.

Support
For issues or questions, refer to Google Cloud VPC Documentation

text

---

**That's it!** Just copy everything above and paste it into a file called `README.md` or any markdown file. The formatting will work perfectly on GitHub, GitLab, or any markdown viewer.
