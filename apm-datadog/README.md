# STEPS:

## 1. Update system and install dependencies (if not already installed):
```bash
sudo apt update
sudo apt upgrade -y
sudo apt install -y curl git docker.io
```

## 2. Start Minikube with Docker driver:
```bash
minikube start --driver=docker
```

## 3. Install Kompose (Docker Compose to Kubernetes converter) (if not already installed):
```bash
curl -L https://github.com/kubernetes/kompose/releases/download/v1.26.1/kompose-linux-amd64 -o kompose
chmod +x kompose
sudo mv ./kompose /usr/local/bin/kompose
```

## 4. Clone GitHub Repository and go to the folder:
```bash
git clone https://github.com/datascientest/apm-datadog.git
cd apm-datadog
```

## 5. Build images in Minikube (if not already built):
```bash
eval $(minikube docker-env)
docker build -t calendar-app -f Dockerfile.calendar .
docker build -t notes-app -f Dockerfile.notes .
```

## 6. Verify images in Minikube:
```bash
minikube image list
```

## 7. Create the values.yaml file:
```bash
touch values.yaml
```
- The file content is contained in the `values.yaml` file.

## 8. Add Datadog Helm repository:
```bash
helm repo add datadog https://helm.datadoghq.com
helm repo update
```

## 9. Install Datadog Agent:
```bash
helm install datadog datadog/datadog -f values.yaml
```

## 10. Verify installation:
```bash
kubectl get pods | grep datadog
```

## 11. Convert Docker Compose to Kubernetes Manifests:
```bash
kompose convert -f docker-compose.yaml -o k8s-manifests/
cd k8s-manifests
```

## 12. Update deployment manifests calendar-app-deployment.yaml and notes-app-deploymant.yaml:
- Replace the image names in both manifests with the local image names `docker.io/library/calendar-app:latest`and `docker.io/library/notes-app:latest`.
- Change the number of replicas in both manifests from 1 to 2.
- In both manifests, add `ÌmagePullPolicy: Never` under `spec:` and `containers:` to use the local images. 

## 13. Rename service manifests (because they are not allowed to contain `_`), replace `_` with `-` in the service names inside the files and come back to main folder:
```bash
cd k8s-manifests/
mv calendar_app-service.yaml calendar-app-service.yaml
mv notes_app-service.yaml notes-app-service.yaml
cd ..
```
- Replace the names `calendar_app`with `calendar-app` and `notes_app` with `notes-app` inside the `calendar-app-service.yaml` and `notes-app-service.yaml` files. 

## 14. Create a service manifest for the database:
```bash
touch db-service.yaml
```
- The file content is contained in the `db-service.yaml` file.

## 15. Apply Kubernetes manifests to deploy the application:
```bash
kubectl apply -f .
```
- or:
```bash
cd ..
kubectl apply -f k8s-manifests/
```

## 16. Verify if all pods have been created:
```bash
kubectl get pods
```

## 17. Verify if all resources have been created:
```bash
kubectl get all
```

## 18. Port-forward to the notes service:
```bash
kubectl port-forward service/notes-app 8080:8080 &
```

## 19. Port-forward to the calendar service:
```bash
kubectl port-forward service/calendar-app 9090:9090
```

## 20. See the apps in browser:
```bash
firefox
```
- To see the notes app enter:
`http://localhost:8080`
- To see the calendar app enter:
`http://localhost:9090`

## 21. Send HTTP requests in a new terminal for the notes service:
- Before you run the following commands, open a new Linux terminal:
```bash
curl -X GET http://localhost:8080/notes
curl -X POST http://localhost:8080/notes -H "Content-Type: application/json" -d '{"text": "Test note"}'
```

## 22. Verify on Datadog:
```bash
https://app.datadoghq.eu/
```
- Go to APM -> Traces. You should see traces for your GET and POST requests.

## 23. Delete all applied resources:
```bash
kubectl delete -f .
```
- or:
```bash
cd ..
kubectl delete -f k8s-manifests/
```

## 24. Delete all persistent volumes and persistent volume claims:
```bash
kubectl delete pv --all
kubectl delete pvc --all
```

## 25. Uninstall Datadog with Helm:
```bash
helm uninstall datadog
```

## 26. Delete all remaining resources:
```bash
kubectl delete daemonset datadog --ignore-not-found
kubectl delete deployment datadog-cluster-agent --ignore-not-found
kubectl delete service datadog-cluster-agent --ignore-not-found
kubectl delete pvc -l app.kubernetes.io/instance=datadog --ignore-not-found
kubectl delete secret -l app.kubernetes.io/instance=datadog --ignore-not-found
```
