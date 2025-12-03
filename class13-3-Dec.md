# ReplicaSets & Pods: Structured Class Notes

## Part I: Cluster & Pod Fundamentals

### 1. Cluster Topology

A minimal Kubernetes cluster requires at least two components to function effectively:

* **Control Plane (Master Node):** The "Brain." It manages the state, scheduling, and API.
* **Worker Node:** The "Muscle." It runs the actual application workloads (Pods).

---

### 2. The Setup Sequence (Hands-On)

**Step A: Initialization (Control Plane)**
The `kubeadm` tool bootstraps the cluster.

```bash
# Run on Master Node
kubeadm init

```

* **Result:** Starts the API Server, Scheduler, and Controller Manager. Generates the Admin Kubeconfig and certificates.
* **Output:** Provides the `kubeadm join` command for workers.
* *Lost the token?* Run `kubeadm token create --print-join-command`.



**Step B: Networking (CNI)**
Kubernetes does not provide networking out of the box. You must install a **Container Network Interface (CNI)** plugin (e.g., Weave, Calico, Flannel) to allow Pods to communicate.

```bash
kubectl apply -f https://github.com/weaveworks/weave/release/download/v2.8.1/weave-daemonset-k8s.yaml

```

**Step C: Validation**
Check if system components (like CoreDNS) are running.

```bash
watch -n 1 kubectl get pods -n kube-system

```

---

### 3. Pod Management Basics

**Imperative Creation (CLI)**
Great for quick tests, but not for production.

```bash
# --restart=Never tells K8s to create a Pod, not a Deployment
kubectl run demo-nginx --image=nginx --restart=Never

```

**Inspection**

```bash
# -o wide shows which Node the pod is on and its internal IP
kubectl get pods -o wide

```

**Cleanup**

```bash
# Deletes everything in the current namespace
kubectl delete pod --all

```

---

## Part II: The ReplicaSet (RS)

### 1. Concept: The Supervisor

A **ReplicaSet** is a controller that guarantees the availability of a specific number of identical Pods.

* **Self-Healing:** If a Pod crashes or is deleted, the RS notices the count is low and spins up a replacement.
* **The Loop:** It continuously monitors: `Current State == Desired State`.

---

### 2. Anatomy of a ReplicaSet Manifest

A ReplicaSet relies on three key fields to function.

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
spec:
  # 1. How many do we want?
  replicas: 3
  
  # 2. Which pods do we own? (The Filter)
  selector:
    matchLabels:
      app: web
      
  # 3. What do we create if we are missing one? (The Blueprint)
  template:
    metadata:
      labels:
        app: web  # MUST match the selector above
    spec:
      containers:
        - name: nginx
          image: nginx

```

> **Critical Rule:** The `spec.selector.matchLabels` must match the `template.metadata.labels`. If they don't match, the ReplicaSet cannot manage the pods it creates.

---

### 3. How It Works (The Control Loop)

When you apply the manifest (`kubectl apply -f rs.yaml`):

1. **API Request:** Sent to the Master Node.
2. **Storage:** Saved in `etcd`.
3. **Controller Check:** The ReplicaSet Controller sees "Desired: 3, Existing: 0".
4. **Creation:** It creates 3 Pod objects based on the **Template**.
5. **Scheduling:** The Scheduler assigns them to nodes.
6. **Kubelet:** Starts the containers.

**The Reconciliation Logic:**

* **Too few pods?**  Create new ones (using Template).
* **Too many pods?**  Terminate extras (Scale down).

---

### 4. Advanced Operations

**Scaling**
You can change the number of replicas dynamically.

```bash
# Imperative scale
kubectl scale rs nginx-rs --replicas=5

```

**Deletion & Cascading**

* **Standard Delete:** Deletes the ReplicaSet *and* all its Pods.
```bash
kubectl delete rs nginx-rs

```


* **Orphan Delete:** Deletes the ReplicaSet resource but *leaves the Pods running*.
```bash
kubectl delete rs nginx-rs --cascade=orphan

```



**Garbage Collection**
Pods created by a ReplicaSet have an `ownerReference` field in their metadata. This links them to the RS, allowing Kubernetes to automatically clean up Pods when the RS is deleted.

---