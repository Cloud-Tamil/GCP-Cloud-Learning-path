# 🌐 Google Cloud VPC Networking - Step-by-Step Guide

## 📘 Overview

This lab teaches you how to:

* Explore the default VPC network
* Create an auto mode network with firewall rules
* Convert an auto mode network to a custom mode network
* Create custom mode VPC networks with firewall rules
* Create VM instances using Compute Engine
* Explore the connectivity for VM instances across VPC networks
  
---

# 🧠 Core Concepts

| Google Cloud Concept | Simple Meaning                              |
| -------------------- | ------------------------------------------- |
| VPC Network          | Virtual private network inside Google Cloud |
| Subnet               | Smaller network inside a VPC                |
| Firewall Rule        | Controls allowed/blocked traffic            |
| VM Instance          | Virtual machine/server                      |
| Internal IP          | Private communication address               |
| External IP          | Public internet address                     |

---

---

# 🚀 Task 1 - Explore the Default Network

## Step 1 - Open VPC Networks

Navigate to:

```text
Navigation Menu
→ VPC Network
→ VPC Networks
```

You will see:

```text
default
```

---

# 🔍 Explore Subnets

Open:

```text
default → Subnets
```

You will notice:

* One subnet per region
* Each subnet has its own IP range

Example:

```text
10.128.0.0/20
```

---

# 🔍 Explore Routes

Navigate to:

```text
VPC Network → Routes
```

Select:

```text
Network = default
```

Important route:

```text
0.0.0.0/0
```

Meaning:

* Internet traffic is allowed

---

# 🔍 Explore Firewall Rules

Navigate to:

```text
VPC Network → Firewall
```

Default firewall rules:

| Rule                   | Purpose                      |
| ---------------------- | ---------------------------- |
| default-allow-ssh      | Allow SSH                    |
| default-allow-rdp      | Allow Windows RDP            |
| default-allow-icmp     | Allow ping                   |
| default-allow-internal | Allow internal communication |

---

# 🗑 Delete Default Firewall Rules

Navigate to:

```text
VPC Network → Firewall policies
```

Delete all:

```text
default-* rules
```

---

# 🗑 Delete Default Network

Navigate to:

```text
VPC Network → VPC Networks
```

Delete:

```text
default
```

Type:

```text
default
```

to confirm deletion.

---

# ⚠ Important

Without a VPC network:

* ❌ You CANNOT create VM instances

---

# 🧪 Verify VM Creation Fails

Navigate to:

```text
Compute Engine → VM Instances
→ Create Instance
```

Expected error:

```text
No more networks available
```

---

# 🚀 Task 2 - Create Auto Mode Network

## Enable Required APIs

Open Cloud Shell and run:

```bash
gcloud services enable \
iap.googleapis.com \
networkmanagement.googleapis.com
```

---

# 🌐 Create Auto Mode VPC

Navigate to:

```text
VPC Network → VPC Networks
→ Create VPC Network
```

Fill:

| Field                | Value     |
| -------------------- | --------- |
| Name                 | mynetwork |
| Subnet creation mode | Automatic |

Select ALL firewall rules.

Click:

```text
Create
```

---

# 🔥 Create IAP Firewall Rule

Navigate to:

```text
VPC Network → Firewall
→ Create Firewall Rule
```

Fill:

| Field      | Value                 |
| ---------- | --------------------- |
| Name       | allow-iap-ssh         |
| Network    | mynetwork             |
| Direction  | Ingress               |
| Action     | Allow                 |
| Targets    | Specified target tags |
| Target tag | iap-gce               |
| Source IP  | 35.235.240.0/20       |
| Protocol   | tcp:22                |

Click Create.

---

# 🖥 Create VM 1 - mynet-us-vm

Navigate to:

```text
Compute Engine → VM Instances
→ Create Instance
```

Use:

| Field        | Value               |
| ------------ | ------------------- |
| Name         | mynet-us-vm         |
| Region       | us-central1         |
| Zone         | us-central1-a       |
| Series       | E2                  |
| Machine Type | e2-medium           |
| OS           | Debian GNU/Linux 12 |

Under Networking:

```text
Network tag = iap-gce
```

Click Create.

---

# 🖥 Create VM 2 - mynet-notus-vm

Use:

| Field  | Value          |
| ------ | -------------- |
| Name   | mynet-notus-vm |
| Region | asia-south1    |
| Zone   | asia-south1-b  |

Keep remaining settings same.

---

# 🔗 SSH into VM

Run:

```bash
gcloud compute ssh mynet-us-vm \
--zone=us-central1-a \
--tunnel-through-iap
```

When prompted:

* Enter `Y`
* Press ENTER twice for passphrase

---

# 🧪 Test Internal Connectivity

Inside VM:

