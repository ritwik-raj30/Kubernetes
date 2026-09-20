# Kubernetes, from a practical point of view

This repo is a learning note: how Kubernetes is used day to day, not a deep internals manual.

If you remember only one sentence:

> You tell Kubernetes the **end state** you want (for example: 3 copies of this app). Kubernetes keeps trying to make reality match that.

---

## What problem it solves

You have an app packed as a **container**. Kubernetes is the **manager** that:

- runs several copies of it
- restarts a copy if it dies
- spreads copies across machines
- gives them a **stable name** even when copies are replaced
- rolls out a new version without taking everything down

---

## Docker vs Kubernetes

**Docker** packs your app (code, runtime, libraries) into a **container** so it runs the same on your laptop and on a server.

| Word | Everyday meaning |
| --- | --- |
| **Image** | The snapshot / recipe. Like a USB installer. It does not run by itself. |
| **Container** | A **running** copy of that image. |
| **Dockerfile** | A short file that says how to build the image. |

Kubernetes does **not** replace that idea. It **manages many containers** on **many machines**.

You need **containers**. You do **not** always need **Docker Engine** specifically. On a laptop, Docker is the easy way to **build** an image. On a real cluster, nodes often use another runner (containerd, CRI-O). Same job: start containers.

Kubernetes is **not** “Docker with extra buttons” and **not** a plugin sitting on top of Docker.

```
Your app
   ↓ packed as a container image
Container runtime (Docker / containerd / …)  ← starts the process
   ↑
Kubernetes  ← boss: “run 3 of these, on these machines, keep them alive”
```

**Kitchen picture:** Docker is packing lunch in a box. Kubernetes is restaurant staff: many boxes, many cooks, if a plate drops they make another.

---

## What Kubernetes is (as a whole)

It is **a set of programs that work together**, not one app like Chrome.

You install it on **several computers** (or one laptop pretending to be several). Together they are a **cluster**.

| Place | What it feels like |
| --- | --- |
| **Laptop** (minikube, kind, Docker Desktop) | One installer. Tiny cluster. Good for learning. |
| **Cloud** (EKS, GKE, AKS) | AWS / Google / Azure run the cluster. You use it. |
| **Your own servers** | You or the platform team install Kubernetes on those machines. |

**Two kinds of machines**

- **Control plane (the brain):** remembers what you asked for, decides where work goes, notices failures, sends orders.
- **Worker nodes (the muscle):** actually run your app containers.

You talk to the cluster with **`kubectl`**. That is only the remote control. Kubernetes itself is the cluster.

---

## Two perspectives

### 1. How Kubernetes itself works

Think of a restaurant kitchen. These programs stay running.

| Component | Role |
| --- | --- |
| **API server** | Front desk. Everything talks to this: you, dashboards, other Kubernetes pieces. |
| **etcd** | The notebook. Stores what you asked for and what is running. Beginners almost never touch this. |
| **Scheduler** | Assigns new work to a worker that has space. |
| **Controller manager** | Anxious manager. Compares **desired** (“3 copies”) vs **actual** (“2 running”) and fixes the gap. |
| **kubelet** | The cook on each worker. Starts what it is told and reports if something died. |
| **Container runtime** | The stove. kubelet does not magically run your app; the runtime starts the process. |
| **kube-proxy / networking** | Waiters and plumbing. Other apps can reach yours through a stable name even when pods move. |

**The loop that *is* Kubernetes**

1. Someone writes **desired state** (3 copies of app X).
2. Front desk saves it in the notebook.
3. Scheduler places work on nodes.
4. kubelet + runtime start containers.
5. Controllers keep watching. If reality ≠ desire, they fix it.

Kubernetes is not a one-shot “start my app” button. It is a **continuous babysitter**.

```
You / kubectl
      ↓
   API server (front desk)
      ↓
   etcd (notebook)
      ↓
 Scheduler + Controllers (brain)
      ↓
   kubelet on workers (cooks)
      ↓
   container runtime (stove)
      ↓
   your app running
```

### 2. What you need to know as a user

You almost never install etcd or talk to kubelet. You live in a smaller world.

**Tools you actually use**

