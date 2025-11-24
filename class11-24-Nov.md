# Kubernetes (K8s) – Conceptual Overview & Architecture

## 1. What is Kubernetes?

Kubernetes is a **Container Orchestration Platform**.
While Docker creates and runs containers, Kubernetes acts as the "Traffic Controller" or "Manager" that handles those containers at massive scale across a fleet of machines.

**Its Job:** To automate the deployment, scaling, and management of containerized applications.

---

## 2. Core Responsibilities (The "Why")

Kubernetes operates on a **Declarative Model**: You define the *Desired State*, and Kubernetes works tirelessly to maintain it.

### A. Scheduling & Optimization

* **Unit of Work:** K8s does not deploy containers directly; it deploys **Pods**.
* **Intelligence:** The Scheduler analyzes resources (CPU/RAM) and decides exactly which node is best suited to run a specific Pod.

### B. Self-Healing (Fault Tolerance)

* **Health Checks:** Continuously monitors if pods are alive.
* **Action:** If a Pod crashes or a Node dies, K8s automatically restarts the Pod or reschedules it on a healthy node.
* **Goal:** `Current State` must always equal `Desired State`.

### C. Scaling Strategies

| Type | Description |
| --- | --- |
| **HPA (Horizontal Pod Autoscaler)** | Adds **more pods** based on traffic (CPU/Memory). |
| **VPA (Vertical Pod Autoscaler)** | Increases the **size (CPU/RAM)** of existing pods. |
| **Cluster Autoscaler** | Adds actual **Worker Nodes (VMs)** to the cluster when resources are low. |

### D. Networking & Discovery

* **Services:** An abstraction that gives Pods a stable IP and DNS name.
* **Load Balancing:** Distributes traffic evenly across all healthy Pods behind a Service.

### E. Secret Management

* Securely stores API Keys, Passwords, and Certificates.
* Injects them into Pods only when needed (via Environment Variables or Files), avoiding hard-coded credentials.

---

## 3. Kubernetes Cluster Architecture

A cluster is split into two distinct roles: The **Brain** (Control Plane) and the **Muscle** (Worker Nodes).

### Part 1: The Control Plane (The Brain)

Manages the cluster, makes decisions, and stores configuration.

| Component | Role | Description |
| --- | --- | --- |
| **API Server** | **The Gatekeeper** | The only entry point. Validates requests, handles auth, and updates `etcd`. Everything talks to this. |
| **etcd** | **The Memory** | A highly available key-value store. It is the "Single Source of Truth" for the cluster state. |
| **Scheduler** | **The Decision Maker** | Watches for new Pods with no assigned node and selects the best node for them to run on. |
| **Controller Manager** | **The Enforcer** | Runs background loops (Controllers) to ensure the current state matches the desired state (e.g., "Are there 3 replicas running?"). |

### Part 2: The Worker Node (The Muscle)

The machines that actually run your applications.

| Component | Role | Description |
| --- | --- | --- |
| **Kubelet** | **The Agent** | The "Captain" of the node. It talks to the API Server and ensures the containers described in the PodSpecs are running and healthy. |
| **Kube-Proxy** | **The Networker** | Maintains network rules (iptables/IPVS) on the node to allow network communication to your Pods. |
| **Container Runtime** | **The Engine** | The software that actually runs the container (e.g., `containerd`, `CRI-O`, or Docker Engine). |

---

## 4. The Lifecycle of a Pod Creation

What happens when you run `kubectl run nginx`?

1. **Request:** User sends command to **API Server**.
2. **Validation:** API Server authenticates user and validates request.
3. **Storage:** API Server writes the new Pod state to **etcd**.
4. **Scheduling:** The **Scheduler** notices a new "Unassigned Pod," picks a suitable Node, and tells the API Server.
5. **Assignment:** API Server updates the Pod record in **etcd** with the Node name.
6. **Execution:** The **Kubelet** on that specific Node sees the assignment.
7. **Runtime:** Kubelet tells the **Container Runtime** (e.g., Docker) to start the container.
8. **Status:** Kubelet reports "Running" status back to the API Server.

---

## 5. Key Concept: The Pod

* **Definition:** The smallest deployable unit in Kubernetes.
* **Composition:** Usually contains 1 container, but can hold multiple "helper" containers (sidecars).
* **Sharing:** All containers in a single Pod share:
* IP Address (Network Namespace).
* Storage Volumes.
* Localhost communication.


* **Ephemeral:** Pods are mortal. They are born, they die, and they are not resurrected—they are **replaced** by new identical Pods.

---

## 6. The Declarative Model

You don't tell Kubernetes *how* to do things; you tell it *what* you want.

**Imperative (Old way):** "Start server A. Then start server B. If A crashes, restart it."
**Declarative (K8s way):** "I want 3 replicas of Nginx running at all times."

* If a node crashes and you lose 1 replica, the **Controller Manager** notices (Current: 2, Desired: 3) and creates a new one automatically to return to the desired state.

---
