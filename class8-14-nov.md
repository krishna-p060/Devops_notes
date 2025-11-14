# Containers & Virtualization Deep Dive: Structured Notes

## 1. What is a Container?

A common misconception is that a container is a "mini virtual machine." It is not.

### The True Definition

A container is simply **a Linux process running with isolation and resource constraints.**

* **Not** a Virtual Machine.
* **Not** an Emulator.
* **Not** a full Operating System.

### Core Components

A container is built from four specific Linux technologies working together:

1. **Host Kernel:** Shared with the host (no separate kernel).
2. **Namespaces:** Provide **Isolation** (what the process can *see*).
3. **Cgroups:** Provide **Limits** (what resources the process can *use*).
4. **Ephemeral Filesystem:** A temporary, layered file structure.

---

## 2. Hypervisors vs. Containers

### The Hypervisor (VM) Model

A Hypervisor enables multiple **Virtual Machines (VMs)** to run on one physical server.

* **Heavyweight:** Each VM is a fully independent computer with its own:
* Full Operating System.
* Independent Kernel.
* Virtual Hardware (CPU, RAM, Network Card).
* Boot sequence (BIOS  Bootloader  Kernel  OS).



### The Container Model

Containers remove the heavy lifting.

* **Lightweight:** They strip away the BIOS, the Bootloader, and the Kernel.
* **Direct Execution:** They use the Host's kernel to run the application directly.
* **Speed:** Startup is measured in **milliseconds**, not minutes.

---

## 3. The Execution Pipeline: What Happens When You Run `docker run`?

When you type `docker run nginx`, a complex chain of events occurs behind the scenes.

### The Chain of Command

1. **Docker Client:** Sends the request.
2. **containerd:** The daemon that manages the container lifecycle.
* Prepares the filesystem (layers).
* Sets up networking.


3. **containerd-shim:** A lightweight process that sits between the daemon and the container.
* Allows `containerd` to restart without killing the container.
* Handles the exit codes (termination).


4. **runc:** The low-level CLI tool that actually spawns the container.
* Interacts directly with the Linux Kernel to create namespaces and cgroups.


5. **The Process (nginx):** Finally, the application starts.

> **Key Insight:** To the Host OS, the container is just another process with a Process ID (PID). Inside the container, that same process thinks it is PID 1 (The Root Process).

---

## 4. How Isolation Works: Linux Namespaces

Isolation is achieved by "lying" to the process about what it can see. Linux uses **Namespaces** to create these separate views.

| Namespace | Isolation Function |
| --- | --- |
| **PID** | Process ID. The container sees its own independent process tree. |
| **NET** | Networking. The container has its own IP, ports, and routing table. |
| **MNT** | Mount. The container has its own filesystem view. |
| **UTS** | Hostname. The container can have its own hostname. |
| **IPC** | Inter-Process Communication. Isolated memory segments. |
| **USER** | User IDs. Root inside the container can be a non-root user on the host. |

---

## 5. Ephemeral Storage

Containers are designed to be temporary.

* **Read-Only Layers:** The base image is never modified.
* **Write Layer:** A thin, temporary layer is created on top for runtime changes.
* **Data Loss:** When a container is removed, the Write Layer is **deleted**.
* **Persistence:** To save data, you must use **Docker Volumes** or **Bind Mounts** to bypass the container filesystem.

---

## 6. Networking & Ports

Since containers have their own Network Namespace, they cannot automatically talk to the outside world.

### Port Mapping

```bash
# Syntax: -p <HostPort>:<ContainerPort>
docker run -d -p 8080:80 nginx

```

* **Host Port (8080):** The door open to the outside world.
* **Container Port (80):** The internal port the app is listening on.
* **Conflict Rule:** You cannot bind two containers to the same **Host Port** (e.g., you can't have two apps both claiming port 8080 on the host).

---

## 7. Lifecycle Summary

1. **Start:** `runc` creates the namespaces and starts the process.
2. **Run:** `containerd-shim` monitors the process.
3. **Stop:** The main process (PID 1) is killed.
4. **Destroy:** The namespaces are deleted, cgroups released, and the temporary filesystem layer is removed.

---

## 8. Essential Commands Checklist

**Installation**

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

```

**Operations**

```bash
# Run detached with port map
docker run -d -p 80:80 nginx

# Check running processes
docker ps

# Check all (including stopped)
docker ps -a

```

---

