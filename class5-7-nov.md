# Docker Fundamentals on AWS EC2: Essential Notes

## 1. Establishing an SSH Connection

Before working with Docker, you must securely access your remote EC2 instance.

### Connection Workflow

1. **Locate Instance:** Go to **AWS Console > EC2 > Instances**.
2. **Get Credentials:** Select your instance, click **Connect**, and choose **SSH Client** to view the connection string.
3. **Prepare Key:** On your local machine, open a terminal and locate your private key file (`.pem`).
4. **Connect:** Run the SSH command (ensure you are in the folder containing the key).

```bash
# syntax: ssh -i <key-file> <user>@<public-dns>
ssh -i your-key.pem ubuntu@ec2-xx-xx-xx-xx.compute.amazonaws.com

```

---

## 2. Validating the Installation

Verify that Docker is operational by running the standard test image.

```bash
docker run hello-world

```

### What happens in the background?

When you execute this command, Docker performs four specific actions:

1. **Check:** Looks for the `hello-world` image locally.
2. **Pull:** Downloads it from the registry (Docker Hub) if missing.
3. **Create:** Builds a new container from that image.
4. **Execute:** Runs the container, prints the success message, and exits.

> **Note:** If you see "Hello from Docker!", the daemon and client are communicating correctly.

---

## 3. The Golden Rule of `docker run`

It is crucial to understand that `docker run` is not just for starting execution.

* **Rule:** Every time you execute `docker run <image>`, Docker creates a **brand new container**.
* It does *not* restart an existing, stopped container.

---

## 4. Concept: Docker Images

Images are the static building blocks of Docker.

* **Analogy:** Think of an Image as a **Class** in programming, and a Container as an **Object**.
* **Function:** They serve as read-only templates used to create containers.
* **Storage:** Images are stored locally or pulled from a remote registry (like Docker Hub).

---

## 5. Concept: Docker Containers

Containers are the active, running instances of images.

* **Architecture:** Unlike Virtual Machines (VMs), containers share the **Host OS Kernel**.
* **Isolation:** Despite sharing the kernel, they maintain strict isolation regarding:
* Filesystems
* Process IDs


* **Benefit:** This shared-kernel architecture makes them significantly lighter and faster than VMs.

---

## 6. Architecture: Client vs. Daemon

Docker operates on a client-server architecture.

### The Docker Client (CLI)

* The tool you use in the terminal (commands starting with `docker`).
* It sends API requests to the Daemon.

### The Docker Daemon (Server)

* A background process running on the host machine.
* **Responsibilities:** building, running, and distributing containers.
* **Flow:** User Command  Client  Daemon  Action (Pull/Run).

---

## 7. Managing the Docker Service

In Linux, background processes are known as **daemons**. You can manage the Docker daemon using `systemctl`.

**Check System Status:**

```bash
systemctl status

```

**Check Docker Specific Status:**

```bash
systemctl status docker

```

---

## 8. Installation Artifacts

When you install Docker, you are essentially installing two distinct binaries:

1. **The Client:** The command-line interface.
2. **The Daemon:** The engine that does the heavy lifting.

**Verify Version:**

```bash
docker version

```

---

## 9. Interactive Mode (Shell Access)

To enter a container and run commands inside it (like a virtual server), use the interactive flags.

```bash
docker run -it ubuntu bash

```

### Breakdown of Flags

* `-i` (**Interactive**): Keeps the Standard Input (STDIN) open.
* `-t` (**TTY**): Allocates a pseudo-terminal (provides the command prompt feel).
* `bash`: The specific command/shell to run inside the container.

> **Visual Cue:** Your command prompt will change from `user@host` to `root@<container-id>`.

---

## 10. Process Isolation

Containers utilize namespaces to hide processes from the host and other containers.

If you run a process check *inside* a container:

```bash
ps -ef

```

You will only see the processes specific to that container (often just PID 1 and the shell), confirming that the environment is isolated.


