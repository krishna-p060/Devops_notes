# Docker Volumes & Data Persistence: Conceptual Notes

## 1. The Core Problem: Ephemeral Design

By default, Docker containers are designed to be temporary (ephemeral).

### The Layered Architecture

* **Image Layers:** Read-only (bottom).
* **Container Layer:** Read-write (top).

When you write data (logs, databases, file uploads) inside a container, it lives in that top **Writable Layer**.

### The Persistence Rule

* **Stop/Start:** If you stop a container (`docker stop`), the data **remains**. The container object still exists.
* **Remove:** If you remove a container (`docker rm`), the writable layer is **destroyed**. All data inside it is lost forever.

> **Solution:** To keep data after a container is deleted, you must use **Docker Volumes**.

---

## 2. What is a Docker Volume?

A volume is a specific directory on the host machine's filesystem that is **managed by Docker** to persist data.

* **Decoupling:** It decouples storage from the container lifecycle.
* **Survival:** Even if you delete the container, the volume (and its data) remains.
* **Sharing:** Multiple containers can attach to the same volume simultaneously.
* **Location:** Standard volumes live in `/var/lib/docker/volumes/`.

---

## 3. The Four Types of Mounts

Docker offers four ways to mount storage into a container.

### A. Named Volumes (Production Standard)

Explicitly created and named by the user. Best for persistent data like databases.

* **Create:** `docker volume create my-data`
* **Run:** `docker run -v my-data:/db/path ...`
* **Behavior:** Data survives container deletion. You can attach this volume to a new container later to access the same data.

### B. Anonymous Volumes

Created automatically if you don't provide a name.

* **Run:** `docker run -v /db/path ...`
* **Behavior:** Docker assigns a random hash ID. Hard to reference later. Usually deleted when the container is removed (if using the `--rm` flag).

### C. Bind Mounts (Development Standard)

Maps a specific folder on your **Host Machine** to a folder in the **Container**.

* **Run:** `docker run -v /home/user/project:/app ...`
* **Behavior:** Changes are instant. If you edit code on your host, it updates in the container immediately.
* **Risk:** Can expose sensitive host system files to the container.

### D. tmpfs Mounts (Security/Speed)

Stores data in the host's **RAM (Memory)**, never writing to disk.

* **Run:** `docker run --tmpfs /app/cache ...`
* **Behavior:** Extremely fast but volatile. Data is lost the moment the container stops.

---

## 4. Comparison Table

| Feature | Named Volume | Bind Mount | tmpfs |
| --- | --- | --- | --- |
| **Persistence** | ✅ Yes | ✅ Yes | ❌ No |
| **Host Location** | `/var/lib/docker/volumes/` | User-defined path | System RAM |
| **Managed By** | Docker CLI | User / Host OS | Host OS Memory |
| **Best For** | Databases, Prod Data | Dev Code, Configs | Secrets, Caches |

---

## 5. Practical Workflow Examples

### Scenario 1: Persisting Database Data (Named Volume)

You want your MySQL data to survive even if you upgrade the MySQL container.

```bash
# 1. Create the volume
docker volume create mysql_data

# 2. Run container attaching volume to DB path
docker run -d \
  -v mysql_data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=pass \
  mysql:8

```

### Scenario 2: Live Code Reloading (Bind Mount)

You are developing a web app and want to see changes without rebuilding the image.

```bash
# Syntax: -v $(pwd)/local_folder:/container_folder
docker run -d \
  -v $(pwd)/html:/usr/share/nginx/html \
  -p 8080:80 \
  nginx

```

### Scenario 3: Sharing Data

Two containers (e.g., an App and a Logger) need to see the same files.

```bash
# Both containers mount the SAME volume
docker run -d -v logs_vol:/var/log/app my-app
docker run -d -v logs_vol:/read-logs my-logger

```

---

