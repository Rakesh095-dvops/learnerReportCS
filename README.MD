# Docker Build and Run Instructions

## Docker Build
```sh
# Build backend (from learnerReportCS_backend directory)
cd learnerReportCS_backend
docker build -t learner-report-backend:latest .
docker tag learner-report-backend:latest crashrik/learner-report-backend:latest
docker push crashrik/learner-report-backend:latest

# Build frontend (from learnerReportCS_frontend directory)
cd ../learnerReportCS_frontend
docker build -t learner-report-frontend:latest .
docker tag learner-report-frontend:latest crashrik/learner-report-frontend:latest
docker push crashrik/learner-report-frontend:latest
```

## Docker Run
```sh
# Run backend
docker run -d --name learner-backend --env-file .env -p 3001:3000 crashrik/learner-report-backend:latest

# Run frontend
docker run -d --name learner-frontend -p 80:3000 crashrik/learner-report-frontend:latest
```

## Troubleshooting
```sh
# List all containers
docker ps -a
docker build --no-cache --progress=plain -t your-frontend .
```

## Error Fixes
### Error: `npm install` fails
If `npm install` fails due to dependency conflicts, use:
```sh
npm install --legacy-peer-deps --force
```

### Error: `react-scripts` not found
Install `react-scripts`:
```sh
npm install react-scripts
```

### Error: Build fails due to outdated `caniuse-lite`
Update `caniuse-lite`:
```sh
npx update-browserslist-db@latest --force
```

---

# Kubernetes Deployment Instructions

## Prerequisites
1. **Start Minikube**:
   ```sh
   minikube start
   ```

2. **Enable Ingress Addon**:
   ```sh
   minikube addons enable ingress
   ```

3. **Create Namespace**:
   ```sh
   kubectl create namespace learner-report
   ```

## Apply Backend Resources
```sh
kubectl apply -f ./misc/k8s/backend/backend-secret.yaml
kubectl apply -f ./misc/k8s/backend/backend-configmap.yaml
kubectl apply -f ./misc/k8s/backend/backend-deployment.yaml
kubectl apply -f ./misc/k8s/backend/backend-service.yaml
kubectl apply -f ./misc/k8s/backend/backend-ingress.yaml
```

## Apply Frontend Resources
```sh
kubectl apply -f ./misc/k8s/frontend/frontend-configmap.yaml
kubectl apply -f ./misc/k8s/frontend/frontend-deployment.yaml
kubectl apply -f ./misc/k8s/frontend/frontend-service.yaml
kubectl apply -f ./misc/k8s/frontend/frontend-ingress.yaml
```

## Verify Resources
Check if all resources are running:
```sh
kubectl get all -n learner-report
```

## Access Services Locally
1. **Get Minikube IP**:
   ```sh
   minikube ip
   ```

2. **Update Hosts File**:
   Add the following entries to your `/etc/hosts` file (or `C:\Windows\System32\drivers\etc\hosts` on Windows):
   ```
   <minikube-ip> backend.local
   <minikube-ip> frontend.local
   ```

3. **Access Services**:
   - Backend: `http://backend.local/student`, `http://backend.local/admin`, etc.
   - Frontend: `http://frontend.local/`

---

# Ingress Troubleshooting

## Check Ingress Resource
```sh
kubectl get ingress -n learner-report
```

## Describe Ingress
If the external IP or hostname is not assigned, describe the Ingress:
```sh
kubectl describe ingress learner-report-cs-backend-ingress -n learner-report
```

## Verify Ingress Controller
Ensure the Ingress controller is running:
```sh
kubectl get pods -n kube-system
kubectl get pods -n ingress-nginx
```
If not running, enable the Ingress addon:
```sh
minikube addons enable ingress
```

If issues persist, delete and reinstall the Ingress jobs:
```sh
kubectl delete job ingress-nginx-admission-create -n ingress-nginx
kubectl delete job ingress-nginx-admission-patch -n ingress-nginx
```
Wait for the pods to start:
```sh
kubectl get pods -n ingress-nginx -w
```
Check logs for details:
```sh
kubectl logs <ingress-nginx-controller-pod-name> -n ingress-nginx
```

## Test Ingress
Start a Minikube tunnel:
```sh
minikube tunnel
```
Get the Minikube IP and test the Ingress:
```sh
minikube ip
minikube service ingress-nginx-controller -n ingress-nginx
```

---

