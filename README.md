Google Kubernetes Engine (GKE): Qwik Start
Show Image
Show Image
Show Image
Show Image

A hands-on guide to deploying a containerized application on Google Kubernetes Engine (GKE).


📋 Table of Contents

Overview
Objectives
Prerequisites
Setup

Task 1: Set a Default Compute Zone
Task 2: Create a GKE Cluster
Task 3: Get Authentication Credentials
Task 4: Deploy an Application to the Cluster
Task 5: Delete the Cluster


Key Concepts
References


📖 Overview
Google Kubernetes Engine (GKE) provides a managed environment for deploying, managing, and scaling containerized applications using Google infrastructure. GKE clusters are powered by the open-source Kubernetes cluster management system.
GKE on Google Cloud includes:

⚖️ Load balancing for Compute Engine instances
🏊 Node pools for flexible cluster management
📈 Automatic scaling of node instance counts
🔄 Automatic upgrades for cluster node software
🔧 Node auto-repair for health and availability
📊 Logging and monitoring via Cloud Monitoring


🎯 Objectives
By the end of this lab, you will be able to:

✅ Create a GKE cluster
✅ Deploy an application to the cluster
✅ Create a Kubernetes Service
✅ Delete the cluster


✅ Prerequisites

Access to a standard internet browser (Chrome recommended)
A Google Cloud account (or Qwiklabs temporary credentials)
Basic familiarity with the command line


⚠️ Note: Use an Incognito/Private browser window to avoid conflicts with personal Google accounts.


🖼️ Architecture Diagram
<!-- TODO: Add architecture diagram showing GKE cluster, nodes, load balancer, and deployed app -->

Show Image
Diagram: GKE cluster with load balancer exposing the hello-server application


⚙️ Setup
Activate Cloud Shell
In the Google Cloud Console, click Activate Cloud Shell at the top right. Cloud Shell is a virtual machine pre-loaded with development tools and gcloud CLI.
Verify your active account and project:
bashgcloud auth list
gcloud config list project

Task 1: Set a Default Compute Zone
Set the default region and zone for your cluster resources:
bash# Set the default compute region
gcloud config set compute/region us-east4

# Set the default compute zone
gcloud config set compute/zone us-east4-b
Expected output:
Updated property [compute/region].
Updated property [compute/zone].

Task 2: Create a GKE Cluster

⚠️ Cluster names must start with a letter, end with an alphanumeric character, and cannot exceed 40 characters.

bashgcloud container clusters create \
  --machine-type=e2-medium \
  --zone=us-east4-b \
  lab-cluster
This may take a few minutes.
Expected output:
NAME: lab-cluster
LOCATION: us-east4-b
MASTER_VERSION: 1.22.8-gke.202
MACHINE_TYPE: e2-medium
NUM_NODES: 3
STATUS: RUNNING
<!-- TODO: Replace with screenshot of GKE cluster in Google Cloud Console -->

Show Image
Screenshot: Newly created lab-cluster visible in the GKE Console


Task 3: Get Authentication Credentials
After the cluster is created, authenticate to interact with it:
bashgcloud container clusters get-credentials lab-cluster
Expected output:
Fetching cluster endpoint and auth data.
kubeconfig entry generated for lab-cluster.

Task 4: Deploy an Application to the Cluster
4a. Create a Deployment
Deploy the hello-app container image as a Kubernetes Deployment:
bashkubectl create deployment hello-server \
  --image=gcr.io/google-samples/hello-app:1.0
Expected output:
deployment.apps/hello-server created
4b. Expose the Deployment as a Service
Create a Kubernetes Service to expose the application to external traffic:
bashkubectl expose deployment hello-server \
  --type=LoadBalancer \
  --port 8080
FlagDescription--portThe port the container exposes--type=LoadBalancerCreates a Compute Engine load balancer
Expected output:
service/hello-server exposed
4c. Verify the Service
bashkubectl get service
Expected output:
NAME           TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)          AGE
hello-server   LoadBalancer   10.39.244.36   35.202.234.26   8080:31991/TCP   65s
kubernetes     ClusterIP      10.39.240.1    <none>          443/TCP          5m13s

⏳ It may take a minute for the EXTERNAL-IP to appear. Re-run the command if the status shows pending.

4d. Access the Application
Open your browser and navigate to:
http://<EXTERNAL-IP>:8080
You should see: Hello, world! along with the version and hostname.
<!-- TODO: Replace with actual browser screenshot of the running app -->

Show Image
Screenshot: hello-server app running at http://<EXTERNAL-IP>:8080


Task 5: Delete the Cluster
Once you are done, clean up your resources to avoid unwanted charges:
bashgcloud container clusters delete lab-cluster
When prompted, type Y and press Enter to confirm.

⏳ Deletion may take a few minutes to complete.


💡 Key Concepts
ConceptDescriptionGKEGoogle's managed Kubernetes serviceClusterA set of nodes (VMs) managed by KubernetesNodeA Compute Engine VM running Kubernetes processesDeploymentManages stateless application podsServiceExposes a deployment to internal/external trafficLoadBalancerA service type that provisions a cloud load balancerkubectlCommand-line tool to interact with Kubernetes clusters

📚 References

Google Kubernetes Engine Documentation
Kubernetes Official Docs
gcloud CLI Reference
GKE Deleting a Cluster
