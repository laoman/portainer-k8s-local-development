# Portainer Kubernetes Local Development

## Table of Contents
- [Prerequisites](#prerequisites)
- [Installing Homebrew 🍺](#-installing-homebrew)
- [Installing kubectl ☸️](# -installing-kubectl)
- [Installing Colima ☸️](# -installing-colima)
- [Installing Portainer 🐳](#-installing-portainer-in-kubernetes)
- [Installing Kubernetes Dashboard ☸️](#-installing-kubernetes-dashboard)
- [Installing Jenkins 🤖](#-installing-jenkins)
- [Additional Resources](#-additional-resources)

## 🐳 Setting up Colima for Kubernetes Development

Colima is a container runtime for macOS that provides a lightweight and fast environment for running containers. It supports Kubernetes, making it a great choice for local development.

### Prerequisites

- macOS
- Homebrew (package manager for macOS)

### Installation Steps

Before installing Colima, you'll need Homebrew package manager. Here's how to install it:

1. **Install Homebrew**
## 🍺 Installing Homebrew

Before installing Colima, you'll need Homebrew package manager. Here's how to install it:

   ```bash
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   ```

2. **Add Homebrew to PATH**

   For MacOS with Apple Silicon (M1/M2):
   ```bash
   echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zshrc
   source ~/.zshrc
   ```

   For Intel Mac:
   ```bash
   echo 'eval "$(/usr/local/bin/brew shellenv)"' >> ~/.zshrc
   source ~/.zshrc
   ```

3. **Verify Installation**

   ```bash
   brew --version
   ```

---
## ☸️ Installing kubectl

Before proceeding with Kubernetes setup, you'll need kubectl command-line tool:

1. **Install kubectl**

   ```bash
   brew install kubectl
   ```

2. **Verify Installation**

   ```bash
   kubectl version --client
   ```

3. **Enable kubectl Shell Completion (optional)**

   For zsh users:
   ```bash
   echo '[[ $commands[kubectl] ]] && source <(kubectl completion zsh)' >> ~/.zshrc
   source ~/.zshrc
   ```

   For bash users:
   ```bash
   echo 'source <(kubectl completion bash)' >> ~/.bashrc
   source ~/.bashrc
   ```

---
## ☸️ Installing Colima
1. **Install Colima**

   ```bash
   brew install colima
   ```

2. **Start Colima with Kubernetes Support**

   ```bash
   colima start --with-kubernetes
   ```

3. **Verify Installation**

   ```bash
   kubectl get nodes
   ```

### Usage

**Start Colima**
```bash
colima start --with-kubernetes
```

**Stop Colima**
```bash
colima stop
```

**Delete Colima Instance**
```bash
colima delete
```

### Troubleshooting

To check Colima status:
```bash
colima status
```

To view Colima logs:
```bash
colima logs
```

---

### 🐳 Installing Portainer in Kubernetes

Portainer provides a web interface to manage your Kubernetes cluster. Follow these steps to install it:

1. **Create Portainer Namespace and RBAC**

   First, apply the cluster role binding that gives Portainer the necessary permissions:

   ```bash
   kubectl apply -f portainer-cluster-admin-role.yaml
   ```

2. **Deploy Portainer**

   Deploy Portainer using the deployment configuration:

   ```bash
   kubectl apply -f portainer-deployment.yaml
   ```

3. **Verify the Installation**

   Check if Portainer pods are running:

   ```bash
   kubectl get pods -n portainer
   ```

   You should see output similar to:
   ```
   NAME                         READY   STATUS    RESTARTS   AGE
   portainer-xxx-xxx           1/1     Running   0          1m
   ```

4. **Access Portainer UI**

   Once deployed, you can access Portainer at:
   - URL: `http://localhost:30777`
   - Default credentials:
     - Username: `admin`
     - Password: You'll be prompted to create one on first login

### Portainer Components

The installation consists of:
- A Kubernetes namespace called `portainer`
- A deployment running the Portainer container
- A NodePort service exposing port 30777
- A PersistentVolumeClaim for data persistence
- RBAC configuration for cluster management

### Troubleshooting Portainer

To check Portainer's status:

```bash
# Check pods
kubectl get pods -n portainer

# Check service
kubectl get svc -n portainer

# View Portainer logs
kubectl logs -n portainer deployment/portainer
```
---
### ☸️ Installing Kubernetes Dashboard

The Kubernetes Dashboard provides a web UI to manage your cluster. Here's how to install and access it:

1. **Create Installation Script**

   Create a file named `install-kubernetes-dashboard.sh`:

   ```bash
   // filepath: install-kubernetes-dashboard.sh
   #!/bin/bash

   # Create the dashboard namespace if it doesn't exist
   kubectl create namespace kubernetes-dashboard

   # Apply the admin role binding
   kubectl apply -f kubernetes-dashboard-admin-role.yaml

   # Install Kubernetes Dashboard
   kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml

   # Wait for the pod to be ready
   echo "Waiting for dashboard pod to be ready..."
   kubectl wait --namespace kubernetes-dashboard \
     --for=condition=ready pod \
     --selector=k8s-app=kubernetes-dashboard \
     --timeout=90s

   # Get the token
   echo "Generating token..."
   kubectl -n kubernetes-dashboard create token admin-user
   ```

2. **Run Installation Script**

   Make the script executable and run it:
   ```bash
   chmod +x install-kubernetes-dashboard.sh
   ./install-kubernetes-dashboard.sh
   ```

   This script will:
   - Create the kubernetes-dashboard namespace
   - Apply the admin role configuration
   - Install the dashboard components
   - Wait for the dashboard to be ready
   - Generate and display an access token
   
   > ⚠️ **Important**: Save the token displayed in the output - you'll need it to log in

3. **Start the Dashboard Proxy**

   ```bash
   kubectl proxy
   ```

4. **Access the Dashboard**

   Open the following URL in your browser:
   ```
   http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
   ```
   > 📝 **Alternative Access Method**  
   > You can also use port forwarding instead of proxy:
   > ```bash
   > kubectl port-forward -n kubernetes-dashboard service/kubernetes-dashboard 8443:443
   > ```
   > Then access the dashboard at: `https://localhost:8443`  
   > Note: You'll need to accept the self-signed certificate warning in your browser.

5. **Login**
   - Select "Token" as authentication method
   - Paste the token from step 2
   - Click "Sign In"

---

### 🤖 Installing Jenkins

Jenkins can be deployed in your Kubernetes cluster using the provided configuration. Here's how to install and access it:

1. **Deploy Jenkins**

   Apply the Jenkins configuration:
   ```bash
   kubectl apply -f jenkins.yaml
   ```

   This will create:
   - Jenkins namespace
   - Persistent Volume Claim (10Gi)
   - Jenkins deployment
   - NodePort service

2. **Wait for Jenkins to Start**

   Check the pod status:
   ```bash
   kubectl get pods -n jenkins
   ```

   Wait until the pod status shows `Running`.

3. **Get Initial Admin Password**

   ```bash
   kubectl exec -n jenkins $(kubectl get pods -n jenkins -o jsonpath='{.items[0].metadata.name}') -- cat /var/jenkins_home/secrets/initialAdminPassword
   ```

4. **Access Jenkins**

   Open in your browser:
   ```
   http://localhost:30000
   ```

   > 📝 **Port Information**  
   > The configuration exposes two ports:
   > - Web UI: `30000` (http://localhost:30000)
   > - Agent Port: `30001` (for Jenkins agents)

5. **Complete Installation**
   - Enter the initial admin password from step 3
   - Install suggested plugins
   - Create your admin user
   - Start using Jenkins

### Troubleshooting Jenkins

Check Jenkins status:
```bash
# View pod status
kubectl get pods -n jenkins

# Check logs
kubectl logs -n jenkins deployment/jenkins

# Check service
kubectl get svc -n jenkins
```

### Additional Resources

- [Colima GitHub Repository](https://github.com/abiosoft/colima)
- [Kubernetes Documentation](https://kubernetes.io/docs/home/)
