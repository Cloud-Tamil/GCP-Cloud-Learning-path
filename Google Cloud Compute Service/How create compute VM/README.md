# Creating a Google Cloud VM (Compute Engine): Manual and Automated

A practical guide to creating a Compute Engine VM in two ways:

1. **Manual**: Console UI and `gcloud` CLI (good for learning, one-off tests)
2. **Automatic**: Terraform + startup script + CI/CD (repeatable, reviewable, production-style)

---

## 1. Prerequisites

- A GCP project with billing enabled
- Compute Engine API enabled
- `gcloud` CLI installed and authenticated
- Terraform >= 1.5 (for the automated method)

```bash
gcloud auth login
gcloud config set project <PROJECT_ID>
gcloud services enable compute.googleapis.com
```

---

## 2. Manual Method

### 2.1 Google Cloud Console (UI)

1. Go to **Compute Engine > VM instances > Create instance**.
2. **Name**: `web-vm-01`
3. **Region / Zone**: `asia-south1` (Mumbai) / `asia-south1-a`
4. **Machine type**: `e2-medium` (2 vCPU, 4 GB) for testing
5. **Boot disk**: Debian 12, 20 GB, Balanced persistent disk
6. **Identity**: use a dedicated service account (not the default Compute SA)
7. **Firewall**: tick HTTP/HTTPS only if needed
8. **Security**: enable Shielded VM (Secure Boot, vTPM, Integrity Monitoring)
9. Click **Create**, then connect with the **SSH** button.

### 2.2 gcloud CLI

```bash
gcloud compute instances create web-vm-01 \
  --zone=asia-south1-a \
  --machine-type=e2-medium \
  --image-family=debian-12 \
  --image-project=debian-cloud \
  --boot-disk-size=20GB \
  --boot-disk-type=pd-balanced \
  --tags=web \
  --service-account=vm-sa@<PROJECT_ID>.iam.gserviceaccount.com \
  --scopes=cloud-platform \
  --shielded-secure-boot \
  --shielded-vtpm \
  --shielded-integrity-monitoring \
  --labels=env=dev,owner=devops
```

Connect and manage:

```bash
gcloud compute ssh web-vm-01 --zone=asia-south1-a
gcloud compute instances list
gcloud compute instances stop web-vm-01 --zone=asia-south1-a
gcloud compute instances delete web-vm-01 --zone=asia-south1-a
```

---

## 3. Automatic Method (Professional)

### 3.1 Project structure

```
gcp-vm/
├── main.tf
├── variables.tf
├── outputs.tf
├── terraform.tfvars
├── scripts/
│   └── startup.sh
└── .github/workflows/terraform.yml
```

### 3.2 `variables.tf`

```hcl
variable "project_id" { type = string }
variable "region"     { type = string, default = "asia-south1" }
variable "zone"       { type = string, default = "asia-south1-a" }
variable "vm_name"    { type = string, default = "web-vm-01" }
variable "machine_type" { type = string, default = "e2-medium" }
```

### 3.3 `main.tf`

```hcl
terraform {
  required_version = ">= 1.5"
  required_providers {
    google = { source = "hashicorp/google", version = "~> 6.0" }
  }
  # Remote state (create the bucket first)
  backend "gcs" {
    bucket = "<STATE_BUCKET>"
    prefix = "gcp-vm"
  }
}

provider "google" {
  project = var.project_id
  region  = var.region
}

resource "google_service_account" "vm_sa" {
  account_id   = "vm-sa"
  display_name = "VM service account"
}

resource "google_compute_firewall" "allow_http" {
  name    = "allow-http"
  network = "default"
  allow {
    protocol = "tcp"
    ports    = ["80", "443"]
  }
  source_ranges = ["0.0.0.0/0"]
  target_tags   = ["web"]
}

resource "google_compute_instance" "vm" {
  name         = var.vm_name
  machine_type = var.machine_type
  zone         = var.zone
  tags         = ["web"]
  labels       = { env = "dev", managed_by = "terraform" }

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-12"
      size  = 20
      type  = "pd-balanced"
    }
  }

  network_interface {
    network = "default"
    access_config {}   # remove this block for a private VM (use IAP/NAT)
  }

  service_account {
    email  = google_service_account.vm_sa.email
    scopes = ["cloud-platform"]
  }

  shielded_instance_config {
    enable_secure_boot          = true
    enable_vtpm                 = true
    enable_integrity_monitoring = true
  }

  metadata_startup_script = file("${path.module}/scripts/startup.sh")
}
```

### 3.4 `outputs.tf`

```hcl
output "vm_name" { value = google_compute_instance.vm.name }
output "external_ip" {
  value = google_compute_instance.vm.network_interface[0].access_config[0].nat_ip
}
```

### 3.5 `scripts/startup.sh`

```bash
#!/bin/bash
set -e
apt-get update -y
apt-get install -y nginx
systemctl enable --now nginx
echo "Hello from $(hostname)" > /var/www/html/index.html
```

### 3.6 Run it

```bash
terraform init
terraform fmt -check
terraform validate
terraform plan -var="project_id=<PROJECT_ID>" -out=tfplan
terraform apply tfplan
```

Destroy when finished:

```bash
terraform destroy -var="project_id=<PROJECT_ID>"
```

### 3.7 CI/CD with GitHub Actions (optional)

`.github/workflows/terraform.yml`

```yaml
name: terraform
on:
  pull_request:
  push:
    branches: [main]

permissions:
  contents: read
  id-token: write   # for Workload Identity Federation

jobs:
  terraform:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: google-github-actions/auth@v2
        with:
          workload_identity_provider: ${{ secrets.WIF_PROVIDER }}
          service_account: ${{ secrets.WIF_SERVICE_ACCOUNT }}
      - uses: hashicorp/setup-terraform@v3
      - run: terraform init
      - run: terraform fmt -check
      - run: terraform validate
      - run: terraform plan -var="project_id=${{ vars.PROJECT_ID }}"
      - if: github.ref == 'refs/heads/main'
        run: terraform apply -auto-approve -var="project_id=${{ vars.PROJECT_ID }}"
```

Use **Workload Identity Federation** instead of storing service account JSON keys.

---

## 4. Manual vs Automatic

| Aspect            | Manual (Console / gcloud) | Automatic (Terraform + CI/CD) |
|-------------------|---------------------------|-------------------------------|
| Speed for one VM  | Fast                      | Slower initial setup          |
| Repeatability     | Low                       | High                          |
| Audit / review    | None                      | Git history + PR review       |
| Drift detection   | No                        | Yes (`terraform plan`)        |
| Best for          | Learning, quick tests     | Dev, staging, production      |

---

## 5. Best Practices Checklist

- [ ] Dedicated service account with least-privilege IAM roles
- [ ] No external IP in production; use IAP TCP forwarding or Cloud NAT
- [ ] Restrict firewall `source_ranges` (avoid `0.0.0.0/0` except for public web ports)
- [ ] OS Login enabled instead of manually managed SSH keys
- [ ] Shielded VM enabled
- [ ] Labels for environment, owner, cost center
- [ ] Remote Terraform state in GCS with versioning
- [ ] Snapshot schedule for persistent disks
- [ ] Budget alerts set on the billing account
- [ ] Use Spot VMs for non-critical workloads to cut cost

---

## 6. Cleanup

```bash
# Manual
gcloud compute instances delete web-vm-01 --zone=asia-south1-a

# Terraform
terraform destroy
```

---

## 7. References

- Compute Engine docs: https://cloud.google.com/compute/docs
- Terraform Google provider: https://registry.terraform.io/providers/hashicorp/google/latest/docs
