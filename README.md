# 🚀 Jenkins + Ansible + Docker + AWS EC2 CI/CD Deployment

## 📌 Project Overview

This project demonstrates an automated **CI/CD deployment pipeline** using **Jenkins, Ansible, Docker, Docker Hub, GitHub, SSH, and AWS EC2**.

The project uses multiple AWS EC2 servers to automate the process of taking application source code from GitHub, building a Docker image, pushing the image to Docker Hub, and deploying the containerized application onto two separate Docker servers.

The complete deployment process is automated using Jenkins and Ansible, reducing the need for manual deployment.

---

## 🏗️ Project Architecture

The project consists of **four AWS EC2 instances**:

| Server | Role                                |
| ------ | ----------------------------------- |
| EC2-1  | Jenkins Server                      |
| EC2-2  | Ansible / Docker Image Build Server |
| EC2-3  | Docker Container Server 1           |
| EC2-4  | Docker Container Server 2           |

### Architecture Flow

```text
                    ┌─────────────────┐
                    │     GitHub      │
                    │   Source Code   │
                    │   Dockerfile    │
                    └────────┬────────┘
                             │
                             │ Git Pull
                             ▼
                    ┌─────────────────┐
                    │  Jenkins EC2    │
                    │     Server      │
                    │                 │
                    │ CI/CD Pipeline  │
                    └────────┬────────┘
                             │
                       SSH + SSH Agent
                             │
                             ▼
                    ┌─────────────────┐
                    │   Ansible EC2   │
                    │     Server      │
                    │                 │
                    │ Build Docker    │
                    │ Image           │
                    └────────┬────────┘
                             │
                             │ Docker Push
                             ▼
                    ┌─────────────────┐
                    │   Docker Hub    │
                    │ Docker Registry │
                    └────────┬────────┘
                             │
                       Docker Pull
                             │
                             ▼
              ┌──────────────────────────┐
              │        Ansible           │
              │      Playbook            │
              └────────────┬─────────────┘
                           │
                 ┌─────────┴─────────┐
                 │                   │
                 ▼                   ▼
        ┌────────────────┐   ┌────────────────┐
        │ Docker Server 1│   │ Docker Server 2│
        │    EC2-3       │   │    EC2-4       │
        │                │   │                │
        │ Docker Pull    │   │ Docker Pull    │
        │       ↓        │   │       ↓        │
        │ Run Container  │   │ Run Container  │
        └────────────────┘   └────────────────┘
```

---

## 🔄 CI/CD Workflow

The deployment process follows these steps:

### 1. Developer Pushes Code to GitHub

The application source code and `Dockerfile` are maintained in a GitHub repository.

```text
Developer
    ↓
GitHub Repository
    ↓
Application Code + Dockerfile
```

---

### 2. Jenkins Pulls the Source Code

Jenkins runs on an AWS EC2 instance.

The Jenkins pipeline connects to the GitHub repository and pulls the latest source code and Dockerfile.

```text
GitHub
   ↓
Jenkins
   ↓
Latest Source Code
```

---

### 3. Jenkins Connects to Ansible Server

Jenkins uses **SSH with an SSH Agent** to securely connect to the Ansible EC2 server.

The required project files are transferred to the Ansible server.

```text
Jenkins EC2
     │
     │ SSH / SSH Agent
     ▼
Ansible EC2
```

---

### 4. Docker Image is Built

The Ansible server uses the received `Dockerfile` to build the Docker image.

```text
Dockerfile
     ↓
docker build
     ↓
Docker Image
```

The image is tagged before being pushed to Docker Hub.

Example:

```bash
docker build -t username/mywebsite:v1 .
```

---

### 5. Docker Image is Pushed to Docker Hub

After building the image, the Ansible server pushes it to Docker Hub.

```text
Ansible Server
      ↓
Docker Image
      ↓
Docker Hub
```

Docker Hub acts as the central image registry.

---

### 6. Ansible Deploys the Image

Ansible uses an **inventory file** and **playbook** to manage the two Docker container servers.

The playbook connects to both target EC2 instances through SSH.

```text
                 Ansible Server
                       │
              Ansible Playbook
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
       Docker Server 1     Docker Server 2
```

---

### 7. Docker Image is Pulled

The two target servers pull the Docker image from Docker Hub.

```text
Docker Hub
    │
    ├──────────► Docker Server 1
    │
    └──────────► Docker Server 2
```

---

### 8. Containers are Started

After pulling the image, Ansible starts containers on both Docker servers.

```text
Docker Image
     ↓
docker run
     ↓
Container
     ↓
Application Running
```