# Port Forwarding (Optional)
If you want to test without Ingress:
```sh
kubectl port-forward svc/learner-report-cs-backend-service 3001:3001 -n learner-report
kubectl port-forward svc/learner-report-cs-frontend-service 3000:3000 -n learner-report
```

---

# Git Branch Management
To find the latest commit across branches:

### Bash
```sh
git branch -[a,l,r] --format="%(committerdate:iso8601), %(committerdate:relative) - %(refname:short)" | grep -v [H]EAD | sort -r
```

### PowerShell
```ps1
git branch -r --format="%(committerdate:iso8601), %(committerdate:relative) - %(refname:short)" | Select-String -NotMatch 'HEAD' | Sort-Object -Descending
```

---

# Logs and Debugging

## Check NGINX Logs
If the application is not working, check the NGINX logs inside the container:
```sh
docker exec -it <container-id> cat /var/log/nginx/access.log
docker exec -it <container-id> cat /var/log/nginx/error.log
```

## Check Pod Logs
Inspect the logs of the Kubernetes pods:
```sh
kubectl logs <pod-name> -n learner-report
kubectl get all -n learner-report -o wide
```

---

# Helm Chart Deployment

## Prerequisites

### For Minikube (Local Development)
1. **Install Prerequisites**:
   ```sh
   # Install Minikube, kubectl, and Helm
   minikube start
   minikube addons enable ingress
   ```

2. **Create Namespace**:
   ```sh
   kubectl create namespace learner-report
   ```

### For AWS EKS (Production)
1. **Install Prerequisites**:
   ```sh
   # Install AWS CLI, eksctl, kubectl, and Helm
   # Create EKS cluster
   eksctl create cluster --name learner-report --region ap-south-1
   ```

2. **Install NGINX Ingress Controller**:
   ```sh
   helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
   helm repo update
   helm install ingress-nginx ingress-nginx/ingress-nginx
   ```

## Deployment Commands

### Minikube Local Development
```sh
# Deploy using development values
helm install learner-report ./learner-report -f ./learner-report/values-development.yaml

# Update deployment
helm upgrade learner-report ./learner-report -f ./learner-report/values-development.yaml

# Update hosts file (Windows: C:\Windows\System32\drivers\etc\hosts)
# Get Minikube IP first
minikube ip
# Add to hosts file:
# <minikube-ip> backend.local
# <minikube-ip> frontend.local
```

### AWS EKS Production
```sh
# Deploy using production values
helm install learner-report ./learner-report -f ./learner-report/values-production.yaml

# Update deployment
helm upgrade learner-report ./learner-report -f ./learner-report/values-production.yaml

# Set up DNS records in Route 53:
# api.learner-report.com -> ALB DNS name
# app.learner-report.com -> ALB DNS name
```

## Helm Management Commands

### Basic Operations
```sh
# List deployments
helm list -n learner-report

# Check status
helm status learner-report -n learner-report

# View deployed values
helm get values learner-report -n learner-report

# Uninstall
helm uninstall learner-report -n learner-report
```

### Debugging
```sh
# Dry run and debug
helm install learner-report ./learner-report -f ./learner-report/values-development.yaml --dry-run --debug

# Template rendering
helm template learner-report ./learner-report -f ./learner-report/values-development.yaml

# Check resources
kubectl get all -n learner-report
kubectl get ingress -n learner-report
```

## Environment-Specific Configuration

### Development (Minikube)
- **Frontend**: `http://frontend.local`
- **Backend**: `http://backend.local`
- **DNS**: Local hosts file mapping
- **TLS**: Disabled for simplicity

### Production (AWS)
- **Frontend**: `https://app.learner-report.com`
- **Backend**: `https://api.learner-report.com`
- **DNS**: Route 53 managed domains
- **TLS**: Automated with cert-manager and Let's Encrypt

## Customizing Values

### Create Custom Values File
```yaml
# values-custom.yaml
global:
  environment: development
  
backend:
  replicaCount: 1
  env:
    CUSTOM_VAR: "custom-value"
    
frontend:
  replicaCount: 1
```

### Deploy with Custom Values
```sh
helm install learner-report ./learner-report -f ./learner-report/values-custom.yaml
```

## Health Checks and Monitoring

### Check Pod Status
```sh
kubectl get pods -n learner-report
kubectl describe pod <pod-name> -n learner-report
kubectl logs <pod-name> -n learner-report
```

### Test Health Endpoints
```sh
# Backend health check
curl http://backend.local/health

# Frontend accessibility
curl http://frontend.local/
```

---

# Helm Deployment Instructions

