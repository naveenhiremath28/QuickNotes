
```
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

- ** API Gateway **

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
```


## 🚀 1. What _is_ Kubernetes?

**Kubernetes (often called “K8s”)** is an open-source platform for automating deployment, scaling, and management of **containerized applications** — the kind built with Docker and other container runtimes. It groups containers into logical units for easy management and scaling. 

---

## 🧠 2. Basic Concepts You Need First

Before we do anything hands-on, here are the core ideas you should understand:

|Term|What it Means|
|---|---|
|**Cluster**|A set of machines (or a single machine) running Kubernetes|
|**Node**|A worker machine in a cluster|
|**Pod**|The **smallest unit** in K8s — one or more containers|
|**Deployment**|A set of pods managed together|
|**Service**|A stable network access point for pods|
|**kubectl**|Command-line tool to talk to Kubernetes|

Understanding these will help you make sense of everything we do next. You can learn more in the **Kubernetes Concepts**section of the official docs. 

---

## 💻 3. Set Up a Local Kubernetes Environment

You’ll need an environment where you can _practice hands-on_. For beginners, the easiest options are:

### 🟢 Option A: **Minikube**

A tool that runs Kubernetes locally on your laptop. It simulates a real Kubernetes cluster. 

👉 Steps (general idea — you can ask me for step-by-step installation on your OS):

1. Install **kubectl** (Kubernetes CLI)
    
2. Install **Minikube**
    
3. Start a cluster:
    
    `minikube start`
    
4. Check cluster:
    
    `kubectl get nodes`
    

---
## 🟢 1. **Cluster**

✔ A **Cluster** is the full Kubernetes environment — like a city.  
✔ It is made up of many **machines (nodes)** working together.  
✔ Kubernetes controls and coordinates everything inside the cluster. 

**Imagine:** A cluster is like a construction site where all work happens. The city managers decide who works where. That's Kubernetes. 

---

## 🔵 2. **Node**

✔ A **Node** is one **machine** inside the cluster — it could be physical or virtual.  
✔ The node is where containers actually _run_. 

**Think of it like:** A worker machine or “worker robot” in your city. 

---

## 🟡 3. **Pod**

✔ A **Pod** is the _smallest unit_ in Kubernetes — it holds one or more containers.  
✔ That’s where your container runs. 

**Easy analogy:**  
Imagine each container is a worker bee, and the **pod** is the hive where they live together. 

👉 Even if a pod contains multiple containers, they **share network and storage inside the same pod**. 

---

## 🧡 4. **Deployment**

✔ A **Deployment** manages _how many copies_ of a Pod you want.  
✔ If one pod fails (it crashes), the deployment makes a **new one**. 

**Analogy:**  
You want 3 similar workers doing the same job. A Deployment ensures you always have 3 running and restarts any that stop. 

---

## ❤️ 5. **Service**

✔ A **Service** gives a _stable way to reach your pods_ — even if pods are added, removed, or restarted. 

**Important point:** Pods get dynamic IPs — that means their network addresses can change each time they start. Services solve this:  
✔ They keep one _stable address_ for clients to send traffic to. 

**Example:**  
Your app has front-end pods and back-end pods. The front end needs to talk to the back end — so you use a Service so the front end can always find the back end even if pods restart.


# Why These Matter

- **Pods** run your apps. 
    
- **Deployments** make sure your app runs enough copies and recovers from failure automatically. 
    
- **Services** let other pods or users _reach your app at stable addresses_. 
    

This set of three (Pods + Deployments + Services) are the **core building blocks** of Kubernetes


---
## Nodes in Kubernetes

A **Node** is a **machine (VM or physical server)** where your containers actually run.

👉 Simple:  
**Node = Worker machine that runs Pods**

---

## Types of Nodes

### 1️⃣ Control Plane Node (Master)

- Manages the cluster
    
- Decides where Pods should run
    
- Stores cluster state
    

### 2️⃣ Worker Node

- Runs your applications (Pods)
    
- Does the actual work
    

---

## What exists inside a Worker Node?

### 🔹 1. Kubelet

- Agent running on the node
    
- Talks to control plane
    
- Makes sure Pods are running correctly
    

### 🔹 2. Container Runtime

- Runs containers
    
- Example: containerd
    

### 🔹 3. Kube-proxy

- Handles networking
    
- Allows Pods to communicate
    

---

## How Node works (Flow)

1. You create a Pod
    
2. Control plane chooses a Node
    
3. Kubelet on that Node creates the Pod
    
4. Container runtime runs containers
    

---

## Important Points

- Each Node has:
    
    - CPU
        
    - Memory
        
    - Storage
        
- If a Node fails → Pods are moved to another Node
    
- More Nodes = More scalability

---

# 1️⃣ What is a Deployment?

A **Deployment** in Kubernetes:

- Manages Pods
    
- Ensures desired number of replicas are running
    
- Supports rolling updates
    
- Automatically replaces failed Pods
    

It internally manages a **ReplicaSet**, which manages Pods.

Example:

```yaml
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
        image: nginx:1.14.2  
        ports:  
        - containerPort: 80
```

---

# 2️⃣ Top-Level Fields

## 🔹 `apiVersion: apps/v1`

- Defines Kubernetes API version used.
    
- `apps/v1` → Required for Deployments.
    

👉 Think of it as schema version.

---

## 🔹 `kind: Deployment`

- Specifies the type of object.
    
- Here → Deployment resource.
    

---

## 🔹 `metadata`

Stores identifying information.

metadata:  
  name: nginx-deployment  
  labels:  
    app: nginx

### ✅ `name`

- Unique name of the Deployment.
    
- Used in commands:
    
    kubectl get deployment nginx-deployment
    

### ✅ `labels`

- Key-value pairs.
    
- Used for grouping and selecting resources.
    

Example:

app = nginx

---

# 3️⃣ `spec` (Desired State)

Defines what you want Kubernetes to maintain.

---

## 🔹 `replicas: 3`

- Run **3 Pods**.
    
- If one crashes → Kubernetes recreates it.
    
- Change to 5 → 5 Pods will run.
    

---

## 🔹 `selector`

selector:  
  matchLabels:  
    app: nginx

- Tells Deployment which Pods it controls.
    
- Must match labels in `template.metadata.labels`.
    

⚠ Important Rule:

selector.matchLabels == template.metadata.labels

---

# 4️⃣ `template` (Pod Blueprint)

Defines how Pods should be created.

Think of it as:

> “This is how every Pod should look.”

---

## 🔹 `template.metadata.labels`

labels:  
  app: nginx

- Every Pod created will have this label.
    
- Must match the selector.
    

---

## 🔹 `template.spec`

Defines what runs inside each Pod.

---

# 5️⃣ Containers Section

containers:  
- name: nginx  
  image: nginx:1.14.2  
  ports:  
  - containerPort: 80

---

## 🔹 `name: nginx`

- Name of container inside the Pod.
    

---

```
## ReplicaSet (Kubernetes)

A **ReplicaSet** ensures a fixed number of **Pods are always running**.

👉 Simple:  
**ReplicaSet = Keeps desired number of Pods alive**

---

## Why we need it?

If:

- A Pod crashes ❌
    
- A Node fails ❌
    

ReplicaSet automatically creates a new Pod ✅

So your app always runs.

---

## How it works

You define:

replicas: 3

Kubernetes ensures:

- Always **3 Pods** are running
    
- If 1 dies → new one is created
    
- If you change to 5 → it creates 2 more
    

---

## Important Concepts

### 🔹 Desired State

Number of Pods you want.

### 🔹 Current State

Number of Pods running.

ReplicaSet always makes current state = desired state.

---

## Very Important

In real projects, we don’t create ReplicaSet directly.  
We use **Deployment**, and Deployment manages ReplicaSet.
```


```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
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
          image: nginx:latest
          ports:
            - containerPort: 80
```


## **Namespace**
```
## Namespace (Kubernetes)

A **Namespace** is a way to **divide a cluster into logical groups**.

👉 Simple:  
**Namespace = Folder inside a cluster**

---

## Why we need it?

To:

- Separate environments (dev, test, prod)
    
- Avoid name conflicts
    
- Control access (RBAC)
    
- Set resource limits
    

---

## Example

You can have:

- `dev` namespace → app v1
    
- `prod` namespace → app v2
    

Both can have a Pod named `my-app`  
No conflict because they are in different namespaces.

---

## Default Namespaces

- `default` → If you don’t specify one
    
- `kube-system` → System components
    
- `kube-public` → Public resources
    

---

## Important

Namespace is only for **logical separation**,  
not for creating separate clusters.

---

### Memory Trick 🧠

Cluster = Big cupboard  
Namespace = Folder  
Pods/Services = Files inside folder

### Example:

- `dev` namespace
    
    - auth-service
        
    - payment-service
        
    - db
        
- `prod` namespace
    
    - auth-service
        
    - payment-service
        
    - db
        

Both can have same names, but they are isolated.

---

## What kind of isolation?

✔ Resource isolation (CPU, Memory limits)  
✔ Access control (RBAC per namespace)  
✔ Logical separation  
✔ Network policies (can restrict communication)

```

