# Wireshark: Packet Operations

---

## Overview

This room focuses on using Wireshark's Statistics menu, protocol filters, and advanced display filters to investigate network traffic captures. Instead of reading packets one by one, a SOC analyst learns to build a picture of network activity quickly and then drill into specific threats using targeted filters.

---

## Learning Objectives

- Investigate network traffic captures (PCAP files)
- View statistics: summary, protocol hierarchy, conversations, endpoints
- Apply packet filtering principles (capture vs. display filters)
- Apply protocol-specific filters (IP, TCP, UDP, HTTP, DNS)
- Apply advanced filters ("contains", "matches", "in", "upper()", "lower()", "string()")

---

## Part 1 : Statistics

### Why Statistics Matter

Large PCAP files can contain tens of thousands of packets. Before diving into individual packets, a SOC analyst uses the Statistics menu to quickly answer:

- Who communicated? → Conversations / Endpoints
- What protocols were used? → Protocol Hierarchy
- Which hosts generated the most traffic? → Endpoints
- Are there any unusual patterns? → Statistics overview

### Key Statistics Features

**Resolved Addresses** (Statistics → Resolved Addresses):

Displays IP-to-hostname mappings extracted from DNS responses in the PCAP. Only available if DNS traffic is present in the capture.

**Protocol Hierarchy** (Statistics → Protocol Hierarchy):

Shows all protocols present, their packet counts, and percentage of total traffic. Useful for detecting unusual protocol usage.

**Conversations** (Statistics → Conversations):

Shows communications between two endpoints (Ethernet, IPv4, IPv6, TCP, UDP views). Answers: "Who talked to whom?"

**Endpoints** (Statistics → Endpoints):

Shows all unique hosts in the capture. Answers: "Who is involved?"

With GeoIP (MaxMind database), can display geographic location and ASN information.


---

## Statistics : Lab Exercises (Exercise.pcapng)

**Q: What is the IP address of the hostname starting with "bbc"?**

Navigation: Statistics → Resolved Addresses → search "bbc"

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Packet_Operations/Captureb.PNG?raw=true">
</div>

Answer: 199.232.24.81

**Q: What is the number of IPv4 conversations?**

Navigation: Statistics → Conversations → IPv4 tab

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Packet_Operations/Capturec.PNG?raw=true">
</div> 

Answer: 435

**Q: How many bytes (k) were transferred from the "Micro-St" MAC address?**

Navigation: Statistics → Endpoints → Ethernet tab

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Packet_Operations/Captured.PNG?raw=true">
</div> 

Answer: 7474 k

**Q: What is the number of IP addresses linked with "Kansas City"?**

Navigation: Statistics → Endpoints → IPv4 tab (with GeoIP enabled)

Filter by City column

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Packet_Operations/Capturee.PNG?raw=true">
</div> 

Answer: 4

**Q: Which IP address is linked with "Blicnet" AS Organisation?**

Navigation: Statistics → Endpoints → IPv4 tab → AS Organization column

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Packet_Operations/Capturef.PNG?raw=true">
</div> 

Answer: 188.246.82.7

**Q: What is the most used IPv4 destination address?**

Navigation: Statistics → IPv4 Statistics → Destinations and Ports

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Packet_Operations/Capturej.PNG?raw=true">
</div> 

Answer: 10.100.1.33

**Q: What is the max service request-response time of the DNS packets?**

Navigation: Statistics → DNS → Service Stats → request-response time (secs) → Max val

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Packet_Operations/Capturek.PNG?raw=true">
</div> 

Answer: 0.467897

**Q: What is the number of HTTP Requests accomplished by "rad[.]msn[.]com"?**

Navigation: Statistics → HTTP → Load Distribution

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Packet_Operations/Capturel.PNG?raw=true">
</div> 

Answer: 39

---

## Part 2 : Packet Filtering

### Two Types of Filters

| Filter Type | When Applied | Effect |
|-------------|-------------|--------|
| Capture Filter | BEFORE capturing | Only saves matching packets; cannot change after start |
| Display Filter | AFTER capture | Hides non-matching packets; original data is preserved |


### Display Filter Syntax

**Comparison Operators:**

| Meaning | Symbol | Example |
|---------|--------|---------|
| Equal | == | ip.src == 10.0.0.1 |
| Not equal | != | ip.src != 10.0.0.1 |
| Greater than | > | ip.ttl > 100 |
| Less than | < | ip.ttl < 10 |
| Greater or equal | >= | ip.ttl >= 64 |
| Less or equal | <= | ip.ttl <= 128 |

**Logical Operators:**

| Meaning | Symbol | Example |
|---------|--------|---------|
| AND | && | ip.src == 1.1.1.1 && tcp.port == 80 |
| OR | \|\| | dns \|\| http |
| NOT | ! | !(ip.src == 1.1.1.1) |


**Filter Toolbar Color Codes:**

- Green → Valid filter
- Red → Invalid filter
- Yellow → Warning (works but not optimal)

### Protocol Filters:

**IP Filters (Network Layer):**


| Filter                     | Description                                       |
| -------------------------- | ------------------------------------------------- |
| `ip`                       | All IP traffic                                    |
| `ip.addr == 10.10.10.111`  | Traffic involving this IP (source or destination) |
| `ip.addr == 10.10.10.0/24` | All hosts in this subnet                          |
| `ip.src == 10.10.10.111`   | Only traffic from this IP                         |
| `ip.dst == 10.10.10.111`   | Only traffic to this IP                           |


**TCP / UDP Filters (Transport Layer):**

