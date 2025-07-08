# Docker Build and Run Instructions

## Docker Build
```sh
docker build -t learner-report-backend:latest .
docker tag learner-report-backend:latest crashrik/learner-report-backend:latest
docker push crashrik/learner-report-backend:latest

docker build -t learner-report-frontend:latest .
docker tag learner-report-frontend:latest crashrik/learner-report-frontend:latest
docker push crashrik/learner-report-frontend:latest
```

## Docker Run
```sh
docker run -d --env-file .env -p 3000:3000 crashrik/learner-report-backend:latest
docker run -it --env-file .env -p 80:80 crashrik/learner-report-frontend:latest
```

## Troubleshooting
```sh
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