## Labels (Kubernetes)
```


A **Label** is a **key-value tag** attached to objects.

👉 Simple:  
**Label = Tag to identify resources**

Example:

app: payment  
env: prod  
tier: backend

### Why Labels?

- Select Pods (Service uses them)
    
- Group resources
    
- Filtering & management
    

---

## Resources (Kubernetes)

Resources = **Objects you create in Kubernetes**

Examples:

- Pod
    
- Service
    
- Deployment
    
- ReplicaSet
    
- ConfigMap
    
- Secret
    

👉 Simple:  
**Resource = Any object inside Kubernetes**

## Important Concept

Labels help connect resources.

Example:

- Service finds Pods using **label selector**
    
- Deployment manages Pods using labels

🧠 Memory Trick:  
Resource = Object  

Label = Tag on object

```
---

## Annotations (Kubernetes)

```


**Annotations** are **key-value pairs** like labels, but used to store extra information.

👉 Simple:  
**Annotation = Note attached to a resource**

---

## Difference from Labels

- **Labels** → Used for selecting & grouping
    
- **Annotations** → Used for metadata (not for selection)
    

---

## Example

annotations:  
  description: "Payment service for handling transactions"  
  owner: "backend-team"

---

## Why use Annotations?

- Store build/version info
    
- Store tool-specific data
    
- Store configuration hints
    
- Used by monitoring / logging tools
    

---

⚠ Important:

- Cannot use annotations to select Pods
    
- Mainly for additional information
  
  
> Annotation → Developers + Kubernetes tools can read

I meant this 👇

### Think like this:

### 🧾 Normal Code Comment

// this function handles payment

Only developer sees it.  
Computer ignores it.

---

### 📌 Kubernetes Annotation

It is like a comment, **but Kubernetes stores it as data**.

Example:

annotations:  
  owner: backend-team

Now:

- 👨‍💻 Developer can see it
    
- 🤖 Kubernetes tools (like monitoring, ingress, CI/CD) can also read it
    

So it is **not ignored** like a normal comment.

---

### Very Simple Difference

Code comment → Ignored  
Annotation → Stored & usable

---

🧠 Final Simple Line:

Annotation = Information attached to a resource that both humans and tools can use.
```

## ConfigMap (Kubernetes)
```


A **ConfigMap** stores **configuration data** for your application.

👉 Simple:  
**ConfigMap = External configuration storage**

---

## Why we need it?

Instead of hardcoding config like:

DB_HOST = "localhost"

We store it in ConfigMap.

So if config changes →  
No need to rebuild container.

---

## What can it store?

- Environment variables
    
- Database URLs
    
- App settings
    
- Config files
    

---

## How it is used?

Pod can read ConfigMap as:

- Environment variable
    
- File inside container
    

---

## Important

ConfigMap is for **non-sensitive data**.

For passwords → use **Secret**.

---

🧠 Memory Trick:

ConfigMap = Settings  
Secret = Passwords
```

## Secret (Kubernetes)
```


A **Secret** stores **sensitive data**.

👉 Simple:  
**Secret = Secure storage for passwords & keys**

---

## Why not ConfigMap?

- ConfigMap → Normal data
    
- Secret → Sensitive data (passwords, tokens, API keys)
    

---

## Simple Example

### 1️⃣ Create Secret

apiVersion: v1  
kind: Secret  
metadata:  
  name: db-secret  
type: Opaque  
stringData:  
  DB_PASSWORD: mypassword

---

### 2️⃣ Use in Pod

env:  
- name: DB_PASSWORD  
  valueFrom:  
    secretKeyRef:  
      name: db-secret  
      key: DB_PASSWORD

Now your container gets the password securely.

---

## Important

- Secrets are base64 encoded
    
- Can be mounted as env variables or files
    

---

🧠 Memory Trick:

ConfigMap = Public settings  
Secret = Private data 🔐
```

## Service (Kubernetes)
```

A **Service** gives a **stable way to access Pods**.

👉 Simple:  
**Service = Fixed address for changing Pods**

---

## Why needed?

Pods:

- Can restart
    
- IP changes
    
- Can scale up/down
    

Service:

- Gives fixed IP
    
- Gives DNS name
    
- Load balances traffic
    

---

## How it works

Service uses **labels** to find Pods.

Example:

- Pods label → `app: payment`
    
- Service selects those Pods
    
- Traffic is distributed
    

---

## Types

### 1️⃣ ClusterIP

Internal only (inside cluster)

### 2️⃣ NodePort

Accessible via Node IP + Port

### 3️⃣ LoadBalancer

External cloud load balancer

---

🧠 Simple understanding:

Pod = Temporary worker  
Service = Reception counter that directs traffic
```


```
# 🌐 What Is a _Service_ in Kubernetes?

A **Service** in Kubernetes is a _networking abstraction_ — it gives you a **stable way to reach one or more Pods** running in your cluster.

### 📌 Why do we need it?

- Pods are **ephemeral** (they can die, restart, get replaced, or move).
    
- Each pod gets an **IP address**, but that IP can _change_ when the pod changes.
    
- If you try to reach the pods directly by IP, your client breaks when the IP changes.
    
- A **Service** gives you a **consistent endpoint** (IP or DNS) that **always works**, even if pods behind it change. 
    

**In simple words:**

> A Service is like a **reception desk** or **phone operator** — you call it, and it directs traffic to the right pod(s), even if those pods were restarted, moved, or replaced. 

---

## 🧠 How It Works

A Service:

✔ Selects a set of Pods based on **labels**  
✔ Gives them a **stable network identity** (IP and sometimes DNS)  
✔ **Routes traffic** to the correct Pods  
✔ Can balance traffic across multiple Pods  
✔ Keeps working even when Pods are replaced or scaled up/down 

So you don’t have to know which pod is running — you send traffic to the **Service**, and Kubernetes delivers it to the right Pods.

---

# 📌 How Does a Service Find Pods?

Kubernetes uses **labels and selectors**:

- You put **labels** like `app=frontend` on your Pods
    
- A Service uses a **selector** to match those labels
    
- Only matching Pods receive traffic from that Service 
    

This means if later you scale your app (add more replicas), the Service _automatically_ includes them in routing — because they share the same label.

```
```
What matters is:

> 🔹 The **Service selector labels** must match the **Pod labels**.

---

### In Your Example

#### Pod Labels

labels:  
  app.kubernetes.io/name: proxy

#### Service Selector

selector:  
  app.kubernetes.io/name: proxy

Since both match → ✅ Service will correctly route traffic to that Pod.

---

### Important Concept

Kubernetes Service works like this:

Service selector  →  finds Pods with matching labels  →  routes traffic

If labels don’t match:

Service → finds 0 Pods → traffic fails

---

### Does It Have to Be `app.kubernetes.io/name`?

❌ No.  
You can use anything like:

labels:  
  app: nginx

And then:

selector:  
  app: nginx

It just needs to match.

---

### Why `app.kubernetes.io/name`?

It is a **recommended Kubernetes labeling convention**, not mandatory.

Kubernetes suggests structured labels like:

- `app.kubernetes.io/name`
    
- `app.kubernetes.io/version`
    
- `app.kubernetes.io/component`
    
- etc.
    

But they are optional.

---

### Final Answer

✔ Yes — the label key and value used in the Service selector must exist in the Pod labels.  
❌ It doesn’t have to be `app.kubernetes.io/name` specifically.  
✔ It just has to match exactly.
```

## StatefulSet (Kubernetes)
```
## StatefulSet (Kubernetes)

A **StatefulSet** is used for applications that need **stable identity and persistent storage**.

👉 Simple:  
**StatefulSet = For stateful applications (data matters)**

---

## Why use StatefulSet?

Some apps need:

- Stable pod name
    
- Stable storage
    
- Ordered startup/shutdown
    

Example apps:

- Databases
    
- Kafka
    
- Zookeeper
    

---

## Example

Pods created like:

db-0  
db-1  
db-2

Each pod:

- Has **fixed name**
    
- Has **its own storage**
    

Even if pod restarts → same identity.

---

## Important Features

✔ Stable pod names  
✔ Persistent volumes  
✔ Ordered pod creation

---

🧠 Memory Trick:

Deployment → Stateless apps  
StatefulSet → Stateful apps (like databases)
```

## Service Networking (Kubernetes)

```

**Service networking** allows **Pods to communicate with each other using Services**.

👉 Simple:  
**Service networking = How Pods talk using Services**

---

## Why needed?

Pods:

- Have **temporary IPs**
    
- Can restart anytime
    

Service provides:

- **Stable IP**
    
- **DNS name**
    

So communication becomes reliable.

---

## Example

Pod A wants to talk to **payment service**.

Instead of Pod IP:

10.244.1.25 (may change)

Pod uses:

payment-service

Kubernetes automatically routes traffic to correct Pods.

---

🧠 Simple Flow:

Pod → Service → Pod(s)
```

