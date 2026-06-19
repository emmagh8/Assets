# Phishing Analysis

## Overview

This room covers how to **analyze and investigate phishing emails** like a real SOC analyst.
It walks through the full investigation lifecycle from reading email headers, to analyzing URLs, to detonating suspicious attachments in
a sandbox. Rather than just theory, the room simulates realistic scenarios that analysts face daily.

**What I learned:**
- How email protocols (SMTP, IMAP, POP3) work under the hood
- How to extract and interpret email header artifacts
- Safe techniques for analyzing malicious URLs and attachments
- How to use industry tools like ANY.RUN, VirusTotal, and PhishTool
- How modern email security controls (SPF, DKIM, DMARC) defend against spoofing

---

## 1. Email Infrastructure Fundamentals

Before analyzing phishing, you need to understand how legitimate email actually works — because attackers exploit every layer of this process.

### Core Protocols

| Protocol | Full Name | Role |
|----------|-----------|------|
| **SMTP** | Simple Mail Transfer Protocol | Transmits emails between mail servers |
| **POP3** | Post Office Protocol v3 | Downloads emails to a local device (removes from server) |
| **IMAP** | Internet Message Access Protocol | Syncs emails across multiple devices (stays on server) |

### The Email Delivery Pipeline

```
[Sender] → SMTP → [Sender's Mail Server] → DNS (MX Lookup) → [Recipient's Mail Server] → POP3/IMAP → [Recipient]
```

Step by step:
1. The sender's client submits the message to their mail server via **SMTP**
2. The sending server queries **DNS** for the recipient domain's **MX record**
3. DNS returns the IP of the recipient's mail server
4. The message is relayed across the Internet to that server
5. The recipient's client retrieves the message via **POP3** or **IMAP**

**Why this matters for analysts:** Every hop in this chain leaves traces in the **email header**. Attackers try to forge or obscure these traces, learning to read headers lets you spot the deception.

### Anatomy of an Email

Every email has two parts:

- **Header** Invisible metadata: sender IP, relay servers, timestamps, authentication results (SPF/DKIM pass/fail). This is where most forensic evidence lives.
- **Body** The visible content: text, HTML, embedded images, links, and attachments. This is where social engineering happens.

---

## 2. Types of Phishing Attacks

Phishing is not a single attack — it is a family of social engineering techniques:

| Attack Type | Target | Vector | Notes |
|-------------|--------|--------|-------|
| **Spam / Malspam** | Mass (anyone) | Email | Bulk unsolicited mail; malspam carries malware payloads |
| **Phishing** | General users | Email | Impersonates trusted brands (banks, Microsoft, PayPal) |
| **Spear Phishing** | Specific individual/org | Email | Uses personal details to appear legitimate |
| **Whaling** | C-Suite executives | Email | High-value targets: CEO, CFO, CISO |
| **Smishing** | Mobile users | SMS | Same concept, different channel |
| **Vishing** | Anyone | Phone call | Voice-based social engineering |

**Key insight:** Spear phishing and whaling are far more dangerous than generic phishing because attackers invest time in research. A targeted email that references your name, your company, and a real ongoing project is much harder to detect than a generic "Dear Customer" message.

---

## 3. Recognizing Phishing Emails

These are the signals to look for when triaging a suspicious email:

| Indicator | What to Look For | Example |
|-----------|-----------------|---------|
| **Spoofed sender** | Domain typos, lookalike domains | "noreply@microsof.com" vs "microsoft.com" |
| **Urgency / fear** | Pressure to act immediately | *"Your account will be suspended in 24 hours"* |
| **Brand impersonation** | Copied logos, color schemes, footers | Fake PayPal email with real PayPal branding |
| **Grammar errors** | Unnatural phrasing, awkward structure | Less common now with AI-generated phishing |
| **Generic greeting** | No personalization | *"Dear Customer"* instead of your actual name |
| **Mismatched URLs** | Hover text does not match actual destination | Link shows "paypal.com" but goes to "paypa1.ru" |
| **Shortened URLs** | Hides the real destination | "bit.ly/secure-login" |
| **Suspicious attachments** | Double extensions, unexpected files | "invoice.pdf.exe", "document.docx.js" |
| **Reply-To mismatch** | Replies routed to a different address | Sent from "support@bank.com", replies go to "attacker@gmail.com" |

---

## 4. Investigation Methodology

When a suspicious email is reported, follow this structured 4-step process:

### Step 1 : Extract Header Artifacts

The header is your first source of truth. Key fields to extract:

```
From:              → Claimed sender address
Reply-To:          → Where replies actually go (often different from From)
Return-Path:       → Where bounces go (another spoofing indicator)
Received:          → Relay chain — read bottom to top for true origin
X-Originating-IP:  → The actual sending IP address
Message-ID:        → Unique email identifier
Date:              → When it was sent
Subject:           → Look for urgency, typos, odd formatting
Authentication-Results: → SPF / DKIM / DMARC pass or fail
```

---

### Step 2 : Analyze the Email Body

**Golden rule: Never click anything directly. Always work on a copy.**

