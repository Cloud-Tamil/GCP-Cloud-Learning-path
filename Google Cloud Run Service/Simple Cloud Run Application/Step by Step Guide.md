# Simple Cloud Run Application

A beginner-friendly Node.js application deployed on Google Cloud Run. This project demonstrates how to create a simple web application, containerize it with Docker, and deploy it as a fully managed serverless service.

## What You'll Build

A responsive web application that displays:

* 🚀 Welcome message
* ☁️ Google Cloud Run deployment information
* 🔄 Dynamic visitor counter
* 📱 Mobile-friendly interface
* ⚡ Serverless architecture

### Sample Output

```text
🚀 Hello from Cloud Run!

This is my first deployment on Google Cloud Run.

Visitor Count: 1

✨ Deployed with zero downtime! ✨
```

---

## Prerequisites

Before you begin, ensure you have:

* A Google Cloud Project
* Billing enabled
* Cloud Shell or a local terminal
* Google Cloud SDK installed
* Basic knowledge of Linux commands

---

## Step 1: Configure Google Cloud Environment

Set your preferred region and zone:

```bash
gcloud config set compute/region us-east4
gcloud config set compute/zone us-east4-c
```

Verify configuration:

```bash
gcloud config list
```

---

## Step 2: Set the Active Project

Replace the example project ID with your own:

```bash
gcloud config set project GCP-PROJECT-ID
```

Verify:

```bash
gcloud config get-value project
```

Example:

```text
my-cloud-project
```

---

## Step 3: Export Project ID

```bash
export PROJECT_ID=$(gcloud config get-value project)
```

Verify:

```bash
echo $PROJECT_ID
```

---

## Step 4: Create Project Directory

```bash
mkdir my-cloudrun-app
cd my-cloudrun-app
```

---

## Step 5: Create Application Files

### Create index.html

```bash
cat > index.html << 'EOF'
<!DOCTYPE html>
<html>
<head>
  <title>My Cloud Run App</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      text-align: center;
      padding: 50px;
      background: linear-gradient(135deg,#667eea,#764ba2);
      color: white;
    }

    h1 {
      font-size: 3rem;
    }

    .counter {
      font-size: 2rem;
      background: rgba(255,255,255,0.2);
      padding: 20px;
      border-radius: 10px;
      display: inline-block;
    }
  </style>
</head>
<body>

<h1>🚀 Hello from Cloud Run!</h1>

<p>This is my first deployment on Google Cloud Run.</p>

<div class="counter">
  Visitor Count: <span id="count">Loading...</span>
</div>

<p>✨ Deployed with zero downtime! ✨</p>

<script>
fetch('/count')
  .then(response => response.text())
  .then(data => {
      document.getElementById('count').innerText = data;
  });
</script>

</body>
</html>
EOF
```

---

### Create server.js

```bash
cat > server.js << 'EOF'
const express = require('express');
const path = require('path');

const app = express();

let visitorCount = 0;

app.use(express.static(__dirname));

app.get('/count', (req, res) => {
  visitorCount++;
  res.send(visitorCount.toString());
});

const PORT = process.env.PORT || 8080;

app.listen(PORT, () => {
  console.log(`Application listening on port ${PORT}`);
});
EOF
```

---

### Create package.json

```bash
cat > package.json << 'EOF'
{
  "name": "my-cloudrun-app",
  "version": "1.0.0",
  "description": "Simple Cloud Run Application",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^4.21.2"
  }
}
EOF
```

---

### Create Dockerfile

```bash
cat > Dockerfile << 'EOF'
FROM node:20-slim

WORKDIR /usr/src/app

COPY package*.json ./

RUN npm install --omit=dev

COPY . .

EXPOSE 8080

CMD ["npm","start"]
EOF
```

---

### Create .dockerignore

```bash
cat > .dockerignore << 'EOF'
node_modules
.git
.gitignore
README.md
npm-debug.log
EOF
```

---

## Step 6: Test Locally

Install dependencies:

```bash
npm install
```

Run application:

```bash
npm start
```

Open:

```text
http://localhost:8080
```

---

## Step 7: Enable Required APIs

```bash
gcloud services enable \
  run.googleapis.com \
  cloudbuild.googleapis.com \
  artifactregistry.googleapis.com
```

---

## Step 8: Deploy to Cloud Run

### Source-Based Deployment (Recommended)