## CoreDNS
```
## CoreDNS (Kubernetes)

**CoreDNS** is the **DNS server used inside a Kubernetes cluster**.  
It allows **Pods and services to find each other using names instead of IP addresses**.

👉 In simple terms:  
**CoreDNS = DNS system of Kubernetes**

---

## Why CoreDNS is Needed

In Kubernetes:

- Pods are **dynamic**
    
- Pod **IP addresses change** when they restart
    
- Pods are **created and deleted frequently**
    

Because of this, applications **cannot rely on IP addresses**.

Instead, they use **service names**, and CoreDNS resolves those names to the correct IP.

Example:

Application calls:

payment-service

CoreDNS converts it to:

10.96.0.12

This IP is the **Service IP**, which forwards traffic to the correct Pods.

---

## Where CoreDNS Runs

CoreDNS runs **inside the cluster as Pods** in the **`kube-system` namespace**.

You can see them using:

kubectl get pods -n kube-system

Example output:

coredns-558bd4d5db-abc12  
coredns-558bd4d5db-def34

Usually **2 replicas** run for reliability.

---

## How Service DNS Works

Kubernetes automatically creates DNS entries for services.

### DNS Format

<service-name>.<namespace>.svc.cluster.local

Example:

payment-service.default.svc.cluster.local

Inside the same namespace, you can just use:

payment-service

CoreDNS resolves this to the service IP.

---

## Example Flow

1. Pod A wants to call **payment-service**
    
2. Pod A asks DNS:  
    "What is the IP of payment-service?"
    
3. Request goes to **CoreDNS**
    
4. CoreDNS returns the **Service ClusterIP**
    
5. Traffic goes to the **Service**
    
6. Service forwards it to one of the Pods
    

Flow:

Pod → CoreDNS → Service → Pod

---

## What CoreDNS Actually Does

CoreDNS:

- Resolves **service names to IP addresses**
    
- Enables **service discovery**
    
- Handles **internal DNS queries**
    
- Allows **pods to communicate using names**
    

---

## Important Points

- Runs as **Pods**
    
- Located in **kube-system namespace**
    
- Automatically configured by Kubernetes
    
- Works with **Services**, not directly with Pods
    

---

## Simple Real-World Analogy

Think of CoreDNS like a **phone contact list**.

Instead of remembering phone numbers:

9876543210

You use a name:

Rahul

Phonebook resolves:

Rahul → 9876543210

Similarly:

payment-service → 10.96.0.12

---

✅ **One-line summary**

CoreDNS is the **internal DNS system of Kubernetes that converts service names into IP addresses so pods can communicate easily.**
```


# 1. What is a Node, Cluster, and External User
```

### Kubernetes Cluster

A **cluster** is a group of machines (Nodes) running Kubernetes.

Inside a cluster you have:

- **Nodes** → machines (VMs or servers)
    
- **Pods** → running containers
    
- **Services** → stable access layer to pods
    

### Node

A **Node** is a machine inside the cluster.

Example:

Cluster  
 ├── Node 1 (192.168.1.10)  
 │     ├── Pod A  
 │     └── Pod B  
 │  
 ├── Node 2 (192.168.1.11)  
 │     └── Pod C

Nodes have **real network IPs** in your infrastructure.

---

# 2. What is an External User?

**External user = anything outside the Kubernetes cluster network**

Examples:

- Your **browser**
    
- A **mobile app**
    
- Another **server outside the cluster**
    
- Your **local laptop**
    

Example:

You opening a website  
curl http://app.example.com

That request comes **from outside the cluster** → so you are an **external user**.

---

# 3. Why ClusterIP Cannot Be Accessed Externally

ClusterIP is **only reachable inside the cluster network**.

Example:

ClusterIP = 10.96.12.45

Only **pods inside Kubernetes** can reach it.

Example:

Pod A  →  http://10.96.12.45

But your laptop cannot.

Why?

Because:

10.x.x.x network  
exists only inside Kubernetes

Your laptop doesn't know how to route to that network.

Example:

Laptop  
   ↓  
Internet  
   ↓  
Cluster  
   ↓  
Service (ClusterIP 10.96.12.45)

Your laptop cannot directly reach that **internal cluster network**.

---

# 4. Why NodePort CAN Be Accessed Externally

Because **Nodes have real IP addresses**.

Example:

Node 1 IP = 192.168.1.10

This IP is part of your **actual network**.

So external traffic can reach it.

Example:

http://192.168.1.10:30007

NodePort opens a port on **every node**.

---

# 5. NodePort Traffic Flow


External User  
     │  
     ▼  
Node IP : NodePort  
(192.168.1.10:30007)  
     │  
     ▼  
Kubernetes Service  
     │  
     ▼  
Pods

Example request:

Browser  
   ↓  
http://192.168.1.10:30007  
   ↓  
Node receives request  
   ↓  
Kubernetes Service  
   ↓  
Pod

---

# 6. ClusterIP vs NodePort Visualization

### ClusterIP

Cluster  
  
 Pod A  
   │  
   ▼  
Service (ClusterIP)  
   │  
   ▼  
 Pod B

Only **internal communication**.

External user ❌ cannot reach.

---

### NodePort

External User  
      │  
      ▼  
Node (192.168.1.10:30007)  
      │  
      ▼  
Service  
      │  
      ▼  
Pods

External user ✅ can reach.

---

# 7. Simple Real-Life Analogy

Think of a **company building**.

### ClusterIP

Office Internal Phone  
  
Employee → Extension 204 → HR  
  
Only people inside office can call.

External people ❌ cannot call.

---

### NodePort

Company Reception Number  
  
Customer → Main Phone Number → Reception → HR

External people ✅ can call.

---

# 8. Why NodePort Exists

NodePort is mainly used for:

- Testing
    
- Development
    
- Basic exposure
    

In production we usually use:

LoadBalancer  
or  
Ingress

---

# 9. One Important Note

Even though **Node is inside cluster**, it still has a **real network interface**.

Example:

Node Internal IP: 10.244.0.12  
Node External IP: 192.168.1.10

External users connect to the **external IP**.

---

✅ **Golden rule**

ClusterIP  → Internal only  
NodePort   → External via Node IP  
LoadBalancer → External via cloud LB


## LoadBalancer

**LoadBalancer** exposes the service using an **external cloud load balancer**.

Used in cloud platforms like:

- GKE
    
- AWS EKS
    
- Azure AKS
    

### Key Points

- Creates a **public IP**
    
- Distributes traffic across pods
    
- Automatically managed by cloud provider
    

### Flow

User  
  ↓  
Cloud LoadBalancer  
  ↓  
Service  
  ↓  
Pods

---

## Quick Comparison

|Service Type|Access|Usage|
|---|---|---|
|ClusterIP|Inside cluster only|Internal communication|
|NodePort|Node IP + Port|Simple external access|
|LoadBalancer|Public IP|Production external access|
```



## Ingress
```


**Ingress** is a Kubernetes resource that manages **external HTTP/HTTPS access to services inside the cluster**.

It allows you to **route traffic to different services using a single entry point**.

---

## Why Ingress is Used

If you expose multiple services using **LoadBalancer**, each service gets its **own external IP**, which is inefficient.

Ingress solves this by providing:

- **One external entry point**
    
- **Routing rules to different services**
    

---

## Example

Suppose the cluster has two services:

- `payment-service`
    
- `order-service`
    

You want requests to go like this:

example.com/payments → payment-service  
example.com/orders → order-service

Ingress checks the request path and routes traffic to the correct service.

---

## How Ingress Works

Traffic flow:

User  
  ↓  
Ingress Controller  
  ↓  
Service  
  ↓  
Pods

1. User sends request to domain
    
2. Ingress receives the request
    
3. Ingress checks routing rules
    
4. Request is forwarded to the correct service
    
5. Service sends it to the appropriate pod
    

---

## Ingress Components

### 1. Ingress Resource

This defines **routing rules**.

Example configuration:

apiVersion: networking.k8s.io/v1  
kind: Ingress  
metadata:  
  name: app-ingress  
spec:  
  rules:  
  - host: example.com  
    http:  
      paths:  
      - path: /payments  
        pathType: Prefix  
        backend:  
          service:  
            name: payment-service  
            port:  
              number: 80

This rule means:

example.com/payments → payment-service

```

### 2. Ingress Controller
```

Ingress rules require a **controller** to work.

The controller watches Ingress resources and configures routing.

Common controllers:

- **NGINX Ingress Controller**
    
- **Traefik**
    
- **Kong**
    

---

## What Ingress Can Do

- Host-based routing (`api.example.com`)
    
- Path-based routing (`/orders`)
    
- TLS/HTTPS termination
    
- Load balancing
    

---

## Simple Summary

Ingress is a **traffic router that directs external HTTP/HTTPS requests to the correct service inside a Kubernetes cluster**.
```

## Forward Proxy
```


A **Forward Proxy** sits **between the client and the internet**.

It forwards **client requests to external servers**.

### Flow

Client → Forward Proxy → Internet Server

### Example

Inside a company network:

Employee → Company Proxy → Google.com

The proxy:

- Sends the request to Google
    
- Returns the response to the employee
    

### Why used

- Security
    
- Content filtering
    
- Hiding client IP
    
- Caching
    

👉 Used to **control client access to the internet**.

```

