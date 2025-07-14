# Learner Report Application

This repository contains the source code and deployment instructions for the Learner Report application, a full-stack project with a separate frontend and backend. The guide covers local development with Docker, deployment to Kubernetes using raw manifests, and a more advanced deployment using a Helm chart for different environments (local Minikube and production AWS EKS).

## Project Structure

The project is organized into distinct directories for the frontend, backend, Kubernetes manifests, and the Helm chart.

```
.
├── learnerReportCS_backend/  # Node.js backend application source
│   ├── ...
│   └── Dockerfile            # Dockerfile for building the backend image
│
├── learnerReportCS_frontend/ # React frontend application source
│   ├── ...
│   └── Dockerfile            # Dockerfile for building the frontend image
│
├── learner-report/           # Helm chart for deploying the application
│   ├── charts/
│   ├── templates/            # Kubernetes manifest templates
│   ├── Chart.yaml            # Chart metadata
│   ├── values.yaml           # Default configuration values
│   ├── values-development.yaml # Values for local Minikube deployment
│   └── values-production.yaml  # Values for production AWS EKS deployment
│
├── misc/
│   └── k8s/                  # Raw Kubernetes manifests (alternative to Helm dev build)
│       ├── backend/
│       └── frontend/
│
└── README.md                 
```

## Table of Contents

