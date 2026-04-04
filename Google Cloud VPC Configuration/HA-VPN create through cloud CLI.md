Task 1: Cloud VPC Setup

🔹 Create VPC
                         gcloud compute networks create vpc-demo --subnet-mode custom

🔹 Create Subnets
                        gcloud compute networks subnets create vpc-demo-subnet1 \
                        --network vpc-demo --range 10.1.1.0/24 --region europe-west4
                        gcloud compute networks subnets create vpc-demo-subnet2 \
                        --network vpc-demo --range 10.2.1.0/24 --region us-central1

🔹 Firewall Rules
                       gcloud compute firewall-rules create vpc-demo-allow-internal \
                      --network vpc-demo \
                      --allow tcp:0-65535,udp:0-65535,icmp \
                      --source-ranges 10.0.0.0/8

                      gcloud compute firewall-rules create vpc-demo-allow-ssh-icmp \
                      --network vpc-demo \
                      --allow tcp:22,icmp

🔹 Create VM Instances
gcloud compute instances create vpc-demo-instance1 \
  --zone europe-west4-b --subnet vpc-demo-subnet1 --machine-type e2-medium

gcloud compute instances create vpc-demo-instance2 \
  --zone us-central1-f --subnet vpc-demo-subnet2 --machine-type e2-medium

🏢 Task 2: Simulate On-Prem Setup
🔹 Create VPC
gcloud compute networks create on-prem --subnet-mode custom
🔹 Create Subnet
gcloud compute networks subnets create on-prem-subnet1 \
  --network on-prem --range 192.168.1.0/24 --region europe-west4
🔹 Firewall Rules
gcloud compute firewall-rules create on-prem-allow-internal \
  --network on-prem \
  --allow tcp:0-65535,udp:0-65535,icmp \
  --source-ranges 192.168.0.0/16

gcloud compute firewall-rules create on-prem-allow-ssh-icmp \
  --network on-prem \
  --allow tcp:22,icmp
🔹 Create VM
gcloud compute instances create on-prem-instance1 \
  --zone europe-west4-a --subnet on-prem-subnet1 --machine-type e2-medium

🔐 Task 3: HA-VPN Setup
🔹 Create VPN Gateways
gcloud compute vpn-gateways create vpc-demo-vpn-gw1 \
  --network vpc-demo --region europe-west4

gcloud compute vpn-gateways create on-prem-vpn-gw1 \
  --network on-prem --region europe-west4
🔹 Create Cloud Routers
gcloud compute routers create vpc-demo-router1 \
  --region europe-west4 --network vpc-demo --asn 65001

gcloud compute routers create on-prem-router1 \
  --region europe-west4 --network on-prem --asn 65002
🔹 Create VPN Tunnels

⚠️ Replace [SHARED_SECRET] with your secret key

# VPC side
gcloud compute vpn-tunnels create vpc-demo-tunnel0 \
  --peer-gcp-gateway on-prem-vpn-gw1 \
  --region europe-west4 --ike-version 2 \
  --shared-secret [SHARED_SECRET] \
  --router vpc-demo-router1 \
  --vpn-gateway vpc-demo-vpn-gw1 --interface 0

gcloud compute vpn-tunnels create vpc-demo-tunnel1 \
  --peer-gcp-gateway on-prem-vpn-gw1 \
  --region europe-west4 --ike-version 2 \
  --shared-secret [SHARED_SECRET] \
  --router vpc-demo-router1 \
  --vpn-gateway vpc-demo-vpn-gw1 --interface 1

# On-prem side
gcloud compute vpn-tunnels create on-prem-tunnel0 \
  --peer-gcp-gateway vpc-demo-vpn-gw1 \
  --region europe-west4 --ike-version 2 \
  --shared-secret [SHARED_SECRET] \
  --router on-prem-router1 \
  --vpn-gateway on-prem-vpn-gw1 --interface 0

gcloud compute vpn-tunnels create on-prem-tunnel1 \
  --peer-gcp-gateway vpc-demo-vpn-gw1 \
  --region europe-west4 --ike-version 2 \
  --shared-secret [SHARED_SECRET] \
  --router on-prem-router1 \
  --vpn-gateway on-prem-vpn-gw1 --interface 1
🔹 Configure BGP Peering
# VPC Side
gcloud compute routers add-interface vpc-demo-router1 \
  --interface-name if-tunnel0 \
  --ip-address 169.254.0.1 --mask-length 30 \
  --vpn-tunnel vpc-demo-tunnel0 --region europe-west4

gcloud compute routers add-bgp-peer vpc-demo-router1 \
  --peer-name bgp-tunnel0 \
  --interface if-tunnel0 \
  --peer-ip-address 169.254.0.2 \
  --peer-asn 65002 --region europe-west4

# On-Prem Side
gcloud compute routers add-interface on-prem-router1 \
  --interface-name if-tunnel0 \
  --ip-address 169.254.0.2 --mask-length 30 \
  --vpn-tunnel on-prem-tunnel0 --region europe-west4

gcloud compute routers add-bgp-peer on-prem-router1 \
  --peer-name bgp-tunnel0 \
  --interface if-tunnel0 \
  --peer-ip-address 169.254.0.1 \
  --peer-asn 65001 --region europe-west4
🔹 Firewall Rules for VPN Traffic
gcloud compute firewall-rules create vpc-demo-allow-subnets-from-on-prem \
  --network vpc-demo \
  --allow tcp,udp,icmp \
  --source-ranges 192.168.1.0/24

gcloud compute firewall-rules create on-prem-allow-subnets-from-vpc-demo \
  --network on-prem \
  --allow tcp,udp,icmp \
  --source-ranges 10.1.1.0/24,10.2.1.0/24
🔹 Verify VPN Tunnels
gcloud compute vpn-tunnels list

gcloud compute vpn-tunnels describe vpc-demo-tunnel0 \
  --region europe-west4
🔹 Test Connectivity
gcloud compute ssh on-prem-instance1 --zone europe-west4-a
🔹 Enable Global Routing
gcloud compute networks update vpc-demo --bgp-routing-mode GLOBAL
🔹 Test High Availability
gcloud compute vpn-tunnels delete vpc-demo-tunnel0 --region europe-west4

🧹 Task 4: Cleanup
🔹 Delete Resources
# VPN Tunnels
gcloud compute vpn-tunnels delete on-prem-tunnel0 --region europe-west4

# Routers
gcloud compute routers delete vpc-demo-router1 --region europe-west4

# Gateways
gcloud compute vpn-gateways delete vpc-demo-vpn-gw1 --region europe-west4

# Instances
gcloud compute instances delete vpc-demo-instance1 --zone europe-west4-b

# Firewall Rules
gcloud compute firewall-rules delete vpc-demo-allow-internal

# Subnets
gcloud compute networks subnets delete vpc-demo-subnet1 --region europe-west4

# VPC
gcloud compute networks delete vpc-demo
