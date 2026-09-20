# ☸️ Kubernetes (K8s) BY =Ritwik Raj 

A practical, beginner-friendly guide to Kubernetes — from understanding the fundamentals to practically undertand how if a college student makes a basic MERN app lets say- TODO app , which initially has just a frontend and a backend folder which that student can host locally easily and use it BUT - BUT : this document takes it to the next step on undertanding how can that student use kubernetes to deplye the same basic mern app .

HOW TO USE THIS DOC :-
1. based on my exp this doc first has basic intro to what kubernetes is and it mostly rely on analogy of practical exp of how if some one knows
   basic operation can he deploy his app .
2. The broader goal is to give a high level idea how deployment happens at enterprise level using simple relatable example.
3. I would advice the reader to use AI to question each section and clear its doubts treat this doc as the basic things that kubernetes user should know
   





The goal of this guide is **not just to memorize Kubernetes commands**.

Instead, we will take one simple application and gradually transform it:

```text
Local MERN Application
        ↓
Docker Containers
        ↓
Kubernetes Pods
        ↓
ReplicaSets
        ↓
Deployments
        ↓
Services
        ↓
Scaling
        ↓
Persistent Storage
        ↓
Multi-Node Kubernetes Cluster
        ↓
Production-style Deployment
```

By the end, you should understand not only **what Kubernetes resources are**, but also **why they exist and what Kubernetes actually does behind the scenes**.

---

# 📚 Table of Contents

