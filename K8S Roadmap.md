

## 🟢 Phase 0 – Prerequisites (Foundation)

- Linux basics (processes, networking, file system)

- Networking fundamentals (TCP/IP, DNS, ports, load balancing)

- YAML

- Docker (images, containers, volumes, networks, multi-stage builds)

- Basic system design concepts


---

## 🟢 Phase 1 – Kubernetes Fundamentals (Core Concepts)

- What is Kubernetes

- Control Plane Components

    - API Server

    - Scheduler

    - Controller Manager

    - etcd

- Node Components

    - Kubelet

    - Kube-proxy

    - Container Runtime

- Kubernetes Architecture

- kubectl basics

- Imperative vs Declarative approach


---

## 🟢 Phase 2 – Core Objects (Hands-On Required)

- Pod & Containers

- ReplicaSet

- StatefulSet

- Deployment

- Namespace

- Labels & Selectors

- Annotations

- Service (ClusterIP, NodePort, LoadBalancer)

- ConfigMap

- Secret

- Resource Requests & Limits


Hands-on:

- Deploy FastAPI/Go app using manifest files

- Expose via Service

- Update image (rolling update)

- Scale replicas


---

## 🟢 Phase 3 – Networking & Storage

### Networking

- Service networking

- CoreDNS

- Ingress

- Ingress Controller (Kong)

- Network Policies


### Storage

- Volumes

- PersistentVolume (PV)

- PersistentVolumeClaim (PVC)

- StorageClass


Hands-on:

- Deploy PostgreSQL with PVC

- Connect backend app to DB inside cluster


---

## 🟢 Phase 4 – Configuration & Workload Management

- Liveness & Readiness Probes

- Startup Probes

- Init Containers

- Multi-container Pods (Sidecar pattern)

- Jobs

- CronJobs

- DaemonSet

- Taints & Tolerations

- Node Selectors & Affinity


Hands-on:

- Add health checks to your backend

- Create CronJob for DB cleanup

- Deploy logging sidecar


---

## 🟢 Phase 5 – Advanced Deployment Strategies [SKIP FOR NOW]

- Rolling Updates

- Recreate Strategy

- Blue-Green Deployment

- Canary Deployment

- HPA (Horizontal Pod Autoscaler)

- VPA

- Pod Disruption Budget (PDB)


Hands-on:

- Enable HPA for CPU-based scaling

- Implement Canary with Ingress


---

## 🟢 Phase 6 – Security

- RBAC

- Service Accounts

- Role & ClusterRole

- Pod Security Standards

- Secrets Management

- Image Security & Scanning [Defer]


Hands-on:

- Create restricted namespace

- Create custom RBAC for developer role


---

## 🟢 Phase 7 – Observability & Debugging (Important for You)

- kubectl logs / exec / describe

- Debugging Pods

- Events

- Metrics Server

- Prometheus

- Grafana

- OpenTelemetry in Kubernetes

- Centralized Logging (Clickstack)


Hands-on:

- Deploy Prometheus + Grafana

- Monitor your FastAPI app

- Add tracing via OTel


---

## 🟢 Phase 8 – Helm & Packaging [SKIP FOR NOW]

- Helm Basics

- Charts

- Values.yaml

- Templates

- Helm Hooks

- Upgrade & Rollback


Hands-on:

- Package your backend app as Helm chart

- Deploy PostgreSQL via Helm


---

## 🟢 Phase 9 – CI/CD with Kubernetes [SKIP FOR NOW]

- Docker build pipeline

- Push to registry

- Deploy via CI (GitHub Actions / GitLab)

- Kubernetes manifests in repo

- Helm in CI

- ArgoCD (GitOps)


Hands-on:

- Auto-deploy on push to main

- Implement GitOps workflow


---

## 🟢 Phase 10 – Production-Level Topics [SKIP FOR NOW]

- Multi-Environment Setup (dev/stage/prod)

- Cluster Autoscaler

- Resource Optimization

- Cost Optimization

- Zero-downtime Deployments

- Multi-Cluster Setup

- Backup & Restore (etcd, Velero)

- Disaster Recovery

- High Availability Control Plane


---

## 🟢 Phase 11 – Cloud Kubernetes [SKIP FOR NOW]

- Managed Kubernetes:

    - Amazon EKS

    - Google Kubernetes Engine

    - Azure Kubernetes Service

- ** Load Balancers **

- Cloud Storage Integration

- IAM integration

- VPC networking


---

# 🧠 Advanced Architecture Concepts [SKIP FOR NOW]

- Service Mesh (Istio / Linkerd)

- **API Gateway**

- Sidecar Pattern

- Operator Pattern

- Custom Resource Definitions (CRD)

- Event-driven architecture in K8s

- Multi-tenant clusters


---

# 🏁 Final Stage – Real Projects (Mandatory)

Build these:

1. Deploy full microservices system (Auth + API + DB)

2. Production-ready monitoring stack

3. Log analyzer inside Kubernetes

4. Auto-scaling backend API

5. CI/CD pipeline with GitOps

6. Secure multi-namespace environment