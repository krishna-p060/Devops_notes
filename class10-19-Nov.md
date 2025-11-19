# Docker Networking & Core Networking Concepts: Structured Notes

## Part 1: Core Networking Fundamentals

Before diving into Docker, it is essential to understand how physical and logical networking works.

### 1. The Basics of Connectivity

* **IP Address (Internet Protocol):** A unique, routable label assigned to every device (e.g., `192.168.10.24`). Think of it as the "mailing address" of a computer.
* **NIC (Network Interface Card):** The physical hardware connecting a computer to the network (converts digital data  electrical/wireless signals).
* **Ethernet Cable:** The physical wire carrying data.
* **Cat5e:** 1 Gbps standard.
* **Cat6/6a:** 10 Gbps standard.



### 2. Network Switch (Layer 2)

A switch connects devices within a Local Area Network (LAN).

* **Intelligence:** Unlike a "hub," a switch is smart. It maintains a **MAC Address Table**.
* **Flow:** When Data arrives  Switch checks Destination MAC  Forwards *only* to the specific port (reducing traffic).

### 3. ARP (Address Resolution Protocol)

Computers know IP addresses (Logical), but switches need MAC addresses (Physical).

* **The Problem:** "I need to send data to IP `192.168.1.20`, but I don't know which physical device that is."
* **The Solution (ARP):** The sender broadcasts "Who owns `192.168.1.20`?". The target replies "I do, here is my MAC address."

### 4. Gateway & NAT

* **Default Gateway:** The "Exit Door" of the network. If a packet's destination is outside the local network (e.g., `8.8.8.8`), it is sent here (usually the Router).
* **NAT (Network Address Translation):**
* **SNAT (Source NAT):** Masks private IPs when going out to the internet (all devices appear as one public IP).
* **DNAT (Destination NAT):** Forwards incoming traffic from a public IP/Port to a specific internal IP/Port (Port Forwarding).



---

## Part 2: Subnetting Simplified

Subnetting is the practice of dividing a large network into smaller, manageable sub-networks.

### CIDR Notation Table

The `/XX` number indicates how many bits are fixed for the network. The remaining bits are for hosts.

| CIDR | Total IPs | Usable IPs (minus Net/Broadcast) |
| --- | --- | --- |
| **/24** | 256 | 254 |
| **/25** | 128 | 126 |
| **/26** | 64 | 62 |
| **/27** | 32 | 30 |
| **/28** | 16 | 14 |
| **/29** | 8 | 6 |
| **/30** | 4 | 2 (Point-to-Point links) |
| **/32** | 1 | 1 (Single Host) |

### Practical Example: Splitting a /24

**Scenario:** You have `192.168.10.0/24` (256 IPs) and need 3 separate networks.
**Solution:** Split into four **/26** subnets (64 IPs each).

1. **Engineering:** `192.168.10.0/26` (Range: .0 - .63)
2. **HR:** `192.168.10.64/26` (Range: .64 - .127)
3. **Reception:** `192.168.10.128/26` (Range: .128 - .191)
4. *(Unused/Spare)*

---

## Part 3: Docker Networking Architecture

Docker replicates physical networking using **Software Defined Networking (SDN)** features in the Linux Kernel.

### Key Components

1. **Network Namespaces:** Provides isolation. Each container gets its own "virtual" network stack (IP, Routing Table, ARP Cache) so it feels like an independent server.
2. **veth Pairs (Virtual Ethernet):** A "virtual cable." One end sits inside the container (`eth0`), and the other plugs into the Docker Bridge on the host.
3. **docker0 Bridge:** A virtual switch created by Docker. All containers on the default bridge plug into this. It usually holds the IP `172.17.0.1`.

---

## Part 4: Docker Network Drivers (Modes)

| Mode | Command Flag | Description |
| --- | --- | --- |
| **Bridge** (Default) | `--network bridge` | Containers connect to a private virtual bridge. They can talk to each other but are isolated from the host LAN. |
| **Host** | `--network host` | Removes isolation. The container shares the **Host's IP and Ports** directly. High performance, but port conflicts are possible. |
| **None** | `--network none` | The container has a loopback interface only. No internet, no LAN. Good for secure, offline batch jobs. |
| **Overlay** | `-d overlay` | Used in **Docker Swarm**. Creates a flat network across multiple physical servers. |

### Why User-Defined Bridges are Better

The default `docker0` bridge is limited. Creating your own bridge enables **Service Discovery** (DNS resolution by container name).

```bash
# 1. Create a custom network
docker network create my-app-net

# 2. Run containers (they can now ping "db" or "web")
docker run -d --network my-app-net --name db mysql
docker run -d --network my-app-net --name web nginx

```

---

## Part 5: Connectivity & Port Mapping

### Outbound Access (Container  Internet)

* **Mechanism:** Docker uses `iptables` rules.
* **Process:** Traffic leaves the container  Hits `docker0` bridge  Host performs **Masquerading (NAT)**  Traffic exits Host NIC to Internet.

### Inbound Access (Internet  Container)

* **Mechanism:** Port Mapping (Publishing).
* **Command:** `docker run -p 8080:80 nginx`
* **Logic:** Docker creates a **DNAT** rule. Any traffic hitting the Host on port `8080` is forwarded to the Container on port `80`.

---

## Part 6: Diagnostic Commands

```bash
# List all networks
docker network ls

# View details (IP ranges, connected containers)
docker network inspect bridge

# Check a specific container's IP/Mac
docker inspect <container_id>

```
