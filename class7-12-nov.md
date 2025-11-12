# Docker Image Building & Dockerfile: Structured Notes

## 1. Anatomy of a Docker Image

A Docker image is not a single file, but a **read-only template composed of stacked layers**.

### Core Concepts

* **Layers:** Every instruction in a Dockerfile adds a new layer.
* **Immutability:** Once built, image layers cannot be changed.
* **The Container Layer:** When you run `docker run <image>`, Docker adds a thin **read-write layer** on top of the read-only image. This is where the container's changes live.

---

## 2. The Dockerfile

A `Dockerfile` is a text-based script containing sequential instructions to automate the image assembly.

### The Foundation: `FROM`

Every Dockerfile **must** start with the `FROM` instruction (unless using global `ARG`).

* **Purpose:** Defines the **Base Image** (OS + Core dependencies).
* **Behavior:** If the image isn't found locally, Docker pulls it from the registry.
* **Best Practice:** Use specific tags (e.g., `ubuntu:22.04`) rather than `latest` for reproducibility. Use `slim` or `alpine` variants to reduce size.

```dockerfile
# Standard Syntax
FROM ubuntu:22.04

```

---

## 3. The Build Process

To convert a Dockerfile into an image, you use the `docker build` command.

### Syntax

```bash
# Standard build (looks for "Dockerfile" in current directory)
docker build -t my-image-name .

# Custom filename build
docker build -t my-image-name -f MyCustomDockerfile .

```

### Key Flags

* `-t`: Tags (names) the image. **Must be lowercase**.
* `-f`: Specifies a file name if it isn't the standard "Dockerfile".
* `.`: The **Build Context**. It tells Docker where to look for files to upload to the Docker daemon.

---

## 4. Instruction Phases: Build vs. Run

It is critical to distinguish when an instruction executes.

| Phase | Description | Instructions |
| --- | --- | --- |
| **Build-Time** | Executed during `docker build`. These commands create the image layers. | `FROM`, `RUN`, `COPY`, `ADD`, `WORKDIR`, `ENV` |
| **Run-Time** | Executed when the container starts via `docker run`. They determine how the container behaves. | `CMD`, `ENTRYPOINT` |

---

## 5. The `RUN` Instruction

Used to install software and configure the environment **inside the image**.

* **Action:** Executes a command and commits the result as a new layer.
* **Optimization:** Chain commands using `&&` to reduce the number of layers and image size.

**Bad Practice (Creates multiple layers):**

```dockerfile
RUN apt-get update
RUN apt-get install -y curl

```

**Good Practice (Single layer, cleans up cache):**

```dockerfile
RUN apt-get update && apt-get install -y curl \
    && rm -rf /var/lib/apt/lists/*

```

---

## 6. Controlling Container Startup: `CMD` vs `ENTRYPOINT`

These two instructions often confuse beginners. They control what happens when the container starts.

### `CMD` (Command)

* **Purpose:** Provides a **default** command or argument.
* **Override:** **Easy**. If the user runs `docker run myimage bash`, the `CMD` is ignored, and `bash` runs instead.
* **Limit:** Only the last `CMD` in a file is valid.

### `ENTRYPOINT`

* **Purpose:** Defines the **main executable** of the container.
* **Override:** **Hard**. Arguments added to `docker run` are *appended* to the entrypoint, not replaced (unless you use the `--entrypoint` flag).

### The Power Combo: `ENTRYPOINT` + `CMD`

You can use them together to create flexible executables.

```dockerfile
# The fixed executable
ENTRYPOINT ["python", "app.py"]

# The default argument (can be changed by user)
CMD ["--help"]

```

**Scenario 1:** `docker run myimage`

* Executes: `python app.py --help`

**Scenario 2:** `docker run myimage --version`

* Executes: `python app.py --version` (Overwrites CMD)

---

## 7. File Management: `COPY` vs `ADD`

| Feature | COPY | ADD |
| --- | --- | --- |
| **Basic File Copying** | ✅ Yes | ✅ Yes |
| **Remote URLs** | ❌ No | ✅ Yes (Downloads file) |
| **Auto-Extraction** | ❌ No | ✅ Yes (Unzips .tar/.gz) |
| **Verdict** | **Preferred**. It is explicit and predictable. | Use only for tar extraction or downloading remote files. |

**Example:**

```dockerfile
# Copies all files from current host dir to /app in container
COPY . /app

```

---

## 8. Analyzing Images

Once built, you can inspect your images using these commands:

* **View Layers:** `docker history <image_name>` (Shows size of each layer).
* **View Metadata:** `docker inspect <image_name>` (Shows env vars, architecture, paths).
* **Security:** `docker scan <image_name>` (Checks for CVE vulnerabilities).

---

## 9. Comprehensive Example

A practical "Best Practice" Dockerfile.

```dockerfile
# 1. Base Image: Use a specific, slim version
FROM python:3.12-slim

# 2. Setup: Define working directory
WORKDIR /app

# 3. Cache Optimization: Copy requirements FIRST
# Docker caches this layer. If requirements.txt doesn't change, 
# it skips installing dependencies again.
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 4. Copy Code: Copy the rest of the application
COPY . .

# 5. Startup: Define the executable and default arg
ENTRYPOINT ["python"]
CMD ["app.py"]

```

---

