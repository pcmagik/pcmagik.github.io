---
tags:
  - Kubernetes
---

# Kubernetes (K3s) Setup Guide

A comprehensive guide for installing and configuring a lightweight Kubernetes cluster using K3s on Ubuntu.

## What is K3s?

K3s is a lightweight, certified Kubernetes distribution designed for resource-constrained environments. It bundles everything needed into a single binary under 100MB.

---

## Server Installation

### Install K3s Server (Control Plane)

```bash
curl -sfL https://get.k3s.io | sh -
```

### Verify Node Readiness

Wait approximately 30 seconds, then verify:

```bash
sudo k3s kubectl get node
```

### Retrieve Node Token

The token is required for joining worker nodes:

```bash
sudo cat /var/lib/rancher/k3s/server/node-token
```

### Get Kubeconfig

```bash
sudo cat /etc/rancher/k3s/k3s.yaml
```

> **Tip:** Copy this file to your local machine for remote `kubectl` access. Replace the `server` address `127.0.0.1` with your server's IP.

```bash
# Copy kubeconfig to local machine
scp your_username@10.0.0.10:/etc/rancher/k3s/k3s.yaml ~/.kube/k3s.yaml
export KUBECONFIG=~/.kube/k3s.yaml
```

---

## Agent (Worker Node) Installation

### Join a Worker Node

```bash
curl -sfL https://get.k3s.io | K3S_URL=https://10.0.0.10:6443 K3S_TOKEN=<your_node_token> sh -
```

Replace `10.0.0.10` with your server's IP and `<your_node_token>` with the token from the server.

### Verify Cluster

On the server node:

```bash
sudo k3s kubectl get nodes
```

---

## kubectl Basics

```bash
# List all pods across namespaces
kubectl get pods -A

# List nodes
kubectl get nodes -o wide

# Describe a specific resource
kubectl describe pod <pod_name> -n <namespace>

# View logs
kubectl logs <pod_name> -n <namespace>
kubectl logs <pod_name> -n <namespace> -f  # follow

# Execute command in a pod
kubectl exec -it <pod_name> -n <namespace> -- /bin/bash
```

---

## Deployments and ReplicaSets

### Create a Deployment

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.25
          ports:
            - containerPort: 80
```

```bash
kubectl apply -f deployment.yaml
kubectl get deployments
kubectl get pods
```

### Scale a Deployment

```bash
kubectl scale deployment nginx-deployment --replicas=5
```

---

## Services

### ClusterIP (Internal Only)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-clusterip
spec:
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
  type: ClusterIP
```

### NodePort (External Access via Node IP)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
  type: NodePort
```

### LoadBalancer

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-lb
spec:
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
  type: LoadBalancer
```

---

## Ingress with Traefik

K3s ships with Traefik as the default Ingress Controller.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
  annotations:
    traefik.ingress.kubernetes.io/router.entrypoints: web
spec:
  rules:
    - host: app.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: nginx-clusterip
                port:
                  number: 80
```

```bash
kubectl apply -f ingress.yaml
kubectl get ingress
```

---

## Persistent Volumes and StorageClasses

### PersistentVolumeClaim

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: local-path
  resources:
    requests:
      storage: 5Gi
```

> **Note:** K3s includes the `local-path` StorageClass by default.

### Use PVC in a Pod

```yaml
spec:
  containers:
    - name: app
      image: nginx:1.25
      volumeMounts:
        - mountPath: /data
          name: app-storage
  volumes:
    - name: app-storage
      persistentVolumeClaim:
        claimName: app-pvc
```

---

## ConfigMaps and Secrets

### ConfigMap

```bash
kubectl create configmap app-config --from-literal=APP_ENV=production --from-literal=LOG_LEVEL=info
```

```yaml
# Or as YAML
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: production
  LOG_LEVEL: info
```

### Secret

```bash
kubectl create secret generic app-secret \
  --from-literal=DB_PASSWORD=your_password_here \
  --from-literal=API_KEY=your_api_token_here
```

---

## Helm Basics

### Install Helm

```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

### Install a Chart (Example: Rancher)

```bash
helm repo add rancher-latest https://releases.rancher.com/server-charts/latest
helm repo update

kubectl create namespace cattle-system
helm install rancher rancher-latest/rancher \
  --namespace cattle-system \
  --set hostname=rancher.example.com \
  --set bootstrapPassword=your_password_here
```

### Verify Installation

```bash
kubectl -n cattle-system rollout status deploy/rancher
kubectl -n cattle-system get deploy rancher
```

---

## Troubleshooting

### Port Conflicts

```bash
# Check what is using a port
sudo lsof -i :6443
sudo lsof -i :6444

# Kill blocking process
sudo kill -9 <PID>
```

### Service Logs

```bash
# Server logs
sudo journalctl -u k3s.service -f

# Agent logs
sudo journalctl -u k3s-agent.service -f
```

### Firewall Ports

Ensure these ports are open:

| Port | Protocol | Description |
|---|---|---|
| 6443 | TCP | Kubernetes API |
| 8472 | UDP | Flannel VXLAN |
| 10250 | TCP | Kubelet metrics |
| 51820 | UDP | Flannel WireGuard |

```bash
sudo ufw status
sudo ufw allow 6443/tcp
sudo ufw allow 8472/udp
sudo ufw allow 10250/tcp
```

### Full Reinstall

```bash
# Uninstall server
sudo /usr/local/bin/k3s-uninstall.sh

# Uninstall agent
sudo /usr/local/bin/k3s-agent-uninstall.sh
```
