# Deep Dive: Kubernetes Pods & Internals

## Part I: Conceptual Foundations

### 1. What is a Pod?

A **Pod** is the smallest, atomic unit of deployment in Kubernetes.

* **Analogy:** If a container is a "Process," a Pod is the "Virtual Host" or "Machine" that wraps around it.
* **Composition:** It can run a single container (most common) or multiple "tightly coupled" containers.

**Shared Context**
Containers inside the *same* Pod share:

* **Network Namespace:** They share the same IP address and Port space. They can talk to each other via `localhost`.
* **Storage Volumes:** They can mount the same disk to share files.
* **Lifecycle:** They are born together, scheduled to the same node together, and die together.

---

### 2. Networking & The CNI

Kubernetes itself does not handle the wiring of networks. It relies on a **CNI (Container Network Interface)** plugin.

**The Role of CNI:**

* Allocates unique IPs to every Pod.
* Sets up the virtual network interfaces.
* Ensures Pods can talk to each other across different nodes without NAT.

**Setup Example (Weave Net):**

```bash
# Deploys a DaemonSet (one agent per node) to handle networking
kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml

```

---

## Part II: Cluster Operations (Hands-On)

### 1. Bootstrapping a Cluster

Setting up a control plane and worker nodes using `kubeadm`.

**Control Plane Init:**

```bash
# Run ONLY on the Master Node
sudo kubeadm init

```

* Generates the certificates (CA).
* Starts the Control Plane components (API, Scheduler, Etcd).
* Outputs the `kubeadm join` command for workers.

**Worker Node Join:**
If you lost the token, regenerate it on the master:

```bash
kubeadm token create --print-join-command

```

### 2. Basic Pod Management

**Create a Pod (Imperative):**

```bash
# --restart=Never ensures it creates a Pod, not a Deployment
kubectl run testpod --image=nginx --restart=Never

```

**Inspect Pods:**

```bash
# -o wide reveals the Node assignment and Pod IP
kubectl get pods -o wide

```

### 3. Namespaces

Logical partitions within the cluster to isolate teams or environments.

* `default`: The standard workspace.
* `kube-system`: Where core K8s components live.
* `kube-public`: Auto-readable by all users (rarely used).

```bash
kubectl create ns demo
kubectl get pods -n demo

```

---

## Part III: Internal Mechanics

### 1. The Pod Object (Etcd State)

When you save a Pod, it is stored in `etcd` as a JSON/YAML object containing:

* **Spec:** The desired state (Images, Volumes, Resources).
* **Status:** The current reality (IP, Phase, Conditions).
* **Metadata:** UID, Labels, ResourceVersion.

### 2. The Creation Lifecycle

The journey from `kubectl apply` to a running container:

1. **API Server:** Receives request, validates auth, writes to `etcd`.
2. **Scheduler:** Watches API, sees a Pod with `nodeName: empty`. Scores nodes and assigns one.
3. **API Server:** Updates the Pod object with the assigned Node.
4. **Kubelet (on Node):** Watches API, sees a Pod assigned to *itself*.
5. **CRI (Runtime):** Kubelet tells Docker/CRI-O to pull images and start containers.
6. **Status Update:** Kubelet reports "Running" back to API Server.

---

### 3. Kubelet Startup Flow

What actually happens on the Worker Node?

1. **Volume Mount:** Prepares storage.
2. **CNI Setup:** Configures the network interface.
3. **Pause Container:** The "Sandbox" container is started first to hold the network namespace.
4. **Image Pull:** Downloads application images.
5. **Init Containers:** Runs any setup scripts sequentially.
6. **App Containers:** Finally, the main application starts.

---

## Part IV: Health & Debugging

### 1. Health Probes

K8s needs to know the difference between "Running" and "Working."

| Probe Type | Question Asked | Action on Failure |
| --- | --- | --- |
| **Liveness** | Is the app dead/deadlocked? | **Restart** the container. |
| **Readiness** | Is the app ready to serve traffic? | Remove IP from **Service** (stop sending traffic). |
| **Startup** | Has the legacy app finished booting? | Pause other probes until this passes. |

### 2. Pod States (Phases)

* **Pending:** Accepted by API, but waiting for scheduling or image pull.
* **Running:** Bound to a node, all containers created.
* **Succeeded/Failed:** Terminated (common in Batch Jobs).
* **Unknown:** API Server lost contact with the Node (Kubelet is down).

### 3. Debugging Cheatsheet

**Common Errors:**

* `ImagePullBackOff`: Cannot find image or wrong credentials.
* `CrashLoopBackOff`: App is starting but immediately dying (check logs).
* `Pending`: No node has enough RAM/CPU to fit the pod.

**Investigation Commands:**

```bash
# 1. The Deep Dive (Events, Config, Errors)
kubectl describe pod <pod-name>

# 2. The Application Output
kubectl logs <pod-name>

# 3. The Timeline
kubectl get events --sort-by='.lastTimestamp'

```

---