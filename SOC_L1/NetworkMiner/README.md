# NetworkMiner 

---

## Overview

NetworkMiner is a **Network Forensics Analysis Tool (NFAT)** designed for passive traffic analysis. Unlike Wireshark (which focuses on deep packet inspection), NetworkMiner is built for rapid triage, extracting hosts, files, credentials, and communications from PCAP files without requiring deep manual filtering.

**Key distinction:**  
NetworkMiner = Quick overview + data extraction  
Wireshark = In-depth packet analysis

---

## Network Forensics Data Types

There are three main data types investigated in Network Forensics:

1.Live Traffic: Real-time sniffing of network packets.

2.Traffic Captures: Pre-recorded PCAP files for forensic analysis.

3.Log Files: System and application event logs.

---

## NetworkMiner Capabilities

* Traffic Sniffing: Intercept, sniff, and log packets passing through the network.
* Parsing PCAP Files: Parse and display packet content in detail.
* Protocol Analysis: Identify protocols used in the capture.
* OS Fingerprinting: Identify host operating systems using Satori and p0f.
* File Extraction: Extract images, HTML files, and emails from PCAPs.
* Credential Grabbing: Extract credentials and password hashes.
* Clear Text Keyword Parsing: Extract cleartext keywords and strings.

---

## Operating Modes

### 1. Sniffer Mode
- Available **only on Windows**
- Not recommended as a primary sniffer (less reliable than Wireshark)
- Intended as a supplementary feature, not the primary use case

### 2. Packet Parsing / Processing Mode 
- Parses existing traffic captures (PCAP files)
- Provides a quick high-level overview
- Best used for **"low-hanging fruit"** identification before deeper investigation

---

## Pros and Cons

| Pros | Cons |
|---|---|
| OS fingerprinting | Not reliable for active sniffing |
| Easy file extraction | Not suited for large-scale investigations |
| Credential grabbing | Limited filtering capabilities |
| Clear text keyword parsing | Not built for manual traffic investigation |
| Quick overall overview | |

---

## Tool Interface Overview

### Menus

**File Menu:** Load PCAP files (`Ctrl+O`) or receive PCAP over IP (`Ctrl+R`). Drag-and-drop is also supported.

**Tools Menu:** Clear the dashboard (`Ctrl+N`) or delete captured data (`Ctrl+Delete`).

**Help Menu:** Check for updates and view the current version.

**Case Panel:** Lists all loaded PCAP files. Right-click a file to view metadata, reload, or remove it from the case.

---

### Main Tabs

#### Hosts Tab:

Displays all identified hosts in the PCAP with:
- IP and MAC address
- OS type (via OS fingerprinting)
- Open TCP ports
- Sent/Received packet counts
- Incoming/Outgoing sessions
- Host details (web server banners, domain names, NTLM usernames)

---

#### Sessions Tab:

Shows detected sessions with:

- Frame number, client/server addresses, ports, protocol, start time

Supports four filter input types: ExactPhrase, AllWords, AnyWord, RegExe

---

#### DNS Tab:

Shows DNS queries with full details:

- Frame number, timestamp, client/server, ports, IP, TTL, DNS time
  
- Transaction ID, query type, DNS query and answer

---

#### Credentials Tab:

Extracts credentials and password hashes. Captured types include:
- Kerberos hashes
- NTLM / NETNTLMv2 hashes
- RDP cookies
- HTTP cookies and requests
- IMAP, FTP, SMTP, MS SQL credentials

Extracted hashes can be cracked with **Hashcat** or **John the Ripper**.

---

#### Files Tab:

Extracted files from the PCAP with full metadata:
- Frame number, filename, extension, size
- Source/destination address and port
- Protocol, timestamp, reconstructed file path

---

#### Images Tab:

Extracted images from the PCAP. Hovering over an image shows source/destination address and file path. Supports zoom in/out.

---

#### Parameters Tab:

Extracted HTTP parameters and other protocol parameters:
- Parameter name and value
- Frame number, source/destination host and port, timestamp

---

#### Keywords Tab:

Extracted cleartext keywords with context. To filter:
1. Add target keywords
2. Reload case files

---

#### Messages Tab:

Extracted emails, chats, and messages:
- Frame number, source/destination host, protocol
- Sender (From), receiver (To), timestamp, size

---

#### Anomalies Tab:

Detected anomalies in the processed PCAP. NetworkMiner includes basic detections for:
- **EternalBlue** exploit activity
- **Spoofing** attempts
- **TLS boundary errors**

---

## Lab Exercises

### Tool Overview: mx-1.pcap + mx-2.pcap

**Q1: What is the total number of frames?**  
Navigation: Case Panel → right-click filename → Show Metadata → Frames field  

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/NetworkMiner/Capture1%20-%20Copy.PNG?raw=true">
</div> 

**Answer:** 460

---

**Q2: How many IP addresses use the same MAC address as host 145.253.2.203?**  

Navigation: Hosts tab → expand host 145.253.2.203 → check MAC: FEFF20000100  

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/NetworkMiner/Capture2%20-%20Copy.PNG?raw=true">
</div> 

**Answer:** 2, (65.208.228.223 and 216.239.59.99 share the same MAC)

Multiple IPs sharing one MAC address can indicate ARP spoofing or a NAT/proxy device. Worth investigating further.

---

**Q3: How many packets were sent from host 65.208.228.223?**  

Navigation: Hosts tab → expand 65.208.228.223 → Sent packets  

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/NetworkMiner/Capture3(1).PNG?raw=true">
</div> 

**Answer:** 72

---

**Q4: What is the web server banner under host 65.208.228.223?**  

Navigation: Hosts tab → expand 65.208.228.223 → Host Details  

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/NetworkMiner/Capture4(3).PNG?raw=true">
</div> 

**Answer:** Apache

---

**Q5: What is the extracted username for the 02694W-WIN10 host?**  

Navigation: Hosts tab → expand 172.16.66.37 [02694W-WIN10] → Host Details  

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/NetworkMiner/Capture6%20-%20Copy.PNG?raw=true">
</div> 

**Answer:** #B\Administrator

---

**Q6: What is the extracted password for the user logged into 02694W-WIN10?**  

Navigation: Credentials tab → find 02694W-WIN10 entry  

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/NetworkMiner/Capture6%20-%20Copy.PNG?raw=true">
</div> 

**Answer:** NETNTLMv2 — #B$136B077D942D9A63$FBFF3C253926907AAAAD670A9037F2A5$01010000000000000094D71AE38CD60170A8D57112

NETNTLMv2 hashes can be cracked offline using Hashcat with a wordlist. Even if not plaintext, capturing this hash means an attacker could attempt credential cracking.

---

**Q7: What is the name of the Linux distro mentioned in the file associated with frame 63602?**  

Navigation: Files tab → search frame 63602 → open file  

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/NetworkMiner/Capture7(4).PNG?raw=true">
</div> 

**Answer:** CentOS

---

**Q8: What name and surname are mentioned in frame 76469?**  

Navigation: Files tab → find frame 76469 → open reconstructed HTML file  

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/NetworkMiner/Capture8(3).PNG?raw=true">
</div> 

**Answer:** Ned Flanders

---

**Q9: What is the source address of the image "ads.bmp.2E5F0FD9[1].bmp"?**  

Navigation: Files tab → search `2E5F0FD9` in filter → check Source host  

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/NetworkMiner/Capture9(4).PNG?raw=true">
</div> 

**Answer:** 80.239.178.187

---

**Q10: What is the frame number of the possible TLS anomaly?**  

Navigation: Anomalies tab  


<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/NetworkMiner/Capture10(3).PNG?raw=true">
</div> 

**Answer:** 36255

---

**Q11: Which platform sent an email with subject starting with "You have more..."?**  

Navigation: Messages tab → filter "You have more"

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/NetworkMiner/Capture11(1).PNG?raw=true">
</div>

**Answer:** Facebook

---

**Q12: What is the email address of Branson Matheson?**  

Navigation: Messages tab → filter "Branson Matheson"

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/NetworkMiner/Capture12(1).PNG?raw=true">
</div>

**Answer:** branson@sandsite.org

---

### Case 1: case1.pcap

**Q1: What is the full OS name of host 131.151.37.122?**  

Navigation: Hosts tab → expand 131.151.37.122 → OS section  

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/NetworkMiner/Capture13(3).PNG?raw=true">
</div>

**Answer:** Windows - Windows NT 4

NetworkMiner uses p0f TCP fingerprinting to identify the OS. Here it detected Windows NT 4.0 SP1+ with 100% confidence via Satori, and 50% each for Windows NT 4.0 / Windows 98 via TCP analysis.