| Filter                | Description                        |
| --------------------- | ---------------------------------- |
| `tcp.port == 80`      | All TCP traffic on port 80         |
| `tcp.srcport == 1234` | Traffic originating from port 1234 |
| `tcp.dstport == 80`   | Traffic destined for port 80       |
| `udp.port == 53`      | DNS traffic                        |
| `udp.dstport == 5353` | mDNS traffic                       |



**Application Layer Filters:**

**HTTP:**

| Filter                          | Description                        |
| ------------------------------- | ---------------------------------- |
| `http`                          | All HTTP traffic                   |
| `http.request.method == "GET"`  | GET requests only                  |
| `http.request.method == "POST"` | POST requests only                 |
| `http.response.code == 200`     | Successful HTTP responses (200 OK) |



**DNS:**
| Filter                    | Description                      |
| ------------------------- | -------------------------------- |
| `dns`                     | All DNS traffic                  |
| `dns.flags.response == 0` | DNS queries only                 |
| `dns.flags.response == 1` | DNS responses only               |
| `dns.qry.type == 1`       | A record (IPv4) lookups          |
| `dns.a`                   | DNS packets containing A records |




**SOC Investigation Workflow Using Filters:**

Full traffic → protocol → IP → port → session → stream

1. Start with IP:      ip.addr == suspicious_ip
2. Narrow to protocol: dns or http
3. Refine:             http.request.method == "POST"
4. Analyze stream:     Follow TCP/HTTP stream


---

## Protocol Filters : Lab Exercises

**Q: What is the number of IP packets?**

Filter: ip

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Packet_Operations/Capturem.PNG?raw=true">
</div> 

Answer: 81420 (Answer visible in status bar as Displayed count)

**Q: What is the number of packets with TTL less than 10?**

Filter: ip.ttl < 10

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Packet_Operations/Capturen.PNG?raw=true">
</div> 

Answer: 66

**Q: What is the number of packets which use TCP port 4444?**

Filter: tcp.port == 4444

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Packet_Operations/Captureo.PNG?raw=true">
</div> 

Answer: 632

**Note:** Port 4444 is commonly associated with Metasploit reverse shells and C2 activity.

**Q: What is the number of HTTP GET requests sent to port 80?**

Filter: http.request.method == "GET" && tcp.port == 80

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Packet_Operations/Capturep.PNG?raw=true">
</div> 

Answer: 527

**Q: What is the number of type A DNS Queries?**

Navigation: Analyze → Display Filter Expression → expand DNS → select dns.a

Filter: dns.a

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Packet_Operations/Captureq.PNG?raw=true">
</div> 

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Packet_Operations/Capturer.PNG?raw=true">
</div> 

Answer: 51

---

## Part 3 : Advanced Filtering

### Advanced Filter Operators

Advanced filters go beyond exact matching they enable pattern-based hunting and anomaly detection.

**1. "contains" Operator**

Searches for a value inside a specific field. Case sensitive.

{http.server contains "Apache"}

To detect specific server technologies, identify malware infrastructure.

**2. "matches" Operator**

Searches using Regular Expressions (Regex). Case insensitive, more flexible than "contains".

{http.host matches "\.(php|html)"}

Tp detect suspicious file types, pattern-based URL hunting.

**3. "in" Operator**

Checks if a value exists within a defined set.

"tcp.port in {80 443 8080}"

To Filter multiple ports at once, detect web traffic variations.

**4. "upper()" Function**

Converts text to uppercase before filtering avoids case sensitivity issues.

{upper(http.server) contains "APACHE"}

**5. "lower()" Function**

Converts text to lowercase normalizes log data.

{lower(http.server) contains "apache"}

**6. "string()" Function**

Converts numeric fields into strings for regex matching.

{string(frame.number) matches "[13579]$"}

Meaning: Find frames with odd numbers.

{string(ip.ttl) matches "[02468]$"}

Meaning: Find packets with even TTL values.

---

## Advanced Filtering : Lab Exercises

**Q: Find all Microsoft IIS servers. How many packets did NOT originate from port 80?**

Filter: http.server contains "IIS" && tcp.srcport != 80

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Packet_Operations/Captures.PNG?raw=true">
</div> 

Answer: 21

**Q: Find all Microsoft IIS servers. How many packets have "version 7.5"?**

Filter: http.server contains "IIS" && http.server contains "7.5"

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Packet_Operations/Capturet.PNG?raw=true">
</div> 

Answer: 71

**Q: What is the total number of packets using ports 3333, 4444 or 9999?**

Filter: tcp.port == 3333 || tcp.port == 4444 || tcp.port == 9999

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Packet_Operations/Capturev.PNG?raw=true">
</div> 

Answer: 2235

**Q: What is the number of packets with even TTL numbers?**

Filter: string(ip.ttl) matches "[02468]$"

why this filter : 0, 2, 4, 6, 8 is even numbers and $ means "End of the string".This filter is a clever of regular expressions to identify even numbers by thier last digit.

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Packet_Operations/Capturew.PNG?raw=true">
</div> 

Answer: 77289

**Q: Change profile to "Checksum Control". How many "Bad TCP Checksum" packets exist?**

Navigation: Edit → Configuration Profiles → Checksum Control

Then: Analyze → Expert Info

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Packet_Operations/Capturewa.PNG?raw=true">
</div> 

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Packet_Operations/Capturewb.PNG?raw=true">
</div> 

Answer: 34185

**Q: Use the existing filtering button (gif/jpeg with http-200). How many packets are displayed?**

Click the pre-configured filter button: gif/jpeg with http-200

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Packet_Operations/Capturexc.PNG?raw=true">
</div> 

Answer: 261

---