* [What is Kubernetes?](#-what-is-kubernetes)
* [The Application We Will Deploy](#-the-application-we-will-deploy)
* [Our MERN Todo Application](#-our-mern-todo-application)
* [Running the Application Locally](#-running-the-application-locally)
* [The Problem Before Kubernetes](#-the-problem-before-kubernetes)
* [Containerizing the Application](#-containerizing-the-application)
* [What is Kubernetes?](#-what-is-kubernetes-1)
* [Kubernetes Cluster](#-kubernetes-cluster)
* [Cluster Architecture](#-cluster-architecture)
* [Control Plane](#-control-plane)
* [API Server](#-api-server-kube-apiserver)
* [etcd](#-etcd)
* [kube-scheduler](#-kube-scheduler)
* [kube-controller-manager](#-kube-controller-manager)
* [Worker Nodes](#-worker-nodes)
* [kubelet](#-kubelet)
* [kube-proxy](#-kube-proxy)
* [Container Runtime](#-container-runtime)
* [Pods](#-pods)
* [Creating a Kubernetes Cluster](#-creating-a-kubernetes-cluster)
* [Single Node Cluster](#-single-node-cluster)
* [Multi Node Cluster](#-multi-node-cluster)
* [kubectl and Contexts](#-kubectl-and-contexts)
* [Deploying Our First Pod](#-deploying-our-first-pod)
* [Problem with Raw Pods](#-problem-with-raw-pods)
* [ReplicaSet](#-replicaset)
* [Deployment](#-deployment)
* [Rollouts and Rollbacks](#-rollouts-and-rollbacks)
* [Internal Flow of a Deployment](#-internal-flow-of-a-deployment)
* [Deploying the React Frontend](#-deploying-the-react-frontend)
* [Services](#-services)
* [ClusterIP](#-clusterip)
* [NodePort](#-nodeport)
* [LoadBalancer](#-loadbalancer)
* [Frontend to Backend Communication](#-frontend-to-backend-communication)
* [Deploying MongoDB](#-deploying-mongodb)
* [ConfigMaps](#-configmaps)
* [Secrets](#-secrets)
* [Persistent Storage](#-persistent-storage)
* [Health Checks](#-health-checks)
* [Scaling](#-scaling)
* [Multi-Node Application Architecture](#-multi-node-application-architecture)
* [Debugging Kubernetes](#-debugging-kubernetes)
* [Complete Application Architecture](#-complete-application-architecture)
* [Production Kubernetes Concepts](#-production-kubernetes-concepts)
* [From Local Kubernetes to Cloud](#-from-local-kubernetes-to-cloud)
* [Useful kubectl Commands](#-useful-kubectl-commands)
* [Final Mental Model](#-final-mental-model)

---

# 🚀 What is Kubernetes?

Kubernetes, often written as **K8s**, is a container orchestration system.

Let's break that sentence down.

### Container

A container is a packaged application that contains the application and the dependencies required to run it.

For example:

```text
Node.js Application
+
Node.js Runtime
+
Dependencies
+
Configuration
        ↓
     Container
```

### Orchestration

Orchestration means automatically managing many containers.

Instead of manually doing:

```bash
docker run my-app
docker run my-app
docker run my-app
```

we can tell Kubernetes:

```text
"I want 3 instances of my application running."
```

Kubernetes then works continuously to maintain that desired state.

It can:

* Deploy applications
* Restart failed containers
* Replace failed Pods
* Scale applications
* Distribute Pods across machines
* Perform rolling updates
* Roll back failed deployments
* Provide service discovery
* Route network traffic
* Manage application configuration
* Manage persistent storage

Think of Kubernetes as a **manager for containerized applications running across multiple machines**.

---

# 📝 The Application We Will Deploy

Instead of learning Kubernetes only through isolated YAML examples, we will use one application throughout this guide.

We will build a simple:

# 📝 MERN Todo Application

The stack is:

```text
Frontend
    ↓
React

Backend
    ↓
Node.js + Express

Database
    ↓
MongoDB
```

The application will allow users to:

* Create todos
* View todos
* Update todos
* Delete todos

The application itself is intentionally simple.

The goal is to understand **how this application moves from a local machine to Kubernetes**.

---

# 🏗️ Our MERN Todo Application

Our initial architecture looks like this:

```text
                Browser
                   │
                   │ HTTP
                   ▼
             React Frontend
                   │
                   │ API Requests
                   ▼
          Node.js + Express
                   │
                   │ MongoDB Driver
                   ▼
                MongoDB
```

We have three major components:

```text
React
  │
  └── Frontend

Node.js + Express
  │
  └── Backend API

MongoDB
  │
  └── Database
```

---

# 💻 Running the Application Locally

Before introducing Kubernetes, we need to understand the application in its simplest form.

A developer might run:

```bash
cd frontend
npm install
npm run dev
```

and separately:

```bash
cd backend
npm install
npm run dev
```

MongoDB may also be running locally.

The architecture is:

```text
Your Laptop
│
├── React
│
├── Node.js
│
└── MongoDB
```

Everything works.

But there are problems.

---

# ❌ The Problem Before Kubernetes

Imagine our Todo application becomes popular.

Initially:

```text
Users
  │
  ▼
Node.js
```

Everything works.

But what happens if the Node.js process crashes?

```text
Users
  │
  ▼
Node.js ❌
```

Someone has to restart it.

---

## Problem 1 — Application Crash

Without an orchestrator:

```bash
npm run dev
```

must be restarted manually.

---

## Problem 2 — Increased Traffic

Suppose 1,000 users arrive.

One backend may not be enough.

We might want:

```text
                ┌── Node.js
Users ──────────┼── Node.js
                └── Node.js
```

Now we need to manage multiple instances.

---

## Problem 3 — Server Failure

Suppose everything runs on one server:

```text
Server 💥

React     ❌
Backend   ❌
MongoDB   ❌
```

We need another machine.

---

## Problem 4 — Deployment

Suppose we release:

```text
todo-backend:v1
```

Then we release:

```text
todo-backend:v2
```

What if `v2` has a bug?

We need a way to:

* Deploy the new version
* Gradually replace the old version
* Detect failures
* Roll back

---

# 🐳 Containerizing the Application

Before Kubernetes, we containerize our application.

The architecture becomes:

```text
Docker
│
├── React Container
│
├── Node.js Container
│
└── MongoDB Container
```

Instead of saying:

```text
"Install Node.js version X."
"Install these dependencies."
"Configure this environment."
```

we package the application into an image.

```text
Application
+
Runtime
+
Dependencies
        ↓
Docker Image
        ↓
Container
```

---

# 🐳 Docker Image vs Container

A useful mental model:

```text
Dockerfile
    ↓
Docker Image
    ↓
Container
```

For example:

```text
Dockerfile
    ↓
todo-backend:1.0
    ↓
Running Node.js Container
```

Kubernetes will eventually manage these containers.

---

# ☸️ What Kubernetes Adds

Docker allows us to run containers.

Kubernetes allows us to manage those containers across a cluster.

```text
Docker
│
└── Runs containers

Kubernetes
│
├── Deploys containers
├── Restarts containers
├── Scales containers
├── Connects containers
├── Replaces failed containers
└── Manages containers across nodes
```

---

# 🎯 Declarative Infrastructure

One of the most important ideas in Kubernetes is:

> **You describe the desired state, and Kubernetes continuously works to achieve it.**

For example:

```text
I want 3 backend Pods.
```

Kubernetes tries to maintain:

```text
Desired:

3 Pods

Actual:

3 Pods
```

If one dies:

```text
Desired: 3
Actual: 2
```

Kubernetes notices the difference.

```text
2 Pods
  ↓
Controller notices
  ↓
New Pod created
  ↓
3 Pods
```

This is called a **reconciliation loop**.

---

# ☸️ Kubernetes Cluster

A Kubernetes environment is called a **Cluster**.

A cluster contains:

```text
Kubernetes Cluster
│
├── Control Plane
│
└── Worker Nodes
    ├── Worker Node
    ├── Worker Node
    └── Worker Node
```

---

# 🧠 Cluster Architecture

The high-level architecture looks like this:

```text
                    Kubernetes Cluster

                ┌──────────────────────┐
                │     Control Plane    │
                │                      │
                │  API Server          │
                │  etcd                │
                │  Scheduler           │
                │  Controllers         │
                └──────────┬───────────┘
                           │
              ┌────────────┼────────────┐
              │            │            │
              ▼            ▼            ▼
         Worker Node   Worker Node   Worker Node
              │            │            │
            Pods         Pods         Pods
```

---

# 🧠 Control Plane

The Control Plane is the management layer of Kubernetes.

It makes decisions about the cluster.

It is responsible for:

* Accepting requests
* Storing cluster state
* Scheduling Pods
* Running controllers
* Maintaining desired state

The major components are:

```text
Control Plane
│
├── API Server
├── etcd
├── kube-scheduler
└── kube-controller-manager
```

---

# 1️⃣ API Server — kube-apiserver

The API Server is the front door of Kubernetes.

When we run:

```bash
kubectl apply -f deployment.yml
```

the request goes to the API Server.

```text
kubectl
   │
   ▼
API Server
```

The API Server:

* Exposes the Kubernetes API
* Authenticates requests
* Authorizes requests
* Validates resource definitions
* Communicates with etcd

A simplified flow:

```text
Developer
    │
    │ kubectl
    ▼
API Server
    │
    ▼
etcd
```

The API Server is also the main communication point between Kubernetes components.

---

# 2️⃣ etcd

`etcd` is Kubernetes' distributed key-value database.

It stores the cluster state.

Think of it as:

```text
Kubernetes
     │
     ▼
   etcd
     │
     ├── Deployments
     ├── Pods
     ├── Services
     ├── Nodes
     ├── Configurations
     └── Cluster metadata
```

For example, if we tell Kubernetes:

```yaml
replicas: 3
```

that desired state becomes part of the cluster state.

---

# 3️⃣ kube-scheduler

The scheduler decides:

> **Which Worker Node should run this Pod?**

Suppose we have:

```text
Worker 1
Worker 2
Worker 3
```

and a new Pod needs to run.

The scheduler evaluates available nodes.

```text
New Pod
   │
   ▼
Scheduler
   │
   ├── Worker 1 ❌
   ├── Worker 2 ✅
   └── Worker 3 ❌
```

The scheduler assigns the Pod to an appropriate node.

---

# 4️⃣ kube-controller-manager

The controller manager contains controllers that continuously compare:

```text
Desired State
      vs
Actual State
```

For example:

```text
Desired = 3 Pods
Actual  = 2 Pods
```

A controller notices the difference and works to correct it.

Important controllers include:

* Node Controller
* ReplicaSet Controller
* Deployment Controller
* Endpoint / EndpointSlice Controller

This is the foundation of Kubernetes' self-healing behavior.

---

# 🖥️ Worker Nodes

Worker Nodes are the machines where our applications actually run.

A worker node can be:

* A physical server
* A virtual machine
* A cloud instance
* A local machine in a development cluster

A worker node contains:

```text
Worker Node
│
├── kubelet
├── kube-proxy
├── Container Runtime
└── Pods
```

---

# 1️⃣ kubelet

The kubelet runs on every worker node.

Its job is to make sure the containers described by Kubernetes are actually running.

```text
API Server
    │
    ▼
 kubelet
    │
    ▼
Container Runtime
    │
    ▼
Container
```

The kubelet:

* Registers the node
* Receives Pod specifications
* Starts containers
* Monitors containers
* Reports status
* Executes health probes

---

# 2️⃣ kube-proxy

`kube-proxy` handles network rules related to Kubernetes Services.

Suppose we have:

```text
Service
   │
   ├── Pod 1
   ├── Pod 2
   └── Pod 3
```

kube-proxy helps route traffic toward the appropriate Pods.

---

# 3️⃣ Container Runtime

Kubernetes does not directly execute containers.

It communicates with a container runtime through the **Container Runtime Interface (CRI)**.

Common runtimes include:

* containerd
* CRI-O

The runtime:

```text
Pull Image
    ↓
Create Container
    ↓
Start Container
    ↓
Stop Container
```

---

# 🫛 Pods

A **Pod** is the smallest deployable unit in Kubernetes.

Kubernetes does not normally deploy containers directly.

It deploys Pods.

```text
Pod
│
└── Container
```

A Pod can also contain multiple tightly coupled containers:

```text
Pod
│
├── Container A
└── Container B
```

Containers inside the same Pod share:

* Network namespace
* IP address
* Ports
* Volumes
* IPC resources

---

# 🌐 Pod Networking

A Pod gets its own IP address.

If our backend Pod gets:

```text
10.244.0.7
```

the backend can be reached using that Pod IP.

But there is a problem.

Pods are ephemeral.

If the Pod dies:

```text
10.244.0.7 ❌
```

and a replacement Pod may receive:

```text
10.244.0.15
```

Therefore, we should not build our application around Pod IP addresses.

This problem will lead us to **Services** later.

---

# 🏗️ Creating a Kubernetes Cluster

For learning locally, we can use:

# kind

`kind` means:

> Kubernetes IN Docker

It allows us to create Kubernetes clusters using Docker containers as nodes.

---

# 🖥️ Single Node Cluster

Create a cluster:

```bash
kind create cluster --name todo-cluster
```

Check the cluster:

```bash
kubectl cluster-info
```

Check nodes:

```bash
kubectl get nodes
```

You should see something similar to:

```text
NAME                     STATUS   ROLES           AGE
todo-cluster-control-plane   Ready    control-plane
```

Conceptually:

```text
Your Laptop
│
└── Docker
    │
    └── Kubernetes
        │
        └── Control Plane
```

---

# 🖥️ Multi Node Cluster

Create a file:

```yaml
# cluster.yml

kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4

nodes:
  - role: control-plane
  - role: worker
  - role: worker
```

Create the cluster:

```bash
kind create cluster --config cluster.yml --name todo-cluster
```

Check nodes:

```bash
kubectl get nodes
```

Now we have:

```text
Kubernetes Cluster
│
├── Control Plane
├── Worker
└── Worker
```

---

# 🔧 kubectl and Contexts

`kubectl` is the command-line tool used to interact with Kubernetes.

```text
You
 │
 │ kubectl
 ▼
API Server
```

Check cluster information:

```bash
kubectl cluster-info
```

List nodes:

```bash
kubectl get nodes
```

List Pods:

```bash
kubectl get pods
```

See configured clusters:

```bash
kubectl config get-clusters
```

Check current context:

```bash
kubectl config current-context
```

See all contexts:

```bash
kubectl config get-contexts
```

Switch context:

```bash
kubectl config use-context kind-todo-cluster
```

---

# 🚀 Deploying Our First Pod

Let's deploy the Node.js backend as a Pod.

Create:

```yaml
# backend-pod.yml

apiVersion: v1
kind: Pod

metadata:
  name: todo-backend

spec:
  containers:
    - name: backend
      image: todo-backend:latest
      ports:
        - containerPort: 5000
```

Apply it:

```bash
kubectl apply -f backend-pod.yml
```

Check:

```bash
kubectl get pods
```

---

# 🔍 Inspecting the Pod

Check details:

```bash
kubectl describe pod todo-backend
```

Check logs:

```bash
kubectl logs todo-backend
```

The important thing to understand is what happened behind the command:

```text
backend-pod.yml
      │
      ▼
    kubectl
      │
      ▼
 API Server
      │
      ▼
    etcd
      │
      ▼
 Scheduler
      │
      ▼
 Worker Node
      │
      ▼
   kubelet
      │
      ▼
Container Runtime
      │
      ▼
Node.js Container
```

---

# ❌ Problem with Raw Pods

Let's delete our Pod:

```bash
kubectl delete pod todo-backend
```

Check:

```bash
kubectl get pods
```

The Pod is gone.

Nothing automatically creates another one.

This gives us a problem.

A raw Pod does not provide the application-level management we need.

We want:

```text
"I want 3 backend instances running."
```

We don't want to manually create them.

This leads us to:

# ReplicaSet

---

# 🔄 ReplicaSet

A ReplicaSet ensures that a specified number of identical Pods are running.

Example:

```yaml
apiVersion: apps/v1
kind: ReplicaSet

metadata:
  name: todo-backend-rs

spec:
  replicas: 3

  selector:
    matchLabels:
      app: todo-backend

  template:
    metadata:
      labels:
        app: todo-backend

    spec:
      containers:
        - name: backend
          image: todo-backend:latest
          ports:
            - containerPort: 5000
```

Apply:

```bash
kubectl apply -f replicaset.yml
```

Check:

```bash
kubectl get rs
kubectl get pods
```

We should now have:

```text
ReplicaSet
    │
    ├── Backend Pod
    ├── Backend Pod
    └── Backend Pod
```

---

# 💥 Self-Healing

Delete one Pod:

```bash
kubectl delete pod <pod-name>
```

Immediately check:

```bash
kubectl get pods
```

You will see a new Pod being created.

Why?

```text
Desired State
3 Pods

Actual State
2 Pods

      ↓

ReplicaSet Controller
      ↓
Create replacement Pod
      ↓

Actual State
3 Pods
```

This is one of the core ideas behind Kubernetes.

---

# 🏷️ Labels and Selectors

ReplicaSets identify Pods using labels.

For example:

```yaml
labels:
  app: todo-backend
```

and:

```yaml
selector:
  matchLabels:
    app: todo-backend
```

The selector means:

> Manage Pods having `app=todo-backend`.

Conceptually:

```text
ReplicaSet
    │
    │ selector:
    │ app=todo-backend
    ▼
┌───────────────┐
│ Pod           │
│ app=todo-backend
└───────────────┘
```

---

# 🚀 Deployment

ReplicaSets solve self-healing and replication.

But we also need:

* Rolling updates
* Rollbacks
* Version management

This is where **Deployment** comes in.

The relationship is:

```text
Deployment
     │
     ▼
ReplicaSet
     │
     ▼
Pods
     │
     ▼
Containers
```

---

# 📦 Deployment Resource

Example:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: todo-backend

spec:
  replicas: 3

  selector:
    matchLabels:
      app: todo-backend

  template:
    metadata:
      labels:
        app: todo-backend

    spec:
      containers:
        - name: backend
          image: todo-backend:v1
          ports:
            - containerPort: 5000
```

Apply:

```bash
kubectl apply -f deployment.yml
```

Check:

```bash
kubectl get deployment
kubectl get rs
kubectl get pods
```

---

# 🔗 Deployment Relationship

The hierarchy is:

```text
Deployment
     │
     ▼
ReplicaSet
     │
     ├── Pod
     ├── Pod
     └── Pod
```

The Deployment manages the ReplicaSet.

The ReplicaSet manages the Pods.

The Pods run the containers.

---

# 🔄 Rollouts and Rollbacks

Suppose we currently have:

```text
todo-backend:v1
```

Our Deployment:

```yaml
image: todo-backend:v1
```

Now we release:

```text
todo-backend:v2
```

Update the Deployment:

```yaml
image: todo-backend:v2
```

Apply:

```bash
kubectl apply -f deployment.yml
```

Kubernetes creates a new ReplicaSet.

```text
Deployment
│
├── Old ReplicaSet
│      ├── v1 Pod
│      ├── v1 Pod
│      └── v1 Pod
│
└── New ReplicaSet
       ├── v2 Pod
       ├── v2 Pod
       └── v2 Pod
```

The new version is rolled out gradually.

---

# 📊 Checking a Rollout

```bash
kubectl rollout status deployment todo-backend
```

Check history:

```bash
kubectl rollout history deployment todo-backend
```

---

# ⏪ Rollback

Suppose `v2` contains a bug.

Rollback:

```bash
kubectl rollout undo deployment todo-backend
```

Check:

```bash
kubectl rollout status deployment todo-backend
```

This is one of the major reasons we use Deployments instead of manually managing Pods.

---

# 🔄 Internal Flow of a Deployment

When we run:

```bash
kubectl apply -f deployment.yml
```

the flow is approximately:

```text
1. kubectl
      ↓
2. API Server
      ↓
3. etcd
      ↓
4. Deployment Controller
      ↓
5. ReplicaSet
      ↓
6. Pods
      ↓
7. Scheduler
      ↓
8. Worker Node
      ↓
9. kubelet
      ↓
10. Container Runtime
      ↓
11. Node.js Container
```

This flow is the most important mental model to understand Kubernetes.

---

# ⚛️ Deploying the React Frontend

Our backend is running.

Now deploy the React frontend.

Create a Deployment:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: todo-frontend

spec:
  replicas: 2

  selector:
    matchLabels:
      app: todo-frontend

  template:
    metadata:
      labels:
        app: todo-frontend

    spec:
      containers:
        - name: frontend
          image: todo-frontend:latest
          ports:
            - containerPort: 80
```

Apply:

```bash
kubectl apply -f frontend-deployment.yml
```

Check:

```bash
kubectl get deployments
kubectl get pods
```

Now:

```text
Kubernetes
│
├── React Deployment
│      ├── React Pod
│      └── React Pod
│
└── Backend Deployment
       ├── Backend Pod
       ├── Backend Pod
       └── Backend Pod
```

---

# 🌐 Services

We now have a problem.

Pods have changing IP addresses.

Suppose:

```text
Backend Pod 1
10.244.0.7
```

The Pod crashes.

A new Pod appears:

```text
Backend Pod 2
10.244.0.19
```

If the frontend directly uses:

```text
10.244.0.7
```

it will break.

We need a stable endpoint.

This is what a Kubernetes **Service** provides.

---

# 🔌 What is a Service?

A Service provides a stable network endpoint for a group of Pods.

```text
              Service
                 │
        ┌────────┼────────┐
        ▼        ▼        ▼
      Pod 1    Pod 2    Pod 3
```

The Service uses labels to identify its Pods.

For example:

```yaml
selector:
  app: todo-backend
```

This means:

> Send traffic to Pods with `app=todo-backend`.

---

# 🟢 ClusterIP

ClusterIP is the default Service type.

It is primarily used for internal cluster communication.

For our application:

```text
React
  │
  ▼
Backend Service
  │
  ├── Backend Pod
  ├── Backend Pod
  └── Backend Pod
```

Example:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: todo-backend-service

spec:
  selector:
    app: todo-backend

  ports:
    - protocol: TCP
      port: 5000
      targetPort: 5000

  type: ClusterIP
```

Apply:

```bash
kubectl apply -f backend-service.yml
```

Check:

```bash
kubectl get services
```

---

# 🔵 NodePort

NodePort exposes a Service through a port on each node.

Example:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: todo-backend-nodeport

spec:
  selector:
    app: todo-backend

  ports:
    - protocol: TCP
      port: 5000
      targetPort: 5000
      nodePort: 30007

  type: NodePort
```

Traffic:

```text
<NodeIP>:30007
       │
       ▼
   Service
       │
       ▼
 Backend Pods
```

NodePort is useful for learning and simple scenarios, but it is generally not the preferred production-facing interface.

---

# 🟣 LoadBalancer

A LoadBalancer Service is commonly used with cloud providers.

Example:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: todo-frontend-service

spec:
  selector:
    app: todo-frontend

  ports:
    - port: 80
      targetPort: 80

  type: LoadBalancer
```

In a managed cloud environment, Kubernetes can request an external load balancer from the cloud provider.

Conceptually:

```text
Internet
   │
   ▼
Cloud Load Balancer
   │
   ▼
Kubernetes Service
   │
   ▼
Frontend Pods
```

---

# 🔗 Frontend to Backend Communication

Now we have:

```text
React
  │
  ▼
Backend Service
  │
  ├── Backend Pod
  ├── Backend Pod
  └── Backend Pod
```

The frontend should not depend on a Pod IP.

Instead, it communicates through the Service.

For example:

```text
http://todo-backend-service:5000
```

Kubernetes provides internal DNS so Services can be discovered by name.

This means:

```text
React
  │
  │ HTTP
  ▼
todo-backend-service
  │
  ├── Pod 1
  ├── Pod 2
  └── Pod 3
```

If Pod 1 dies:

```text
Pod 1 ❌
Pod 2 ✅
Pod 3 ✅
Pod 4 ✅
```

The Service continues routing traffic to available Pods.

---

# 🍃 Deploying MongoDB

Now let's introduce our database.

For learning purposes, MongoDB can also be deployed inside Kubernetes.

Conceptually:

```text
Frontend
   │
   ▼
Backend
   │
   ▼
MongoDB
```

We can create a MongoDB Deployment:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: mongodb

spec:
  replicas: 1

  selector:
    matchLabels:
      app: mongodb

  template:
    metadata:
      labels:
        app: mongodb

    spec:
      containers:
        - name: mongodb
          image: mongo:latest
          ports:
            - containerPort: 27017
```

Create a Service:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: mongodb

spec:
  selector:
    app: mongodb

  ports:
    - port: 27017
      targetPort: 27017
```

Now our backend can connect to:

```text
mongodb:27017
```

rather than a Pod IP.

---

# ⚠️ Important Database Note

Running MongoDB inside Kubernetes is useful for learning Kubernetes concepts.

In production, databases are often operated differently.

For example, teams may use managed database services.

The important Kubernetes concept is:

```text
Application
    ↓
Service
    ↓
Database
```

and, when the database is managed inside the cluster:

```text
Database Pod
    ↓
Persistent Storage
```

---

# ⚙️ ConfigMaps

Applications often need configuration.

For example:

```text
PORT=5000
NODE_ENV=production
```

A ConfigMap can store non-sensitive configuration.

Example:

```yaml
apiVersion: v1
kind: ConfigMap

metadata:
  name: backend-config

data:
  NODE_ENV: production
  PORT: "5000"
```

The application can consume these values as environment variables.

Conceptually:

```text
ConfigMap
    │
    ▼
Backend Pod
    │
    ▼
Environment Variables
```

---

# 🔐 Secrets

Sensitive information should not be placed directly into application images.

Examples:

```text
MongoDB password
API keys
Credentials
Tokens
```

Kubernetes provides Secrets for storing sensitive configuration.

Example:

```yaml
apiVersion: v1
kind: Secret

metadata:
  name: backend-secret

type: Opaque

stringData:
  MONGO_USERNAME: admin
  MONGO_PASSWORD: password
```

The application can then consume these values.

> Kubernetes Secrets provide a mechanism for handling sensitive configuration, but production deployments should also consider encryption at rest, access control, secret rotation, and external secret-management systems.

---

# 💾 Persistent Storage

Pods are ephemeral.

Suppose MongoDB is running inside a Pod:

```text
MongoDB Pod
    │
    └── Database Data
```

If the Pod disappears, we don't want the database data to disappear with it.

Therefore, we need persistent storage.

The basic Kubernetes concepts are:

```text
PersistentVolume
        │
        ▼
PersistentVolumeClaim
        │
        ▼
Pod
```

The application uses a PVC to request persistent storage.

Conceptually:

```text
MongoDB
   │
   ▼
PVC
   │
   ▼
Persistent Storage
```

---

# ❤️ Health Checks

Kubernetes can check whether our application is healthy.

There are three important types of probes.

---

## Liveness Probe

Question:

> Is the application alive?

If the container repeatedly fails its liveness check, Kubernetes can restart it.

---

## Readiness Probe

Question:

> Is the application ready to receive traffic?

If a Pod is not ready, the Service should avoid sending normal traffic to it.

```text
Service
   │
   ├── Ready Pod       ← traffic
   ├── Ready Pod       ← traffic
   └── Not Ready Pod   ← no traffic
```

---

## Startup Probe

Useful for applications that take significant time to start.

It gives the application time to initialize before normal liveness checking becomes important.

---

# 📈 Scaling

Suppose our backend initially has:

```yaml
replicas: 1
```

We can scale it:

```bash
kubectl scale deployment todo-backend --replicas=3
```

Now:

```text
Backend Service
       │
 ┌─────┼─────┐
 ▼     ▼     ▼
Pod 1 Pod 2 Pod 3
```

Check:

```bash
kubectl get pods
```

---

# 📉 Scaling Down

We can reduce replicas:

```bash
kubectl scale deployment todo-backend --replicas=1
```

Now Kubernetes removes unnecessary Pods.

```text
3 Pods
  ↓
1 Pod
```

The desired state changed.

Kubernetes reconciles the actual state.

---

# 🤖 Horizontal Pod Autoscaling

Instead of manually changing replicas, Kubernetes can automatically scale Pods based on metrics.

Conceptually:

```text
Low Traffic
    ↓
2 Pods

High Traffic
    ↓
5 Pods

Very High Traffic
    ↓
10 Pods
```

This is handled using the **Horizontal Pod Autoscaler (HPA)**.

---

# 🖥️ Multi-Node Application Architecture

Now let's move beyond a single machine.

Our cluster:

```text
                    Control Plane
                         │
             ┌───────────┴───────────┐
             │                       │
             ▼                       ▼
        Worker Node 1           Worker Node 2
             │                       │
        ┌────┴────┐              ┌───┴────┐
        ▼         ▼              ▼        ▼
      React      API            API      Mongo
```

The scheduler decides where Pods should run.

For example:

```text
New Backend Pod
       │
       ▼
Scheduler
       │
       ├── Worker 1
       └── Worker 2
```

It considers things such as:

* Available CPU
* Available memory
* Scheduling constraints
* Taints and tolerations
* Affinity rules
* Resource requirements

---

# 💥 What Happens When a Node Dies?

Suppose:

```text
Worker Node 1 💥
```

Pods running there may disappear.

But the desired state still says:

```text
Backend replicas = 3
```

Kubernetes notices:

```text
Desired = 3
Actual = 1
```

It can schedule replacement Pods on another available node.

```text
Worker Node 2
│
├── Backend Pod
├── Backend Pod
└── Backend Pod
```

This is the combination of:

* Controllers
* Scheduler
* kubelet
* ReplicaSets / Deployments

working together.

---

# 🐞 Debugging Kubernetes

Kubernetes applications can fail for many reasons.

Here are the commands you should know.

---

## Get Pods

```bash
kubectl get pods
```

---

## Get Pods with Node Information

```bash
kubectl get pods -o wide
```

---

## Describe a Pod

```bash
kubectl describe pod <pod-name>
```

This is extremely useful when troubleshooting scheduling or startup issues.

---

## View Logs

```bash
kubectl logs <pod-name>
```

---

## Follow Logs

```bash
kubectl logs -f <pod-name>
```

---

## Get Deployments

```bash
kubectl get deployments
```

---

## Get ReplicaSets

```bash
kubectl get rs
```

---

## Get Services

```bash
kubectl get services
```

or:

```bash
kubectl get svc
```

---

## Get Events

```bash
kubectl get events
```

---

# ❗ Common Kubernetes Problems

## ImagePullBackOff

Usually means Kubernetes is having trouble pulling the container image.

Possible causes:

```text
Wrong image name
Wrong image tag
Private registry
Authentication issue
Image unavailable
```

---

# ❗ CrashLoopBackOff

The container starts and repeatedly crashes.

Check:

```bash
kubectl logs <pod-name>
```

and:

```bash
kubectl describe pod <pod-name>
```

Typical causes:

```text
Application error
Missing environment variable
Database unavailable
Incorrect configuration
Port/configuration issue
```

---

# ❗ Pending Pod

If a Pod remains:

```text
Pending
```

the scheduler may not have been able to place it.

Check:

```bash
kubectl describe pod <pod-name>
```

Possible reasons include:

```text
Insufficient CPU
Insufficient memory
Scheduling constraints
Taints
Node availability
```

---

# ❗ Service Not Reaching Pods

If the Service cannot reach the backend, check:

```text
Service
   │
   ▼
Selector
   │
   ▼
Pod Labels
```

For example, Service:

```yaml
selector:
  app: todo-backend
```

must match:

```yaml
labels:
  app: todo-backend
```

You can inspect:

```bash
kubectl get svc
kubectl get endpoints
kubectl get pods --show-labels
```

---

# 🏗️ Complete Todo Application Architecture

At this point, our application looks like:

```text
                         Internet
                            │
                            ▼
                   Frontend Service
                            │
                            ▼
                  React Deployment
                       │       │
                       ▼       ▼
                     Pod     Pod
                            │
                            │ HTTP
                            ▼
                  Backend Service
                            │
                 ┌──────────┼──────────┐
                 ▼          ▼          ▼
              API Pod    API Pod    API Pod
                            │
                            │ MongoDB
                            ▼
                   MongoDB Service
                            │
                            ▼
                      MongoDB Pod
                            │
                            ▼
                    Persistent Storage
```

---

# 🧩 Kubernetes Object Relationships

The most important hierarchy to remember is:

```text
Deployment
    │
    ▼
ReplicaSet
    │
    ▼
Pod
    │
    ▼
Container
```

For networking:

```text
Service
    │
    ▼
Selector
    │
    ▼
Pods
```

For configuration:

```text
ConfigMap ────────┐
                  │
                  ▼
              Application
                  ▲
                  │
Secret ───────────┘
```

For storage:

```text
Application
    │
    ▼
PVC
    │
    ▼
Persistent Storage
```

---

# 🔄 Complete Request Flow

Suppose a user opens our Todo application.

The request might look like:

```text
User
 │
 ▼
Load Balancer
 │
 ▼
Frontend Service
 │
 ▼
React Pod
 │
 │ API Request
 ▼
Backend Service
 │
 ▼
Backend Pod
 │
 │ Database Query
 ▼
MongoDB Service
 │
 ▼
MongoDB
 │
 ▼
Persistent Storage
```

This is what a complete Kubernetes application starts to look like.

---

# 🧠 Production Kubernetes Concepts

Once you understand the fundamentals above, there are several concepts worth learning next.

---

## Namespaces

Namespaces logically separate resources.

```text
Cluster
│
├── development
│
├── staging
│
└── production
```

---

## Ingress

Ingress provides HTTP/HTTPS routing into a Kubernetes cluster.

Instead of exposing every application separately:

```text
Internet
  │
  ▼
Ingress
  │
  ├── /api     → Backend
  │
  └── /        → Frontend
```

---

## Resource Requests and Limits

Containers can specify:

```text
CPU
Memory
```

Requests help the scheduler understand the resources required.

Limits restrict how much a container can consume.

---

## Horizontal Pod Autoscaler

Automatically changes the number of Pods based on metrics.

```text
Traffic ↑
   ↓
Pods ↑
```

---

## Rolling Updates

Deploy new versions gradually.

```text
v1 v1 v1
 ↓
v1 v1 v2
 ↓
v1 v2 v2
 ↓
v2 v2 v2
```

---

## Rollbacks

If the new version is problematic:

```text
v2 ❌
 ↓
Rollback
 ↓
v1 ✅
```

---

## RBAC

Role-Based Access Control determines:

> Who can perform which actions on which Kubernetes resources?

For example:

```text
Developer
   ↓
Can read Pods

Admin
   ↓
Can create Deployments

Restricted User
   ↓
Limited permissions
```

---

## Network Policies

Network Policies can restrict which Pods are allowed to communicate.

For example:

```text
Frontend ─────→ Backend
                 │
                 ▼
              MongoDB
```

while preventing unwanted communication between unrelated applications.

---

# ☁️ From Local Kubernetes to Cloud

Everything we learned using `kind` applies conceptually to managed Kubernetes.

Common managed Kubernetes services include:

```text
AWS
 └── EKS

Google Cloud
 └── GKE

Microsoft Azure
 └── AKS
```

The infrastructure changes, but the Kubernetes concepts remain:

```text
Pods
Deployments
Services
ConfigMaps
Secrets
Persistent Volumes
Namespaces
Ingress
RBAC
```

---

# 💻 Local vs Cloud Kubernetes

Local:

```text
Laptop
 │
 └── kind
      │
      └── Kubernetes
```

Cloud:

```text
Cloud Provider
 │
 └── Managed Kubernetes
      │
      ├── Control Plane
      └── Worker Nodes
```

The application manifests can remain conceptually similar.

---

# 🛠️ Useful kubectl Commands

## Cluster

```bash
kubectl cluster-info
kubectl get nodes
kubectl describe node <node>
```

## Pods

```bash
kubectl get pods
kubectl get pods -o wide
kubectl describe pod <pod>
kubectl logs <pod>
kubectl logs -f <pod>
kubectl delete pod <pod>
```

## Deployments

```bash
kubectl get deployments
kubectl describe deployment <deployment>
kubectl rollout status deployment <deployment>
kubectl rollout history deployment <deployment>
kubectl rollout undo deployment <deployment>
```

## ReplicaSets

```bash
kubectl get rs
kubectl describe rs <replicaset>
```

## Services

```bash
kubectl get svc
kubectl describe svc <service>
kubectl get endpoints
```

## Configuration

```bash
kubectl get configmaps
kubectl get secrets
```

## Resources

```bash
kubectl get all
```

## Apply Configuration

```bash
kubectl apply -f file.yml
```

## Delete Configuration

```bash
kubectl delete -f file.yml
```

---

# 🧹 Cleaning Up the Local Cluster

Delete the kind cluster:

```bash
kind delete cluster --name todo-cluster
```

---

# 🧠 Final Mental Model

If you remember only one thing from this guide, remember this:

```text
                     Kubernetes Cluster
                            │
                            ▼
                    ┌──────────────┐
                    │ Control Plane│
                    └───────┬──────┘
                            │
              ┌─────────────┼─────────────┐
              ▼             ▼             ▼
           Worker         Worker        Worker
            Node           Node          Node
              │             │             │
             Pods          Pods          Pods
              │
              ▼
         Containers
```

And remember the resource hierarchy:

```text
Deployment
    ↓
ReplicaSet
    ↓
Pod
    ↓
Container
```

Networking:

```text
Service
    ↓
Pod Selector
    ↓
Pods
```

Configuration:

```text
ConfigMap / Secret
        ↓
      Pod
```

Storage:

```text
Pod
 ↓
PVC
 ↓
Persistent Storage
```

---

# 🎯 The Complete Kubernetes Story

We started with:

```text
MERN Todo App
```

running locally:

```text
React
 ↓
Node.js
 ↓
MongoDB
```

Then we containerized it:

```text
React Container
Node Container
MongoDB Container
```

Then Kubernetes entered:

```text
Kubernetes Cluster
```

We learned that:

```text
Pod
```

is the basic execution unit.

Then:

```text
ReplicaSet
```

gave us replication and self-healing.

Then:

```text
Deployment
```

gave us controlled updates and rollbacks.

Then:

```text
Service
```

gave our Pods stable networking.

Then:

```text
ConfigMaps + Secrets
```

gave us configuration management.

Then:

```text
Persistent Volumes
```

gave applications persistent storage.

Then:

```text
Health Checks
```

allowed Kubernetes to understand application health.

Then:

```text
Scaling
```

allowed us to run multiple application instances.

Finally:

```text
Multi-Node Cluster
```

allowed our application to run across multiple machines.

The entire journey can be summarized as:

```text
                 MERN Todo Application
                          │
                          ▼
                     Docker
                          │
                          ▼
                  Kubernetes Cluster
                          │
          ┌───────────────┼────────────────┐
          │               │                │
          ▼               ▼                ▼
      Frontend         Backend          MongoDB
      Deployment       Deployment       Deployment
          │               │                │
          ▼               ▼                ▼
        Pods            Pods             Pod
                          │                │
                          │                ▼
                          │          Persistent Storage
                          │
                          ▼
                     Backend Service
                          ▲
                          │
                          │
                  Frontend Service
                          ▲
                          │
                       Users
```

---

# 🚀 What You Should Understand After This Guide

After completing this tutorial, you should be able to explain:

* What Kubernetes is
* Why Kubernetes is needed
* What a Kubernetes cluster is
* What the Control Plane does
* What Worker Nodes do
* What the API Server does
* What etcd stores
* What the scheduler does
* What controllers do
* What kubelet does
* What kube-proxy does
* What a Pod is
* Why Pods are ephemeral
* Why ReplicaSets exist
* Why Deployments exist
* How rolling updates work
* How rollbacks work
* Why Services exist
* Difference between ClusterIP, NodePort and LoadBalancer
* How Pods communicate through Services
* How Kubernetes DNS works
* How ConfigMaps work
* How Secrets work
* Why persistent storage is needed
* How health probes work
* How Kubernetes scales applications
* How Pods are distributed across nodes
* How Kubernetes responds to failures
* How to debug common Kubernetes problems
* How local Kubernetes relates to managed cloud Kubernetes

---

# ⭐ The Core Idea

Kubernetes can initially look like a huge collection of YAML files and commands.

But the underlying idea is simple:

```text
You describe what you want.
              ↓
       Kubernetes observes
              ↓
       Kubernetes compares
              ↓
     Desired State vs Actual State
              ↓
       Kubernetes reconciles
              ↓
        System moves toward
        the desired state
```

For our Todo application:

```text
"I want 3 backend Pods."
```

Kubernetes continuously works toward:

```text
3 healthy backend Pods
```

If one crashes:

```text
3 → 2
```

Kubernetes notices.

```text
2 → 3
```

If we deploy a new version:

```text
v1 → v2
```

Deployment manages the transition.

If Pods change IPs:

```text
Service
```

provides a stable endpoint.

If a node fails:

```text
Scheduler + Controllers
```

can work together to restore the desired state elsewhere.

That is the core idea behind Kubernetes:

> **Declare the state you want, and Kubernetes continuously works to keep the cluster in that state.**

---

# ☸️ Kubernetes Learning Path

```text
Docker
  ↓
Kubernetes Basics
  ↓
Cluster Architecture
  ↓
Pods
  ↓
ReplicaSets
  ↓
Deployments
  ↓
Services
  ↓
ConfigMaps
  ↓
Secrets
  ↓
Persistent Storage
  ↓
Health Checks
  ↓
Scaling
  ↓
Ingress
  ↓
RBAC
  ↓
Network Policies
  ↓
Helm
  ↓
Monitoring
  ↓
Cloud Kubernetes
```

**The best way to learn Kubernetes is not to memorize every resource.**

Build something.

Break it.

Deploy it.

Scale it.

Delete a Pod.

Kill a Node.

Change the image.

Rollback the deployment.

Watch what Kubernetes does.

That's when Kubernetes starts making sense.
