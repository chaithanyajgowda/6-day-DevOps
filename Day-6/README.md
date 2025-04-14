# day 6 summary
---

# Kubernetes Deployment and Jenkins CI/CD Automation (Summary Notes) 
Date: 03/04/2025  

---

# 1. Kubernetes Deployment & Service Setup

# 1.1 Deployment YAML Configuration
- This YAML configures a Deployment and ReplicaSet using the `apps/v1` API version.
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: blue-deploy
  namespace: blue-ns
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
        - name: c-1
          image: daviddocker526/ipl-srh:latest
          ports:
            - containerPort: 80
```

---

# 1.2 Killercoda Free Labs
- Log in using Google.
- Basic command to verify nodes:
```bash
kubectl get nodes
```

---

# 1.3 EC2 Setup for Kubernetes

# Initial Commands
```bash
sudo su
kubectl create ns blue-ns
kubectl get ns
vi blue-deployment.yaml
cat blue-deployment.yaml
kubectl apply -f blue-deployment.yaml
kubectl get deployment -n blue-ns
kubectl get rs -n blue-ns
kubectl get pods -n blue-ns
kubectl describe pod <pod-name> -n blue-ns
```

# Service YAML Configuration
```yaml
apiVersion: v1
kind: Service
metadata:
  name: blue-service
  namespace: blue-ns
spec:
  selector:
    app: ipl
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer
```

> Note: Killercoda doesn’t support LoadBalancer type services (free version). The EXTERNAL-IP will show as pending.

# Apply & Check All Resources
```bash
kubectl apply -f blue-service.yaml
kubectl get all -n blue-ns
```

# Replica and Pod Management
```bash
kubectl delete pod <pod-name> -n blue-ns
# Edit replica count in YAML (replicas: 1)
vi blue-deployment.yaml
kubectl apply -f blue-deployment.yaml
kubectl get pods -n blue-ns
```

# File Editing Reminder
- In `vi`, press `i` to insert/edit.
- After editing, press `Esc`, type `:wq` to save and exit.

# Cleanup
```bash
kubectl delete deployment blue-deploy -n blue-ns
kubectl delete svc blue-service -n blue-ns
```

---

# 2. Jenkins Setup on EC2 for CI/CD

# 2.1 EC2 Configuration
- Use instance type `t3.medium`, 20 GB storage.
- Enable port `22` (SSH) and allow All TCP ports in security groups.

# 2.2 Putty Access
- Paste public IP.
- Upload `.ppk` key pair and connect.

---

# 2.3 Jenkins Installation (Ubuntu)
```bash
sudo su
apt update -y

# Add Jenkins key and source
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt-get update
sudo apt-get install jenkins
```

# 2.4 Java Installation (Required by Jenkins)
```bash
sudo apt update
sudo apt install openjdk-17-jdk -y
java --version
```

---

# 2.5 Start Jenkins Server
- Default Jenkins port: `8080`
- Access in browser:  
  `http://<your-ec2-public-ip>:8080`
- Retrieve password:
```bash
cat /var/lib/jenkins/secrets/initialAdminPassword
```

- Paste password in browser → Install Suggested Plugins
- Create Admin User (username, password, email)

---

# 2.6 Essential Jenkins Plugins to Install
From: Manage Jenkins > Plugins > Available Plugins

Install:
1. AWS Credentials  
2. Kubernetes Client API  
3. Kubernetes Credentials Version  
4. Kubernetes  
5. Kubernetes CLI  
6. Kubernetes Credentials Provider  
7. Kubernetes :: Pipeline :: DevOps Steps  

>  Important: Login again after installing plugins to avoid HTTP ERROR 403.

---

# 3. AWS Access & Kubeconfig Setup

# 3.1 AWS Keys and Jenkins Credentials
- Generate Access Keys (AWS Console > IAM > Users)
- In Jenkins:  
  *Manage Jenkins > Credentials > Global Credentials > Add Credentials > AWS Credentials*  
  - Paste Access Key & Secret  
  - Give ID and Description

---

# 3.2 Install AWS CLI and Kubectl on Jenkins Server
```bash
sudo apt install unzip -y

# Install AWS CLI
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Configure AWS
aws configure
# Enter access key, secret key, region

# Install kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
mv kubectl /usr/local/bin/
kubectl version --client
```

---

# 3.3 Setup Kubeconfig File
- Get the kubeconfig file using EKS commands (refer to the instructor’s Word document).
- Switch to Jenkins user:
```bash
sudo su - jenkins
aws configure
```
- Copy kubeconfig to Jenkins home directory.
- Verify:
```bash
ls
cat config
```
- Add this file as secret file credential in Jenkins under Global Credentials

---

# 4. Jenkins Pipeline for Kubernetes Deployment

# 4.1 Create New Item
- Select type: Pipeline
- In the script section: Paste the EKS deployment pipeline code(2 stages)
- Update GitHub repo URL
- Save & Apply

# 4.2 Run the Pipelines
- Click Build Now
- Monitor Console Output

---

# 4.3 Verifying Deployment
Back in EC2 terminal:
```bash
kubectl get all
```
- Confirm that pods, services, and deployments are running as expected.

---
