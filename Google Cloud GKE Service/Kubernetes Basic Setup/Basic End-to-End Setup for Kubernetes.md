# End-to-End: Google Kubernetes Engine
- ## Google Kubernetes Engine provides a fully managed platform to deploy, scale, and manage containerized applications with high availability, automatic scaling, self-healing, and integrated networking and monitoring.
  
```bash
User (Browser)
     ↓
Load Balancer (External IP)
     ↓
Kubernetes Service
     ↓
Deployment
     ↓
Pods (Containers running app)
     ↓
Nodes (VMs in Cluster)
```
## Step 1: Set Region & Zone
- Before creating resources, you need to configure the region and zone.
- You configure region & zone first because cloud resources must know where to be created, and that choice affects performance, cost, and reliability.
  
```
gcloud config set compute/region europe-west4
gcloud config set compute/zone europe-west4-a
```
## Step 2: Create GKE Cluster
- Use the following commands to build your own cluster in your environment, where you can define the type and create the components as needed.
  
```
gcloud container clusters create lab-cluster \
--machine-type=e2-medium \
--zone=europe-west4-a
```
## Step 3: Connect to Cluster
- Used to connect your local environment (kubectl) to your GKE cluster.
  
```
gcloud container clusters get-credentials lab-cluster
```
## Step 4: Deploy Application
- Using these commands, you can deploy and run containerized applications in Kubernetes.
  
```
kubectl create deployment hello-server \
--image=gcr.io/google-samples/hello-app:1.0
```
- Check the Deployment and pods.
  
```
kubectl get deployments
kubectl get pods
```
## Step 5: Expose Application (Very Important)
- We use this command to expose the application to the outside world so users can access it using a public IP address.
- We use a port to direct incoming traffic to the correct application running on a server or container.
  
```
kubectl expose deployment hello-server \
--type=LoadBalancer \
--port 8080
```
## Step 6: Get External IP
- Commands to get external IP.
  
```
kubectl get services
```
## Step 7: Access Application
- Copyed the external IP and paste it to Brower.

```
http://<EXTERNAL-IP>:8080
```
## Step 8: Delete Cluster (Important for cleanup)
- If you no longer need this cluster, you can delete it using the command below.

```
gcloud container clusters delete lab-cluster
```
