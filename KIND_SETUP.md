# KIND CLUSTER SETUP STEP BY STEP:-
Kind is a tool for running local Kubernetes clusters using Docker container "nodes". kind was primarily designed for testing Kubernetes itself, but may be used for local development or CI.

- For more details see the design documentation and installation guide.
  1. https://kind.sigs.k8s.io/docs/design/initial
  2. https://kind.sigs.k8s.io/

## Overview
This guide explains how to set up a local Kubernetes cluster using:
- Docker (Container Runtime)
- Kind (Kubernetes IN Docker)
- kubectl (Kubernetes CLI)

## Architecture

```text
+----------------------+
|      Ubuntu VM       |
+----------------------+
           |
           v
+----------------------+
|       Docker         |
|  (Container Runtime) |
+----------------------+
           |
           v
+----------------------+
|        Kind          |
| Kubernetes in Docker |
+----------------------+
           |
           v
+----------------------+
| Kubernetes Cluster   |
| 1 Control Plane      |
| 2 Worker Nodes       |
+----------------------+
```

---

## Prerequisites

| Resource | Minimum | Recommended |
|----------|----------|-------------|
| CPU | 2 vCPU | 4 vCPU |
| RAM | 4 GB | 8 GB |
| Disk | 20 GB | 40 GB |

---

## Step 1: Update Ubuntu

```bash
sudo apt update
sudo apt upgrade -y
```

Verify Ubuntu version:

```bash
lsb_release -a
```

---

## Step 2: Install Docker

```bash
sudo apt install docker.io -y
```

Enable Docker:

```bash
sudo systemctl enable docker
sudo systemctl start docker
```

Verify:

```bash
docker --version
sudo systemctl status docker
```

---

## Step 3: Configure Docker for Non-Root User

```bash
sudo usermod -aG docker $USER
newgrp docker
docker ps
```

---

## Step 4: Install kubectl

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client
```

---

## Step 5: Install Kind

```bash
curl -Lo ./kind https://kind.sigs.k8s.io/dl/latest/kind-linux-amd64
chmod +x kind
sudo mv kind /usr/local/bin/
kind version
```

---

## Step 6: Create Kind Configuration

Create file:

```bash
nano kind-config.yaml
```

Paste:

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4

nodes:
- role: control-plane
- role: worker
- role: worker
```

---

## Step 7: Create Kubernetes Cluster

```bash
kind create cluster --name dev-cluster --config kind-config.yaml
```

Verify:

```bash
kubectl get nodes
```

Expected:

```text
dev-cluster-control-plane   Ready
dev-cluster-worker          Ready
dev-cluster-worker2         Ready
```

---

## Step 8: Cluster Verification

```bash
kubectl cluster-info
kubectl get nodes
kubectl get pods -A
```

---

## Step 9: Verify Docker Containers

```bash
docker ps
```

Example:

```text
dev-cluster-control-plane
dev-cluster-worker
dev-cluster-worker2
```

---

## Step 10: Deploy Your First Application

Create deployment:

```bash
kubectl create deployment nginx --image=nginx
```

Verify:

```bash
kubectl get deployments
kubectl get pods
```

---

## Step 11: Expose the Application

```bash
kubectl expose deployment nginx --port=80 --type=NodePort
```

Check service:

```bash
kubectl get svc
```

---

## Useful Commands

### Nodes

```bash
kubectl get nodes
```

### Pods

```bash
kubectl get pods -A
```

### Deployments

```bash
kubectl get deployments
```

### Services

```bash
kubectl get svc -A
```

### Describe Pod

```bash
kubectl describe pod <pod-name>
```

### View Logs

```bash
kubectl logs <pod-name>
```

### Docker Containers

```bash
docker ps
```

---

## Delete Cluster

```bash
kind delete cluster --name dev-cluster
```

Verify:

```bash
kind get clusters
```

---

## Troubleshooting

### Docker Permission Denied

```bash
sudo usermod -aG docker $USER
newgrp docker
```

### kubectl Cannot Connect

```bash
kubectl cluster-info
kubectl config get-contexts
```

### Check Cluster Logs

```bash
docker ps
docker logs <container-id>
```

---

## Learning Roadmap

1. Pods
2. ReplicaSets
3. Deployments
4. Services
5. ConfigMaps
6. Secrets
7. Persistent Volumes
8. Ingress
9. Helm
10. RBAC
11. Monitoring (Prometheus + Grafana)
12. CI/CD Integration
13. Kubernetes Troubleshooting

---

## Conclusion

You now have a fully functional Kubernetes cluster running locally using Kind and Docker. This setup is ideal for DevOps learning, Kubernetes practice, home labs, and interview preparation.
