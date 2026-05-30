# MuchToDo - Cloud-Native Task Management API

**Author:** John Akinola

## Project Overview

MuchToDo is a cloud-native task management backend designed to demonstrate modern software engineering, cloud infrastructure, containerization, and Kubernetes orchestration practices.

The application provides a RESTful API for managing tasks while showcasing how production-oriented applications can be deployed using containerized microservices, persistent storage, caching, secrets management, ingress routing, and infrastructure automation.

This project was developed to simulate real-world deployment patterns used by cloud engineers, DevOps engineers, site reliability engineers (SREs), and platform engineers.

---

## Why This Project Matters

Modern applications rarely run on a single server. Organizations increasingly rely on distributed systems that require:

* Containerized workloads
* Persistent databases
* High-performance caching
* Secure secrets management
* Automated deployments
* Service discovery
* Traffic routing and ingress control

This project demonstrates practical experience with these concepts by deploying a complete application stack on Kubernetes.

Key skills demonstrated include:

* Kubernetes administration
* Container orchestration
* Infrastructure as Code (IaC)
* Docker image management
* Service networking
* Stateful application deployment
* Secrets management
* Cloud-native architecture principles
* Troubleshooting distributed systems

---

# Architecture

## High-Level Architecture

```text
                    ┌─────────────────────┐
                    │   NGINX Ingress     │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │   Go Backend API    │
                    └───────┬─────┬───────┘
                            │     │
                            │     │
                            ▼     ▼
                   ┌───────────┐ ┌───────────┐
                   │ MongoDB   │ │  Redis    │
                   │ Database  │ │  Cache    │
                   └───────────┘ └───────────┘
```

---

# Technology Stack

| Layer              | Technology                      |
| ------------------ | ------------------------------- |
| Backend API        | Go (Golang)                     |
| Database           | MongoDB 4.4                     |
| Cache              | Redis 7.2                       |
| Containerization   | Docker                          |
| Orchestration      | Kubernetes                      |
| Local Kubernetes   | Kind                            |
| Ingress Controller | NGINX Ingress                   |
| Secrets Management | Kubernetes Secrets              |
| Persistent Storage | Persistent Volume Claims (PVCs) |

---

# Components

## Backend API

The backend service is written in Go and exposes RESTful endpoints for task management operations.

Responsibilities:

* Request handling
* Business logic
* Data validation
* Database interactions
* Redis caching
* Authentication token management

---

## MongoDB

MongoDB serves as the primary persistent datastore.

Features:

* Persistent Volume Claim (PVC)
* Stateful data storage
* Document-based schema
* Authentication-enabled deployment

Data remains available even when containers are restarted.

---

## Redis

Redis provides an in-memory caching layer to reduce database load and improve response times.

Benefits:

* Faster data retrieval
* Reduced database queries
* Improved scalability
* Lower latency

Redis is deployed as a stateless service.

---

## NGINX Ingress Controller

The NGINX Ingress Controller acts as the external entry point into the cluster.

Responsibilities:

* HTTP routing
* Load balancing
* Service exposure
* Centralized ingress management

---

# Kubernetes Resources Used

This project utilizes several Kubernetes resource types:

| Resource              | Purpose                          |
| --------------------- | -------------------------------- |
| Deployment            | Application lifecycle management |
| Service               | Internal service discovery       |
| Secret                | Sensitive configuration storage  |
| ConfigMap             | Non-sensitive configuration      |
| Ingress               | External routing                 |
| PersistentVolumeClaim | Persistent storage               |
| Namespace             | Resource isolation               |

---

# Security Considerations

A security-first approach was taken throughout the deployment.

## Kubernetes Secrets

Sensitive values such as:

* MongoDB credentials
* JWT signing keys
* Application configuration

are stored using Kubernetes Secrets rather than ConfigMaps.

Benefits:

* Separation of sensitive and non-sensitive data
* Reduced risk of accidental exposure
* Production-aligned security practices

---

## Principle of Least Exposure

Services are exposed only where necessary.

Internal communication occurs through Kubernetes Services rather than direct pod access.

---

# Architectural Decisions

