# Creating a Google Cloud VM (Compute Engine): Manual and Automatic

Two ways to create a VM, with no Terraform needed:

1. **Manual**: Console UI or a single `gcloud` command
2. **Automatic**: reusable `gcloud` script + startup script + GitHub Actions

---

## 1. Prerequisites

- A GCP project with billing enabled
- `gcloud` CLI installed and authenticated

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
4. **Machine type**: `e2-medium` (2 vCPU, 4 GB)
5. **Boot disk**: Debian 12, 20 GB, Balanced persistent disk
6. **Identity**: use a dedicated service account, not the default one
7. **Firewall**: tick HTTP/HTTPS only if needed
8. **Security**: enable Shielded VM (Secure Boot, vTPM, Integrity Monitoring)
9. Click **Create**, then connect with the **SSH** button.

### 2.2 gcloud CLI (single command)

```bash
gcloud compute instances create web-vm-01 \
  --zone=asia-south1-a \
  --machine-type=e2-medium \
  --image-family=debian-12 \
  --image-project=debian-cloud \
  --boot-disk-size=20GB \
  --boot-disk-type=pd-balanced \
  --tags=web \
  --shielded-secure-boot \
  --shielded-vtpm \
  --shielded-integrity-monitoring \
  --labels=env=dev,owner=devops
```

Manage it:

```bash
gcloud compute ssh web-vm-01 --zone=asia-south1-a
gcloud compute instances list
gcloud compute instances stop web-vm-01 --zone=asia-south1-a
gcloud compute instances delete web-vm-01 --zone=asia-south1-a
```

---

## 3. Automatic Method (script + CI/CD)

### 3.1 Project structure

```
gcp-vm/
├── create-vm.sh
├── delete-vm.sh
├── scripts/
│   └── startup.sh
└── .github/workflows/create-vm.yml
```

### 3.2 `scripts/startup.sh` (runs on first boot)

```bash
#!/bin/bash
set -e
apt-get update -y
apt-get install -y nginx
systemctl enable --now nginx
echo "Hello from $(hostname)" > /var/www/html/index.html
```

### 3.3 `create-vm.sh` (idempotent, reusable)

```bash
#!/bin/bash
set -euo pipefail

PROJECT_ID="${PROJECT_ID:?Set PROJECT_ID}"
VM_NAME="${VM_NAME:-web-vm-01}"
ZONE="${ZONE:-asia-south1-a}"
MACHINE_TYPE="${MACHINE_TYPE:-e2-medium}"
SA_NAME="vm-sa"
SA_EMAIL="${SA_NAME}@${PROJECT_ID}.iam.gserviceaccount.com"

# 1. Service account (create if missing)
if ! gcloud iam service-accounts describe "$SA_EMAIL" --project "$PROJECT_ID" >/dev/null 2>&1; then
  gcloud iam service-accounts create "$SA_NAME" \
    --display-name="VM service account" --project "$PROJECT_ID"
fi

# 2. Firewall rule for web traffic (create if missing)
if ! gcloud compute firewall-rules describe allow-web --project "$PROJECT_ID" >/dev/null 2>&1; then
  gcloud compute firewall-rules create allow-web \
    --allow=tcp:80,tcp:443 --target-tags=web --project "$PROJECT_ID"
fi

# 3. VM (skip if it already exists)
if gcloud compute instances describe "$VM_NAME" --zone "$ZONE" --project "$PROJECT_ID" >/dev/null 2>&1; then
  echo "VM $VM_NAME already exists. Skipping."
  exit 0
fi

gcloud compute instances create "$VM_NAME" \
  --project="$PROJECT_ID" \
  --zone="$ZONE" \
  --machine-type="$MACHINE_TYPE" \
  --image-family=debian-12 \
  --image-project=debian-cloud \
  --boot-disk-size=20GB \
  --boot-disk-type=pd-balanced \
  --tags=web \
  --service-account="$SA_EMAIL" \
  --scopes=cloud-platform \
  --shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring \
  --labels=env=dev,managed_by=script \
  --metadata-from-file=startup-script=scripts/startup.sh

gcloud compute instances describe "$VM_NAME" --zone "$ZONE" --project "$PROJECT_ID" \
  --format="get(networkInterfaces[0].accessConfigs[0].natIP)"
```

Run it:

```bash
chmod +x create-vm.sh
PROJECT_ID=<PROJECT_ID> ./create-vm.sh
```

### 3.4 `delete-vm.sh`

```bash
#!/bin/bash
set -euo pipefail
gcloud compute instances delete "${VM_NAME:-web-vm-01}" \
  --zone="${ZONE:-asia-south1-a}" --project="${PROJECT_ID:?Set PROJECT_ID}" --quiet
```

### 3.5 GitHub Actions (`.github/workflows/create-vm.yml`)

```yaml
name: create-vm
on:
  workflow_dispatch:
    inputs:
      vm_name:
        description: VM name
        default: web-vm-01

permissions:
  contents: read
  id-token: write   # Workload Identity Federation

jobs:
  create:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: google-github-actions/auth@v2
        with:
          workload_identity_provider: ${{ secrets.WIF_PROVIDER }}
          service_account: ${{ secrets.WIF_SERVICE_ACCOUNT }}
      - uses: google-github-actions/setup-gcloud@v2
      - run: ./create-vm.sh
        env:
          PROJECT_ID: ${{ vars.PROJECT_ID }}
          VM_NAME: ${{ inputs.vm_name }}
```

Use **Workload Identity Federation** instead of storing service account JSON keys.

---

## 4. Manual vs Automatic

| Aspect           | Manual                | Automatic (script + CI/CD)  |
|------------------|-----------------------|-----------------------------|
| Speed for one VM | Fast                  | Slightly slower to set up   |
| Repeatability    | Low                   | High                        |
| Audit / review   | None                  | Git history + workflow logs |
| Best for         | Learning, quick tests | Dev, staging, team use      |

---

## 5. Best Practices Checklist

- [ ] Dedicated service account with least-privilege IAM roles
- [ ] No external IP in production; use IAP TCP forwarding or Cloud NAT
- [ ] Restrict firewall source ranges where possible
- [ ] OS Login enabled instead of manual SSH keys
- [ ] Shielded VM enabled
- [ ] Labels for environment and owner
- [ ] Snapshot schedule for disks
- [ ] Budget alerts on the billing account
- [ ] Spot VMs for non-critical workloads

---

## 6. Cleanup

```bash
PROJECT_ID=<PROJECT_ID> ./delete-vm.sh
```

---

## 7. References

- Compute Engine docs: https://cloud.google.com/compute/docs
- gcloud reference: https://cloud.google.com/sdk/gcloud/reference/compute/instances/create
