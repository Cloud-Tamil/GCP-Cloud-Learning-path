# Develop Your Google Cloud Network : Challenge 

## Overview

This guide provides a complete end-to-end walkthrough for the **Develop Your Google Cloud Network: Challenge**.

You will create:

* Development VPC
* Production VPC
* Bastion Host
* Cloud SQL Instance
* GKE Kubernetes Cluster
* WordPress Deployment
* Monitoring
* IAM Access

---

# Pre-requisites

| Item       | Value                                            |
| ---------- | ------------------------------------------------ |
| Set Project| Your_Project ID                                  |              
| Region     | `asia-south1`                                    |
| Zone       | `asia-south1-b`                                  |

---

# Task 1 - Create Development VPC

## Create VPC

```bash
gcloud compute networks create griffin-dev-vpc \
    --subnet-mode=custom
```

## Create Subnets

### Development WordPress Subnet

```bash
gcloud compute networks subnets create griffin-dev-wp \
    --network=griffin-dev-vpc \
    --region=asia-south1 \
    --range=192.168.16.0/20
```

### Development Management Subnet

```bash
gcloud compute networks subnets create griffin-dev-mgmt \
    --network=griffin-dev-vpc \
    --region=asia-south1 \
    --range=192.168.32.0/20
```

---

# Task 2 - Create Production VPC

## Create VPC

```bash
gcloud compute networks create griffin-prod-vpc \
    --subnet-mode=custom
```

## Create Subnets

### Production WordPress Subnet

```bash
gcloud compute networks subnets create griffin-prod-wp \
    --network=griffin-prod-vpc \
    --region=asia-south1 \
    --range=192.168.48.0/20
```

### Production Management Subnet

```bash
gcloud compute networks subnets create griffin-prod-mgmt \
    --network=griffin-prod-vpc \
    --region=asia-south1 \
    --range=192.168.64.0/20
```

---

# Task 3 - Create Bastion Host

## Create Bastion VM

```bash
gcloud compute instances create bastion \
    --zone=us-central1-b \
    --machine-type=e2-medium \
    --subnet=griffin-dev-mgmt \
    --network-interface=subnet=griffin-prod-mgmt,no-address
```

## Create Firewall Rule

```bash
gcloud compute firewall-rules create allow-ssh \
    --network=griffin-dev-vpc \
    --allow=tcp:22
```

## SSH Into Bastion Host

```bash
gcloud compute ssh bastion --zone=us-central1-b
```

---

# Task 4 - Create and Configure Cloud SQL

## Enable API

```bash
gcloud services enable sqladmin.googleapis.com
```

## Create Cloud SQL Instance

```bash
gcloud sql instances create griffin-dev-db \
    --database-version=MYSQL_5_7 \
    --tier=db-f1-micro \
    --region=us-central1 \
    --root-password=password123
```

> Wait approximately 5–10 minutes for provisioning.

## Connect to SQL Instance

```bash
gcloud sql connect griffin-dev-db --user=root
```

Password:

```text
password123
```

## Configure WordPress Database

```sql
CREATE DATABASE wordpress;

CREATE USER 'wp_user'@'%' IDENTIFIED BY 'stormwind_rules';

GRANT ALL PRIVILEGES ON wordpress.* TO 'wp_user'@'%';

FLUSH PRIVILEGES;

EXIT;
```

---

# Task 5 - Create Kubernetes Cluster

## Create GKE Cluster

```bash
gcloud container clusters create griffin-dev \
    --zone=asia-south1-b \
    --num-nodes=2 \
    --machine-type=e2-standard-4 \
    --network=griffin-dev-vpc \
    --subnetwork=griffin-dev-wp
```

## Get Cluster Credentials

```bash
gcloud container clusters get-credentials griffin-dev \
    --zone=us-central1-b
```

---

# Task 6 - Prepare Kubernetes Cluster

## Copy Kubernetes Files

```bash
gsutil cp -r gs://spls/gsp321/wp-k8s .
```

```bash
cd wp-k8s
```

---

## Edit wp-env.yaml

```bash
nano wp-env.yaml
```

Update values:

| Key      | Value             |
| -------- | ----------------- |
| username | `wp_user`         |
| password | `stormwind_rules` |

## Apply Environment File

```bash
kubectl apply -f wp-env.yaml
```

---

## Create Service Account Key

```bash
gcloud iam service-accounts keys create key.json \
    --iam-account=cloud-sql-proxy@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com
```

## Create Kubernetes Secret

```bash
kubectl create secret generic cloudsql-instance-credentials \
    --from-file key.json
```

---

## Task 7 - Deploy WordPress

### Get SQL Connection Name

```bash
gcloud sql instances describe griffin-dev-db \
    --format="value(connectionName)"
```

Example:

```text
qwiklabs-gcp-xxxx:asia-south1:griffin-dev-db
```

---

### Edit Deployment File

```bash
nano wp-deployment.yaml
```

Replace:

```text
YOUR_SQL_INSTANCE
```

With:

```text
YOUR_PROJECT_ID:us-central1:griffin-dev-db
```

---

### Deploy WordPress

```bash
kubectl apply -f wp-deployment.yaml
```

### Create Service

```bash
kubectl apply -f wp-service.yaml
```

### Check Service

```bash
kubectl get svc
```

Wait until:

```text
EXTERNAL-IP
```

is assigned.

---

## Task 8 - Enable Monitoring

### Open Monitoring

Navigate:

```text
Monitoring → Uptime Checks
```

## Create Uptime Check

| Setting       | Value             |
| ------------- | ----------------- |
| Name          | `wordpress-check`
|Protocol       | HTTP               |
| Resource Type | URL               |
| URL           | External IP       |
| Path          | `/`               |

Save the configuration.

---

## Task 9 - Provide IAM Access

Navigate:

```text
IAM & Admin → IAM
```

Grant the second lab user:

```text
Editor
```

role.

---

## Validation Commands

### Kubernetes Pods

```bash
kubectl get pods
```

### Kubernetes Services

```bash
kubectl get svc
```

### Deployments

```bash
kubectl get deployments
```

### GKE Clusters

```bash
gcloud container clusters list
```

### Cloud SQL Instances

```bash
gcloud sql instances list
```

---

## Common Errors and Fixes

| Problem                     | Solution                              |
| --------------------------- | ------------------------------------- |
| SQL instance not connecting | Wait for provisioning to complete     |
| No EXTERNAL-IP              | Wait 5–10 minutes                     |
| Pod CrashLoopBackOff        | Check secrets and SQL connection name |
| Firewall SSH issue          | Verify firewall rule                  |
| Cluster creation fails      | Retry after quota refresh             |

---

## Final Verification Checklist

| Resource           | Expected Name      |
| ------------------ | ------------------ |
| Development VPC    | `griffin-dev-vpc`  |
| Production VPC     | `griffin-prod-vpc` |
| Bastion Host       | `bastion`          |
| Cloud SQL          | `griffin-dev-db`   |
| Kubernetes Cluster | `griffin-dev`      |
| Monitoring         | Enabled            |
| IAM Role           | Editor             |

---

## Completion

After all tasks are validated successfully:

```text
Congratulations! You completed the GSP321 Challenge Lab.
```
