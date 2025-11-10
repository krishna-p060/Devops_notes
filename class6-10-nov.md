# System Virtualization & Containerization: Structured Notes

## 1. Bare Metal Architecture

This is the traditional model where applications run directly on the physical hardware without any virtualization layer.

### The Stack Layers

1. **Hardware:** The physical foundation (CPU, RAM, HDD, Network Card).
2. **Kernel:** The core of the OS that manages hardware resources.
3. **System Libraries/Binaries:** Shared files required for programs to run.
4. **Application:** The actual user software.

### Key Characteristics

* **Performance:** Maximum efficiency (no middleman).
* **Limitation:** Only one Operating System (OS) can run at a time; low flexibility.

---

## 2. Hypervisor-Based Virtualization

A **Hypervisor** is software that creates an abstraction layer over hardware, allowing a single physical machine to be split into multiple **Virtual Machines (VMs)**.

### Types of Hypervisors

#### Type 1: Bare Metal

* **Location:** Installed directly on the hardware.
* **Efficiency:** High performance (no host OS overhead).
* **Examples:** VMware ESXi, Microsoft Hyper-V, Xen.

#### Type 2: Hosted

* **Location:** Installed as an application on top of an existing Host OS.
* **Efficiency:** Lower performance due to extra layers.
* **Examples:** Oracle VirtualBox, VMware Workstation.

### The Heavy Cost of VMs

Every VM is an isolated computer that requires:

* Its own **Virtual Hardware** (CPU, RAM, Network).
* Its own **Full Kernel**.
* Its own **Full Operating System**.

---

## 3. The Virtual Machine Boot Process (Why it's Slow)

When you start a VM, it must go through the entire boot sequence of a physical computer.

1. **BIOS:** Hardware checks and boot device selection.
2. **MBR (Master Boot Record):** Reads the partition table.
3. **Bootloader (e.g., GRUB):** Loads the OS into memory.
4. **Kernel Initialization:** Drivers loaded, memory managed.
5. **Init Process (e.g., Systemd):** Starts background services.
6. **User Space:** Finally, the application starts.

> **Result:** This redundancy makes VMs slow to start (minutes) and resource-heavy (GBs of RAM).

---

## 4. Containers vs. Virtual Machines

Docker takes a fundamentally different approach by eliminating the redundancy of the OS.

### The Architecture Shift

* **VM Model:** Hardware  Host OS  Hypervisor  Guest OS  App
* **Container Model:** Hardware  Host OS  Docker Engine  App

### Why Containers win on Speed

* **No Booting:** Containers do *not* run BIOS, MBR, or a Kernel.
* **Shared Kernel:** They utilize the existing **Host OS Kernel**.
* **Startup:** Since they skip the boot sequence, they start in **milliseconds**.

---

## 5. Docker Architecture Components

Docker uses a client-server architecture to manage the lifecycle of containers.

### 1. Docker Client

The interface (CLI) you interact with. It sends API requests (like `docker build`, `docker run`) to the Daemon.

### 2. Docker Daemon (`dockerd`)

The background process that listens for requests and manages:

* Images
* Containers
* Networks
* Volumes

### 3. Docker Registry

The central storage for Docker Images.

* **Public:** Docker Hub.
* **Private:** AWS ECR, Azure ACR, etc.

---

## 6. Practical Docker Operations

### Installation (Linux)

```bash
# Download and install script
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Verify
docker run hello-world

```

### Essential Commands Cheatsheet

| Action | Command |
| --- | --- |
| **List Running** | `docker ps` |
| **List All** (incl. stopped) | `docker ps -a` |
| **Run (Foreground)** | `docker run nginx` |
| **Run (Background/Detached)** | `docker run -d nginx` |
| **Inspect Details** | `docker inspect <container-id>` |
| **Enter Container** | `docker exec -it <container-id> bash` |

### Registry & Image Management

**1. Authenticate**

```bash
docker login -u <username>

```

**2. Save Container State as Image**

```bash
# Syntax: docker commit <container-id> <user>/<image-name>
docker commit -m "Added new config" a1b2c3d4 myuser/my-app

```

**3. Upload to Registry**

```bash
docker push myuser/my-app

```

**4. Clean Up**

```bash
docker rmi <image-id>

```

---
