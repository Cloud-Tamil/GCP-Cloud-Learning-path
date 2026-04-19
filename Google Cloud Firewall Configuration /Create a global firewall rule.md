# Setup and requirements
- Set your region and zone
   - Certain Compute Engine resources live in regions and zones. A region is a specific geographical location where you can run your resources. Each region has one or more zones.
     
- Run the following gcloud commands in Cloud Shell to set the default region&Zone for your lab       
```        
gcloud config set compute/region "us-central1"
export REGION=$(gcloud config get compute/region)

gcloud config set compute/zone "us-central1-f"
export ZONE=$(gcloud config get compute/zone)
```
- Set your Project ID as an environment variable.
```
export PROJECT_ID=$(gcloud config get-value project)
```
- List the networks in your project.
```
gcloud compute networks list
```
**Identify the number of existing VPC firewall rules in your network**
- Go to Navigation Menu → VPC Network
- Review the list of available networks
- Click on **external-network**
- Navigate to the Firewalls tab
- Expand each firewall rule to view.
- List the networks in your project.
```
Using CLI mode
gcloud compute firewall-rules list --filter=network:external-network
```
        
## Task 1. Create a global firewall rule

- Global network firewall policy rules must be created in a global network firewall policy. The rules are not active until you associate the policy that contains those rules with a VPC network.
- Each global network firewall policy rule can include either IPv4 or IPv6 ranges, but not both.

   - Create firewall rules
   - That will later be migrated or equivalent to NGFW
   - Creating baseline rules (tag-based)

## Step-by-Step
Step 1: Create SSH Rule
```
gcloud compute firewall-rules create allow-ssh \
  --direction=INGRESS \
  --action=ALLOW \
  --rules=tcp:22 \
  --source-ranges=10.1.0.0/24,10.2.0.0/24 \
  --description="allow-ssh" \
  --network=external-network \
  --target-tags=ssh
```
### *Explanation* 
- **Protocol/Port:** SSH (TCP 22)  
- **Allowed Source Ranges:**  
  - 10.1.0.0/24  
  - 10.2.0.0/24  
- **Target:**  
  - VMs with tag `ssh`
---

Step 2: Create Web Rule
```
gcloud compute firewall-rules create allow-web \
  --allow tcp:80,tcp:443 \
  --description="allow-web" \
  --source-ranges=10.0.0.0/16 \
  --network=external-network \
  --target-tags=web
```
### *Explanation*  
- **Protocol/Port:** Allows
  - HTTP (80)
  - HTTPS (443)
- **Allowed Source Ranges:**  
  - 10.0.0.0/16
- **Target:**  
  - VMs with tag = web
---

Step 3: Verify Rules
```
gcloud compute firewall-rules list
```