## 1. Makefile Instead of Shell Scripts

The project requirements suggested storing deployment commands inside individual shell scripts.

Instead, a centralized Makefile was implemented.

Benefits:

* Cleaner repository structure
* Easier maintenance
* Consistent developer experience
* Industry-standard approach for Go projects

Examples:

```bash
make build
make deploy
make test
make clean
```

---

## 2. Kubernetes Secrets Instead of ConfigMaps

The backend relies on sensitive values including:

* MongoDB credentials
* JWT secrets

Storing these values inside ConfigMaps would expose them as plaintext configuration.

To align with industry best practices, Kubernetes Secrets are used instead.

This mirrors how production environments manage application secrets.

---

## 3. Local Development Stability

While configuring MongoDB authentication and Kubernetes networking, issues were encountered involving environment variable interpolation and MongoDB connection initialization.

To ensure consistent local deployment behavior within the Kind cluster, backend connection settings were simplified and standardized.

This decision prioritizes deployment reliability while maintaining a production-like architecture.

---

# Deployment Instructions

## Prerequisites

Install:

* Docker Desktop
* kubectl
* kind
* Go (optional for local development)

Verify installation:

```bash
docker --version
kubectl version --client
kind version
```

---

## Step 1: Create the Kubernetes Cluster

```bash
kind create cluster \
  --name muchtodo-cluster \
  --config kubernetes/kind-config.yaml
```

Verify:

```bash
kubectl cluster-info
```

---

## Step 2: Deploy the NGINX Ingress Controller

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
```

Wait until the controller is ready:

```bash
kubectl get pods -n ingress-nginx
```

---

## Step 3: Create Namespace

```bash
kubectl apply -f kubernetes/namespace.yaml
```

---

## Step 4: Deploy MongoDB

```bash
kubectl apply -f kubernetes/mongodb/
```

Verify:

```bash
kubectl get pods -n muchtodo
```

---

## Step 5: Deploy Redis

```bash
kubectl apply -f kubernetes/backend/redis-deployment.yaml
kubectl apply -f kubernetes/backend/redis-service.yaml
```

Verify:

```bash
kubectl get pods -n muchtodo
```

---

## Step 6: Create Backend Secrets

```bash
kubectl create secret generic backend-env-file \
  --from-file=.env \
  -n muchtodo
```

---

## Step 7: Deploy Backend

```bash
kubectl apply -f kubernetes/backend/backend-deployment.yaml
kubectl apply -f kubernetes/backend/backend-service.yaml
```

Verify:

```bash
kubectl get deployments -n muchtodo
kubectl get pods -n muchtodo
```

---

## Step 8: Deploy Ingress

```bash
kubectl apply -f kubernetes/ingress.yaml
```

Verify:

```bash
kubectl get ingress -n muchtodo
```

---

# Accessing the Application

Once all services are running successfully:

```text
http://localhost/
```

You can also inspect cluster resources:

```bash
kubectl get all -n muchtodo
```

---

# Operational Commands

View logs:

```bash
kubectl logs deployment/muchtodo-backend -n muchtodo
```

Check services:

```bash
kubectl get svc -n muchtodo
```

Inspect pods:

```bash
kubectl describe pod <pod-name> -n muchtodo
```

Restart backend:

```bash
kubectl rollout restart deployment/muchtodo-backend -n muchtodo
```

---

# Future Improvements

Potential enhancements include:

* Helm chart packaging
* GitHub Actions CI/CD pipeline
* ArgoCD GitOps deployment
* Horizontal Pod Autoscaling (HPA)
* Prometheus monitoring
* Grafana dashboards
* TLS certificates with cert-manager
* Distributed tracing with OpenTelemetry
* MongoDB replica set deployment
* Cloud deployment on AWS EKS or Azure AKS

---

# Learning Outcomes

This project demonstrates practical experience with:

* Kubernetes deployments
* Container orchestration
* Infrastructure troubleshooting
* Service networking
* Stateful applications
* Secrets management
* Cloud-native application design
* DevOps and platform engineering workflows

The project serves as a strong foundation for cloud engineering, DevOps engineering, platform engineering, and cloud security roles.