As a result, the same application runs as a container on both EC2 instances.

---

# 🛠️ Technologies Used

* **AWS EC2** – Cloud infrastructure
* **GitHub** – Source code management
* **Jenkins** – CI/CD automation
* **SSH / SSH Agent** – Secure server communication
* **Ansible** – Configuration management and deployment automation
* **Docker** – Application containerization
* **Docker Hub** – Docker image registry
* **Linux/Ubuntu** – Server operating system

---

# 📂 Project Structure

```text
project/
│
├── Dockerfile
├── Jenkinsfile
├── index.html
├── ansible/
│   ├── inventory
│   └── playbook.yml
│
└── README.md
```

---

# 🔐 Server Communication

The project uses SSH-based communication between the servers.

```text
Jenkins Server
      │
      │ SSH
      ▼
Ansible Server
      │
      │ SSH + Ansible
      ├──────────────► Docker Server 1
      │
      └──────────────► Docker Server 2
```

SSH keys are configured to allow secure communication between the required EC2 instances.

---

# ⚙️ Jenkins Pipeline Stages

The Jenkins pipeline performs the following major stages:

```text
┌─────────────────────┐
│   Start Pipeline    │
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│ Clone GitHub Repo   │
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│ Transfer Files to   │
│ Ansible Server      │
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│ Build Docker Image  │
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│ Tag Docker Image    │
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│ Push Image to       │
│ Docker Hub          │
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│ Run Ansible         │
│ Playbook            │
└──────────┬──────────┘
           ↓
     ┌─────┴─────┐
     ↓           ↓
 Server 1     Server 2
     ↓           ↓
 Container    Container
     └─────┬─────┘
           ↓
   Application Deployed
```

---

# 📦 Ansible Deployment

Ansible manages the two Docker servers from the Ansible control node.

The playbook is responsible for tasks such as:

* Connecting to the Docker servers
* Installing Docker if required
* Starting Docker service
* Pulling the latest Docker image
* Removing the previous container
* Creating and starting the new container
* Ensuring the application is running

Example deployment concept:

```text
Ansible Control Node
        │
        │ Ansible
        ▼
┌───────────────────────┐
│ Docker Server 1       │
│ Pull Image            │
│ Remove Old Container  │
│ Run New Container     │
└───────────────────────┘

        │
        │ Ansible
        ▼

┌───────────────────────┐
│ Docker Server 2       │
│ Pull Image            │
│ Remove Old Container  │
│ Run New Container     │
└───────────────────────┘
```

---

# ☁️ AWS Infrastructure

The infrastructure is hosted on AWS EC2.

```text
                    AWS Cloud
                       │
        ┌──────────────┼──────────────┐
        │              │              │
     Jenkins        Ansible        Docker
      EC2             EC2          EC2 Instances
                                    │
                              ┌─────┴─────┐
                              │           │
                           Server 1    Server 2
```

Security groups are configured to allow the required communication between the instances.

Common ports used include:

| Port | Purpose                |
| ---- | ---------------------- |
| 22   | SSH                    |
| 80   | HTTP / Web Application |
| 8080 | Jenkins Web Interface  |

---

# 🎯 Project Objectives

The main objectives of this project are:

* Automate application deployment
* Understand Jenkins CI/CD pipelines
* Learn Docker image creation and management
* Understand Docker Hub image repositories
* Automate server configuration using Ansible
* Deploy applications across multiple servers
* Practice AWS EC2 infrastructure management
* Understand SSH-based server communication
* Reduce manual deployment steps

---

# 🚀 Key Learning Outcomes

Through this project, I gained practical experience with:

* CI/CD pipeline implementation
* Jenkins pipeline automation
* Docker containerization
* Docker image lifecycle
* Docker Hub
* Ansible inventory and playbooks
* SSH authentication
* AWS EC2 server management
* Multi-server application deployment
* Automated container deployment
* Linux server administration

---

# 🔮 Future Improvements

The project can be further enhanced by integrating:

* Kubernetes for container orchestration
* Prometheus and Grafana for monitoring
* Jenkins webhooks for automatic pipeline triggering
* AWS ECR instead of Docker Hub
* Load Balancer for traffic distribution
* HTTPS using SSL/TLS
* Ansible roles for better project organization
* Automated testing in the CI/CD pipeline
* Blue-Green or Rolling Deployment

---

# 👨‍💻 Author

**Mohammad Sadhiq KV**

BE Computer Science & Engineering

Interested in **DevOps, Cloud, Automation, Python, Docker, Kubernetes, and CI/CD**.
 
