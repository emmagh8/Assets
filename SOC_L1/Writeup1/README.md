# Summit Writeup

**Challenge:** Summit

**Platform:** TryHackMe

**Category:** SOC / Threat Detection

**Difficulty:** Easy

---

## Summary

In this challenge, I played the role of a SOC analyst defending against a simulated attacker named **Sphinx**. The goal was to progressively block each malware sample by applying the **Pyramid of Pain** framework. Starting from the easiest indicator (hash values) and climbing all the way to the hardest (TTPs). Each time I successfully blocked a sample, Sphinx was forced to adapt and climb higher up the pyramid, until they finally gave up at the top.

---

## Environment

| Component | Details |
|-----------|---------|
| Platform | PicoSecure (TryHackMe simulated SOC environment) |
| Tools Used | Malware Sandbox, Manage Hashes, Firewall Manager, DNS Filter, Sigma Rule Builder |
| Framework | Pyramid of Pain, MITRE ATT&CK |

---

## Investigation Steps

### Step 1 : Blocking sample1.exe

**Pyramid of Pain Level:** Hash Values (Trivial)

I started by submitting "Sample1.exe" to the **Malware Sandbox** for analysis. The sandbox returned the following details:

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Writeup1/Malware_sandbox.PNG?raw=true">
</div>

I copied the **SHA256 hash** from the results, navigated to **Manage Hashes**, selected SHA256 as the algorithm, pasted the value, and submitted it to block the file.

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Writeup1/Manage_hashes.PNG?raw=true">
</div>


**Why it works:** Hash values are unique fingerprints for a specific file. Blocking them is fast and high-confidence.

**Why it's not enough:** The attacker only needs to recompile the malware, even changing a single bit generates a completely new hash, bypassing this detection instantly.

Flag obtained for Step 1.

---

### Step 2 : Blocking "sample2.exe"

**Pyramid of Pain Level:** IP Addresses (Easy)

As expected, Sphinx recompiled their malware and bypassed the hash block. I submitted "sample2.exe" to the sandbox and examined the **HTTP Requests** and **Connections** sections:

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Writeup1/Capture6.PNG?raw=true">
</div>

The suspicious IP "154.35.10.113" (ASN: Intrabuzz Hosting Limited) was clearly a C2 server. I navigated to the **Firewall Manager** and created a new outbound deny rule:

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Writeup1/Capture7.PNG?raw=true">
</div>


**Why it works:** Blocking the C2 IP cuts off the malware's communication channel.

**Why it's not enough:** Attackers using cloud providers can rotate to a new IP address within minutes, making IP-based blocking a short-term solution.

Flag obtained for Step 2.

---

### Step 3 : Blocking "sample3.exe"

**Pyramid of Pain Level:** Domain Names (Simple)

Sphinx pivoted to using a cloud service provider to rotate IP addresses, making firewall rules impractical. I analyzed "sample3.exe" and looked at the **DNS Requests**:

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Writeup1/Capture9.PNG?raw=true">
</div>

The domain "emudyn.bresonicz.info" was clearly malicious, it was being used to map Sphinx's regularly-rotated C2 IP addresses. I navigated to the **DNS Filter** and created a new rule:

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Writeup1/Capture10.PNG?raw=true">
</div>

**Why it works:** Blocking the domain prevents resolution regardless of what IP it points to, which is more resilient than blocking a single IP.

**Why it's not enough:** Attackers can register a new domain cheaply and quickly.

Flag obtained for Step 3.

---

### Step 4 : Blocking "sample4.exe"

**Pyramid of Pain Level:** Network/Host Artifacts (Annoying)

Sphinx moved past domains and began leaving **artifacts on the victim host**. The sandbox revealed registry modification events:

| Process | Registry Key | Name | Value |
|---------|-------------|------|-------|
| sample4.exe | HKLM\SOFTWARE\Microsoft\Windows Defender\Real-Time Protection | DisableRealtimeMonitoring | 1 |

This maps to **MITRE ATT&CK TA0005 — Defense Evasion**: Sphinx was disabling Windows Defender to avoid detection.

I used the **Sigma Rule Builder** to create a detection rule:

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Writeup1/Capture13.PNG?raw=true">
</div>

**Why it works:** Detecting specific registry modifications catches this evasion technique regardless of the malware's hash, IP, or domain.

**Why it's not enough:** The attacker could achieve the same goal through a different registry key or method.

Flag obtained for Step 4.

---

### Step 5 : Analysing "outgoing_connections.log"

**Pyramid of Pain Level:** Tools / Behavioral Artifacts (Challenging)

Sphinx moved their malware logic server-side to leave no artifacts on the host. They provided a log file of outgoing network connections. Examining the log, I identified a clear **beaconing pattern**:

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Writeup1/Capture16.PNG?raw=true">
</div>

This behavior is consistent with **C2 beaconing** — the infected host checking in with the attacker's server at regular intervals for instructions.

I created a Sigma rule in the **Sigma Rule Builder**:

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Writeup1/Capture17.PNG?raw=true">
</div>

**Why it works:** Behavioral detection based on timing and packet size catches the attacker regardless of which IP or port they use.

**Why it's harder to bypass:** Changing beaconing behavior requires modifying the core logic of the malware.

Flag obtained for Step 5.

---

### Step 6 : Analysing "commands.log"

**Pyramid of Pain Level:** Tactics, Techniques & Procedures (Tough!)

Sphinx was now at the **very top of the Pyramid of Pain**. They sent command logs showing what actions they perform on victim machines after gaining remote access. The log revealed systematic **host discovery and data collection** commands:

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Writeup1/Capture19.PNG?raw=true">
</div>

All output was being written to the hardcoded file "%temp%\exfiltr8.log". This maps to **MITRE ATT&CK TA0007-Discovery**.

I created a final Sigma rule:

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Writeup1/Capture20.PNG?raw=true">
</div>

**Why it works:** Detecting the creation of a specific output file catches the attacker's post-exploitation behavior regardless of the delivery method.

**Why TTPs are the most painful to change:** To evade this detection, Sphinx would need to completely redesign their post-exploitation methodology a costly and time-consuming operation.

Flag obtained for Step 6. **Sphinx gave up.**

---

## Outcome

After successfully climbing all 6 levels of the Pyramid of Pain, Sphinx sent a final email:

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Writeup1/Capture21.PNG?raw=true">
</div>

---

## Key Takeaways

- **Low-level indicators** (hashes, IPs, domains) are easy and fast to block, but attackers can bypass them trivially.
- **High-level indicators** (artifacts, tools, TTPs) take more analysis effort but cause significantly more pain for the attacker to work around.
- **Sigma rules** are a powerful way to codify behavioral detections that map directly to MITRE ATT&CK techniques.
- **MITRE ATT&CK** provides the context that turns raw observations into actionable intelligence.
- Forcing an attacker to climb the Pyramid of Pain increases their **operational cost** until it's no longer worth targeting you.

---