## Reverse Proxy
```
A **Reverse Proxy** sits **in front of servers**.

Clients send requests to the proxy, and the proxy forwards them to backend servers.

### Flow

Client → Reverse Proxy → Backend Servers

### Example

User → NGINX → App Server

The reverse proxy:

- Receives request
    
- Chooses a backend server
    
- Returns response to user
    

### Why used

- Load balancing
    
- SSL termination
    
- Security
    
- Hiding backend servers
    

👉 Used to **manage access to servers**.

---

## Simple Difference

|Forward Proxy|Reverse Proxy|
|---|---|
|Protects **clients**|Protects **servers**|
|Client-side proxy|Server-side proxy|
|Client knows proxy|Client usually doesn’t know|

---

## Kubernetes Relation

**Ingress Controller** acts like a **Reverse Proxy**.
```


## Persistent Volume (PV)
```
A **Persistent Volume (PV)** is a **piece of storage in a Kubernetes cluster** that can be used by Pods to store data **persistently**.

Normally, containers are **stateless**, meaning when a Pod is deleted or restarted, all data inside it is lost.  
PV solves this problem by providing **external storage that survives Pod restarts**.

**Simple idea:**

Pod dies → Data still exists in PV

---

## Why Persistent Volumes Are Needed

Pods are **ephemeral (temporary)**.

Example:

A database Pod stores data inside the container filesystem.

Database Pod deleted → All database data lost

To avoid this, Kubernetes stores data **outside the Pod** using Persistent Volumes.

So even if the Pod restarts:

New Pod → reconnects to same storage → data remains

---

## Storage Architecture in Kubernetes

Kubernetes separates **storage provisioning** from **storage usage**.

The flow looks like this:

Pod → Persistent Volume Claim (PVC) → Persistent Volume (PV) → Actual Storage

### 1. Persistent Volume (PV)

Actual storage resource available in the cluster.

Examples:

- AWS EBS volume
    
- Google Persistent Disk
    
- NFS storage
    
- Local disk
    

Example PV definition:

apiVersion: v1  
kind: PersistentVolume  
metadata:  
  name: my-pv  
spec:  
  capacity:  
    storage: 10Gi  
  accessModes:  
    - ReadWriteOnce  
  hostPath:  
    path: /data/storage

This creates **10GB storage available in the cluster**.

---

### 2. Persistent Volume Claim (PVC)

A **PVC is a request for storage** made by a Pod.

Instead of directly using a PV, Pods request storage like this:

"I need 10GB storage"

Kubernetes then finds a suitable PV and **binds it to the claim**.

Example PVC:

apiVersion: v1  
kind: PersistentVolumeClaim  
metadata:  
  name: my-pvc  
spec:  
  accessModes:  
    - ReadWriteOnce  
  resources:  
    requests:  
      storage: 10Gi

Once created:

PVC → gets matched with PV

---

### 3. Pod Using the Storage

The Pod mounts the PVC as a **volume**.

Example:

apiVersion: v1  
kind: Pod  
metadata:  
  name: app  
spec:  
  containers:  
  - name: app-container  
    image: nginx  
    volumeMounts:  
    - mountPath: "/data"  
      name: storage  
  volumes:  
  - name: storage  
    persistentVolumeClaim:  
      claimName: my-pvc

Now inside the container:

/data → persistent storage

Even if the Pod is deleted, the data remains.

---

## Access Modes

These define **how the storage can be mounted**.

### ReadWriteOnce (RWO)

- Mounted by **one node**
    
- Most common mode
    

Example:

Single database Pod using storage

---

### ReadOnlyMany (ROX)

- Multiple nodes can **read**
    
- No writing allowed
    

Example:

Multiple services reading shared data

---

### ReadWriteMany (RWX)

- Multiple nodes can **read and write**
    

Example:

Shared file storage

Requires network storage like **NFS**.

---

## Storage Lifecycle

Persistent volumes have a lifecycle:

### 1. Provision

Storage is created (manually or dynamically).

### 2. Bind

PVC gets connected to a matching PV.

### 3. Use

Pods mount the PVC.

### 4. Release

PVC is deleted.

### 5. Reclaim

Storage is handled based on policy.

---

## Reclaim Policies

Defines what happens to storage after PVC deletion.

### Retain

Data stays in storage.

Example:

Admin manually cleans data

### Delete

Storage is deleted automatically.

Common in cloud environments.

### Recycle (deprecated)

Old data is scrubbed and reused.

---

## Static vs Dynamic Provisioning

### Static Provisioning

Admin creates PV manually.

Admin → PV  
User → PVC

---

### Dynamic Provisioning

Kubernetes automatically creates storage when PVC is requested.

Uses **StorageClass**.

PVC → StorageClass → Auto-created PV

This is the **most common modern approach**.

---

## Example Use Cases

Persistent Volumes are essential for:

- Databases (MySQL, PostgreSQL)
    
- Stateful applications
    
- Logging systems
    
- File storage
    
- Machine learning datasets
    

Stateless apps (like simple APIs) usually **don't require PVs**.

---

## Real-World Analogy

Think of a **Pod as a laptop** and a **Persistent Volume as an external hard drive**.

Laptop breaks → external hard drive still has the data

Similarly:

Pod deleted → PV still stores the data

---

## Key Points to Remember

- PV provides **persistent storage in Kubernetes**
    
- Data survives **Pod restarts**
    
- Pods do **not directly use PV**
    
- Pods request storage using **PVC**
    
- PV can be backed by **cloud disks, NFS, or local storage**
    

---

✅ **Summary**

A **Persistent Volume (PV)** is a cluster-wide storage resource that allows Kubernetes Pods to store data persistently, independent of the Pod lifecycle.
```

## **StorageClass**
```
**StorageClass** is a way to define **how storage should be dynamically provisioned** for a Persistent Volume (PV).

Instead of manually creating PVs, you define a StorageClass, and Kubernetes automatically creates PVs when a PersistentVolumeClaim (PVC) requests storage.

---

## 🔹 Simple Idea

Think of **StorageClass = storage template / policy**

It tells Kubernetes:

- Which storage backend to use (AWS EBS, GCP PD, Azure Disk, etc.)
- What type of disk (SSD, HDD, etc.)
- How to provision it (automatically or not)

---

## 🔹 Why StorageClass is needed

Without StorageClass:

- You must manually create PVs
- Hard to scale and manage

With StorageClass:

- PVC → triggers automatic PV creation
- Fully dynamic provisioning

---

## 🔹 How it works (flow)

1. You create a **StorageClass**
2. You create a **PVC** referencing that StorageClass
3. Kubernetes:
    - Uses the StorageClass
    - Dynamically creates a PV
    - Binds PV ↔ PVC

---

## 🔹 Example

### StorageClass

apiVersion: storage.k8s.io/v1  
kind: StorageClass  
metadata:  
  name: fast-storage  
provisioner: kubernetes.io/aws-ebs  
parameters:  
  type: gp3  
reclaimPolicy: Delete  
volumeBindingMode: Immediate

---

### PVC using StorageClass

apiVersion: v1  
kind: PersistentVolumeClaim  
metadata:  
  name: my-pvc  
spec:  
  accessModes:  
    - ReadWriteOnce  
  storageClassName: fast-storage  
  resources:  
    requests:  
      storage: 10Gi

---

## 🔹 Key Fields Explained

### 1. provisioner

- Defines **which storage system**
- Example:
    - `kubernetes.io/aws-ebs`
    - `kubernetes.io/gce-pd`

---

### 2. parameters

- Storage-specific configs
- Example:
    - SSD vs HDD
    - disk type, IOPS

---

### 3. reclaimPolicy

What happens when PVC is deleted:

- `Delete` → delete the disk
- `Retain` → keep the disk

---

### 4. volumeBindingMode

- `Immediate` → PV created instantly
- `WaitForFirstConsumer` → created when Pod is scheduled (better for topology-aware scheduling)

---

## 🔹 Types of provisioning

### 1. Static provisioning

- You manually create PV
- No StorageClass needed

### 2. Dynamic provisioning (most common)

- StorageClass automatically creates PV

---

## 🔹 Default StorageClass

- Kubernetes cluster can have a **default StorageClass**
- If PVC doesn’t specify one → default is used

---

## 🔹 One-line definition

👉 **StorageClass defines _how_ storage is created, while PVC defines _what_ storage is needed.**
```

```
**StatefulSet is not mainly about sharing a single volume; it is about giving each pod its own stable identity and its own storage.**

### If all Pods use the same PV

If multiple pods point to the **same PV**, they are sharing the same data.

Example:

Pod1  
Pod2  → same PV  
Pod3

Problems for databases or stateful apps:

- Data corruption
    
- Conflicts in writes
    
- No clear ownership of data
    

This is usually **not suitable for databases**.

---

### What StatefulSet Actually Does

StatefulSet gives **each pod its own dedicated storage**.

Example with 3 replicas:

db-0 → PV-0  
db-1 → PV-1  
db-2 → PV-2

Each pod:

- Has **stable name** (`db-0`, `db-1`, `db-2`)
    
- Gets **its own persistent volume**
    
- Keeps the **same storage even after restart**
    

---

### Why This Is Important

Stateful applications like:

- MySQL
    
- PostgreSQL
    
- Kafka
    
- Zookeeper
    

Need:

- Stable identity
    
- Dedicated storage
    
- Ordered startup
    

For example:

db-0 = primary  
db-1 = replica  
db-2 = replica

Each database must have **its own data directory**.

---

### Deployment vs StatefulSet

|Deployment|StatefulSet|
|---|---|
|Pods identical|Pods have identity|
|Pods share config|Pods have unique storage|
|Good for stateless apps|Good for databases|

---

### Simple Way to Remember

Deployment:

3 pods → interchangeable

StatefulSet:

3 pods → each has its own identity + storage

---

✅ **Answer to your doubt**

If all pods share one PV, they share the same data, which is unsafe for most stateful apps. **StatefulSet ensures each pod gets its own persistent volume and stable identity.**
```

