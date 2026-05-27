# ☁️ Cloud Networking in Google Cloud Platform (GCP)

### 📘 Overview

This project demonstrates the fundamentals of networking in Google Cloud Platform (GCP) using both default and custom Virtual Private Cloud (VPC) networks.

The lab covers:

- Understanding default and custom VPC networks
- Creating subnet-based custom networks
- Configuring firewall rules
- Applying network tags to virtual machines
- Securing communication between instances

---

## 🚀 What You'll Learn

- Basic concepts and constructs of Google Cloud networking
- Difference between default and custom VPC networks
- How subnet-based networks work in GCP
- How firewall rules control traffic flow
- How to use instance tags for firewall policies
- How to create secure cloud network architectures

---

## 📋 Prerequisites

Before starting this lab, ensure you have:

- A Google Cloud account
- Billing enabled
- Access to Google Cloud Console
- Basic understanding of networking concepts
- Compute Engine API enabled

---

# 🌍 Set Default Region and Zone in Google Cloud

Certain Compute Engine resources live in **regions** and **zones**.

- A **region** is a specific geographical location where you can run your resources.
- Each **region** contains one or more **zones**.

You can learn more from the official Google Cloud documentation:
- Regions and Zones Documentation

---

## ⚙️ Configure Default Region and Zone

Run the following commands in Cloud Shell to set the default region and zone for your lab environment.

### Step 1: Set the Default Zone

```bash
gcloud config set compute/zone "ZONE"
```

Example:

```bash
gcloud config set compute/zone "us-central1-a"
```

Export the zone variable:

```bash
export ZONE=$(gcloud config get compute/zone)
```

---

### Step 2: Set the Default Region

```bash
gcloud config set compute/region "REGION"
```

Example:

```bash
gcloud config set compute/region "us-central1"
```

Export the region variable:

```bash
export REGION=$(gcloud config get compute/region)
```

---

## ✅ Verify Configuration

Check the configured values:

```bash
echo $ZONE
echo $REGION
```

---

## 📝 Notes

- Replace `"ZONE"` with your assigned lab zone.
- Replace `"REGION"` with your assigned lab region.
- These settings help avoid repeatedly specifying the region and zone in future commands.

---

# 🛡️ CloudMart GCP Networking Lab

Built for learning and hands-on practice in Google Cloud Networking and Infrastructure.
