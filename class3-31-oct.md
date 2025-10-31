# CI/CD, Testing, and Deployment – DevOps Notes

## Introduction to CI/CD

**CI/CD** represents a collection of DevOps practices used to automate the **build, test, and release** phases of software development.

- **Continuous Integration (CI):** Automates code integration and validation.
- **Continuous Delivery / Deployment (CD):** Automates software release processes.

The primary aim is to achieve **rapid, reliable, and low-risk deployments**.

---

## Continuous Integration (CI)

### What is Continuous Integration?
Continuous Integration is a development practice where developers frequently merge code changes into a shared repository, triggering **automated builds and validations**.

### Importance of CI
- Detects bugs at an early stage
- Avoids integration conflicts
- Improves overall code quality
- Reduces manual testing effort
- Provides faster developer feedback

### Typical CI Pipeline
1. Developer pushes code to a version control system.
2. CI server fetches the latest code.
3. Required dependencies are installed.
4. Application is built or compiled.
5. Linting and static code analysis are executed.
6. Unit tests are triggered.
7. Static security analysis (SAST) is performed.
8. Dependency vulnerability scanning (SCA) is done.
9. Application is packaged into a Docker image.
10. Artifacts are stored in an artifact repository.

**Core Objective:** Identify issues as early as possible in the development lifecycle.

---

## Continuous Delivery (CD)

### What is Continuous Delivery?
Continuous Delivery ensures that the application is always in a **production-ready state**.  
All deployment steps are automated until production, where **manual approval** is required.

### Delivery Workflow
1. Fetch build artifacts from the repository.
2. Deploy application to the SIT (System Integration Testing) environment.
3. Validate inter-service communication.
4. Run performance and load tests.
5. Conduct dynamic security testing (DAST).
6. Obtain manual approval for production release.

**Key Principle:** The software is always ready to be released.

---

## Continuous Deployment

Continuous Deployment takes automation a step further by **deploying every successful build directly to production** without manual intervention.

### Advantages
- Faster feature rollouts
- Reduced human errors
- Continuous user feedback
- Simple rollback strategies

**Common CI/CD Tools:**  
Jenkins, GitHub Actions, GitLab CI/CD, ArgoCD, Spinnaker, CircleCI

---

## CI/CD Pipeline Overview

Source Code Repository
|
v
Continuous Integration

Build

Unit Testing

Static Code Analysis

Security Scanning

Docker Image Creation
|
v
Continuous Delivery

Deploy to SIT

Integration Testing

Performance Testing

DAST
|
v
Continuous Deployment

Production Deployment

Monitoring & Alerts


---

## Testing Strategies in CI/CD Pipelines

| Test Type | Purpose | Tools |
|---------|---------|-------|
| Unit Testing | Validate individual components | JUnit, pytest |
| Integration Testing | Verify interaction between modules | Postman, REST Assured |
| System Testing | End-to-end application validation | Selenium, Cypress |
| Load & Performance Testing | Assess system behavior under load | JMeter, Locust |
| Security Testing | Detect vulnerabilities | OWASP ZAP |
| SAST | Static source code security analysis | SonarQube |
| DAST | Runtime application security testing | OWASP ZAP |
| SCA | Third-party dependency scanning | Black Duck, JFrog Xray |

---

## Unit Testing and Mocking

### Unit Testing
Unit testing validates **individual units of code**, such as methods or classes, in isolation.

### Mocking Explained
Mocking is the practice of replacing dependent components with simulated implementations.

Example:
- While testing function `A`, dependent function `B` is mocked.
- Function `B` is assumed to work correctly.
- Testing focuses solely on function `A`.

### Popular Mocking Libraries
- Java: Mockito
- Python: unittest.mock
- JavaScript: Sinon.js

---

# Linux Fundamentals

## Evolution of Operating Systems

| Timeline | System | Overview |
|-------|--------|----------|
| 1969–1990 | UNIX | Multi-user OS developed at Bell Labs |
| 1985–1995 | Windows | GUI-based OS for personal computers |
| 1991 | Linux | Open-source kernel created by Linus Torvalds |

Linux is the backbone of modern cloud and server infrastructure.

---

## What is Linux?
Linux is an **open-source operating system kernel** that manages system resources and hardware interactions.

### Key Kernel Functions
- CPU process scheduling
- Memory allocation and management
- File system operations
- Device and I/O handling
- Security and access control

### Kernel Architectures
- **Monolithic Kernel:** Most services run in kernel space
- **Microkernel:** Minimal kernel with services running in user space

---

## Core Linux Components

| Component | Function |
|--------|----------|
| Shell | Command-line user interface |
| File System | Hierarchical directory structure starting from `/` |
| Process Management | Controls running processes |
| User Management | Manages permissions and access control |

---

# AWS IP Address Types

Amazon EC2 instances support different IP address categories:

| IP Address Type | Description |
|---------------|-------------|
| Private IP | Internal communication within a VPC |
| Public IP | Temporary internet-facing IP |
| Elastic IP | Static public IP assigned to AWS resources |

- **Private IPs** enable internal networking.
- **Public IPs** allow temporary internet access.
- **Elastic IPs** provide consistent external access.
