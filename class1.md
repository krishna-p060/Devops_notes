# Microservice Architecture — Notes

## Overview
**Microservice Architecture** is the practice of breaking down a **monolithic application** into multiple smaller, independent, and modular services.  
Each service focuses on a single business functionality and can be developed, deployed, and scaled independently.

---

## DNS (Domain Name System)
- DNS maps **domain names** (like `scaler.com`) to **IP addresses**.
- When a user enters a domain name in a browser:
  1. The request goes to the **DNS server** to resolve the IP.
  2. The resolved IP forwards the request to a **Load Balancer (LB)**.
  3. The LB routes the request to the appropriate **microservice** (e.g., Authentication, Account Service, etc.).

---

## Load Balancing

### **Purpose**
Distribute incoming requests efficiently among multiple instances of services to ensure:
- High availability  
- Scalability  
- Fault tolerance  

### **Types**
1. **Path-Based Routing**  
   - Routes requests based on the **URL path**.  
     Example:  
     ```
     /auth → Authentication Service  
     /accounts → Account Service
     ```
2. **Instance-Based Routing**  
   - Distributes requests among multiple **instances** of the same service.  
     Example:  
     ```
     /accounts → Account Service Instance 1, Instance 2, Instance 3
     ```

### **Service Discovery**
- Every Load Balancer must support **service discovery**.  
- It detects when:
  - A new service instance **comes up**, or  
  - An existing service instance **goes down**,  
  and updates its routing table automatically.

---

## Database Design Principle

### **Single Database Service Principle**
- Each **microservice** should have **its own database**.  
- This ensures **loose coupling**, **data isolation**, and **independent scalability**.  

---

## Three Axes of Scalability
 *Read and understand this topic in detail.*

The concept of scaling can be broken down along **three dimensions (axes)**:
1. **X-Axis** — Cloning (Horizontal Scaling)  
2. **Y-Axis** — Functional Decomposition (Microservices)  
3. **Z-Axis** — Data Partitioning (Sharding)

---

## Polyglot Programming
- **Polyglot** = A person who can speak multiple languages.  
- In **software**, it means using **multiple programming languages or technologies** across different services.  
  Example:  
  - Auth Service → Node.js  
  - Payment Service → Go  
  - Reporting Service → Python  

---

## Communication Between Microservices

### 1. **Synchronous Communication**
- The sender **waits for a response** before proceeding.  
- Example: REST API calls or RPC.  
- Analogy: Talking to someone **on a phone call**.

### 2. **Asynchronous Communication**
- The sender **does not wait** for a response.  
- Often used for **event-driven** or **background tasks**.  
- Analogy: Sending a **WhatsApp message**.

#### Common Models:
- **Pub/Sub Model** – Services publish events to a topic; subscribers listen to relevant events.  
- **Queue Model** – Requests are pushed into a queue and processed asynchronously.

---

## Logs & Observability
 *Read about **Loki Logs** — a log aggregation system for microservices.*

Loki is a **prometheus-style** logging system that helps in collecting, storing, and querying logs efficiently across distributed services.

---

## Twelve-Factor App Methodology
*Read this topic thoroughly.*

The **Twelve-Factor App** is a set of best practices for building **scalable, maintainable, and portable** microservices.  
It covers topics like:
- Config management  
- Dependency isolation  
- Build & release processes  
- Stateless processes  

---

## Stateless Microservices
 *Read: Why microservices should be stateless.*

- A **stateless service** doesn’t store session or user data locally.  
- All state-related data is stored in a **shared data store** (e.g., Redis, DB).  
- Benefits:
  - Easy horizontal scaling  
  - Fault tolerance  
  - Simpler load balancing  

---

## Types of Dependencies

1. **Service Dependency**  
   - One microservice depends on another service.  
   - Example: Payment Service depends on User Service.

2. **Component Dependency**  
   - Internal dependencies within a single service’s components.  
   - Example: Controller depends on Repository or DAO layer.

---