## Liveness & Readiness Probes
```

In Kubernetes, Pods run your applications, but Kubernetes needs a way to **continuously check their health**.

That’s where **Probes** come in.

They are **periodic checks performed by Kubelet** to understand:

- Is the app working?
- Is it ready to handle traffic?

There are two main types:

---

# 1️⃣ Liveness Probe

### Purpose

To check whether the **application is still running correctly**.

It answers:

> “Is this container alive, or should I restart it?”

---

### When is it useful?

Sometimes:

- App enters **deadlock**
- Infinite loop
- Stops responding
- Internal error (but process is still running)

Kubernetes **cannot detect this automatically**.

Liveness probe solves this.

---

### What happens if it fails?

If Liveness probe fails:

Kubernetes → kills container → restarts it

---

### Example Scenario

App starts → works fine  
↓  
App gets stuck (no response)  
↓  
Liveness probe fails  
↓  
Container restarted

---

### Example Config

livenessProbe:  
  httpGet:  
    path: /health  
    port: 8080  
  initialDelaySeconds: 10  
  periodSeconds: 5

Kubernetes:

- Calls `/health` every 5 seconds
- If it fails repeatedly → restart container

---

# 2️⃣ Readiness Probe

### Purpose

To check whether the **application is ready to serve traffic**.

It answers:

> “Should this Pod receive requests?”

---

### When is it useful?

When:

- App is **starting up**
- App depends on **DB connection**
- App is temporarily overloaded
- App is initializing cache

Even though container is running, it may not be ready.

---

### What happens if it fails?

If Readiness probe fails:

Pod is removed from Service  
→ No traffic sent  
→ Pod is NOT restarted

---

### Example Scenario

Pod starts  
↓  
App initializing (not ready)  
↓  
Readiness = false → no traffic  
↓  
App ready → Readiness = true → traffic starts

---

### Example Config

readinessProbe:  
  httpGet:  
    path: /ready  
    port: 8080  
  initialDelaySeconds: 5  
  periodSeconds: 3

---

# Key Differences

|Feature|Liveness|Readiness|
|---|---|---|
|Purpose|Check if app is alive|Check if app is ready|
|Failure Action|Restart container|Stop traffic|
|Use Case|Deadlock / crash|Startup / dependency delay|

---

# Types of Probes

Both Liveness & Readiness can use:

### 1. HTTP Check

httpGet:  
  path: /health  
  port: 8080

---

### 2. TCP Check

tcpSocket:  
  port: 3306

---

### 3. Command Check

exec:  
  command:  
    - cat  
    - /tmp/healthy

---

# Important Parameters

- `initialDelaySeconds` → wait before first check
- `periodSeconds` → how often to check
- `failureThreshold` → how many failures before action
- `timeoutSeconds` → max time for each check

---

# Real-World Analogy

Think of a restaurant:

### Liveness

- Chef is alive or not
- If chef collapses → replace chef

### Readiness

- Restaurant open or not
- If kitchen not ready → don’t send customers

---

# Common Mistakes

- Not using probes → Kubernetes can't detect issues
- Using only Liveness → traffic may hit unready app
- Using aggressive settings → frequent restarts

---

# When to Use

|Scenario|Use|
|---|---|
|Stateless apps|Both probes|
|Database|Carefully configured probes|
|Startup delay|Readiness probe|

---

# Final Understanding

- Liveness ensures **app keeps running correctly**
- Readiness ensures **only healthy pods get traffic**

---

## How Probes Work (Internally)

Probes are executed by **Kubelet** (agent running on each node).

👉 Simple flow:

Kubelet → checks container → takes action

---

## Step-by-Step Working

### 1️⃣ Pod is running on a Node

- Kubelet is already running on that node
- It watches the Pod configuration

---

### 2️⃣ Kubelet sees probe config

Example:

livenessProbe:  
  httpGet:  
    path: /health  
    port: 8080  
  periodSeconds: 5

Kubelet understands:

- Every **5 seconds**, check `/health`

---

### 3️⃣ Kubelet executes probe

Depending on type:

- **HTTP** → sends HTTP request
- **TCP** → tries to open port
- **Exec** → runs command inside container

Example:

GET http://pod-ip:8080/health

---

### 4️⃣ Kubelet checks result

- Success (200 OK) → ✅ healthy
- Failure (timeout / error) → ❌ unhealthy

---

### 5️⃣ Based on Probe Type

#### 🔹 Liveness Probe

If it fails repeatedly:

Kubelet → restarts container

---

#### 🔹 Readiness Probe

If it fails:

Kubelet → marks pod as NOT READY  
Service → stops sending traffic

(No restart happens)

---

## Important Detail: Continuous Checking

Probes are **not one-time checks**.

They run continuously:

Every N seconds → check → decide

---

## Failure Handling

Kubernetes doesn’t react on first failure.

It uses:

failureThreshold: 3

Meaning:

Fail 3 times → then take action

---

## Full Flow Example

Pod starts  
↓  
Kubelet waits (initialDelaySeconds)  
↓  
Runs probe every few seconds  
↓  
If success → continue  
↓  
If fails multiple times:  
    Liveness → restart  
    Readiness → stop traffic

---

## Key Understanding

- Kubelet = **health checker**
- Probes = **rules for checking**
- Actions = **restart or remove from traffic**

---

## Simple Analogy

Think of a **doctor checking a patient regularly**:

- Checks pulse → (Liveness)
- Checks if patient can work → (Readiness)

---

## When Kubernetes Restarts Pods Automatically

Kubernetes restarts containers **only if the process crashes**.

Example:

App crashes (process exits)  
→ Kubelet detects it  
→ Restarts container

This is based on **restartPolicy (usually Always)**.

---

## Problem: What if App is NOT crashed?

Sometimes:

- App is **stuck (deadlock)**
- App is **not responding**
- App is **running but broken**

Example:

while(true) {}  // infinite loop

- Process is still running ❗
- Kubernetes thinks → "everything is fine" ❗

👉 No restart happens ❌

---

## Why Liveness Probe is Needed

Liveness detects these **hidden failures**.

App stuck  
→ Liveness probe fails  
→ Kubernetes restarts container

---

## Readiness Case

Even if app is running:

- DB not connected
- App still starting

Without readiness:

Traffic → goes to pod → errors ❌

With readiness:

Pod not ready → no traffic ✅

---

## Key Difference

|Situation|Without Probe|With Probe|
|---|---|---|
|App crash|Restarted|Restarted|
|App stuck|❌ No restart|✅ Restart|
|App not ready|❌ Gets traffic|✅ No traffic|

---

## Final Understanding

- Kubernetes restarts **only when container exits**
- It **cannot detect internal issues**
- Probes help detect:
    - Stuck apps (Liveness)
    - Not-ready apps (Readiness)

```

# 🧠 Core Difference (one-liner each)
```
- **Liveness Probe** → _“Is my app broken?”_ → restart it
- **Readiness Probe** → _“Can this app handle traffic?”_ → route traffic or not
- **Startup Probe** → _“Has the app finished starting?”_ → delay other probes
```

```bash

Container starts
  │
  ├─ Startup Probe runs repeatedly ──→ SUCCESS ──→ Startup probe stops
  │   (if keeps failing, container     │
  │    gets killed & restarted)        │
  │                                    ├─ Liveness Probe starts (runs forever)
  │                                    │   Fails? → restart container
  │                                    │
  │                                    └─ Readiness Probe starts (runs forever)
  │                                        Fails? → stop sending traffic

```

