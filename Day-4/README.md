# Day 4- Summary
---

# DevOps Deployment Using Docker – Summary

# 01/04/2025  
Roles Involved:
- DevOps Engineers  
- Testers  

# Objective:
- Use repositories from both engineers (DevOps & Tester)
- Deploy application first in Docker (containerized environment)
- Then move it to production stage

---

# Docker
- Docker is a containerization tool that packages an application and its dependencies into a single unit (container).
- Simplifies application deployment across environments.

# Docker Architecture:
1. Dockerfile– Contains a set of instructions to build an image.
2. Docker Image – Built from Dockerfile; contains source code & dependencies.
3. Docker Registry (e.g., Docker Hub)** – Stores and distributes images.
4. Docker Container – A running instance of a Docker image.

---

# Ubuntu Instance Setup (for Docker Deployment)

# Commands:
```bash
sudo
sudo su
apt update -y
```

# Install Docker:
- Visit: [https://docs.docker.com/engine/install/ubuntu](https://docs.docker.com/engine/install/ubuntu)
- Install Docker and verify:
```bash
docker --version
```

---

# Create and Edit Files:
```bash
touch index.html
vi index.html    # Write your HTML content
# Press ESC, then type :wq and press Enter to save
cat index.html
```

```bash
touch Dockerfile
vi Dockerfile    # Create Dockerfile and insert the following content
```

# Sample Dockerfile:
```Dockerfile
FROM ubuntu:latest
WORKDIR /COLOR
RUN apt-get update -y
RUN apt-get install apache2 -y
COPY . /var/www/html
EXPOSE 80
CMD ["apachectl", "-D", "FOREGROUND"]
```

# Build and Run Docker Image:
```bash
docker image build -t red .
docker run -d --name blue -p 80:80 red
```

---

# Docker Management Commands

| Command | Purpose |
|--------|---------|
| `docker ps` | View running containers |
| `docker ps -a` | View all containers |
| `docker logs <container>` | Check logs/errors |
| `service apache2 status` | Check Apache status |
| `docker inspect <container>` | Get container info |
| `curl <container-ip>:80` | Test container via IP |

# If error: Update Dockerfile
- Fix syntax issues, e.g.,:
```Dockerfile
CMD ["apachectl", "-D", "FOREGROUND"]
```
- Rebuild:
```bash
docker image build -t green-v3 .
docker run -d --name yellow -p 81:80 green-v3
```

# Open App to Public
- Go to AWS Console > Instance > Security Groups > Edit Inbound Rules
- Add:  
  - Type:All Traffic  
  - Source: 0.0.0.0/0  
- Access app via:  
  `http://<public-ip>:<port>`  
  e.g., `http://3.39.226.214:81/`

---

# DockerHub Integration

# Push to Docker Hub
1. [Sign up](https://hub.docker.com)
2. Log in from terminal:
```bash
docker login
```

3. Tag & Push Image:
```bash
docker tag green-v2:latest chaithanya97/green-v2:latest
docker push chaithanya97/green-v2:latest
```

4. Repeat for other images:
```bash
docker tag red:latest chaithanya97/red:latest
docker push chaithanya97/red:latest
```

---

# Docker Cleanup

| Command | Description |
|--------|-------------|
| `docker rmi red` | Remove image |
| `docker rm black` | Remove container |
| `docker pause black` / `docker unpause black` | Pause/Resume container |
| `docker stop black` / `docker restart black` | Stop/Restart container |
| `docker rm $(docker ps -aq)` | Remove all containers |

---

# Pull Common Images

```bash
docker pull ubuntu:latest
docker pull node:latest
docker pull mysql:latest
docker pull nginx:latest
docker pull python:latest
```

---

# Example Deployment from DockerHub

```bash
docker run -d --name ipl-v2 -p 86:80 daviddocker526/ipl
```
Access at:  
`http://3.39.226.214:86/`

# Enter into Container:
```bash
docker exec -it ipl-v2 bash
cd /var/www/html
ls
```

---

