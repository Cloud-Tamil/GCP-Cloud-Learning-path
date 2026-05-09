# 
## Challenge scenario.
 - ### As a cloud engineer at Jooli Inc. and recently trained with Google Cloud and Kubernetes, you have been asked to help a new team (Griffin) set up their environment. The team has asked for your help and has done some work, but needs you to complete the work.

   #### *You need to complete the following tasks:*
    - Create a development VPC with three subnets manually.
    - Create a production VPC with three subnets manually.
    - Create a bastion that is connected to both VPCs.
    - Create a development Cloud SQL Instance and connect and prepare the WordPress environment.
    - Create a Kubernetes cluster in the development VPC for WordPress.
    - Prepare the Kubernetes cluster for the WordPress environment.
    - Create a WordPress deployment using the supplied configuration.
    - Enable monitoring of the cluster.
    - Provide access for an additional engineer.

#### *Pre-requisites*
- Before Starting we needs to be set Default our project ID,Region & Zone.
```
gcloud config set project $(gcloud config get-value project)

gcloud config set compute/region asia-south1
gcloud config set compute/zone asia-south1-b
```

## Task 1. Create development VPC.

```
Create VPC for Dev
gcloud compute networks create griffin-dev-vpc \
    --subnet-mode=custom
---
Create Subnet - 1
gcloud compute networks subnets create griffin-dev-wp \
    --network=griffin-dev-vpc \
    --region=asia-south1 \
    --range=192.168.16.0/20
---
Create Subnet - 2
gcloud compute networks subnets create griffin-dev-mgmt \
    --network=griffin-dev-vpc \
    --region=asia-south1 \
    --range=192.168.32.0/20
```
## Task 2. Create production VPC.

```
Create VPC for Prod
gcloud compute networks create griffin-prod-vpc \
    --subnet-mode=custom
---
Create Subnet - 1
gcloud compute networks subnets create griffin-prod-wp \
    --network=griffin-prod-vpc \
    --region=asia-south1 \
    --range=192.168.48.0/20
---
Create Subnet - 2
gcloud compute networks subnets create griffin-prod-mgmt \
    --network=griffin-prod-vpc \
    --region=asia-south1 \
    --range=192.168.64.0/20
```
## Task 3. Create bastion host.

```
Dev SSH Rule

gcloud compute firewall-rules create griffin-dev-allow-ssh \
    --network=griffin-dev-vpc \
    --allow=tcp:22
---
Prod SSH Rule

gcloud compute firewall-rules create griffin-prod-allow-ssh \
    --network=griffin-prod-vpc \
    --allow=tcp:22
---
Create Bastion VM

gcloud compute instances create griffin-bastion \
    --machine-type=e2-medium \
    --zone=asia-south1-b \
    --network-interface subnet=griffin-dev-mgmt \
    --network-interface subnet=griffin-prod-mgmt
---
Test SSH

gcloud compute ssh griffin-bastion --zone=asia-south1-b    -    exit
```

# Cloud SQL MySQL Instance Setup (Manual UI Method)

## Task 4 — Create Cloud SQL Instance

> **Important Configuration**
>
> * **Region:** `asia-south1`
> * **Database Engine:** `MySQL`
> * **Instance Name:** `griffin-dev-db`
> * **Root Password:** `password123`

---

# Step 1 — Enable Cloud SQL Admin API

1. Open the Google Cloud Console:

   [https://console.cloud.google.com](https://console.cloud.google.com)

2. Navigate to:

   ```
   APIs & Services → Library
   ```

3. Search for:

   ```
   Cloud SQL Admin API
   ```

4. Click:

   ```
   ENABLE
   ```

---

# Step 2 — Open Cloud SQL

1. Navigate to:

   ```
   Databases → SQL
   ```

2. Click:

   ```
   CREATE INSTANCE
   ```

---

# Step 3 — Select Database Engine

Choose:

```text
MySQL
```

---

# Step 4 — Configure SQL Instance

Fill the configuration exactly like below:

| Setting          | Value            |
| ---------------- | ---------------- |
| Instance ID      | `griffin-dev-db` |
| Database Version | `MySQL 5.7`      |
| Region           | `us-central1`    |
| Machine Type     | `db-f1-micro`    |
| Root Password    | `password123`    |

---

# Step 5 — Machine Configuration

Under:

```text
Choose a configuration
```

Select:

* Shared Core
* Lightweight / Sandbox
* Equivalent to:

```text
db-f1-micro
```

---

# Step 6 — Create Instance

Click:

```text
CREATE INSTANCE
```

Wait approximately:

```text
5–10 minutes
```

Status should become:

```text
Runnable
```

---

# Step 7 — Open Cloud Shell

Open Cloud Shell from the top-right corner of Google Cloud Console.

---

# Step 8 — Connect to Cloud SQL

Run the following command:

```bash
gcloud sql connect griffin-dev-db --user=root
```

When prompted for password:

```text
password123
```

---

# Step 9 — Create Database and User

Run the following SQL commands:

```sql
CREATE DATABASE wordpress;

CREATE USER 'wp_user'@'%' IDENTIFIED BY 'stormwind_rules';

GRANT ALL PRIVILEGES ON wordpress.* TO 'wp_user'@'%';

FLUSH PRIVILEGES;

EXIT;
```

---

# Verification

Navigate to:

```text
Cloud SQL → griffin-dev-db
```

Verify:

* Instance Status = `Runnable`
* Database = `wordpress`
* User = `wp_user`

---

# Architecture Flow

```text
Google Cloud Console
        │
        ▼
Cloud SQL
        │
        ▼
Create MySQL Instance
        │
        ├── Instance Name → griffin-dev-db
        ├── Version → MySQL 5.7
        ├── Region → us-central1
        ├── Tier → db-f1-micro
        └── Root Password → password123
        │
        ▼
Connect using Cloud Shell
        │
        ▼
Create Database & User
        │
        ├── wordpress
        └── wp_user
```

---

# Optional — Create Database & User from UI

Inside the SQL instance:

## Databases Tab

Create database:

```text
wordpress
```

## Users Tab

Add user:

| Field    | Value             |
| -------- | ----------------- |
| Username | `wp_user`         |
| Password | `stormwind_rules` |

Click:

```text
ADD
```




















