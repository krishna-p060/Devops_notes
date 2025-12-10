# Class 15 — Kubernetes Services: Structured Notes

## PART 1: Conceptual Foundation

### 1. What is a Service?

A **Kubernetes Service** is an abstraction layer that defines a logical set of Pods and a policy by which to access them.

* **The Problem:** Pods are ephemeral. They die, get rescheduled, and change IP addresses constantly.
* **The Solution:** A Service provides a **Stable IP Address** and **DNS Name** that never changes, regardless of what happens to the Pods behind it.

### 2. The Core Functions

A Service acts as a bridge between a client (like a Frontend App) and a backend (like a Database App), providing:

1. **Service Discovery:** "Where is the database?"  "It's at `db-service`."
2. **Load Balancing:** Spreads traffic evenly across all healthy backend Pods.

---

### 3. Scenario: The "Blue vs. Red" Problem

Imagine a **Blue App** needs to talk to a **Red App**.

* **Bad Approach:** Blue talks directly to Red's Pod IP (`10.42.0.5`).
* *Why it fails:* If Red crashes and restarts at `10.42.0.9`, Blue is now talking to a dead IP.


* **Good Approach:** Blue talks to the **Red Service** (`10.96.0.100`).
* *Why it works:* The Service tracks the new Red Pod IP automatically. Blue never needs to update its config.



> **Key Rule:** Never rely on Pod IPs for communication. Always use the Service IP or DNS.

---

## PART 2: Port Terminology

Understanding ports in Kubernetes manifests is crucial.

| Term | Definition | Context |
| --- | --- | --- |
| **port** | The port the **Service** listens on. | This is what other internal apps hit. |
| **targetPort** | The port the **Container** listens on. | This is where the app runs inside the pod. |
| **nodePort** | The static port opened on the **Worker Node**. | This allows external access (Range: 30000-32767). |

**Traffic Flow:**
`Client`  `ServiceIP:port`  `PodIP:targetPort`

---

## PART 3: Service Types

Kubernetes offers four main ways to expose an application.

### 1. ClusterIP (The Default)

* **Visibility:** **Internal Only**.
* **Behavior:** Assigns a virtual IP reachable *only* from within the cluster.
* **Use Case:** Databases, internal APIs, backend microservices.

### 2. NodePort

* **Visibility:** **External (Local Network)**.
* **Behavior:** Opens a specific port (e.g., `30005`) on the IP of **every** Node in the cluster.
* **Flow:** `External Client`  `NodeIP:NodePort`  `ClusterIP`  `Pod`.
* **Use Case:** Demos, development, or on-premise setups without a Load Balancer.

### 3. LoadBalancer

* **Visibility:** **Public Internet**.
* **Behavior:** Provisions a real Cloud Load Balancer (like AWS ELB or Google Cloud LB) that points to the NodePorts.
* **Use Case:** Production web servers, public APIs.

### 4. ExternalName

* **Visibility:** **DNS Alias**.
* **Behavior:** Instead of proxying traffic, it returns a CNAME record (e.g., mapping `my-db-service`  `db.aws-rds.com`).
* **Use Case:** Migrating legacy apps or connecting to external managed databases.

---

## PART 4: Under the Hood

### 1. Service Discovery (DNS)

When you create a Service named `my-backend` in the `default` namespace, CoreDNS automatically creates a DNS entry:

**Format:** `<service-name>.<namespace>.svc.cluster.local`
**Example:** `my-backend.default.svc.cluster.local`

This allows any Pod in the cluster to reach it simply by calling `http://my-backend`.

### 2. Kube-Proxy & The Logic

How does the traffic actually get routed?

* **Component:** `kube-proxy` runs on every worker node.
* **Mechanism:** It watches the API Server for new Services and Pods.
* **Action:** It updates the Node's **iptables** or **IPVS** rules to redirect traffic destined for the Service IP (`10.96.x.x`) to the actual healthy Pod IPs.

---

## PART 5: Essential Commands

**Viewing Services**

```bash
# List services to see IPs and Ports
kubectl get svc

# See which pods a service is actually pointing to (Endpoints)
kubectl get endpoints <service-name>

```

**Creating Resources (Imperative)**

```bash
# Create a namespace
kubectl create ns project-a

# Run a pod inside that namespace
kubectl run nginx-pod --image=nginx -n project-a

# Expose that pod as a Service
kubectl expose pod nginx-pod --port=80 --name=nginx-svc -n project-a

```

**Testing Connectivity**

```bash
# From inside the cluster, test the service DNS
kubectl run curl-test --image=curlimages/curl --restart=Never -- curl http://nginx-svc

```

---