---

**Q2: How many bytes were sent by the client (131.151.32.91) through port 1065?**  

Navigation: Hosts tab → expand 131.151.37.122 → Incoming sessions → find TCP 1065 session  

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/NetworkMiner/Capture14(2).PNG?raw=true">
</div>

**Answer:** 192

---

**Q3: How many bytes were sent back by the server (131.151.37.122) through port 143?**  

Navigation: Hosts tab → Sessions → find TCP 143 (IMAP) session between the two hosts  

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/NetworkMiner/Capture15(1).PNG?raw=true">
</div>

**Answer:** 20769

---

**Q4: What is the sequence number of frame 9?**  

Versions above 1.6 do not display frame-level details. I used **NetworkMiner 1.6.1** for this question.

Navigation: Frames tab (v1.6.1) → expand Frame 9 → TCP → Sequence Number  

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/NetworkMiner/Capture16(3).PNG?raw=true">
</div>

**Answer:** 2AD77400

---

**Q5: What is the number of detected content types?**  

Navigation: Parameters tab → filter keyword `content-type` → select "Parameter name" column  

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/NetworkMiner/Capture17(3).PNG?raw=true">
</div>

**Answer:** 2 (multipart/mixed and text/plain)

---

### Case 2: case2.pcap

**Q1: What is the USB product's brand name?**  

Navigation: Parameters tab → filter `usb` → check parameter values  

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/NetworkMiner/Capture18(1).PNG?raw=true">
</div> 

**Answer:** asix

---

**Q2: What is the name of the phone model?**  

Navigation: Images tab → scroll to find phone selfie image → check filename  


<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/NetworkMiner/Capture19(2).PNG?raw=true">
</div> 

**Answer:** Lumia 535

---

**Q3: What is the source IP of the fish image?**  

Navigation: Files tab → filter `fish` → filter by Filename column → check Source host  

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/NetworkMiner/Capture20(1).PNG?raw=true">
</div> 

**Answer:** 50.22.95.9

---

**Q4: What is the password of "homer.pwned.se@gmx.com"?**  

Navigation: Credentials tab → find the homer.pwned.se@gmx.com entry  

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/NetworkMiner/Capture21(1).PNG?raw=true">
</div> 

**Answer:** spring2015

---

**Q5: What is the DNS query of frame 62001?**  

Navigation: DNS tab → filter keyword `62001`  

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/NetworkMiner/Capture22.PNG?raw=true">
</div> 

**Answer:** pop.gmx.com

---

## Key Takeaways

### When to Use Each (NetworkMiner vs Wireshark)

| Scenario | Best Tool |
|---|---|
| Quick host discovery | NetworkMiner |
| Extracting files from PCAP | NetworkMiner |
| Grabbing credentials / hashes | NetworkMiner |
| OS fingerprinting | NetworkMiner |
| Deep packet analysis | Wireshark |
| Protocol-level filtering | Wireshark |
| Detecting DNS tunneling | Wireshark |
| Stream reconstruction | Wireshark |

### SOC Investigation Workflow

1. Load PCAP into NetworkMiner
2. Check Hosts tab → identify all systems, OS types, open ports
3. Check Credentials tab → look for exposed hashes or cleartext passwords
4. Check Files/Images tab → extract and inspect transferred files
5. Check DNS tab → look for suspicious domains
6. Check Anomalies tab → flag TLS errors, spoofing, EternalBlue
7. Switch to Wireshark for deep-dive on flagged indicators


### Why Credential Extraction Matters

Even hashed credentials (NTLM, NETNTLMv2, Kerberos) are dangerous because:
- They can be cracked offline with tools like Hashcat
- They can be used in **pass-the-hash** attacks without cracking
- Their presence in network traffic indicates authentication activity that may be unauthorized

### OS Fingerprinting Insight

NetworkMiner uses **passive fingerprinting** (reading TCP/IP stack behavior from existing packets), no active scanning required. This means it can identify OS types without alerting the target.

---

## Tools Used

- **TryHackMe:** NetworkMiner room
- **NetworkMiner 2.7.2:** primary analysis tool
- **NetworkMiner 1.6.1:** frame-level analysis (Frames tab)
- **Exercise PCAPs:** mx-1.pcap, mx-2.pcap, case1.pcap, case2.pcap

---