## Quick Start - Minikube
```sh
# Start Minikube
minikube start

# Enable ingress
minikube addons enable ingress

# Create namespace
kubectl create namespace learner-report

# Deploy using Helm
helm install learner-report ./learner-report -f ./learner-report/values-development.yaml

# Get Minikube IP and update hosts file
minikube ip
# Add to C:\Windows\System32\drivers\etc\hosts:
# <minikube-ip> backend.local
# <minikube-ip> frontend.local
```

## Quick Start - AWS EKS
```sh
# Deploy to EKS
helm install learner-report ./learner-report -f ./learner-report/values-production.yaml

# Configure DNS records in Route 53
# api.learner-report.com -> Load Balancer
# app.learner-report.com -> Load Balancer
```

## Helm Management Commands
```sh
# Update deployment
helm upgrade learner-report ./learner-report -f ./learner-report/values-development.yaml

# Check status
helm status learner-report

# Uninstall
helm uninstall learner-report

# Dry run to validate templates
helm install learner-report ./learner-report --dry-run --debug
```

## Kubernetes Troubleshooting

### Check Pod Status and Logs
```sh
# Check all resources
kubectl get all -n learner-report

# Check pod status
kubectl describe pod <pod-name> -n learner-report

# Check pod logs
kubectl logs <pod-name> -n learner-report

# Follow logs in real-time
kubectl logs -f <pod-name> -n learner-report
```

### Health Check Issues
```sh
# Test health endpoint manually
kubectl port-forward svc/learner-report-backend-service 3001:3001 -n learner-report
curl http://localhost:3001/health

# Check if backend is responding
kubectl exec -it <backend-pod> -n learner-report -- wget -qO- http://localhost:3000/health
```

### Common Issues and Solutions

#### 1. **Readiness/Liveness Probe Failures**
```sh
# Symptoms: Pod keeps restarting, shows "Unhealthy" status
# Check if the health endpoint exists
kubectl exec -it <backend-pod> -n learner-report -- curl http://localhost:3000/health

# If 404, the backend needs the /health endpoint added
```

#### 2. **Image Pull Issues**
```sh
# Symptoms: "ImagePullBackOff" or "ErrImagePull"
# Check if image exists
docker pull crashrik/learner-report-backend:latest
docker pull crashrik/learner-report-frontend:latest

# Force image pull
kubectl delete pod <pod-name> -n learner-report
```

#### 3. **Database Connection Issues**
```sh
# Check environment variables
kubectl describe pod <backend-pod> -n learner-report | grep -A 10 "Environment:"

# Test MongoDB connection
kubectl exec -it <backend-pod> -n learner-report -- node -e "
const mongoose = require('mongoose');
mongoose.connect(process.env.ATLAS_URI)
  .then(() => console.log('Connected to MongoDB'))
  .catch(err => console.error('MongoDB connection error:', err));
"
```

#### 4. **Ingress Issues**
```sh
# Check ingress status
kubectl get ingress -n learner-report
kubectl describe ingress <ingress-name> -n learner-report

# Check if ingress controller is running
kubectl get pods -n ingress-nginx

# Test ingress from inside cluster
kubectl run test-pod --image=curlimages/curl -it --rm -- /bin/sh
curl http://backend.local/health
```

#### 5. **Service Discovery Issues**
```sh
# Test service connectivity
kubectl exec -it <frontend-pod> -n learner-report -- nslookup learner-report-backend-service.learner-report.svc.cluster.local

# Test service endpoints
kubectl get endpoints -n learner-report
```

### Rebuild and Redeploy After Code Changes
```sh
# 1. Rebuild Docker images
cd learnerReportCS_backend
docker build -t crashrik/learner-report-backend:latest .
docker push crashrik/learner-report-backend:latest

cd ../learnerReportCS_frontend  
docker build -t crashrik/learner-report-frontend:latest .
docker push crashrik/learner-report-frontend:latest

# 2. Force Kubernetes to pull new images
kubectl rollout restart deployment/learner-report-backend -n learner-report
kubectl rollout restart deployment/learner-report-frontend -n learner-report

# 3. Check rollout status
kubectl rollout status deployment/learner-report-backend -n learner-report
kubectl rollout status deployment/learner-report-frontend -n learner-report
```

### Debug Network Connectivity
```sh
# Test frontend to backend communication
kubectl exec -it <frontend-pod> -n learner-report -- wget -qO- http://learner-report-backend-service:3001/health

# Test external access
curl http://backend.local/health
curl http://frontend.local/
```

