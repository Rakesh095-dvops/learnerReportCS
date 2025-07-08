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

7. **Access the Services locally**:
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

- If this doesn't work then it could be minikube ingress issue 
To ensure that the `learner-report-cs-backend-ingress` is listed and has an external IP or hostname assigned, follow these steps:

### 1. **Check the Ingress Resource**
```sh
kubectl get ingress -n learner-report
```
This will display the Ingress resource and its associated external IP or hostname.
### 2. **Describe the Ingress**
If the external IP or hostname is not assigned, describe the Ingress to check for any issues:
```sh
kubectl describe ingress learner-report-cs-backend-ingress -n learner-report 
```
### 3. **Verify Ingress Controller**
Ensure that the Ingress controller is running. Check the pods in the `kube-system` ,`ingress-nginx` namespace:
```sh
kubectl get pods -n kube-system
kubectl get pods -n ingress-nginx
```
Look for a pod with a name like `ingress-nginx-controller`. If it is not running, you may need to enable the Ingress addon in Minikube:
```sh
minikube addons enable ingress
```
If issue still persist ,then delete the batch job and reinstall again 
```sh
kubectl delete job ingress-nginx-admission-create -n ingress-nginx
kubectl delete job ingress-nginx-admission-patch -n ingress-nginx
```
wait for pod to start and check logs 
```sh
kubectl get pods -n ingress-nginx -w
kubectl logs <ingress-nginx-controller-pod-name> -n ingress-nginx
```
>***TIPS*** - If ingress is successful the logs will reflect the ingress routing details correct 
```log
I0706 19:40:17.432903       7 status.go:304] "updating Ingress status" namespace="learner-report" ingress="learner-report-cs-backend-ingress" currentValue=null newValue=[{"ip":"192.168.49.2"}]
I0706 19:40:17.443917       7 event.go:377] Event(v1.ObjectReference{Kind:"Ingress", Namespace:"learner-report", Name:"learner-report-cs-backend-ingress", UID:"c4c0b5a0-25ad-4332-8b1c-5da162c3f3cf", APIVersion:"networking.k8s.io/v1", ResourceVersion:"19853", FieldPath:""}): type: 'Normal' reason: 'Sync' Scheduled for sync
```
```log 
I0706 20:11:26.796453       7 controller.go:213] "Backend successfully reloaded"
I0706 20:11:26.796867       7 event.go:377] Event(v1.ObjectReference{Kind:"Pod", Namespace:"ingress-nginx", Name:"ingress-nginx-controller-56d7c84fd4-tdgsv", UID:"cac45de3-1aa7-465a-98d8-07683285e9cb", APIVersion:"v1", ResourceVersion:"19642", FieldPath:""}): type: 'Normal' reason: 'RELOAD' NGINX reload triggered due to a change in configuration
I0706 20:12:17.082543       7 status.go:304] "updating Ingress status" namespace="learner-report" ingress="learner-report-cs-frontend-ingress" currentValue=null newValue=[{"ip":"192.168.49.2"}]
I0706 20:12:17.106739       7 event.go:377] Event(v1.ObjectReference{Kind:"Ingress", Namespace:"learner-report", Name:"learner-report-cs-frontend-ingress", UID:"c03d3bcc-142c-4898-9a14-aa5cfbf8ab0a", APIVersion:"networking.k8s.io/v1", ResourceVersion:"21667", FieldPath:""}): type: 'Normal' reason: 'Sync' Scheduled for sync
W0706 20:12:53.184984       7 controller.go:1216] Service "learner-report/learner-report-cs-frontend-service" does not have any active Endpoint.

```
---

### 4. **Test the Ingress**
If the Ingress is configured correctly start tunnel `minikube tunnel`
```sh
minikube ip
minikube service ingress-nginx-controller -n ingress-nginx
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
