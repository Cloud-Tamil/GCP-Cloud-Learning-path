## Task 1. Create a global firewall rule

- Global network firewall policy rules must be created in a global network firewall policy. The rules are not active until you associate the policy that contains those rules with a VPC network.
- Each global network firewall policy rule can include either IPv4 or IPv6 ranges, but not both.

   - Create firewall rules
   - That will later be migrated or equivalent to NGFW
   - Creating baseline rules (tag-based)

# *Step-by-Step* 
Step 1: Create SSH Rule
  
```bash
gcloud compute firewall-rules create allow-ssh \
  --direction=INGRESS \
  --action=ALLOW \
  --rules=tcp:22 \
  --source-ranges=10.1.0.0/24,10.2.0.0/24 \
  --description="allow-ssh" \
  --network=external-network \
  --target-tags=ssh

# What this does:
Allows SSH (port 22)
Only from:
10.1.0.0/24
10.2.0.0/24
Applies to:
VMs with tag = ssh #
