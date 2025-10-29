# DevOps Fundamentals

## What is DevOps?
DevOps is a **culture, practice, and set of tools** that brings together **software development (Dev)** and **IT operations (Ops)** teams.  
The primary goal is to deliver software **faster, more reliably, and at scale**.

> **Definition**  
> DevOps is a collaborative approach that aligns people, processes, and technology to enable **continuous and dependable software delivery**.

## Why DevOps?
- Faster and more frequent releases  
- Improved collaboration between teams  
- Early detection of defects through automation  
- Higher system reliability and uptime  
- Reduced manual effort and operational overhead  
- Quicker recovery from failures  

## Core DevOps Principles
1. **Shared Ownership**  
   Development and operations teams jointly own the product lifecycle.

2. **Automation Everywhere**  
   Build, test, and deployment processes are automated using CI/CD pipelines.

3. **Continuous Integration (CI)**  
   Code is merged frequently with automated builds and tests.

4. **Continuous Delivery / Deployment (CD)**  
   Software is always in a deployable state, with automated releases.

5. **Continuous Monitoring & Feedback**  
   Systems are constantly observed to improve performance and reliability.

---

# Software Development Life Cycle (SDLC)

## SDLC Phases
1. **Requirements Analysis**  
   - Understand business needs and constraints  
   - Clearly define project scope  

2. **System Design**  
   - Create architectural and technical designs  

3. **Development**  
   - Implement features based on design specifications  

4. **Testing (Quality Assurance)**  
   - Perform unit, integration, and system testing  

5. **Deployment**  
   - Release applications using strategies like **Blue-Green** or **Canary** to avoid downtime  

6. **Monitoring & Maintenance**  
   - Monitor application health and fix production issues  

---

# DevOps Lifecycle Mapping to SDLC

| Phase | Objective | Common Tools |
|------|----------|--------------|
| **Planning** | Task definition and sprint planning | Jira, Asana |
| **Development** | Source code creation and version control | Git, GitHub |
| **Build** | Application compilation and packaging | Maven, Gradle |
| **Testing** | Automated and manual testing | JUnit, Selenium |
| **Release** | Artifact preparation and versioning | Jenkins |
| **Deployment** | Application deployment | Docker, Kubernetes |
| **Operations** | Infrastructure and configuration management | Ansible, Puppet |
| **Monitoring** | Logging, metrics, and alerting | Prometheus, Grafana |

---

# DevSecOps: Security in DevOps

DevSecOps integrates **security at every stage of the DevOps pipeline**, rather than treating it as a final step.  
This concept is commonly known as **Shift-Left Security**.

## Security Techniques & Tools
- **Static Application Security Testing (SAST)**  
  - Scans source code for vulnerabilities  
  - Tools: SonarQube, Checkmarx  

- **Dynamic Application Security Testing (DAST)**  
  - Tests running applications for security flaws  
  - Tools: OWASP ZAP, Burp Suite  

- **Software Composition Analysis (SCA)**  
  - Detects vulnerabilities in third-party libraries  
  - Tools: Black Duck, JFrog Xray  

- **Linting & Code Quality Checks**  
  - Enforces coding standards and detects issues early  

- **Container Image Scanning**  
  - Identifies vulnerabilities in Docker images before deployment  

---

# Infrastructure as Code (IaC)

Infrastructure as Code enables teams to **define, provision, and manage infrastructure using code**, ensuring consistency across environments.

## Popular IaC Tools
- **Terraform** – Multi-cloud infrastructure provisioning  
- **AWS CloudFormation** – AWS-native infrastructure automation  
- **Ansible, Chef, Puppet** – Configuration management tools  
- **Boto3** – Python SDK for AWS service automation  

---

# Amazon VPC Fundamentals

## What is a VPC?
A **Virtual Private Cloud (VPC)** is an isolated virtual network in AWS that allows secure hosting of cloud resources.

## Key VPC Components

| Component | Description |
|---------|------------|
| **VPC** | Logical network container |
| **Subnet** | Segmented network within a VPC |
| **Public Subnet** | Allows internet-facing resources |
| **Private Subnet** | Hosts internal resources |
| **Internet Gateway (IGW)** | Enables internet access |
| **NAT Gateway** | Provides outbound internet for private subnets |
| **Route Table** | Defines traffic routing rules |
| **Security Group** | Instance-level firewall |
| **Network ACL** | Subnet-level firewall |

> **Defense-in-Depth Security**  
> - Security Groups protect instances  
> - NACLs protect subnets  
> - Route tables manage network flow  

---

# Sample VPC Architecture

**Important Highlights:**
- All resources within a VPC can communicate privately  
- Public subnets route traffic via an Internet Gateway  
- Private subnets access the internet using a NAT Gateway  

---

# Artifacts and Deployment Strategies

## Artifact Management
Artifacts are packaged outputs such as binaries or container images.  
Common artifact repositories include:
- Nexus Repository  
- JFrog Artifactory  

## Deployment Strategies
- **Blue-Green Deployment** – Switch traffic between two identical environments  
- **Canary Deployment** – Release updates gradually to a small user group  
- **Rolling Deployment** – Update instances incrementally without downtime  
