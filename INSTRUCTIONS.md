# 📋 Local Setup Instructions

This guide will walk you through setting up the **Kubernetes React DevOps Project** on your local machine.

---

## 📋 Prerequisites

Before you begin, ensure you have the following installed:

| Tool | Version | Purpose |
|------|---------|---------|
| **Git** | Latest | Clone the repository |
| **Node.js** | 14.x or higher | React app dependency |
| **npm** | 6.x or higher | Package manager |
| **Docker** | Latest | Build and run containers |
| **Docker Compose** | Latest | Multi-container orchestration |
| **Minikube** | Latest | Local Kubernetes cluster |
| **kubectl** | Latest | Kubernetes CLI |

### Installation Links
- [Git](https://git-scm.com/downloads)
- [Node.js & npm](https://nodejs.org/)
- [Docker Desktop](https://www.docker.com/products/docker-desktop) (includes Docker & Docker Compose)
- [Minikube](https://minikube.sigs.k8s.io/docs/start/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)

---

## 🚀 Step 1: Clone the Repository

```bash
git clone https://github.com/Vamsi-1409/kubernetes-react-devops-project.git
cd kubernetes-react-devops-project
```

---

## 📦 Step 2: Install Dependencies

Navigate to the app directory and install Node.js dependencies:

```bash
cd app
npm install
cd ..
```

---

## 🐳 Step 3: Run Locally with Docker Compose (Recommended for Quick Testing)

### Build and Start Services

```bash
docker-compose up --build
```

The application will be available at: **http://localhost:8082**

### Stop Services

```bash
docker-compose down
```

---

## 🏃 Step 4: Run React App Directly (Without Docker)

If you prefer to run the React app directly on your machine:

```bash
cd app
npm start
```

The application will automatically open at **http://localhost:3000**

### Run Tests

```bash
cd app
npm test
```

---

## ⸱ Step 5: Build Docker Image Manually

If you want to build the Docker image yourself:

```bash
docker build -t react-app:latest .
docker run -p 3000:3000 react-app:latest
```

---

## ☸️ Step 6: Deploy to Local Kubernetes (Minikube)

### 6.1 Start Minikube

```bash
minikube start --cpus 4 --memory 4096
```

### 6.2 Verify Minikube is Running

```bash
minikube status
kubectl cluster-info
```

### 6.3 Build Docker Image in Minikube

```bash
# Use Minikube's Docker daemon
eval $(minikube docker-env)

# Build the image
docker build -t react-app:latest .
```

### 6.4 Deploy to Kubernetes

```bash
# Apply all Kubernetes manifests
kubectl apply -f k8s/

# Verify deployments
kubectl get pods
kubectl get svc
kubectl get ingress
```

### 6.5 Access the Application

```bash
# Option 1: Using Minikube service command
minikube service react-service

# Option 2: Port forward
kubectl port-forward svc/react-service 3000:80

# Option 3: Check service details
kubectl get svc react-service
```

Then visit the URL provided by the command above.

---

## 📊 Step 7: Monitor with Prometheus & Grafana (Optional)

### 7.1 Install Helm (if not already installed)

```bash
# On macOS with Homebrew
brew install helm

# Or download from: https://helm.sh/docs/intro/install/
```

### 7.2 Add Prometheus Helm Repository

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

### 7.3 Install Prometheus Stack

```bash
helm install monitoring prometheus-community/kube-prometheus-stack -n monitoring --create-namespace
```

### 7.4 Access Grafana

```bash
# Port forward Grafana
kubectl port-forward svc/monitoring-grafana 3000:80 -n monitoring
```

Visit: **http://localhost:3000**

**Default Credentials:**
- Username: `admin`
- Password: `prom-operator`

### 7.5 Access Prometheus

```bash
# Port forward Prometheus
kubectl port-forward svc/monitoring-kube-prometheus 9090:9090 -n monitoring
```

Visit: **http://localhost:9090**

---

## 📈 Step 8: Enable Horizontal Pod Autoscaler (HPA)

### 8.1 Ensure Metrics Server is Running

```bash
# Check if metrics-server is installed
kubectl get deployment metrics-server -n kube-system

# If not installed, install it
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

### 8.2 Create HPA

```bash
kubectl autoscale deployment react-app \
  --cpu-percent=50 \
  --min=2 \
  --max=5

# Verify HPA
kubectl get hpa
```

### 8.3 Monitor HPA Status

```bash
kubectl get hpa -w
```

Wait a few minutes for metrics to appear (CPU usage may show `<unknown>` initially).

---

## 🔍 Useful Commands

### View Logs

```bash
# Get pod logs
kubectl logs <pod-name>

# Stream logs in real-time
kubectl logs -f <pod-name>

# Logs from deployment
kubectl logs -l app=react-app
```

### Access Pod Terminal

```bash
kubectl exec -it <pod-name> -- /bin/sh
```

### Describe Resources

```bash
# Describe a pod
kubectl describe pod <pod-name>

# Describe a service
kubectl describe svc react-service

# Describe an ingress
kubectl describe ingress <ingress-name>
```

### Delete Deployments

```bash
# Delete all Kubernetes resources
kubectl delete -f k8s/

# Or delete specific resources
kubectl delete deployment react-app
kubectl delete svc react-service
```

### Stop Minikube

```bash
minikube stop
minikube delete  # To remove cluster completely
```

---

## 🐛 Troubleshooting

### Issue: Pod in CrashLoopBackOff

**Solution:**
```bash
# Check pod logs
kubectl logs <pod-name>

# Describe the pod for detailed info
kubectl describe pod <pod-name>

# Common causes: Image not found, application error, missing environment variables
```

### Issue: ImagePullBackOff

**Solution:**
```bash
# Ensure Docker image exists in Minikube
eval $(minikube docker-env)
docker images

# Rebuild image if necessary
docker build -t react-app:latest .
```

### Issue: Service not accessible

**Solution:**
```bash
# Verify service is running
kubectl get svc

# Check service endpoints
kubectl get endpoints

# Use port-forward as workaround
kubectl port-forward svc/react-service 3000:80
```

### Issue: HPA shows `<unknown>` CPU

**Solution:**
```bash
# Wait a few minutes for metrics to populate
# Check if metrics are available
kubectl top pods
kubectl top nodes

# If still not working, reinstall metrics-server
kubectl delete -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

### Issue: Minikube Docker image not found

**Solution:**
```bash
# Always use Minikube's Docker daemon for building
eval $(minikube docker-env)
docker build -t react-app:latest .

# Disable image pull policy for local testing
# Update k8s deployment to use: imagePullPolicy: Never
```

---

## 📁 Project Structure

```
kubernetes-react-devops-project/
├── app/                          # React application
│   ├── src/                      # Source files
│   ├── public/                   # Static assets
│   └── package.json              # Node dependencies
├── k8s/                          # Kubernetes manifests
│   ├── deployment.yaml           # Deployment configuration
│   ├── service.yaml              # Service configuration
│   ├── ingress.yaml              # Ingress rules
│   ├── configmap.yaml            # Config maps
│   └── hpa.yaml                  # Horizontal Pod Autoscaler
├── Dockerfile                    # Production Dockerfile
├── docker-compose.yaml           # Docker Compose configuration
├── README.md                     # Project overview
└── INSTRUCTIONS.md               # This file
```

---

## ✅ Verification Checklist

After setup, verify everything is working:

- [ ] Git repository cloned successfully
- [ ] Node modules installed (`npm install` completed)
- [ ] Docker image builds successfully
- [ ] Docker Compose services start (`docker-compose up`)
- [ ] React app runs at http://localhost:3000 (direct) or http://localhost:8082 (Docker)
- [ ] Minikube cluster starts successfully
- [ ] Kubernetes deployment is running (`kubectl get pods`)
- [ ] Service is accessible (`minikube service react-service`)
- [ ] Metrics server is running (for HPA)
- [ ] Prometheus & Grafana are accessible (optional)

---

## 🆘 Getting Help

If you encounter issues:

1. **Check logs:** `kubectl logs <pod-name>`
2. **Describe resources:** `kubectl describe pod <pod-name>`
3. **Verify prerequisites:** Ensure all tools are installed
4. **Restart Minikube:** `minikube stop && minikube start`
5. **Check Docker:** `docker ps` and `docker images`
6. **Review GitHub Issues:** Check if similar issues exist

---

## 📚 Additional Resources

- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Docker Documentation](https://docs.docker.com/)
- [React Documentation](https://react.dev/)
- [Minikube Documentation](https://minikube.sigs.k8s.io/)
- [Helm Documentation](https://helm.sh/docs/)

---

## 🎯 Next Steps

After successful setup, consider:

- Modify the React app in `app/src/`
- Update Kubernetes manifests in `k8s/`
- Push Docker image to Docker Hub
- Set up CI/CD pipeline with GitHub Actions
- Configure custom monitoring dashboards in Grafana

---

## 👨‍💻 Contributing

Feel free to contribute! Submit issues and pull requests to improve this project.

---

**Last Updated:** June 2026  
**Maintained by:** Vamsi Krishna