## Pod Status
```
Pods have a **lifecycle with different statuses (phases)**.

---

## 1️⃣ Pending

👉 Pod is created but not running yet

Reasons:

- Node not assigned
- Image pulling
- Resources not available

---

## 2️⃣ Running

👉 Pod is running on a node

- Container is started
- But app **may or may not be ready**

---

## 3️⃣ Succeeded

👉 Pod finished successfully

- All containers exited with **exit code 0**

Example:

- Batch job completed

---

## 4️⃣ Failed

👉 Pod finished with error

- Container exited with **non-zero code**

Example:

- Script failed

---

## 5️⃣ Unknown

👉 Kubernetes can’t determine status

- Node communication issue

---

## Important Note

These are **Pod phases**, not detailed states.

You may also see:

- `ContainerCreating`
- `CrashLoopBackOff`
- `ImagePullBackOff`

👉 These are **detailed conditions (kubectl output)**, not actual phases.

---

## Simple Flow

Pending → Running → Succeeded / Failed

---

## Key Understanding

- **Running ≠ Ready**
- Readiness is controlled by probes

---

🧠 Memory Trick:

Pending = waiting  
Running = started  
Succeeded = done  
Failed = error


## 1️⃣ ContainerCreating

👉 Means: **Container is being set up**

### What is happening:

- Image is being pulled
- Volume is being attached
- Network is being configured

Pod → Pending → ContainerCreating → Running

### Key Point:

- This is a **temporary state**
- Nothing is wrong

---

## 2️⃣ CrashLoopBackOff

👉 Means: **Container is crashing again and again**

### What is happening:

Container starts → crashes → restart  
→ crashes again → restart  
→ Kubernetes slows down retries (backoff)

### Reasons:

- App error
- Wrong config
- Missing env variables
- DB connection failure

### Key Point:

- Kubernetes is trying to restart, but failing repeatedly

---

## 3️⃣ ImagePullBackOff

👉 Means: **Kubernetes failed to pull container image**

### What is happening:

Kubernetes tries to pull image → fails  
→ retries with delay (backoff)

### Reasons:

- Wrong image name
- Image not found
- Private repo (no credentials)
- Network issue

---

## Quick Comparison

|Status|Meaning|
|---|---|
|ContainerCreating|Setting up container|
|CrashLoopBackOff|App keeps crashing|
|ImagePullBackOff|Image download failed|

---

## Simple Way to Remember

- **ContainerCreating** → “Setting up…”
- **CrashLoopBackOff** → “App is broken”
- **ImagePullBackOff** → “Can’t download image”

---

## Important

These are **runtime states shown by kubectl**, not official Pod phases.

---

✅ **Summary**

These states help you debug issues:

- ContainerCreating → normal startup
- CrashLoopBackOff → app crash loop
- ImagePullBackOff → image pull failure
```

## Init Containers
```
**Init Containers** are containers that run **before the main application container starts**.

👉 Simple:  
**Init Containers = Setup containers that run first**

---

## Why they are needed

Sometimes your app needs setup before starting:

- Wait for database
- Download config/data
- Run migrations
- Prepare files

Instead of putting this logic in main app, we use init containers.

---

## How they work

Init Container 1 → completes  
↓  
Init Container 2 → completes  
↓  
Main Container starts

- Run **one by one (sequentially)**
- Must **complete successfully**
- Only then main container starts

---

## Important Behavior

- If init container fails → Pod **restarts**
- Main container **will NOT start** until all init containers succeed

---

## Example

initContainers:  
- name: wait-for-db  
  image: busybox  
  command: ['sh', '-c', 'until nc -z db 3306; do sleep 2; done']

This waits until DB is available.

---

## Use Cases

- Wait for dependent service
- Set permissions
- Load initial data
- Configuration setup

---

## Key Difference

|Init Container|Main Container|
|---|---|
|Runs first|Runs after init|
|Temporary|Long-running|
|Sequential|Runs normally|

---

## Simple Analogy

Think of cooking:

- Init containers → **Preparation (cutting, cleaning)**
- Main container → **Actual cooking**
  

For example, we have Kafka-init what it can do?

## Kafka Init Container – What it can do

A **Kafka init container** is used to **prepare everything needed before your app starts using Kafka**.

👉 Simple:  
**Kafka-init = setup Kafka-related things before main app runs**

---

## Common Things Kafka Init Can Do

### 1️⃣ Wait for Kafka to be ready

Kafka might not be up yet.

Check: kafka:9092 is reachable  
→ only then start app

Prevents app from failing at startup.

---

### 2️⃣ Create Topics

Instead of manually creating topics:

Create topic: orders  
Create topic: payments

Init container ensures topics exist.

---

### 3️⃣ Setup Config / Permissions

- Configure ACLs (access control)
- Set replication configs
- Initialize schemas (if using Schema Registry)

---

### 4️⃣ Run Migrations / Setup Scripts

- Preload messages
- Setup environment required for app

---

## Example Flow

Kafka-init container starts  
↓  
Waits for Kafka  
↓  
Creates topics  
↓  
Exits successfully  
↓  
Main app container starts

---

## Why Use Init Instead of Main App?

Without init:

App starts → Kafka not ready → crash ❌

With init:

Init waits → setup done → app starts safely ✅

---

## Simple Example Command

initContainers:  
- name: kafka-init  
  image: confluentinc/cp-kafka  
  command: ["sh", "-c", "kafka-topics --create --topic orders --bootstrap-server kafka:9092"]

---

## Simple Understanding

- Init container = **setup Kafka environment**
- Main app = **use Kafka**
```


## Multi-Container Pods (Sidecar Pattern)
```
A **multi-container Pod** means **multiple containers run inside the same Pod**.

👉 Simple:  
**Sidecar = Helper container that runs alongside main app**

---

## Why multiple containers in one Pod?

Containers in same Pod:

- Share **network (same IP, localhost)**
- Share **storage (volumes)**

So they can work closely together.

---

## Sidecar Pattern

One **main container** + one (or more) **helper containers**

Main App Container  
        +  
Sidecar Container (helper)

---

## What Sidecar Can Do

### 1️⃣ Logging

- Sidecar collects logs from app
- Sends to monitoring system

---

### 2️⃣ Proxy / Networking

- Sidecar acts as proxy (like Envoy)
- Handles traffic, security

---

### 3️⃣ Config Reloading

- Watches config changes
- Updates app dynamically

---

### 4️⃣ Data Sync

- Sync files from external storage

---

## Example

containers:  
- name: app  
  image: my-app  
  
- name: log-sidecar  
  image: busybox  
  command: ["sh", "-c", "tail -f /var/log/app.log"]

- App writes logs
- Sidecar reads and processes logs

---

## How They Work

Pod starts  
↓  
All containers start together  
↓  
Sidecar supports main container continuously

---

## Important Points

- All containers share:
    - IP (localhost)
    - Volumes
- They are **tightly coupled**
- If Pod dies → all containers die

---

## Real Example

- Istio → uses sidecar proxy (Envoy)
- Logging agents → Fluentd sidecar

---

## Simple Analogy

Think of a **driver and assistant**:

- Driver (main container) → does main work
- Assistant (sidecar) → helps (navigation, support)

---

✅ **Summary**

Sidecar pattern is a design where a helper container runs alongside the main container in the same Pod to provide additional functionality like logging, proxying, or configuration.
```

## Jobs
```
## Jobs (Kubernetes)

A **Job** is used to run a **task that completes once and then stops**.

👉 Simple:  
**Job = Run a task → finish → exit**

---

## Why use Job?

For tasks like:

- Batch processing
- Database migration
- Sending emails
- Data processing scripts

These are **not long-running apps**.

---

## How it works

Job starts  
↓  
Creates Pod  
↓  
Pod runs task  
↓  
Task completes → Pod stops

---

## Important Behavior

- If Pod fails → Job **retries**
- If Pod succeeds → Job **completes**
- Ensures task runs **successfully at least once**

---

## Example

apiVersion: batch/v1  
kind: Job  
metadata:  
  name: my-job  
spec:  
  template:  
    spec:  
      containers:  
      - name: job-container  
        image: busybox  
        command: ["echo", "Hello World"]  
      restartPolicy: Never

---

## Key Properties

- `completions` → how many times to run
- `parallelism` → how many pods run at once
- `backoffLimit` → retry count on failure

---

## Example Behavior

completions: 3  
parallelism: 2

- 3 total runs needed
- 2 pods run in parallel

---

## Job vs Deployment

|Job|Deployment|
|---|---|
|Runs once|Runs forever|
|Completes|Keeps running|
|Batch work|Long-running apps|

---

## Simple Analogy

- Job = **Exam** (once done, finished)
- Deployment = **Office job** (runs continuously)

---

✅ **Summary**

A Job runs a task until completion, ensuring it finishes successfully, and then stops.

where did you mention in Job

Ah got it 👍 you’re asking **where/how these are defined in a Job**.

They are part of the **Job spec (configuration)**.

---






## Where they are mentioned

They are defined inside:

spec:

---

## Example with all of them

apiVersion: batch/v1  
kind: Job  
metadata:  
  name: example-job  
spec:  
  completions: 3  
  parallelism: 2  
  backoffLimit: 4  
  
  template:  
    spec:  
      containers:  
      - name: job-container  
        image: busybox  
        command: ["echo", "Hello"]  
      restartPolicy: Never

---

## What each means (in this example)

### 🔹 `completions: 3`

- Job must run **3 successful executions**

---

### 🔹 `parallelism: 2`

- At a time, **2 Pods run in parallel**

Flow:

Run 2 pods → complete  
Run 1 more pod → complete

---

### 🔹 `backoffLimit: 4`

- If a Pod fails → retry up to **4 times**

---

## Key Understanding

All these are part of:

Job → spec → controls execution behavior

---

## Simple Summary

- `completions` → total work
- `parallelism` → speed (how many at once)
- `backoffLimit` → retry count
```