```bash
ping -c 3 INTERNAL_IP
```

Expected:

* ✅ Ping works because both VMs are inside same VPC network

---

# 🧪 Test External Connectivity

```bash
ping -c 3 EXTERNAL_IP
```

Expected:

* ✅ Ping works because ICMP firewall rule allows it

---

# ❓ Quiz Answer

## Which firewall rule allows external ping?

✅ Answer:

```text
mynetwork-allow-icmp
```

---

# 🔄 Convert Auto Mode → Custom Mode

Navigate to:

```text
VPC Network → VPC Networks
→ mynetwork
→ Edit
```

Change:

```text
Automatic → Custom
```

Click Save.

---

# 🧠 Why Use Custom Mode?

| Auto Mode                     | Custom Mode                |
| ----------------------------- | -------------------------- |
| Subnets created automatically | Full subnet control        |
| Easier setup                  | Better production design   |
| Less flexible                 | Recommended for production |

---

# 🚀 Task 3 - Create Custom Networks

Create:

| Network       | Type   |
| ------------- | ------ |
| managementnet | Custom |
| privatenet    | Custom |

---

# 🌐 Create managementnet

Navigate to:

```text
VPC Network → Create VPC Network
```

Fill:

| Field | Value         |
| ----- | ------------- |
| Name  | managementnet |
| Mode  | Custom        |

Subnet:

| Field      | Value               |
| ---------- | ------------------- |
| Name       | managementsubnet-us |
| Region     | us-central1         |
| IPv4 Range | 10.240.0.0/20       |

Click Create.

---

# 🌐 Create privatenet Using CLI

## Create VPC

```bash
gcloud compute networks create privatenet --subnet-mode=custom
```

---

## Create Subnet 1

```bash
gcloud compute networks subnets create privatesubnet-us \
--network=privatenet \
--region=us-central1 \
--range=172.16.0.0/24
```

---

## Create Subnet 2

```bash
gcloud compute networks subnets create privatesubnet-notus \
--network=privatenet \
--region=asia-south1 \
--range=172.20.0.0/20
```

---

# 🔥 Firewall Rule for managementnet

Allow:

* SSH
* RDP
* Ping

Create rule:

```text
managementnet-allow-icmp-ssh-rdp
```

Protocols:

```text
tcp:22
tcp:3389
icmp
```

Source:

```text
0.0.0.0/0
```

---

# 🔥 Firewall Rule for privatenet

Run:

```bash
gcloud compute firewall-rules create privatenet-allow-icmp-ssh-rdp \
--direction=INGRESS \
--priority=1000 \
--network=privatenet \
--action=ALLOW \
--rules=icmp,tcp:22,tcp:3389 \
--source-ranges=0.0.0.0/0
```

---

# 🖥 Create managementnet-us-vm

Use:

| Field      | Value               |
| ---------- | ------------------- |
| Name       | managementnet-us-vm |
| Network    | managementnet       |
| Subnetwork | managementsubnet-us |

---

# 🖥 Create privatenet-us-vm

Run:

```bash
gcloud compute instances create privatenet-us-vm \
--zone=us-central1-a \
--machine-type=e2-micro \
--subnet=privatesubnet-us \
--image-family=debian-12 \
--image-project=debian-cloud
```

---

# 🚀 Task 4 - Connectivity Testing

# 🌍 External IP Connectivity

From:

```text
mynet-us-vm
```

Ping:

* mynet-notus-vm external IP
* managementnet-us-vm external IP
* privatenet-us-vm external IP

Expected:

* ✅ ALL pings work

Reason:

* Firewall allows ICMP traffic

---

# 🔒 Internal IP Connectivity

## ✅ This Works

```text
mynet-us-vm → mynet-notus-vm
```

Reason:

* Same VPC network

Even though:

* Different regions
* Different zones
* Different continents

---

# ❌ These Fail

```text
mynet-us-vm → managementnet-us-vm
mynet-us-vm → privatenet-us-vm
```

Reason:

* Different VPC networks

---

# 🧠 Key Learning

Google Cloud VPCs are:

* Private
* Isolated
* Secure by default

Different VPCs cannot communicate internally unless using:

* VPC Peering
* VPN
* Interconnect

---

# ✅ Final Quiz Answers

## Q1. Can you create VM without VPC?

✅ Answer:

```text
False
```

---

## Q2. Which firewall rule allows external ping?

✅ Answer:

```text
mynetwork-allow-icmp
```

---

## Q3. Which internal ping works?

✅ Answer:

```text
mynet-notus-vm
```

# 💡 Real World Best Practices

Production environments usually use:

* ✅ Custom Mode VPCs
* ✅ Strict firewall rules
* ✅ Separate networks for:

  * Management
  * Private workloads
  * Public workloads

---
