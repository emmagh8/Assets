# Wireshark: The Basics

## Overview

This room covers network traffic analysis using Wireshark one of the most essential tools in a SOC analyst's toolkit. It moves beyond log-based detection and teaches how to capture, filter, and inspect raw network packets to uncover threats that logs alone cannot reveal. The room covers traffic observation across all TCP/IP layers, PCAP file analysis, display filtering techniques, and stream reconstruction.

What I learned:

- Why network traffic analysis goes deeper than log monitoring
- How to observe traffic at every TCP/IP layer and what each layer reveals
- The difference between North-South and East-West traffic flows
- How to navigate and use Wireshark's GUI effectively
- How to apply capture filters vs. display filters and when to use each
- Practical packet investigation techniques: marking, commenting, exporting, and stream following
- How to use Expert Info for quick anomaly triage

---

## 1. Why Network Traffic Analysis?

Logs are the foundation of SOC monitoring but they are not the whole picture. Log entries typically record metadata: source IP, destination IP, domain, timestamp, and query type. They do not capture the **full content** of a communication.

Network traffic analysis fills that gap by inspecting the actual payload of every packet crossing the wire.

### The Limits of Logs

| What Logs Show | What Logs Miss |
|---|---|
| Source / Destination IP | Full packet payload |
| Domain queried | Data transferred inside the session |
| Timestamp | Whether a file was downloaded |
| Query type | C2 commands embedded in DNS responses |

### What Traffic Analysis Enables

| Capability | Description |
|---|---|
| Detect malicious activity | Identify suspicious connections in real time |
| Validate alerts | Confirm or dismiss SIEM alerts with packet-level evidence |
| Investigate incidents | Reconstruct exactly what happened and when |
| Reconstruct attacks | Build a complete attack timeline from raw packets |
| Detect DNS tunneling | Spot data exfiltration hidden inside DNS queries |
| Detect beaconing | Identify periodic C2 check-ins from compromised hosts |
| Extract malicious files | Pull transferred files directly out of PCAP captures |

### Key Concepts: DNS Tunneling & Beaconing

**DNS Tunneling:** Attackers abuse the DNS protocol to smuggle data or command-and-control (C2) instructions through DNS queries and responses, bypassing firewalls that only inspect web traffic.

**Beaconing:** A compromised host periodically contacts a C2 server at regular intervals to receive instructions. The regularity of these connections is the key detection signal.

If a host is making DNS queries every 60 seconds like clockwork, that regularity is the anomaly — not the protocol itself.

---

## 2. What Traffic Can We Observe?

Wireshark captures headers and payloads across **all four TCP/IP layers**, enabling analysts to detect attack techniques that are completely invisible in log data.

### TCP/IP Layers: What Each Reveals

| Layer | What We Observe | SOC Use Cases |
|---|---|---|
| **Application** | HTTP requests/responses, DNS queries/replies, email traffic, application payloads | Malware downloads, DNS tunneling, data exfiltration |
| **Transport** | TCP/UDP headers, source/destination ports, TCP flags, sequence & acknowledgment numbers | Session hijacking detection, connection analysis |
| **Internet** | Source/destination IPs, TTL values, fragment offsets, packet fragmentation | Fragmentation attack detection, routing analysis |
| **Link** | MAC addresses, ARP traffic | ARP poisoning detection, MAC spoofing detection, MitM attacks |

### TLS Note

**TLS (Transport Layer Security)** encrypts HTTPS communications at the Transport layer. Encrypted sessions will show as opaque in Wireshark unless the session keys are available for decryption.

---

## 3. Network Traffic Sources and Flows

Understanding where traffic comes from and how it moves through the network is essential for placing Wireshark in the right location to capture relevant data.

### Traffic Sources

**Endpoint Devices:** Generate the majority of network traffic.

Examples: workstations, servers, mobile devices, printers, IoT devices, cloud resources.

**Intermediary Devices:** Forward and manage traffic between endpoints.

Examples: firewalls, routers, switches, IDS/IPS, proxies, access points.

### Network Flows

#### North-South Traffic
Traffic entering or leaving the network perimeter.

- **Examples:** HTTPS, DNS, SSH, VPN, SMTP, RDP
- **Characteristics:** Crosses the firewall; easier to monitor at the perimeter

#### East-West Traffic
Traffic moving laterally inside the network between internal hosts.

- **Examples:** SMB, Active Directory, Kerberos, internal applications
- **Characteristics:** Critical for detecting lateral movement; often under-monitored

 East-West traffic is where attackers move after initial compromise. If your monitoring only covers North-South, you will miss the lateral phase of an attack entirely.

#### SMB and Kerberos Authentication Flow

Internal Windows environments rely on Kerberos for authentication before SMB sessions are established:

```
User → Kerberos (authentication) → Service Ticket issued → SMB Access granted
```

Anomalies in Kerberos traffic ( excessive ticket requests, unusual service names) are key indicators of credential-based attacks such as Pass-the-Ticket or Kerberoasting.

---

## 4. How We Observe Network Traffic

Three complementary methods exist for observing network traffic, each with different levels of visibility and storage requirements.

| Method | What It Captures | Limitations |
|---|---|---|
| **Logs** | Event records (authentication, web, firewall) | No full packet content; vendor-specific format |
| **Full Packet Capture (PCAP)** | Complete packet headers and payloads→  show full truth | High storage requirements |
| **Network Flow Data** | Traffic metadata (IPs, ports, bytes, timing) | No payload; pattern-level visibility only |

### Full Packet Capture (PCAP)