```bash
gcloud run deploy my-simple-app \
  --source . \
  --region us-east4 \
  --allow-unauthenticated
```

Cloud Build automatically:

1. Creates the container image
2. Stores the image
3. Deploys to Cloud Run
4. Generates a public HTTPS URL

---

## Step 9: Verify Deployment

List Cloud Run services:

```bash
gcloud run services list --region us-east4
```

Get service URL:

```bash
gcloud run services describe my-simple-app \
  --region us-east4 \
  --format='value(status.url)'
```

Example:

```text
https://my-simple-app-xxxxx-uc.a.run.app
```

---

## Step 10: Update the Application

Modify application files:

```bash
nano index.html
```

Redeploy:

```bash
gcloud run deploy my-simple-app \
  --source . \
  --region us-east4 \
  --allow-unauthenticated
```

Cloud Run performs rolling updates automatically with no service interruption.

---

## Monitoring and Logs

View recent logs:

```bash
gcloud logging read \
'resource.type=cloud_run_revision' \
--limit 20
```

Tail logs in real time:

```bash
gcloud beta run services logs tail my-simple-app \
  --region us-east4
```

---

## Scaling Configuration

Configure minimum and maximum instances:

```bash
gcloud run services update my-simple-app \
  --region us-east4 \
  --min-instances 0 \
  --max-instances 5
```

| Setting       | Description                     |
| ------------- | ------------------------------- |
| Min Instances | Minimum containers kept warm    |
| Max Instances | Maximum containers allowed      |
| Concurrency   | Requests processed per instance |

---

## Automated Deployment Script

Create `deploy.sh`:

```bash
#!/bin/bash

SERVICE_NAME="my-simple-app"
REGION="us-east4"

echo "Deploying to Cloud Run..."

gcloud run deploy $SERVICE_NAME \
  --source . \
  --region $REGION \
  --allow-unauthenticated

echo "Deployment completed."
```

Run:

```bash
chmod +x deploy.sh
./deploy.sh
```

---

## Quick Reference

| Command                      | Purpose              |
| ---------------------------- | -------------------- |
| npm start                    | Run locally          |
| gcloud run deploy --source . | Deploy application   |
| gcloud run services list     | List services        |
| gcloud run services describe | View service details |
| gcloud logging read          | View logs            |

---

## Cost Optimization Tips

#### Scale to Zero

```bash
--min-instances=0
```

No instances run when there is no traffic.

#### Free Tier

Cloud Run includes free monthly CPU, memory, and request quotas.

#### Efficient Container Images

Use slim base images to reduce startup time and image size.

---

## Conclusion

In this project, we successfully built a simple Node.js web application and deployed it to Google Cloud Run using a fully managed serverless architecture.

Throughout this guide, we:

* Configured the Google Cloud environment and project settings.
* Created a responsive web application using HTML, CSS, and JavaScript.
* Built a lightweight Node.js and Express server to serve the application and handle visitor requests.
* Containerized the application using Docker to ensure consistent deployments across environments.
* Enabled the required Google Cloud services, including Cloud Run and Cloud Build.
* Tested the application locally before deployment.
* Deployed the application to Cloud Run directly from source code using Cloud Build.
* Verified the deployment by accessing the public HTTPS endpoint generated by Cloud Run.
* Learned how to update the application and redeploy new versions with zero downtime.
* Explored monitoring, logging, and scaling capabilities available within Cloud Run.

By deploying the application on Cloud Run, we gained several cloud-native benefits:

* **Serverless Infrastructure** – No virtual machines or servers to manage.
* **Automatic Scaling** – The application automatically scales based on incoming traffic.
* **Scale to Zero** – No running instances when there is no traffic, helping reduce costs.
* **Built-in HTTPS** – Secure access through automatically managed SSL certificates.
* **Container-Based Deployment** – Consistent and portable deployments using Docker containers.
* **High Availability** – Managed by Google Cloud's reliable infrastructure.
* **Zero-Downtime Releases** – New revisions can be deployed without interrupting users.
* **Pay-As-You-Go Pricing** – Pay only for the resources consumed by the application.

This project demonstrates the complete lifecycle of a modern cloud-native application—from development and containerization to deployment and operations—using Google Cloud Run. The same deployment pattern can be extended to host APIs, microservices, enterprise applications, and production-grade workloads in Google Cloud.

---
