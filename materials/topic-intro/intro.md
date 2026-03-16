
# Introduction to Distributed Systems
+ **Felix García Carballeira and Alejandro Calderón Mateos** @ arcos.inf.uc3m.es
+ [![License: CC BY-NC-SA 4.0](https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-blue.svg)](https://github.com/acaldero/uc3m_sd/blob/main/LICENSE)


## Contents
 
1. Introduction
     * 1.2. [Basic Concepts](#11-basic-concepts)
     * 1.2. [Distributed System](#12-distributed-system)
     * 1.3. [Distributed Application](#13-distributed-application)
     * 1.4. [Evolution toward distributed systems](#14-evolution-toward-distributed-systems)
      
 2. Elements in distributed applications
     * 2.1  [Main characteristics](#21-main-characteristics)
     * 2.2 [Objectives of a distributed system](#22-objectives-of-a-distributed-system)
       * 2.2.1. [Transparency](#221-transparency)
       * 2.2.2. [Scalability](#222-scalability)
       * 2.2.3. [Consistency](#223-consistency)
       * 2.2.4. [Fault Tolerance](#224-fault-tolerance)
       * 2.2.5. [Security](#225-security)
     * 2.3  [Typical Elements](#23-typical-elements)
       * 2.3.1. [Communication mechanism](#231-communication-mechanisms)
       * 2.3.2. [Synchronization mechanism](#232-synchronization-mechanisms)
       * 2.3.3. [Concurrency support](#233-concurrency-support)
       * 2.3.4. [Common communication architecture models](#234-communication-architectures)


## 1. Introduction

### 1.1 Basic Concepts

* ***Connected host***: processor + memory + communication subsystem
     * **Communication subsystem**: set of hardware and software components that provide communication services in a distributed system 

* **Computer network**: set of *hosts* connected via a network 
   * ***Host***: a computer or other device that uses the network for communication purposes.

* **Protocol**: set of rules and instructions governing communication in a distributed system, i.e., the exchange of packets and messages.
  * **Message**: logical object exchanged between two or more processes 
     * Its size can be quite large 
     * A message is broken down into packets
  * **Packet**: a type of message exchanged between two communication devices 
     * Size limited by the hardware


### 1.2 Distributed System

* **Computer network**: 
   * A computer network is a set of computers connected by an interconnection network.
   * Common objective: sharing a printer or remote disk.
* **Distributed system**:
   * **System** consisting of **computing resources** (hardware and software) that are **physically distributed** and **interconnected** via a **network**, which communicate by exchanging messages and **cooperate** to perform a specific **task**.
     * Common objective: sharing resources and collaborating.
   * A set of independent computers that appear to users as a single coherent system.
* **Parallel system**:
   * **Distributed system** designed to perform a specific **task** as quickly as possible.
   * Common objective:
     * High performance (*High Performance Computing*)
     * High throughput (*High Throughput Computing*)


### 1.3 Distributed Application

 * **Monolithic (non-distributed) applications**:
    * A set of processes running on a single computer, which can collaborate and communicate via shared memory.
 * **Distributed applications**:
     * A set of processes running on one or more computers that collaborate and communicate by exchanging messages.
     * Some examples:
       * World Wide Web (WWW)
       * Email (IMAP, POP) 
       * File transfer (FTP)


### 1.4 Evolution toward distributed systems

<html>
<table border="1"><tr valign="top"><td>
</html>
 
**~1950: Centralized computing**
   * Main characteristics:
     - **Central mainframes**
     - **Time-sharing systems**
     - **Fully centralized resources**
     - **Simple terminals** connected to the mainframe
     - **Unfriendly interfaces**
  
* Technological context:
     - First **computer networks**
     - Users were completely dependent on the central system

<html></td><td></html>
 
**~1960s/70s:  Emergence of TCP/IP**
  - **1969:** creation of **ARPANET**
  - **1974:** design of the **TCP/IP** protocol
* Key contributors:
  - Vinton G. Cerf
  - Robert E. Kahn
* Impact:
  - Foundation of the Internet
  - Enables communication between heterogeneous networks

<html></td><td></html>

**~1980: 
Early distributed computing**
 
* Major changes:
   - Emergence of **PCs and workstations**
   - **Hardware standardization**
   - Growth in the number of computers
 * Characteristics:
   - Applications run **locally**
   - More user-friendly interfaces
   - Emergence of **local area networks (LAN)**
 * Use of networks:
   - Resource sharing
     - printers
     - file systems
 * First **distributed operating systems**:
    - Mach
    - Sprite
    - Chorus

<html></td><td></html>

 **~1990: 
Internet and the client-server model**

* Characteristics:
  - Growth of **client-server architectures**
  - Greater **decentralization**
  - Applications distributed between client and server
* Key event:
   - **1991: emergence of the Web**
   - Created by **Tim Berners-Lee at CERN**
* Consequences:
   - Explosion in Internet use
   - New applications:
* Examples:
   - E-commerce
   - Multimedia applications
   - Control systems
   - Medical applications
   - Distributed supercomputing

<html></td><td></html>

 **~now: 
Modern trends**
  - Massive distributed computing
  - Global networked systems
  - Cloud computing
  - Mobile systems
  - Internet of Things (IoT)

<html>
</td></tr></table>
</html>


### 1.5 Review of Computer Networks

* A computer network enables:
   - Communication between systems
   - Resource sharing
   - Data transfer
 
* Main **types** of networks:
   * **LAN (Local Area Network)**
      * Characteristics:
         - Small geographic area
         - High speed
         - Low latency
      * Examples:
         - Office networks
         - University networks
   * **WAN (Wide Area Network)**
      * Characteristics:
         - Large geographic area
         - Higher latency
         - Interconnection of multiple networks
      * Example:
         - Internet
 
* Most important **communication protocols**:
   * **TCP**
      - Reliable communication
      - Error control
      - Connection-oriented
   * **UDP**
      - Connectionless
      - Lower overhead
      - Higher speed


### 1.6 Review: Shared Memory vs. Distributed Memory

* **Shared-memory systems**
  * Characteristics:
     - All processors access **a common memory**
     - Communication via **memory read/write**
  * Advantages:
     - Simpler programming
     - Fast communication
  * Problems:
     - Limited scalability
     - Memory contention
     - High cost
  * Examples:
     - Multiprocessors
     - SMP systems

* **Distributed memory systems**
  * Characteristics:
     - Each node has **its own local memory**
     - Communication via **message passing**
  * Advantages:
     - Greater scalability
     - Better fault tolerance
     - Lower cost
  * Problems:
     - More complex programming
     - Communication cost
  * Examples:
     - Clusters
     - Distributed systems


## 2. Elements in Distributed Applications

### 2.1 Main Characteristics

* **Multiple autonomous components**
  * Sole means of communication and synchronization: message passing.
* **Concurrency**
   - Multiple processes running simultaneously.
* **Absence of a global clock**
   - There is no single clock to coordinate the actions of the programs running on the system.
* **Independent failures**
   - Each node can fail independently.
* More complex software


### 2.2 Objectives of a distributed system

Objectives and design aspects of a distributed system:
* <ins>Resource sharing</ins>
* <ins>Better performance</ins>
* <ins>Transparency</ins>: to appear as a single system as much as possible
* <ins>Scalability</ins>: ability to grow
* <ins>Fault tolerance</ins>: continuing to function in the presence of failures
* <ins>Security</ins>: secure access to resources
* <ins>Quality of Service (QoS)</ins>


#### 2.2.1 Transparency

* Transparency aims to make the distributed system **appear as a single system**.
* Main types:
   * **Access transparency**
     * Uniform access to local or remote resources.
   * **Location transparency**
     * The user does not need to know where the resource is located.
   * **Migration transparency**
     * Resources can change location without affecting the user.
   * **Replication transparency**
     * Multiple copies of resources without the user noticing.
   * **Concurrency transparency**
     * Multiple users access resources simultaneously.
   * **Fault transparency**
     * The system hides component failures.


#### 2.2.2 Scalability

* The system’s ability to scale in terms of:
   - Number of users
   - Resources
   - Nodes
* For a system with ``n`` users to be scalable,
   the amount of resources required to support it should be proportional to ``n`` or ``O(n)``.
* Types:
   - Size scalability
   - Geographic scalability
   - Administrative scalability
* Scalability techniques:
   * **Replication**
      * Data: multiple copies of the same data
      * Processes: multiple executions of the same computation
   * **Caching**
      * Local copies of recently or frequently used data
   * **Distribution**
      * Load balancing across multiple nodes
         * DNS, DHT
* Replication and caching techniques can introduce consistency and coherence issues


#### 2.2.3 Consistency

The consistency problem arises when multiple processes access and update data concurrently
* Coherence of updates
* Coherence of replication
* Coherence of caches
* Coherence in the face of failures
* Consistent clocks


#### 2.2.4 Fault tolerance

* The system must continue to function when:
   - A node fails
   - A network fails
   - A component fails
*  Techniques:
   - Replication
   - Recovery
   - Redundancy


#### 2.2.5 Security

 - **Authentication**:  verifies the user’s identity.
 - **Confidentiality**: ensures that only authorized users access the data.
 - **Integrity**: guarantees that data is not improperly modified.
 - **Access control**: defines and limits what each authenticated user can do.
 

### 2.3 Typical Elements

* <ins>Communication and synchronization</ins>
   * **Communication** mechanism: exchange of information between processes
   * **Synchronization** mechanism: events that control execution in a process dependent on an event
* <ins>Concurrency support</ins>:
     * Sequential: requests are executed one at a time
     * Concurrent: requests are executed simultaneously.
       * Concurrency model: master/workers, readers/writers, producer/consumer, etc.
* Software structure: <ins>distributed architecture model</ins>:
      * Client/server in about 80% of cases.
      * Publish/subscribe, Peer-to-peer,  etc.
* <ins>Heterogeneity</ins> of components
   * This refers to the variety and differences among the following components: networks, programming languages, etc.
   * Helper: use of open systems and standards
* <ins>Naming</ins>
    * Human notation -> machine notation
      * Users designate objects using a name (e.g., www.uc3m.es)
      * Programs designate objects using an identifier (e.g., 163.117.100.10)
      * Resolving a name involves obtaining the identifier from the name.
    * Important goal: names must be independent of their location.


#### 2.3.1 Communication Mechanisms

* **Types** of mechanisms:
    * B BasicB :
       * **Shared memory**
       * **Message passing**
    * B CollateralB :
      * Files
      * Email
      * Etc.

* **Paradigms** (models/patterns): using basic communication mechanisms, other communication mechanisms have been built that can be classified according to the communication paradigm used:
  * **Message passing**
     * Typical model in distributed systems.
     * Characteristics:
        - Explicit communication
        - Sending and receiving messages
     * Example:
        - Sockets
  * **Remote Procedure Call (RPC)**
     * Allows functions on another computer to be called as if they were local.
     * Advantages:
       - Simplifies programming
     * Problems:
        - Latency
        - Fault handling
  * **Message-oriented communication**
     * Intermediate systems are used:
        - Message queues
        - Middleware
     * Example:
        - Messaging systems


#### 2.3.2. Synchronization Mechanisms

* **Shared memory** mechanisms:
   * Semaphores 
   * Mutexes and condition variables 
* **Distributed memory** communication mechanisms also used for synchronization (**side-effect** mechanisms):
   * Pipes (FIFOs)
   * Message passing
* **Others**:
   * Signals (asynchrony)
   * Etc.


#### 2.3.3. Concurrency Support

* **Sequential** servers:
  ```mermaid
  graph LR
  A{Resource}
  B((Server))
  C[Client_1]
  D[Client_2]
  E[...]
  
  C <-- request/response --> B
  D <-- request/response --> B
  E <-- request/response --> B
  B --- A
  ```

* **Concurrent** servers:
  ```mermaid
  graph TD
  A@{ shape: cyl, label: "Resource" }
  B((Server))
  C@{ shape: procs, label: "Service thread"}
  D[Client_1]
  
  D <-- request --> B
  C <-- response --> D
  B --- A
  B <-- create service thread --> C
  ```

     * Most common **concurrency models**:
       * Producer/Consumer
       * Readers/Writers
       * Etc.


#### 2.3.4. Communication Architectures

* Most well-known architectural models in distributed applications:
   * **Client-Server**
     * Most common model.
     * Components:
        - **Client**: requests services
        - **Server**: provides services
     * Advantages:
        - Clear organization
        - Easy maintenance
     * Problems:
        - Potential bottleneck
        - Dependence on the server
   * **Peer-to-Peer (P2P)**
     * All nodes can act as:
        - Clients
        - Servers
     * Characteristics:
        - Decentralization
        - Greater scalability
     * Examples:
        - File-sharing networks
* Other models:
  * Publish/subscribe (Pub/Sub)
  * Etc.
* It is possible to use well-known models or various combinations of these models.