**URL Analysis:**
1. Right-click → Copy link address (do NOT click)
2. Expand any shortened URLs: [Unshorten.it](https://unshorten.it)
3. Scan the full URL: [URLScan.io](https://urlscan.io), [VirusTotal](https://www.virustotal.com)
4. Use a URL extraction tool to catch obfuscated or hidden URLs in the HTML source

**Attachment Analysis:**
1. Note the filename and extension — watch for **double extensions**: "invoice.pdf.exe"
2. Generate a SHA256 hash (do not open the file):

```bash
# Linux / macOS
sha256sum suspicious_file.pdf

# PowerShell (Windows)
Get-FileHash suspicious_file.pdf -Algorithm SHA256
```

3. Submit the hash to threat intelligence platforms:

| Platform | Use Case |
|----------|----------|
| [VirusTotal](https://www.virustotal.com) | Check hash against 70+ antivirus engines |
| [Talos Intelligence](https://talosintelligence.com) | Cisco threat intel: IP, domain, and file reputation |
| [MalwareBazaar](https://bazaar.abuse.ch) | Malware hash database by Abuse.ch |

---

### Step 3 : Sandbox Detonation

If static analysis is not conclusive, execute the file or URL in a **controlled sandbox**:

| Sandbox | Type | Highlights |
|---------|------|------------|
| [**ANY.RUN**](https://any.run) | Interactive | Real-time execution — interact with the VM, watch processes spawn, see network calls live |
| [**Hybrid Analysis**](https://www.hybrid-analysis.com) | Automated | Free, detailed behavioral reports including network IOCs and memory analysis |
| [**JOESandbox**](https://www.joesecurity.org) | Advanced | Static + dynamic analysis with MITRE ATT&CK mapping |

**What to look for in sandbox results:**
- Processes created (especially "cmd.exe", "powershell.exe", "wscript.exe")
- Outbound network connections (potential C2 communication)
- Files dropped or modified on disk
- Registry changes (persistence mechanisms)
- DNS queries to suspicious domains

---

### Step 4 : Correlate & Close the Case

Using **PhishTool** or your SIEM/ticketing system:
1. Compile all artifacts: sender IP, domain, URLs, file hashes
2. Cross-reference with threat intelligence feeds
3. Determine verdict: **malicious / suspicious / benign**
4. Document findings with evidence
5. Escalate if needed, or close the ticket with full notes

---

## 5. Tools Used

| Tool | Purpose | Link |
|------|---------|------|
| MXToolbox | Email header analysis | [mxtoolbox.com](https://mxtoolbox.com) |
| Google Admin Toolbox | Header visualization | [toolbox.googleapps.com](https://toolbox.googleapps.com/apps/messageheader/) |
| URLScan.io | Safe URL scanning and screenshot | [urlscan.io](https://urlscan.io) |
| Unshorten.it | Expand shortened URLs | [unshorten.it](https://unshorten.it) |
| VirusTotal | Hash, URL, and file reputation | [virustotal.com](https://www.virustotal.com) |
| Talos Intelligence | Cisco threat intel (IP/domain/hash) | [talosintelligence.com](https://talosintelligence.com) |
| AbuseIPDB | IP reputation and abuse reports | [abuseipdb.com](https://www.abuseipdb.com) |
| MalwareBazaar | Malware sample database | [bazaar.abuse.ch](https://bazaar.abuse.ch) |
| ANY.RUN | Interactive malware sandbox | [any.run](https://any.run) |
| Hybrid Analysis | Automated sandbox analysis | [hybrid-analysis.com](https://www.hybrid-analysis.com) |
| JOESandbox | Advanced static + dynamic analysis | [joesecurity.org](https://www.joesecurity.org) |
| PhishTool | Phishing case management | [phishtool.com](https://www.phishtool.com) |

---

## 6. Phishing Prevention & Defenses

### Email Authentication Protocols

These three protocols work together to verify that an email is legitimate before it reaches the inbox:

| Protocol | What It Does | What Happens on Failure |
|----------|-------------|------------------------|
| **SPF** | Checks that the sending server is authorized for the domain | Email may be marked spam or rejected |
| **DKIM** | Cryptographic signature verifying the email was not tampered with | Signature mismatch flags the email as suspicious |
| **DMARC** | Policy layer on top of SPF + DKIM — defines what to do with failures | Reject, quarantine, or report to the domain owner |
| **S/MIME** | End-to-end encryption and digital signing of email content | No automatic enforcement — requires setup on both ends |

**Analyst tip:** Always check the "Authentication-Results" field in the header — it shows whether SPF, DKIM, and DMARC passed or failed for the email you are investigating.

### Technical Controls

| Defense Layer | How It Works |
|--------------|-------------|
| **Email Filtering** | Blocks or quarantines based on IP/domain reputation scores |
| **Secure Email Gateways (SEGs)** | Deep inspection for spoofing, impersonation, and zero-day links |
| **Link Rewriting** | Wraps URLs to scan them at click-time, not just at delivery-time |
| **Sandboxing** | Detonates attachments in isolation before delivering to the user |

### Human Controls

Even perfect technical defenses will eventually be bypassed. The human layer matters:

| Measure | Why It Works |
|---------|-------------|
| **Warning banners** | *"External Sender"* banners prime users to be skeptical before reading |
| **Easy reporting buttons** | Lower friction = more reports = faster SOC detection |
| **Security awareness training** | Helps users recognize red flags before clicking |
| **Phishing simulation campaigns** | Regular drills keep skills sharp and measure improvement over time |

**The mindset shift:** The goal of user training is not to make users the last line of defense — it is to make them an active part of your detection pipeline. A user who reports a phishing email is generating threat intelligence.

---

## 7. Key Takeaways

1. **Start with the header, always.** The body is designed to deceive. The header tells the truth.
2. **We never interact with artifacts directly.** Copy URLs, hash files, use sandboxes.
3. **Reputation checks are fast wins.** A quick VirusTotal or Talos lookup can confirm maliciousness in seconds.
4. **Sandboxes reveal what static analysis misses.** Obfuscated scripts and packed executables hide their behavior and let them run safely in isolation.
6. **Defense is multi-layered.** SPF + DKIM + DMARC + SEG + trained users together are far stronger than any single control alone.
7. **Document everything.** A well-documented case helps the whole team and build detection knowledge base over time.

---

