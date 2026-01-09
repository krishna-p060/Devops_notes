# Terraform Fundamentals: Part 1 — Structured Notes

## 1. What is Terraform?

Terraform is an open-source **Infrastructure as Code (IaC)** tool developed by **HashiCorp**. It allows you to build, change, and version infrastructure safely and efficiently.

* **Model:** It uses a **Declarative** approach. You define *what* you want (the end state), and Terraform figures out *how* to achieve it.
* **Scope:** It is platform-agnostic, supporting AWS, Azure, GCP, Kubernetes, Docker, and hundreds of other providers.

---

## 2. Infrastructure as Code (IaC) vs. Manual Setup

### The Old Way (Manual)

* Clicking through web consoles (GUI).
* **Risks:** High human error, difficult to replicate, no version history ("ClickOps").

### The New Way (IaC)

* Writing configuration files to define resources.
* **Benefits:**
* **Consistency:** Eliminates configuration drift.
* **Version Control:** Code is stored in Git (history, code reviews, rollbacks).
* **Automation:** Speed and scalability.



---

## 3. Core Architecture & Components

### A. Providers

Plugins that allow Terraform to interact with external APIs.

* Examples: `aws`, `google`, `docker`, `kubernetes`.
* Without a provider, Terraform cannot talk to any cloud.

### B. Resources

The fundamental building blocks. A resource describes a specific piece of infrastructure object.

* Examples: An EC2 instance, an S3 bucket, a Docker container.

### C. The State File (`terraform.tfstate`)

* **Purpose:** Terraform's "brain." It maps your code configuration to real-world resources.
* **Function:** It tracks metadata and attributes of deployed resources to know what to update or destroy.

---

## 4. The HCL Syntax (HashiCorp Configuration Language)

Terraform code is written in HCL, designed to be human-readable.

### Block Structure Anatomy

```hcl
# <BLOCK TYPE> "<RESOURCE TYPE>" "<LOCAL NAME>"
resource "aws_instance" "web_server" {
  # <ARGUMENT> = <VALUE>
  ami           = "ami-12345678"
  instance_type = "t2.micro"
}

```

* **Block Type:** What kind of object is this? (`resource`, `variable`, `provider`, `data`).
* **Resource Type:** The specific component from the provider (e.g., `aws_instance`).
* **Local Name:** How you refer to this resource inside your code (e.g., `web_server`).

---

## 5. The Terraform Lifecycle (Workflow)

The standard procedure for applying changes involves three main steps.

### Step 1: Initialize (`init`)

Prepares the directory.

```bash
terraform init

```

* Downloads the necessary Provider plugins.
* Sets up the backend for the state file.

### Step 2: Plan (`plan`)

The "Dry Run."

```bash
terraform plan

```

* Compares your code against the current state.
* Shows exactly what actions will be taken (Create, Update, or Destroy) without actually making changes.

### Step 3: Apply (`apply`)

Execution.

```bash
terraform apply

```

* Executes the plan.
* Provisions the infrastructure.
* Updates the State file.

---

## 6. Practical Example: Docker Provider

A configuration to launch an Nginx container using Terraform.

```hcl
# 1. Define the Provider
terraform {
  required_providers {
    docker = {
      source  = "kreuzwerker/docker"
      version = "~> 2.21.0"
    }
  }
}

provider "docker" {}

# 2. Pull the Image
resource "docker_image" "nginx" {
  name = "nginx:latest"
}

# 3. Run the Container
resource "docker_container" "nginx_app" {
  image = docker_image.nginx.latest
  name  = "nginx-container"
  
  ports {
    internal = 80
    external = 8080
  }
}

```

---

## 7. Variables & Data Types

Hardcoding values is bad practice. Variables make your code reusable.

### Declaration (`variables.tf`)

```hcl
variable "container_name" {
  description = "Name of the docker container"
  type        = string
  default     = "my-app"
}

variable "external_port" {
  type    = number
  default = 8080
}

```

### Usage (`main.tf`)

Use the `var.` prefix to access variables.

```hcl
resource "docker_container" "app" {
  name = var.container_name
  ports {
    internal = 80
    external = var.external_port
  }
}

```

---

## 8. Outputs

Outputs are like "return values" for your infrastructure modules. They print useful information to the console after an apply.

```hcl
output "container_id" {
  value = docker_container.nginx_app.id
}

```

---

## 9. Essential Command Cheatsheet

| Command | Description |
| --- | --- |
| `terraform init` | Downloads plugins & sets up backend. |
| `terraform fmt` | Automatically formats code to HCL standards. |
| `terraform validate` | Checks code for syntax errors. |
| `terraform plan` | Previews changes. |
| `terraform apply` | Creates/Updates infrastructure. |
| `terraform destroy` | Deletes all infrastructure managed by this config. |
| `terraform state list` | Lists all resources currently tracked in the state file. |

---