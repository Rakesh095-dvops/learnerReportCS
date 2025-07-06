# Docker file build 
```sh
docker build -t learner-report-backend:latest .  
docker tag  learner-report-backend:latest  crashrik/learner-report-backend:latest 
docker push crashrik/learner-report-backend:latest
docker build -t learner-report-frontend:latest .  
docker tag  learner-report-frontend:latest  crashrik/learner-report-frontend:latest 
```
# Docker run file 
```sh
docker run -d --env-file .env -p 3000:3000 crashrik/learner-report-backend:lates
docker run -it --env-file .env -p 80:3000 crashrik/learner-report-frontend:latest 
```
# Troubleshoot 
```sh 
docker ps -a 
docker build --no-cache --progress=plain -t your-frontend .
```
## Error building frontend fix applied
```
#8 [4/5] RUN npm install --silent
#8 3.361 npm notice 
#8 3.361 npm notice New major version of npm available! 8.19.4 -> 11.4.2
#8 3.362 npm notice Changelog: <https://github.com/npm/cli/releases/tag/v11.4.2>
#8 3.362 npm notice Run `npm install -g npm@11.4.2` to update!
#8 3.364 npm notice
#8 ERROR: process "/bin/sh -c npm install --silent" did not complete successfully: exit code: 1
------
 > [4/5] RUN npm install --silent:
 notice New major version of npm available! 8.19.4 -> 11.4.2
3.362 npm notice Changelog: <https://github.com/npm/cli/releases/tag/v11.4.2>
3.362 npm notice Run `npm install -g npm@11.4.2` to update!
3.364 npm notice
------

```
## k8s manifest files 

The `Ingress` files for both backend and frontend have been created:

- **Backend Ingress**: backend-ingress.yaml
- **Frontend Ingress**: frontend-ingress.yaml

### Steps to Run and Test Locally with Minikube:

1. **Start Minikube**:
   ```sh
   minikube start
   ```

2. **Enable Ingress Addon**:
   ```sh
   minikube addons enable ingress
   ```

3. **Apply the Namespace**:
   If the namespace learner-report is not already created, create it:
   ```sh
   kubectl create namespace learner-report
   ```

4. **Apply Backend Resources**:
   ```sh
   kubectl apply -f .\misc\k8s\backend\backend-secret.yaml
   kubectl apply -f .\misc\k8s\backend\backend-configmap.yaml
   kubectl apply -f .\misc\k8s\backend\backend-deployment.yaml
   kubectl apply -f .\misc\k8s\backend\backend-service.yaml
   kubectl apply -f .\misc\k8s\backend\backend-ingress.yaml
   ```

5. **Apply Frontend Resources**:
   ```sh
   kubectl apply -f .\misc\k8s\frontend\frontend-configmap.yaml
   kubectl apply -f .\misc\k8s\frontend\frontend-deployment.yaml
   kubectl apply -f .\misc\k8s\frontend\frontend-service.yaml
   kubectl apply -f .\misc\k8s\frontend\frontend-ingress.yaml
   ```
6. **Verify Resources**:
   Check if all resources are running:
   ```sh
   kubectl get all -n learner-report
   ```

7. **Access the Services**:
   - Get the Minikube IP:
     ```sh
     minikube ip
     ```
   - Add the following entries to your `/etc/hosts` file (or hosts on Windows):
     ```
     <minikube-ip> backend.local
     <minikube-ip> frontend.local
     ```
   - Test the backend and frontend:
     - Backend: `http://backend.local/student`, `http://backend.local/admin`, etc.
     - Frontend: `http://frontend.local/`

8. **Port Forwarding (Optional)**:
   If you want to test without Ingress:
   ```sh
   kubectl port-forward svc/learner-report-cs-backend-service 3001:3001 -n learner-report
   kubectl port-forward svc/learner-report-cs-frontend-service 3000:3000 -n learner-report
   ```
-----
>
**Note - Port Forwarding (Optional)**:
If you want to test without Ingress:
```sh
kubectl get all -n learner-report -o wide
kubectl port-forward svc/learner-report-cs-backend-service 3001:3001 -n learner-report
kubectl port-forward svc/learner-report-cs-frontend-service 3000:3000 -n learner-report
```
***Note - git branch how to do see which has latest commit ***

```sh
# #!/bin/bash
- git branch -[a,l,r] --format="%(committerdate:iso8601), %(committerdate:relative) - %(refname:short)" | grep -v [H]EAD | sort -r
```
```ps1
# PowerShell 
- git branch -r --format="%(committerdate:iso8601), %(committerdate:relative) - %(refname:short)" | Select-String -NotMatch 'HEAD' | Sort-Object -Descending
```
----
