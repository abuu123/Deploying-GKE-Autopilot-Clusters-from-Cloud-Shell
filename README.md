# Deploying-GKE-Autopilot-Clusters-from-Cloud-Shell
#!/bin/bash
# =============================================================
# GKE Autopilot Cluster Deployment and Nginx Pod Demo
# =============================================================
# Author: Abu Godo
# Lab: GKE Autopilot - Cloud Shell
# =============================================================

# ------------------------------
# 1️⃣ Set Project & Cluster Variables
# ------------------------------
export MY_PROJECT=$(gcloud config get-value project)
export MY_REGION="us-east1"
export MY_CLUSTER="autopilot-cluster-1"
echo "ℹ️ Project: $MY_PROJECT | Region: $MY_REGION | Cluster: $MY_CLUSTER"

# ------------------------------
# 2️⃣ Create Autopilot Cluster
# ------------------------------
echo "🔹 Creating GKE Autopilot Cluster..."
gcloud container clusters create-auto $MY_CLUSTER --region $MY_REGION
echo "✅ Cluster created."

# ------------------------------
# 3️⃣ Get Cluster Credentials
# ------------------------------
echo "🔹 Fetching cluster credentials..."
gcloud container clusters get-credentials $MY_CLUSTER --region $MY_REGION
kubectl config use-context gke_${MY_PROJECT}_${MY_REGION}_${MY_CLUSTER}
echo "✅ Context set for kubectl."

# ------------------------------
# 4️⃣ Enable kubectl bash completion
# ------------------------------
source <(kubectl completion bash)
echo "✅ kubectl bash completion enabled."

# ------------------------------
# 5️⃣ Deploy Nginx Pod
# ------------------------------
echo "🔹 Deploying Nginx pod..."
kubectl create deployment --image=nginx nginx-demo
kubectl get pods

# ------------------------------
# 6️⃣ Expose Nginx Pod as LoadBalancer
# ------------------------------
kubectl expose pod nginx-demo --port 80 --type LoadBalancer
kubectl get services

# ------------------------------
# 7️⃣ Copy Custom HTML to Pod
# ------------------------------
export MY_POD=$(kubectl get pods -l app=nginx-demo -o jsonpath="{.items[0].metadata.name}")
echo "ℹ️ Nginx Pod: $MY_POD"

# Create a test HTML file locally
echo "<html><body><h1>Hello GKE Autopilot!</h1></body></html>" > ~/test.html

# Copy to Pod
kubectl cp ~/test.html $MY_POD:/usr/share/nginx/html/test.html
echo "✅ Custom HTML copied to pod."

# ------------------------------
# 8️⃣ Optional: Port Forward for Local Testing
# ------------------------------
kubectl port-forward $MY_POD 10081:80 &
echo "✅ Port forwarding started at http://127.0.0.1:10081/test.html"

# ------------------------------
# 9️⃣ Check Pod & Node Resources
# ------------------------------
kubectl get pods
kubectl top pods
kubectl top nodes

echo "🎉 GKE Autopilot cluster deployment complete!"
echo "Test your Nginx pod via LoadBalancer IP or port-forwarded URL."
