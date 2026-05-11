# Computer Networking Fundamentals

Welcome to my Networking study notes! This section documents my journey of understanding how computers communicate, how data travels across the globe, and how networks are secured.

## Introduction to Networking

### What is a Computer Network?
At its core, a **computer network** is a collection of interconnected devices such as computers, servers, routers, and switches that communicate with one another to share resources, exchange data, and facilitate services.

### Why Do We Need Networking?
Without networking, every computer would be an isolated island. Networking enables:
* **Resource Sharing:** Sharing printers, storage drives, and applications.
* **Information Exchange:** Sending emails, streaming videos, and browsing websites.
* **Centralized Management:** Allowing administrators to manage security and access from a single point.

### How Data Travels: The Physical Reality

Every piece of data we send (an email, a video, or a webpage) is broken down into its simplest digital form: **bits (0s and 1s)**.

To travel from one device to another, these bits must be converted into physical signals depending on the transmission medium:
* **Electrical Signals:** Sent as voltage fluctuations over copper cables (like Ethernet/UTP cables).
* **Light Waves:** Sent as pulses of light through fiber-optic cables.
* **Radio Waves:** Sent through the air for wireless communication (Wi-Fi, Cellular, Bluetooth).

### The Core Communication Model
For any network communication to happen, we always need three essential elements:
1. **The Sender (Source):** Converts digital data into physical signals and transmits them.
2. **The Medium (Channel):** The physical path (cable or air) through which the signal travels.
3. **The Receiver (Destination):** Captures the incoming signals, decodes them back into digital bits, and reconstructs the original data.

### How Data Finds Its Destination

To ensure data reaches the correct receiver without getting lost, computer networks rely on two critical types of addressing, functioning much like a physical postal system:

#### 1. MAC Address (The Physical Address)
* **What it is:** A unique, permanent hardware identifier burned into every network card NIC by the manufacturer for exampel `00:1A:2B:3C:4D:5E`.
* **Purpose:** Used for communication within the **same local network (LAN)**.
* **How it works:** An intelligent device called a **Switch** maintains a table of MAC addresses and forwards the electrical/optical signals only to the specific physical port where the receiver is connected.

#### 2. IP Address (The Logical Address)
* **What it is:** A dynamic logical address assigned to a device when it connects to a network for exampel `192.168.1.15` or `8.8.8.8`.
* **Purpose:** Used for routing data across **different networks (WAN / Internet)**.
* **How it works:** A device called a **Router** reads the destination IP address of the incoming packet and determines the best path to forward the data across the internet until it safely reaches the target network.
### Network Types & Topologies:
----
#### 1.Types of Computer Networks:

<div align="center">
  <p>A visual hierarchy of computer networks from personal devices to global infrastructures.</p>
  <img src="https://github.com/emmagh8/Assets/blob/main/Fundamentals/Networking/Types%20of%20computer%20network.png?raw=true" width="500" height="auto">
</div>


----
#### 2. Network Topologies (Physical Layout)
The choice of topology is critical because it determines how efficiently data moves and how the network handles a failure. Here is a breakdown of how those "lines and objects" are typically arranged:

----
<div style="overflow: hidden; margin-bottom: 20px;">
  <img src="https://github.com/emmagh8/Assets/blob/main/Fundamentals/Networking/Star_Topology.jpeg?raw=true" 
       align="right" 
       width="150" 
       style="margin-left: 20px; margin-bottom: 10px;">

  <h3 style="margin-top: 0;">Star Topology</h3>
  <ul>
    <li><b>Concept:</b> Every node is connected to a central hub/switch.</li>
    <li><b>Reliability:</b> High (single node failure doesn't affect others).</li>
    <li><b>Usage:</b> Standard for modern LANs.</li>
  </ul>
</div>

<img src="[YOUR_NEXT_IMAGE_URL](https://github.com/emmagh8/Assets/blob/main/Fundamentals/Networking/Star_Topology.jpeg?raw=true)" width="100%">

<hr>

<!-- MESH TOPOLOGY -->
<div style="overflow: hidden; margin-bottom: 25px;">
  <img src="https://github.com/emmagh8/Assets/blob/main/Fundamentals/Networking/Mesh_Topology.jpeg?raw=true" 
       align="right" 
       width="150" 
       style="margin-left: 20px; border-radius: 5px;">
  <h3>Mesh Topology</h3>
  <ul style="margin-top: 0;">
    <li><b>Concept:</b> Nodes are interconnected with multiple redundant paths.</li>
    <li><b>Reliability:</b> Extremely High (multiple paths available).</li>
    <li><b>Usage:</b> Basis for the Internet (WAN/GAN).</li>
  </ul>
</div>

<hr>

<!-- TREE TOPOLOGY -->
<div style="overflow: hidden; margin-bottom: 25px;">
  <img src="https://github.com/emmagh8/Assets/blob/main/Fundamentals/Networking/Tree_Topology.jpeg?raw=true" 
       align="right" 
       width="150" 
       style="margin-left: 20px; border-radius: 5px;">
  <h3>Tree Topology</h3>
  <ul style="margin-top: 0;">
    <li><b>Concept:</b> A hierarchical structure combining Star and Bus characteristics.</li>
    <li><b>Reliability:</b> High.</li>
    <li><b>Usage:</b> Large Corporate Sites and Campus Area Networks (CAN).</li>
  </ul>
</div>

<hr>

<!-- BUS TOPOLOGY -->
<div style="overflow: hidden; margin-bottom: 25px;">
  <img src="https://github.com/emmagh8/Assets/blob/main/Fundamentals/Networking/Bus_Topology.jpeg?raw=true" 
       align="right" 
       width="150" 
       style="margin-left: 20px; border-radius: 5px;">
  <h3>Bus Topology</h3>
  <ul style="margin-top: 0;">
    <li><b>Concept:</b> All nodes share a single communication line.</li>
    <li><b>Reliability:</b> Low (backbone failure stops the entire network).</li>
    <li><b>Usage:</b> Legacy systems or simple low-cost setups.</li>
  </ul>
</div>

---

To master networking from both a defensive (SOC) and offensive (Security Specialist) perspective, I am diving deep into the following core concepts:
* [The OSI Model & TCP/IP](./Fundamentals/Networking/OSI-Model-TCP-IP/README.md) 
* [IP Addressing & Subnetting](./Fundamentals/Networking/IP-Addressing-Subnetting/README.md)
* [Core Protocols](./Fundamentals/Networking/Core-Protocols/README.md)
* [Network Devices](./Fundamentals/Networking/Network-Devices/README.md) 
* [Traffic Analysis & Tools](./Fundamentals/Networking/Traffic-Analysis-Tools/README.md) 