## CronJob (Kubernetes)
```
A **CronJob** is used to run a **Job on a schedule (time-based)**.

👉 Simple:  
**CronJob = Job + Schedule**

---

## Why use CronJob?

For tasks like:

- Daily backups
- Cleanup jobs
- Reports generation
- Periodic data sync

---

## How it works

CronJob triggers  
↓  
Creates a Job  
↓  
Job creates Pod  
↓  
Pod runs task → completes

---

## Schedule Format

Uses **cron format**:

* * * * *  
| | | | |  
| | | | └ Day of week  
| | | └── Month  
| | └──── Day of month  
| └────── Hour  
└──────── Minute

### Example

*/5 * * * *

👉 Runs every 5 minutes

---

## Example YAML

apiVersion: batch/v1  
kind: CronJob  
metadata:  
  name: my-cronjob  
spec:  
  schedule: "*/5 * * * *"  
  jobTemplate:  
    spec:  
      template:  
        spec:  
          containers:  
          - name: job-container  
            image: busybox  
            command: ["echo", "Hello"]  
          restartPolicy: Never

---

## Important Fields

- `schedule` → when to run
- `jobTemplate` → what to run (Job spec)
- `successfulJobsHistoryLimit` → how many success jobs to keep
- `failedJobsHistoryLimit` → how many failed jobs to keep

---

## Key Behavior

- Creates a **new Job each time**
- Jobs run independently
- Old Jobs can be cleaned up

---

## Job vs CronJob

|Job|CronJob|
|---|---|
|Runs once|Runs repeatedly|
|Manual trigger|Scheduled trigger|

---

## Simple Analogy

- Job = **Run task now**
- CronJob = **Alarm clock ⏰ that runs task repeatedly**

---

✅ **Summary**

A CronJob schedules and automatically runs Jobs at specified times using cron expressions.
```

## When to Use Job vs CronJob
```

## 🟢 Use **Job** when

👉 You want to **run a task once (on-demand)**

### Examples:

- Database migration
- One-time data processing
- Sending emails once
- Initial setup scripts

### Behavior:

kubectl apply → Job runs → completes → done

---

## 🔵 Use **CronJob** when

👉 You want to **run a task repeatedly on a schedule**

### Examples:

- Daily backups
- Log cleanup every hour
- Generate reports every day
- Sync data every 5 minutes

### Behavior:

Time matches schedule → CronJob triggers → Job runs

---

## Key Difference

|Feature|Job|CronJob|
|---|---|---|
|Execution|One-time|Repeated (scheduled)|
|Trigger|Manual|Automatic (time-based)|
|Use case|Immediate task|Periodic task|

---

## Simple Rule

- Job → **“Run now”**
- CronJob → **“Run every X time”**

---

## Real-World Example

- Job → “Backup database **right now**”
- CronJob → “Backup database **every day at 2 AM**”
```


## DaemonSet
```
A **DaemonSet** ensures that a **Pod runs on every Node in the cluster**.

👉 Simple:  
**DaemonSet = One Pod per Node**

---

## Why it is needed

Some applications must run on **all nodes**, not just some.

Examples:

- Log collection (Fluentd)
- Monitoring agents (Prometheus node exporter)
- Security agents

---

## How it works

Cluster has 3 Nodes  
↓  
DaemonSet creates:  
Node1 → Pod  
Node2 → Pod  
Node3 → Pod

If a new node is added:

New Node added  
↓  
DaemonSet automatically creates Pod on it

---

## Important Behavior

- Ensures **exactly one Pod per Node**
- Automatically handles:
    - Node added → Pod created
    - Node removed → Pod removed

---

## Example

apiVersion: apps/v1  
kind: DaemonSet  
metadata:  
  name: log-agent  
spec:  
  selector:  
    matchLabels:  
      app: log-agent  
  template:  
    metadata:  
      labels:  
        app: log-agent  
    spec:  
      containers:  
      - name: fluentd  
        image: fluentd

---

## DaemonSet vs Deployment

|Deployment|DaemonSet|
|---|---|
|Runs N replicas|Runs 1 per node|
|Manual scaling|Auto per node|
|App workloads|Node-level services|

---

## Simple Analogy

- Deployment → “Run 3 copies anywhere”
- DaemonSet → “Run 1 copy on every machine”

---

✅ **Summary**

A DaemonSet ensures that a specific Pod runs on every node in the cluster, typically used for system-level services like logging and monitoring.
```

## Taints & Tolerations (Kubernetes)
```
They control **which Pods can run on which Nodes**.

👉 Simple:  
**Taint = Node says “don’t come here”**  
**Toleration = Pod says “I can come”**

---

## Why needed

To:

- Reserve nodes for specific workloads
- Prevent unwanted Pods on certain nodes
- Control scheduling strictly

---

## Taints (on Node)

A **taint** is applied to a Node to **repel Pods**.

Example:

key=value:NoSchedule

Meaning:

- Don’t schedule any Pod on this node unless it tolerates it

---

## Tolerations (on Pod)

A **toleration** allows a Pod to **ignore the taint**.

Example:

tolerations:  
- key: "key"  
  operator: "Equal"  
  value: "value"  
  effect: "NoSchedule"

Now this Pod **can be scheduled** on that Node.

---

## Effects of Taints

### 1️⃣ NoSchedule

- Pod **will not be scheduled** on node

---

### 2️⃣ PreferNoSchedule

- Try to avoid scheduling (not strict)

---

### 3️⃣ NoExecute

- Pod will be **removed (evicted)** if already running

---

## Example Flow

Node has taint  
↓  
Pod without toleration → blocked ❌  
Pod with toleration → allowed ✅

---

## Real Use Cases

- Dedicated nodes for:
    - Databases
    - GPUs
    - Critical workloads
- Prevent normal apps from using special nodes

---

## Simple Analogy

- Node = “VIP area 🚫”
- Taint = “No entry” sign
- Toleration = “VIP pass 🎟️”

---

## Key Point

Toleration **does not force scheduling**,  
it only **allows** it.

Scheduler still decides placement.

---

✅ **Summary**

Taints restrict Pods from being scheduled on Nodes, and Tolerations allow specific Pods to bypass those restrictions.
```


