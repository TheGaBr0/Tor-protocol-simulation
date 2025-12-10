# Tor Network Simulator

This project implements a modular and extensible simulator of the Tor network.  
It reproduces the core logic of onion routing, multi-hop circuit formation, relay cell forwarding, and several categories of attacks that target anonymity and network integrity.

Communication occurs locally through **TCP sockets**.  
Each node, client, and server is assigned a **symbolic IP address**, managed by an internal oracle, which is used for logging, circuit analysis, and attack evaluation.

An interactive interface is included:  
each node, client, and server exposes a **terminal console** where commands can be executed directly.  
The interface also displays circuits visually using **arrows between nodes**, along with symbolic IPs and compromised-node indicators.

---

## Main Components

- **Directory Server** – registers nodes and distributes descriptors.
- **Client** – constructs circuits of three or more hops and forwards traffic.
- **Tor Nodes** – operate as Guard, Middle, or Exit relays with persistent socket connections.
- **External Server** – destination for traffic exiting the network.
- **Graphical Interface** – visualizes node topology, symbolic IPs, and circuits.

All entities run locally and communicate over socket pairs such as `(127.0.0.1, port)`.

---

## Circuit Construction and Node Selection

The client selects nodes according to a weighted and constrained mechanism inspired by Tor’s consensus.

### Weighted Selection Criteria

**1. Bandwidth (99.9% of the weight)**  
- Normal nodes: bandwidth in the range **0–3**  
- Compromised nodes: bandwidth fixed at **3**

Higher bandwidth yields higher selection probability.

**2. Uptime (0.1% of the weight)**  
- Normal nodes: **10–100**  
- Compromised nodes: **80–100**

Uptime provides a mild bias toward stable nodes.

---

### Diversity Constraints

Node selection enforces multiple diversity rules:

- **Owner diversity**  
  - No two nodes in a circuit may belong to the same owner.  
  - Normal nodes come from a pool of **5 owners**.  
  - Compromised nodes come from **40 owners**.

- **Subnet diversity**  
  - Nodes must belong to **different /16 subnets**, preventing grouping of hops under the same hosting provider or network segment.

---

### Circuit Axioms

- Guard nodes are **reused** when already established.  
- Middle and Exit nodes must respect all weighting and diversity constraints.  
- Circuit length is **extendible**, allowing more than three hops.

---

## Implemented Attacks

Multiple attack vectors are implemented, covering both active and passive strategies.

### Correlation Attacks
- Timing correlation  
- Traffic volume matching  
- Statistical inference  
- Guard–Exit flow comparison  

Compromised nodes log timestamps, packet metadata, and symbolic IP paths.

### DoS Attacks
- Circuit flooding via continuous `CREATE` requests  
- Resource exhaustion targeting routing tables or socket pools  

### Redirection Attacks
- Exit-node redirection to attacker-controlled destinations  

### Node Compromise
- Full plaintext inspection after onion-layer removal  
- Manipulation of routing behavior  
- Logging of circuit metadata  

### Node Descriptor Falsification
- Bandwidth inflation  
- Fake uptime reports  
- Manipulation of identity attributes to increase selection probability  

---

## Project Structure

