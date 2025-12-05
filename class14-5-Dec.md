# Class 14 — Kubernetes Deployment Deep Dive

## Section 1: Conceptual Foundation

### 1. The Hierarchy of Power

In a production Kubernetes environment, you rarely manage Pods or ReplicaSets directly. You manage a **Deployment**, which acts as a high-level manager.

* **The Chain of Command:**
* **Deployment:** Manages releases (updates/rollbacks). It controls 
* **ReplicaSet:** Manages scale (number of copies). It controls 
* **Pod:** Manages the running container.



### 2. Deployment vs. ReplicaSet

If you look at the YAML for both, they are 95% identical. The difference lies in **capability**, not structure.

* **ReplicaSet (RS):** Great at keeping  number of pods alive. Terrible at updates (you have to manually delete old pods to create new ones).
* **Deployment:** Wraps around the RS. It enables:
* **Rolling Updates:** Zero-downtime version changes.
* **Rollbacks:** Reverting to a previous stable state.
* **Pausing:** stopping a rollout mid-way to check health.



---

## Section 2: The Reliability Inheritance Stack

Reliability in Kubernetes is built in layers, with each layer adding a specific "superpower."

| Layer | Responsibility | Added Capability |
| --- | --- | --- |
| **Container** | Runtime | Process isolation & fast startup. |
| **Pod** | Co-location | **Kubelet** restarts container if process crashes. |
| **ReplicaSet** | Availability | **Controller** recreates Pod if Node crashes. Guaranteed Count. |
| **Deployment** | Lifecycle | **Deployment Controller** manages versions, history, and updates. |

> **Key Takeaway:** A Deployment inherits all the resilience of the layers below it.

---

## Section 3: Cluster Initialization (Recap)

A quick refresher on bringing up a cluster to test deployments.

1. **Host Config:** Ensure unique hostnames (`hostnamectl set-hostname controlplane`).
2. **Bootstrap:** Run `kubeadm init` on the master node.
3. **Network:** The cluster stays `NotReady` until a CNI is installed.
```bash
kubectl apply -f https://github.com/weaveworks/weave/release/download/v2.8.1/weave-daemonset-k8s.yaml

```


4. **Verification:** Watch the system pods. Once `coredns` is Running, the cluster is live.
```bash
watch -n 1 kubectl get pods -n kube-system

```



---

## Section 4: Deployment Operations

### 1. Creating a Deployment

The manifest looks like a ReplicaSet but with `kind: Deployment`.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webapp # Must match template labels
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: app
        image: nginx:1.27

```

**Apply and Monitor:**

```bash
kubectl apply -f deploy.yaml
watch -n 1 kubectl get deploy,rs,po

```

### 2. The Internal Workflow

When you apply the above YAML:

1. **API Server:** Validates auth and syntax; writes to **etcd**.
2. **Deployment Controller:** Notices the new Deployment. It creates a **ReplicaSet**.
3. **ReplicaSet Controller:** Notices the new RS. It creates 3 **Pods**.
4. **Scheduler:** Assigns Pods to Nodes.
5. **Kubelet:** Starts containers.

### 3. Scaling Options

* **Imperative (CLI):** `kubectl scale deployment webapp --replicas=5`
* **Declarative (YAML):** Edit the file to `replicas: 5` and run `kubectl apply`.
* **Autoscaling:**
* **HPA:** Scales Pods based on CPU/RAM.
* **Cluster Autoscaler:** Scales Nodes (VMs) when cluster is full.



---

## Section 5: Updates & Rollbacks (The Core Feature)

This is the primary reason we use Deployments over ReplicaSets.

### 1. Rolling Update Strategy (Default)

When you change the image version (e.g., `nginx:1.27`  `1.28`), the Deployment does **not** kill all pods at once.

* **Logic:** It spins up a **New ReplicaSet** and scales it *up* slowly, while scaling the **Old ReplicaSet** *down*.
* **Result:** Zero downtime.
* **Config:**
* `maxUnavailable`: How many pods can die during update (default 25%).
* `maxSurge`: How many extra pods can be created during update (default 25%).



### 2. Recreate Strategy

* **Logic:** Kills **ALL** old pods, then starts **ALL** new pods.
* **Use Case:** When the app cannot support running two versions simultaneously (e.g., database schema incompatibility).

### 3. Managing Rollouts

**Trigger an Update:**

```bash
kubectl set image deployment/webapp app=nginx:1.28

```

**Check Status:**

```bash
kubectl rollout status deployment/webapp

```

**View History:**

```bash
kubectl rollout history deployment/webapp

```

**Rollback (Undo):**
Oh no! Version 1.28 is crashing. Go back to the previous stable version.

```bash
kubectl rollout undo deployment/webapp
# Or go to specific revision
kubectl rollout undo deployment/webapp --to-revision=2

```

**Pause/Resume (Canary-ish):**

```bash
kubectl rollout pause deployment/webapp
# ... perform manual checks ...
kubectl rollout resume deployment/webapp

```

---

## Section 6: Under the Hood (Etcd)

Kubernetes stores everything as objects in `etcd`. During an update, you will see multiple ReplicaSets.

* `/registry/deployments/default/webapp` (The Manager)
* `/registry/replicasets/default/webapp-oldhash` (Scaled to 0)
* `/registry/replicasets/default/webapp-newhash` (Scaled to 3)

A **Rollback** is simply the Deployment Controller flipping the desired replica counts: it scales the "Old Hash" RS back up to 3 and the "New Hash" RS down to 0.

---