1.  [Docker Build and Run](#docker-build-and-run-instructions)
    *   [Build Docker Images](#1-build-docker-images)
    *   [Run Docker Containers](#2-run-docker-containers)
2.  [Kubernetes Deployment (Raw Manifests)](#kubernetes-deployment-with-raw-manifests)
    *   [Prerequisites](#prerequisites)
    *   [Apply Kubernetes Resources](#apply-kubernetes-resources)
    *   [Verify and Access Services](#verify-and-access-services)
    *   [Port Forwarding (Optional)](#port-forwarding-optional)
3.  [Helm Chart Deployment](#helm-chart-deployment)
    *   [Prerequisites](#helm-prerequisites)
    *   [Deployment Commands](#deployment-commands)
    *   [Helm Management](#helm-management-commands)
    *   [Environment-Specific Configuration](#environment-specific-configuration)
    *   [Customizing Values](#customizing-values)
4.  [Troubleshooting and Debugging](#troubleshooting-and-debugging)
    *   [General Docker/NGINX Logs](#general-dockernginx-logs)
    *   [General Kubernetes Debugging](#general-kubernetes-debugging)
    *   [Common Kubernetes Issues & Solutions](#common-kubernetes-issues-and-solutions)
    *   [Ingress Troubleshooting](#ingress-troubleshooting)
    *   [Rebuilding and Redeploying](#rebuilding-and-redeploying-after-code-changes)
    *   [Local Build Error Fixes](#local-build-error-fixes)
5.  [Developer Utilities](#developer-utilities)
    *   [Git Branch Management](#git-branch-management)

---

## Docker Build and Run Instructions

Follow these instructions to build and run the application services locally using Docker.

### 1. Build Docker Images

First, build the Docker images for the backend and frontend services and push them to a container registry (e.g., Docker Hub).

```sh
# Build backend (from the learnerReportCS_backend directory)
cd learnerReportCS_backend
docker build -t learner-report-backend:latest .
docker tag learner-report-backend:latest crashrik/learner-report-backend:latest
docker push crashrik/learner-report-backend:latest

# Build frontend (from the learnerReportCS_frontend directory)
cd ../learnerReportCS_frontend
docker build -t learner-report-frontend:latest .
docker tag learner-report-frontend:latest crashrik/learner-report-frontend:latest
docker push crashrik/learner-report-frontend:latest
```

### 2. Run Docker Containers

Run the containers locally. Ensure you have a `.env` file in your project root for the backend environment variables.

```sh
# Run backend container
docker run -d --name learner-backend --env-file .env -p 3001:3000 crashrik/learner-report-backend:latest

# Run frontend container
docker run -d --name learner-frontend -p 80:3000 crashrik/learner-report-frontend:latest
```

---

## Kubernetes Deployment with Raw Manifests

This method uses `kubectl` to apply raw manifest files. For a more manageable approach, see the [Helm Chart Deployment](#helm-chart-deployment) section.

### Prerequisites

1.  **Start Minikube**:
    ```sh
    minikube start
    ```
2.  **Enable Ingress Addon**:
    ```sh
    minikube addons enable ingress
    ```
3.  **Create Namespace**:
    ```sh
    kubectl create namespace learner-report
    ```

### Apply Kubernetes Resources

Apply the manifest files to create the deployments, services, secrets, configmaps, and ingresses.

```sh
# Apply Backend Resources
kubectl apply -f ./misc/k8s/backend/backend-secret.yaml
kubectl apply -f ./misc/k8s/backend/backend-configmap.yaml
kubectl apply -f ./misc/k8s/backend/backend-deployment.yaml
kubectl apply -f ./misc/k8s/backend/backend-service.yaml
kubectl apply -f ./misc/k8s/backend/backend-ingress.yaml

# Apply Frontend Resources
kubectl apply -f ./misc/k8s/frontend/frontend-configmap.yaml
kubectl apply -f ./misc/k8s/frontend/frontend-deployment.yaml
kubectl apply -f ./misc/k8s/frontend/frontend-service.yaml
kubectl apply -f ./misc/k8s/frontend/frontend-ingress.yaml
```

### Verify and Access Services

1.  **Check Resources**: Verify that all pods, services, and other resources are running correctly.
    ```sh
    kubectl get all -n learner-report
    ```
2.  **Get Minikube IP**:
    ```sh
    minikube ip
    ```
3.  **Update Hosts File**: Add the following entries to your `/etc/hosts` file (or `C:\Windows\System32\drivers\etc\hosts` on Windows), replacing `<minikube-ip>` with the output from the previous command.
    ```
    <minikube-ip> backend.local
    <minikube-ip> frontend.local
    ```
4.  **Access Services**:
    *   Backend API: `http://backend.local/`
    *   Frontend App: `http://frontend.local/`

### Port Forwarding (Optional)

If you need to access a service directly without setting up Ingress, use `port-forward`.

```sh
# Forward backend service to localhost:3001
kubectl port-forward svc/learner-report-cs-backend-service 3001:3001 -n learner-report

# Forward frontend service to localhost:3000
kubectl port-forward svc/learner-report-cs-frontend-service 3000:3000 -n learner-report
```

---

## Helm Chart Deployment

This is the recommended method for deploying the application to Kubernetes, as it simplifies management and configuration for different environments.

### Helm Prerequisites

#### For Minikube (Local Development)

1.  **Install Tools**: Ensure you have Minikube, kubectl, and Helm installed.
2.  **Start Minikube and Enable Ingress**:
    ```sh
    minikube start
    minikube addons enable ingress
    ```
3.  **Create Namespace**:
    ```sh
    kubectl create namespace learner-report
    ```

#### For AWS EKS (Production)

1.  **Install Tools**: Ensure you have AWS CLI, eksctl, kubectl, and Helm installed.
2.  **Create EKS Cluster**:
    ```sh
    eksctl create cluster --name learner-report --region ap-south-1
    ```
3.  **Install NGINX Ingress Controller**:
    ```sh
    helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
    helm repo update
    helm install ingress-nginx ingress-nginx/ingress-nginx
    ```

### Deployment Commands

#### Minikube Local Development

```sh
# Deploy using development values
helm install learner-report ./learner-report -n learner-report -f ./learner-report/values-development.yaml

# To update an existing deployment
helm upgrade learner-report ./learner-report -n learner-report -f ./learner-report/values-development.yaml

# Update your hosts file with the Minikube IP
# <minikube-ip> backend.local
# <minikube-ip> frontend.local
```

#### AWS EKS Production

```sh
# Deploy using production values
helm install learner-report ./learner-report -n learner-report -f ./learner-report/values-production.yaml

# To update an existing deployment
helm upgrade learner-report ./learner-report -n learner-report -f ./learner-report/values-production.yaml

# After deployment, configure DNS records in Route 53 to point to the ALB created by the Ingress controller
# api.learner-report.com -> ALB DNS name
# app.learner-report.com -> ALB DNS name
```

### Helm Management Commands

```sh
# List all Helm releases in the namespace
helm list -n learner-report

# Check the status of a release
helm status learner-report -n learner-report

# View the computed values for a release
helm get values learner-report -n learner-report

# Uninstall a release
helm uninstall learner-report -n learner-report

# Perform a dry run to see what would be deployed
helm install learner-report ./learner-report -f ./values-development.yaml --dry-run --debug
```

### Environment-Specific Configuration

-   **Development (Minikube)**:
    -   Frontend: `http://frontend.local`
    -   Backend: `http://backend.local`
    -   DNS: Local `/etc/hosts` file mapping.
    -   TLS: Disabled.
-   **Production (AWS)**:
    -   Frontend: `https://app.learner-report.com`
    -   Backend: `https://api.learner-report.com`
    -   DNS: Managed via Route 53.
    -   TLS: Automated with cert-manager and Let's Encrypt (configured in `values-production.yaml`).

### Customizing Values

To override default settings, create a custom values file (`values-custom.yaml`) and apply it during installation.

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

```sh
# Deploy with your custom values
helm install learner-report ./learner-report -f ./learner-report/values-custom.yaml
```

---

## Troubleshooting and Debugging

### General Docker/NGINX Logs

If an application running in Docker is not working, check the NGINX logs inside the container.

```sh
# Replace <container-id> with the actual ID from `docker ps`
docker exec -it <container-id> cat /var/log/nginx/access.log
docker exec -it <container-id> cat /var/log/nginx/error.log
```

### General Kubernetes Debugging

```sh
# Get an overview of all resources and their status
kubectl get all -n learner-report -o wide

# Get detailed information about a specific pod
kubectl describe pod <pod-name> -n learner-report

# View the logs of a pod
kubectl logs <pod-name> -n learner-report

# Follow logs in real-time
kubectl logs -f <pod-name> -n learner-report
```

### Common Kubernetes Issues and Solutions

*   **Readiness/Liveness Probe Failures**: Pods keep restarting or show `Unhealthy`.
    *   **Solution**: Ensure the `/health` endpoint is implemented in the backend and correctly configured in the `deployment.yaml` or Helm chart. Test it manually:
        ```sh
        kubectl exec -it <backend-pod> -n learner-report -- curl http://localhost:3000/health
        ```
*   **Image Pull Issues**: Pods show `ImagePullBackOff` or `ErrImagePull` status.
    *   **Solution**: Verify the image name and tag are correct and that the image exists in the registry. Ensure your Kubernetes cluster has credentials to pull from the private registry if needed. Force a re-pull:
        ```sh
        kubectl delete pod <pod-name> -n learner-report
        ```
*   **Database Connection Issues**: Backend fails to connect to the database.
    *   **Solution**: Check that the database URL, username, and password are correct in the Kubernetes Secret and are properly mounted as environment variables in the backend pod.
        ```sh
        kubectl describe pod <backend-pod> -n learner-report | grep -A 10 "Environment:"
        ```
*   **Service Discovery Issues**: Frontend cannot communicate with the backend.
    *   **Solution**: Test DNS resolution from within the frontend pod. The backend should be reachable at `http://<service-name>.<namespace>.svc.cluster.local`.
        ```sh
        kubectl exec -it <frontend-pod> -n learner-report -- nslookup learner-report-backend-service.learner-report.svc.cluster.local
        ```

### Ingress Troubleshooting

If you cannot access your application via the Ingress hostnames (`frontend.local`, `backend.local`):

1.  **Check Ingress Resource**: Ensure it has an assigned address.
    ```sh
    kubectl get ingress -n learner-report
    ```
2.  **Describe Ingress**: If there's no address, check for errors.
    ```sh
    kubectl describe ingress <ingress-name> -n learner-report
    ```
3.  **Verify Ingress Controller**: Make sure the NGINX Ingress controller pods are running.
    ```sh
    kubectl get pods -n ingress-nginx
    ```
    If not, you may need to re-enable or reinstall it:
    ```sh
    minikube addons enable ingress
    ```
4.  **Check Controller Logs**: Look for errors in the controller logs.
    ```sh
    kubectl logs <ingress-nginx-controller-pod-name> -n ingress-nginx
    ```

### Rebuilding and Redeploying After Code Changes

1.  **Rebuild and Push Docker Images**: After making code changes, you must rebuild and push new images with the `latest` tag (or a new version tag).
    ```sh
    # Rebuild and push backend
    cd learnerReportCS_backend
    docker build -t crashrik/learner-report-backend:latest .
    docker push crashrik/learner-report-backend:latest

    # Rebuild and push frontend
    cd ../learnerReportCS_frontend  
    docker build -t crashrik/learner-report-frontend:latest .
    docker push crashrik/learner-report-frontend:latest
    ```
2.  **Force a Rolling Update**: Kubernetes will not automatically pull an image that has the same tag (`:latest`). You must trigger a rollout to force it to pull the new version.
    ```sh
    kubectl rollout restart deployment/learner-report-backend -n learner-report
    kubectl rollout restart deployment/learner-report-frontend -n learner-report
    ```
3.  **Check Rollout Status**:
    ```sh
    kubectl rollout status deployment/learner-report-backend -n learner-report
    ```

### Local Build Error Fixes

If you encounter errors while running `npm install` locally:

*   **Dependency Conflicts**:
    ```sh
    npm install --legacy-peer-deps --force
    ```
*   **`react-scripts` Not Found**:
    ```sh
    npm install react-scripts
    ```
*   **Outdated `caniuse-lite`**:
    ```sh
    npx update-browserslist-db@latest --force
    ```

---

## Developer Utilities

### Git Branch Management

This script helps find the most recently updated branches in the repository.

#### Bash
```sh
git branch -a --sort=-committerdate --format="%(committerdate:relative)%09%(refname:short)"
```

#### PowerShell
```ps1
git branch -r --format="%(committerdate:iso8601), %(committerdate:relative) - %(refname:short)" | Select-String -NotMatch 'HEAD' | Sort-Object -Descending
```