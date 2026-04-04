## Overview ##
  - HA-VPN (High Availability VPN) is a highly available IPSec VPN solution in Google Cloud that enables secure connectivity between on-premises environments and VPC networks.

## Today what we will going to learn ##
  - How to configure high availability ha-vpn gateways
  - How to configure dynamic routing with vpn tunnels
  - How to configure global dynamic routing mode
  - How to verify high availability ha-vpn gateways
<img width="1500" height="604" alt="image" src="https://github.com/user-attachments/assets/758c3730-b8f1-491b-897a-40d45f54f0c9" />

## *Task 1. Cloud VPC setup* ##
- Open Cloud Shell, create a vpc network called vpc-demo:
   
      '''
      gcloud compute networks create vpc-demo --subnet-mode custom
      '''
    
    
