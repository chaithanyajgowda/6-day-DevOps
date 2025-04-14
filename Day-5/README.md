# day 5 summary
---

# Docker: Drawbacks

1. No Self-Maintenance of Containers:
   Docker containers don't manage themselves. If a container crashes or stops, Docker doesn't automatically restart or replace it unless managed externally.

2. No Built-in Load Balancer:
   Docker alone cannot distribute traffic evenly across multiple containers or servers.

3. No Auto-Scaling Support: 
   Docker doesn't adjust the number of running containers automatically based on load or traffic.

---

# Kubernetes (K8s)

Kubernetes is an open-source container orchestration platform used to automate the deployment, scaling, and management of containerized applications.


# Background

- Originally developed by Google.
- Donated to the Cloud Native Computing Foundation (CNCF) in 2014.

---

# Key Features of Kubernetes

1. Auto-Scaling 
   Automatically increases or decreases the number of running containers based on current traffic.

2. Auto-Healing  
   Replaces and restores failed or deleted containers automatically.

3. Load Balancing  
   Distributes incoming traffic across containers to ensure even resource usage.

4. Platform Independence  
   Works across different operating systems and cloud platforms.

5. Rollbacks  
   Enables switching back to previous application versions with ease.

6. Health Monitoring  
   Continuously monitors the health of containers and replaces unhealthy ones.

7. Fault Tolerance  
   Notifies about node failures and maintains high availability.

8. Orchestration  
   Manages containerized workloads and services automatically.

---

# Kubernetes Architecture

# Cluster
A cluster is a collection of nodes. It includes:

# Master Node (Control Plane)
- API Server – Entry point for all administrative tasks.
- etcd – A key-value store that stores cluster state.
- Controller Manager – Governs controllers and sends updates to the API server.
- Scheduler – Assigns workloads (pods) to nodes based on resource availability.

# Worker Node
- Kubelet – Registers the node with the cluster and manages pods.
- Pod – The smallest deployable unit, created by the Kubelet.
- Container Runtime – Executes containers inside pods (e.g., Docker, containerd).
- Kube-Proxy – Maintains network rules and enables communication.

---

# Cluster Setup on AWS EC2

# 1. Launch an EC2 Instance
- OS: Ubuntu
- Instance Type: `t2.medium` (K8s requires more CPU)
- Ports: Enable SSH and HTTP
- Storage: 25 GB
- Connect to your instance:
```bash
sudo su
apt update -y
```

---

# Installing Tools

# Install `kubectl`
```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
mkdir -p ~/.local/bin
mv ./kubectl ~/.local/bin/kubectl
kubectl version --client
```

# Install `eksctl`
```bash
ARCH=amd64
PLATFORM=$(uname -s)_$ARCH
curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"
tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz
sudo mv /tmp/eksctl /usr/local/bin
```

# Install AWS CLI
```bash
apt install unzip -y
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version
```

---

# Configure AWS Access Keys

1. Go to AWS Console → IAM → Create a new user.
2. Assign Administrator Access.
3. Generate and download the Access Keys (.csv).
4. On your EC2 server:
```bash
aws configure
```

---

# Create a Kubernetes Cluster using `eksctl`
```bash
eksctl create cluster \
--name blue-cluster \
--region sa-east-1 \
--zone sa-east-1a,sa-east-1b \
--nodegroup-name red-node \
--node-type t2.medium \
--nodes 2
```

> Takes about 10 minutes to create the cluster.

---

# Use Killercoda (Alternative to EC2)
- Visit: [https://killercoda.com/](https://killercoda.com/)
- Choose: Kubernetes → Kubernetes 1.32
- Sign in with Google, start a Playground (session lasts 1 hour)

---

# Working with Namespaces and Pods (Imperative vs Declarative)

# Create a Namespace (Using YAML)
```yaml
# blue.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: white-2
```
```bash
kubectl apply -f blue.yaml
kubectl get ns
```

# Create a Pod
```yaml
# blue-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: blue-pod2
  namespace: white-2
spec:
  containers:
  - name: c-1
    image: daviddocker526/ipl
    ports:
    - containerPort: 80
```
```bash
kubectl apply -f blue-pod.yaml
kubectl get pods -n white-2
kubectl describe pod blue-pod2 -n white-2
```

---

# ReplicaSet – Ensures Desired Number of Pods

# Create a ReplicaSet
```yaml
# white-replica.yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: blue-replica1
  namespace: white-2
spec:
  replicas: 5
  selector:
    matchLabels:
      app: ipl
  template:
    metadata:
      labels:
        app: ipl
    spec:
      containers:
      - name: c-2
        image: daviddocker526/ipl
        ports:
        - containerPort: 80
```
```bash
kubectl apply -f white-replica.yaml
kubectl get rs -n white-2
kubectl get pods -n white-2
```

> ReplicaSets help reduce downtime and maintain availability by ensuring the specified number of pods are always running.

---