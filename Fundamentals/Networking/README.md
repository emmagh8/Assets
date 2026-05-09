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
#### 3. Network Types & Topologies.

----

To master networking from both a defensive (SOC) and offensive (Security Specialist) perspective, I am diving deep into the following core concepts:
* [The OSI Model & TCP/IP](./Fundamentals/Networking/OSI-Model-TCP-IP/README.md) 
* [IP Addressing & Subnetting](./Fundamentals/Networking/IP-Addressing-Subnetting/README.md)
* [Core Protocols](./Fundamentals/Networking/Core-Protocols/README.md)
* [Network Devices](./Fundamentals/Networking/Network-Devices/README.md) 
* [Traffic Analysis & Tools](./Fundamentals/Networking/Traffic-Analysis-Tools/README.md) 