PCAP captures every packet on the wire — headers and full payload.

**Collection methods:**
- **Network TAP:** Passive hardware device; zero impact on network performance
- **Port Mirroring (SPAN):** Switch-level copy of traffic to a monitoring port

**Tools:** Wireshark, tcpdump, Suricata, Zeek

### Network Flow Data

Provides traffic metadata without storing full packet content. Useful for detecting patterns at scale.

**Protocols:** NetFlow (Cisco), IPFIX (vendor-neutral)

**Data includes:** Source/destination IPs, ports, byte counts, timing information

**Use cases:** Detecting C2 communication patterns, identifying data exfiltration, detecting lateral movement

---

## 5. Wireshark: Interface & Core Features

Wireshark is a network packet analyzer used to capture and inspect traffic in real time or from saved PCAP files, it analyzes packets passively, it does not block traffic or generate alerts automatically.

### GUI Components

| Component | Function |
|---|---|
| **Toolbar** | Start, stop, and restart live capture; open and save PCAP files |
| **Display Filter Bar** | Apply filters to show only relevant packets |
| **Packet List Pane** | Summary of all captured packets (source, destination, protocol, info) |
| **Packet Details Pane** | Protocol-layer breakdown of the selected packet: "Ethernet → IP → TCP → Application" |
| **Packet Bytes Pane** | Raw packet data displayed in hex and ASCII |

### Three Levels of Packet Inspection

Packet List    →  Overview (what is there)
Packet Details →  Structure (how it is built)
Packet Bytes   →  Raw data (what it actually contains)

### Packet Coloring

Wireshark uses a color-coded system to visually distinguish traffic types and highlight anomalies (TCP, HTTP, DNS, errors...). Colors allow analysts to spot suspicious traffic at a glance without reading every packet individually.

### Investigation Features

| Feature | Description | SOC Relevance |
|---|---|---|
| **Packet Numbers** | Each packet has a unique sequential ID | Navigation and investigation tracking in large captures |
| **Find Packets** | Search by string, regex, hex, or display filter across List / Details / Bytes panes | Locate specific credentials, domains, or file signatures |
| **Mark Packets** | Flag important packets (turns black); temporary lost when file is closed | Quick highlighting during live investigation |
| **Packet Comments** | Persistent notes attached to individual packets inside the PCAP file | SOC documentation and evidence annotation |
| **Export Packets** | Extract a subset of packets to a new PCAP file | Sharing specific evidence with incident responders |
| **Export Objects** | Extract transferred files from protocol sessions (HTTP, SMB, FTP, TFTP, DICOM) | Recovering malware samples from captures |
| **Time Display Format** | Switch from relative time to UTC | Attack timeline reconstruction and log correlation |
| **Expert Info** | Wireshark auto-flags anomalies by severity | First-pass triage for protocol errors and suspicious behavior |


---

## 6. Packet Filtering

Filtering is one of the most critical skills in Wireshark analysis. A raw packet capture during an active incident can contain millions of packets. Without filtering, finding the relevant traffic is like searching for a needle in a haystack.

### Two Types of Filters

#### Capture Filters
Applied **before** capture begins. Determines what gets recorded.

- Reduces the volume of data written to disk
- Must be defined before starting the capture session
- **Risk:** An overly narrow filter can cause you to miss critical evidence use with caution

#### Display Filters
Applied **after** capture. Hides non-matching packets without deleting them from the file.

- Does not remove any data from the capture
- Can be modified freely during analysis
- The primary filtering method in SOC investigations

### Advanced Filtering Techniques

| Technique | How It Works | Use Case |
|---|---|---|
| **Apply as Filter** | Right-click any field → Apply as Filter | Instantly filter by IP, port, or protocol with one click |
| **Conversation Filter** | Shows only traffic between two specific endpoints | Analyze a full client-server session in isolation |
| **Colourise Conversation** | Highlights endpoint pair without hiding other traffic | Visual session tracking without losing context |
| **Prepare as Filter** | Builds a filter expression without applying it immediately | Review and edit complex filters before execution |
| **Apply as Column** | Adds any packet field as a visible column | Compare source ports, hostnames, or TTLs across packets |
| **Follow Stream** | Reconstructs the full application-layer conversation | Read full HTTP exchanges, detect cleartext credentials, analyze C2 sessions |


### Common Display Filter Syntax

**By Protocol:** http, dns, ftp, ssh

**By Port:** tcp.port == 80, udp.port == 53, tcp.port == 443

**By IP Address:** ip.addr == 192.168.1.10

---

## 7. Key Takeaways

**Logs tell what happened. Packets show everything.** Network traffic analysis is the difference between knowing an alert fired and understanding exactly what an attacker did, downloaded, or exfiltrated.

- **Start with traffic context.** Understand whether we are looking at North-South or East-West traffic before placing our capture point location determines visibility.
- **PCAP is the ground truth.** Logs and flow data are summaries. When the investigation requires certainty, PCAP delivers the full picture.
- **Follow Stream is our most powerful tool.** It reconstructs human-readable conversations from raw packets and can expose cleartext credentials, C2 communications, and complete file transfers in seconds.
- **Export Objects to recover evidence.** Files transferred over HTTP, SMB, or FTP can be extracted directly from the PCAP and submitted to sandbox or threat intelligence platforms.
- **Expert Info is a triage accelerator, not a verdict.** Use it to surface anomalies quickly, then validate manually before escalating.
- **Document inside the PCAP.** Packet comments persist with the file annotate key packets so our findings are preserved when we share the capture with our team.