- **`kubectl`** — remote control. Talks only to the API server.
- **YAML files (manifests)** — order tickets. Not a script of steps. You describe the **end state**. Kubernetes figures out the steps.
- **kubeconfig / context** — which cluster you are talking to (laptop vs cloud). Wrong context means you deploy to the wrong kitchen. Check this before you apply anything important.

**Objects you should learn first**

| You say | What it means |
| --- | --- |
| **Pod** | One running instance of your app (usually one container). Short-lived. Can die and be replaced. Do not treat it as a pet. |
| **Deployment** | “Keep N healthy copies, and when I change the image, roll it out.” This is what you create most of the time. |
| **Service** | A stable door so others can find pods even when pod IPs change. |
| **Namespace** | A folder / team fence: `dev`, `prod`. |
| **ConfigMap / Secret** | Settings and passwords injected into the app, not baked into the image. |
| **Ingress** (a bit later) | How the internet or company network reaches the Service. The front door. |

**You create Deployments. Kubernetes creates Pods for you.**  
That is the most useful sentence in beginner Kubernetes.

**A normal day**

1. Build an image (often with Docker).
2. Put it in a registry so the cluster can pull it.
3. Write a Deployment + Service.
4. `kubectl apply -f ...`
5. `kubectl get pods` / `kubectl logs` / `kubectl describe` when something is wrong.
6. Change the image tag, apply again, watch a rollout.

You **declare desire**, then **observe**, then **fix the declaration** if the cluster cannot match it (bad image name, not enough memory, typo in YAML).

---

## How the objects actually nest

This is the part people mix up.

**Nested (parent owns child):**

```
Deployment          ← you usually create this
  └── ReplicaSet    ← Kubernetes creates this for you
        └── Pod(s)  ← Kubernetes creates these
              └── Container(s)  ← your actual app process
```

- **Deployment:** “Always have 3 plates of this dish, and when the recipe changes, replace them safely.”
- **ReplicaSet:** “Right now, keep exactly 3 pods of *this version* alive.”
- **Pod:** one plate.
- **Container:** the food on the plate.

You almost never create a ReplicaSet by hand. When you roll out a new version, you briefly get **two** ReplicaSets: old pods shrinking, new pods growing.

**Service is not inside that stack.** It stands **beside** the pods and **points at them** with labels (for example `app=my-web`).

```
Service  ──────────►  Pod  Pod  Pod
  (stable name)         ▲    ▲    ▲
                        └── selected by labels
```

A Service does **not** contain a Deployment. A Deployment does **not** contain a Service. You usually create **both**.

The waiter station is not inside the pasta.

---

## How the two views meet (one example)

You: “Deployment, 3 replicas of `my-web`,” plus a Service.

Behind the curtain:

1. API server stores that.
2. Deployment controller makes a ReplicaSet / 3 pods.
3. Scheduler puts pods on workers.
4. kubelet pulls the image and starts containers.
5. The Service sends traffic to those pods.
6. You delete a pod. The controller makes a new one. You still hit the same Service.

**You** cared about: Deployment, Service, `kubectl`.  
**Kubernetes** cared about: API, store, schedule, kubelet, runtime, networking.

---

## What to ignore at first

- How etcd is clustered
- How CNI plugins work
- Operators and CRDs
- Helm internals
- Installing Kubernetes from scratch on several VMs

That is admin / SRE later. First job: **read cluster state, ship a Deployment, debug a CrashLoop.**

---

## What to practice next

1. Run a tiny cluster locally (minikube, kind, or Docker Desktop Kubernetes).
2. Deploy a simple web app (even nginx).
3. Use `kubectl get pods` until that feels normal.
4. Scale to 3 copies.
5. Delete a pod and watch it come back.
6. Expose it with a Service and open it in a browser.

That loop is most of junior / mid practical Kubernetes.

---

## Tiny example (optional)

You do not need this YAML memorized. It is here so the words above look like something you would actually apply.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-web
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-web
  template:
    metadata:
      labels:
        app: my-web
    spec:
      containers:
        - name: web
          image: nginx:1.27
          ports:
            - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: my-web
spec:
  selector:
    app: my-web
  ports:
    - port: 80
      targetPort: 80
```

Apply with `kubectl apply -f` after you have a cluster. Then `kubectl get deploy,rs,pods,svc`.