# Helm
```
---

## 1. Helm Basics

Helm is the **package manager for Kubernetes** — like `apt` for Ubuntu or `npm` for Node.js, but for K8s applications.

### Core Components

|Component|Description|
|---|---|
|**Helm CLI**|Command-line tool. Reads charts, fills values, talks to K8s API|
|**Chart**|A packaged bundle of K8s templates + default config|
|**Release**|A named, versioned instance of a chart running in a cluster|
|**Repository**|Remote registry where charts are published (e.g. Artifact Hub)|
|**Release History**|Stored as Secrets inside the cluster — enables rollbacks|

### How it works

1. You run `helm install`
2. Helm reads the chart + your value overrides
3. Renders Go templates into plain K8s YAML
4. Applies them to the cluster via the K8s API
5. Stores the release state as a Secret in the cluster

### Common Commands

helm install <name> <chart>         # Deploy a chart
helm upgrade <name> <chart>         # Upgrade a release
helm rollback <name> <revision>     # Roll back to a revision
helm uninstall <name>               # Remove a release
helm list                           # List all releases
helm history <name>                 # Show revision history
helm status <name>                  # Show release status
helm repo add <name> <url>          # Add a chart repository
helm search repo <keyword>          # Search charts in repos


---

## 2. Charts

A chart is a directory (or `.tgz` tarball) with a specific layout.

### Chart Directory Structure


mychart/
├── Chart.yaml          # Chart metadata (name, version, description)
├── values.yaml         # Default configuration values
├── charts/             # Dependencies / subcharts
├── templates/          # Go-templated K8s manifest files
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── configmap.yaml
│   ├── _helpers.tpl    # Reusable template snippets (not rendered)
│   └── NOTES.txt       # Printed to terminal after install
└── .helmignore         # Files to ignore when packaging


### Chart.yaml


apiVersion: v2              # Always v2 for Helm 3
name: mychart
version: 1.0.0              # Chart version (semver)
appVersion: "2.3.1"         # App version inside (informational only)
description: A sample Helm chart
type: application           # 'application' (default) or 'library'
dependencies:               # External chart dependencies
  - name: postgresql
    version: "12.1.0"
    repository: "https://charts.bitnami.com/bitnami"


### Chart Types

|Type|Description|
|---|---|
|`application`|Default. Installs K8s resources when deployed|
|`library`|Contains only reusable helpers. Cannot be installed directly|

### Managing Dependencies


helm dependency update ./mychart    # Download declared dependencies into charts/
helm dependency list ./mychart      # List all dependencies

---

## 3. values.yaml

`values.yaml` is the configuration layer of a chart. It provides defaults that users override per environment.

### Defining Values


# values.yaml
replicaCount: 2

image:
  repository: nginx
  tag: "latest"
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80

ingress:
  enabled: false
  host: "myapp.example.com"

env:
  APP_ENV: production
  LOG_LEVEL: info

resources:
  limits:
    cpu: 500m
    memory: 128Mi
  requests:
    cpu: 250m
    memory: 64Mi


### Accessing Values in Templates


replicas: {{ .Values.replicaCount }}
image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"


### Override Priority (lowest → highest)

1. `values.yaml` — chart defaults
2. Parent chart values — if this is a subchart
3. `-f myvals.yaml` — user-supplied values file
4. `--set key=value` — inline CLI flag (highest priority)

### Override Patterns


# Override with a file (recommended for environments)
helm install myapp ./chart -f prod-values.yaml

# Override inline
helm install myapp ./chart --set image.tag=v2.1.0

# Combine both
helm install myapp ./chart -f prod.yaml --set image.tag=$CI_TAG

# Set a nested key
helm install myapp ./chart --set ingress.host=myapp.example.com

# Set a list value
helm install myapp ./chart --set "hosts[0]=foo.com,hosts[1]=bar.com"


### Built-in Objects Available in Templates

|Object|Description|
|---|---|
|`.Values`|All values from values.yaml + overrides|
|`.Release.Name`|Name of the release|
|`.Release.Namespace`|Target namespace|
|`.Release.IsInstall`|True if this is a fresh install|
|`.Release.IsUpgrade`|True if this is an upgrade|
|`.Chart.Name`|Chart name from Chart.yaml|
|`.Chart.Version`|Chart version|
|`.Files`|Access non-template files in chart|
|`.Capabilities`|Info about the K8s cluster|

---

## 4. Templates

Templates are K8s manifest files with **Go template syntax**. Helm uses Go's `text/template` engine plus the **Sprig** library (100+ utility functions).

### Basic Syntax


# {{ }} — output a value
name: {{ .Release.Name }}

# {{- }} — trim preceding whitespace
# {{ -}} — trim trailing whitespace
name: {{- .Release.Name -}}


### Conditionals


{{- if .Values.ingress.enabled }}
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{ .Release.Name }}-ingress
{{- end }}

# if / else if / else
{{- if eq .Values.service.type "LoadBalancer" }}
  # LoadBalancer config
{{- else if eq .Values.service.type "NodePort" }}
  # NodePort config
{{- else }}
  # ClusterIP config
{{- end }}


### Loops (range)


# Loop over a list
ports:
{{- range .Values.service.ports }}
  - port: {{ .port }}
    protocol: {{ .protocol }}
{{- end }}

# Loop over a map
env:
{{- range $key, $val := .Values.env }}
  - name: {{ $key }}
    value: {{ $val | quote }}
{{- end }}


### Named Templates (_helpers.tpl)


# Define in _helpers.tpl
{{- define "mychart.labels" -}}
app.kubernetes.io/name: {{ .Chart.Name }}
app.kubernetes.io/instance: {{ .Release.Name }}
app.kubernetes.io/version: {{ .Chart.AppVersion }}
{{- end }}

# Use in any template with 'include' + nindent
metadata:
  labels:
    {{- include "mychart.labels" . | nindent 4 }}


> Use `include` (not `template`) because `include` can be piped to functions like `nindent`.

### Useful Sprig Functions


{{ .Values.name | upper }}              # MYAPP
{{ .Values.name | lower }}              # myapp
{{ .Values.name | quote }}              # "myapp"
{{ .Values.name | trunc 63 }}           # truncate to 63 chars
{{ .Values.replicas | default 1 }}      # fallback if not set
{{ .Values.tag | required "tag required!" }}  # fail if empty
{{ now | date "2006-01-02" }}           # current date
{{ .Values.name | replace "-" "_" }}    # string replace
{{ list "a" "b" "c" | join "," }}       # join list → a,b,c
{{ .Values.port | int }}                # cast to integer


### Debugging Templates


helm template myapp ./chart                     # render locally, no install
helm template myapp ./chart --debug             # show computed values too
helm template myapp ./chart -f prod.yaml        # render with override file
helm lint ./chart                               # validate syntax
helm get manifest myapp                         # see what's deployed in cluster


---

## 5. Helm Hooks

Hooks let you run K8s Jobs/Pods at specific points in the release lifecycle. They are defined by adding a special annotation to any K8s resource.

### Defining a Hook


apiVersion: batch/v1
kind: Job
metadata:
  name: {{ .Release.Name }}-db-migrate
  annotations:
    "helm.sh/hook": pre-upgrade
    "helm.sh/hook-weight": "0"
    "helm.sh/hook-delete-policy": hook-succeeded
spec:
  template:
    spec:
      restartPolicy: Never
      containers:
        - name: migrate
          image: myapp:{{ .Values.image.tag }}
          command: ["./migrate.sh"]


### Available Hook Types

|Hook|When it runs|
|---|---|
|`pre-install`|Before any resources are installed|
|`post-install`|After all resources are installed|
|`pre-upgrade`|Before the upgrade begins|
|`post-upgrade`|After the upgrade completes|
|`pre-rollback`|Before a rollback|
|`post-rollback`|After a rollback|
|`pre-delete`|Before uninstall|
|`post-delete`|After uninstall|
|`test`|Runs when `helm test` is called|

### Hook Weights

When multiple hooks share the same type, they run in **ascending weight order**. Negative weights are allowed.

```yaml
"helm.sh/hook-weight": "-5"   # runs first
"helm.sh/hook-weight": "0"    # runs second
"helm.sh/hook-weight": "10"   # runs third


### Hook Delete Policies

|Policy|Behaviour|
|---|---|
|`before-hook-creation`|Delete old hook resource before creating new one (default)|
|`hook-succeeded`|Delete after successful completion|
|`hook-failed`|Delete even if the hook fails|

### Common Hook Use Cases

- `pre-install` / `pre-upgrade` → database migrations
- `post-install` → seed initial data, send notifications, warm caches
- `pre-delete` → drain connections, take a data backup
- `test` → run integration tests against a live release

> Helm **blocks** the install/upgrade until the hook Job reaches `Completed`. If a hook fails, the release fails — great for safe DB migrations.

---

## 6. Upgrade & Rollback

### Upgrade

Upgrade deploys a new version of a chart to an existing release.


# Basic upgrade
helm upgrade myapp ./chart

# Upgrade with new image tag
helm upgrade myapp ./chart --set image.tag=v2.0.0

# Upgrade with a values file
helm upgrade myapp bitnami/nginx -f prod-values.yaml

# Upgrade or install if not present (upsert — very common in CI/CD)
helm upgrade --install myapp ./chart

# Recommended CI/CD upgrade command
helm upgrade --install myapp ./chart \
  --set image.tag=$IMAGE_TAG \
  --atomic \
  --timeout 3m \
  --cleanup-on-fail


### Upgrade Flags

|Flag|Behaviour|
|---|---|
|`--atomic`|Auto-rollback if upgrade fails. Best for CI/CD|
|`--cleanup-on-fail`|Delete new resources created during a failed upgrade|
|`--timeout 5m`|Wait up to 5m for resources to become ready|
|`--dry-run`|Simulate upgrade, show rendered YAML, do not apply|
|`--history-max 10`|Keep only last 10 revisions|
|`--force`|Force resource updates (deletes and recreates pods)|
|`--reset-values`|Reset all values to chart defaults before applying|
|`--reuse-values`|Reuse last release's values + apply new overrides|

### Rollback


# View revision history
helm history myapp

# Roll back to a specific revision
helm rollback myapp 2

# Roll back to the previous revision
helm rollback myapp

# Roll back in a specific namespace
helm rollback myapp 2 -n production


### How Rollback Works

- Rollback **creates a NEW revision** — it does not rewind the counter.
- Example: on revision 3, rolling back to revision 2 → creates revision 4 (a copy of revision 2's state).
- History is always **append-only and auditable**.


Revision 1  →  Revision 2  →  Revision 3 (failed)
                    ↑                 |
                    └── Rollback ─────┘
                    
After rollback: Revision 4 = copy of Revision 2


### Inspecting a Release


helm get values myapp               # values used in last install/upgrade
helm get values myapp --revision 2  # values used at revision 2
helm get manifest myapp             # rendered K8s YAML currently deployed
helm get all myapp                  # everything (values + manifest + hooks)
helm status myapp                   # current status and NOTES.txt output


---

## Quick Reference

### Full Workflow


# 1. Add a repo
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

# 2. Search for a chart
helm search repo nginx

# 3. Inspect a chart before installing
helm show values bitnami/nginx > my-values.yaml
# Edit my-values.yaml...

# 4. Install
helm install my-nginx bitnami/nginx -f my-values.yaml -n production

# 5. Check status
helm status my-nginx -n production
helm list -n production

# 6. Upgrade
helm upgrade my-nginx bitnami/nginx -f my-values.yaml --atomic

# 7. Roll back if needed
helm history my-nginx -n production
helm rollback my-nginx 1 -n production

# 8. Uninstall
helm uninstall my-nginx -n production


### Cheat Sheet

|Command|Description|
|---|---|
|`helm install`|Deploy a chart as a new release|
|`helm upgrade`|Upgrade an existing release|
|`helm upgrade --install`|Install if not exists, upgrade if exists|
|`helm rollback`|Roll back to a previous revision|
|`helm uninstall`|Remove a release from the cluster|
|`helm list`|List all releases|
|`helm history`|Show revision history of a release|
|`helm status`|Show status of a release|
|`helm get values`|Show values used in a release|
|`helm get manifest`|Show rendered manifests of a release|
|`helm template`|Render templates locally without installing|
|`helm lint`|Validate a chart for errors|
|`helm dependency update`|Download/update chart dependencies|
|`helm repo add`|Add a chart repository|
|`helm repo update`|Refresh repo index|
|`helm search repo`|Search for charts in added repos|
|`helm show values`|Display a chart's default values|
|`helm package`|Package a chart directory into a .tgz|
```