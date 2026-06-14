# Eviction Writeup

**Challenge:** Eviction

**Platform:** TryHackMe

**Category:** Threat Intelligence / APT Analysis

**Difficulty:** Easy–Medium

---

## Summary

The Eviction room explores the world of **Advanced Persistent Threat (APT)** groups and their Tactics, Techniques, and Procedures (TTPs). Through a real-world scenario involving the fictional company **E-corp** and analyst **Sunny**, I investigated the attack lifecycle of **APT28 (G0007)** a well-known Russian state-sponsored threat group using the **MITRE ATT&CK Navigator**.

The objective was to trace APT28's full attack chain: from initial reconnaissance all the way to attempted data exfiltration, and identify which techniques Sunny should monitor to detect and respond to each stage.

---

## Environment

| Component | Details |
|-----------|---------|
| Platform | TryHackMe - Eviction Room |
| Framework | MITRE ATT&CK Navigator |
| Threat Actor | APT28 / Fancy Bear (G0007) |
| Target Organization | E-corp (fictional) |
| Analyst | Sunny |

---

## APT28 Threat Actor Overview

**APT28**, also known as **Fancy Bear**, is a Russian military intelligence (GRU) sponsored threat group active since at least 2004. They are known for targeting government, military, and critical infrastructure organizations through sophisticated phishing campaigns and custom malware. Their TTPs are well-documented in MITRE ATT&CK under group ID **G0007**.

---

## Attack Chain Analysis

The following questions trace APT28's attack lifecycle across the **MITRE ATT&CK kill chain**, from Reconnaissance to Command & Control:

### Q1: Reconnaissance and Initial Access

**Question:** What technique did APT28 use for both reconnaissance and initial access?

**Answer:** "Spearphishing Link"

**Explanation:**
Spearphishing Link appears in both the **Reconnaissance** phase (gathering victim information by luring them to click links) and the **Initial Access** phase (using those same links to deliver malicious payloads). This dual-use nature makes it one of APT28's most effective techniques - one action serves two objectives simultaneously.

- **MITRE Reference:** T1566.002 - Phishing: Spearphishing Link

---

### Q2 : Resource Development

**Question:** Which accounts might APT28 compromise while developing resources?

**Answer:** "Email Accounts"

**Explanation:**
During the **Resource Development** phase, APT28 compromises legitimate email accounts to make their phishing campaigns more convincing. Sending spearphishing emails from a trusted, real account dramatically increases the chance that the target will click the link or open the attachment.

- **MITRE Reference:** T1586.002 - Compromise Accounts: Email Accounts

---

### Q3 : Execution via User Interaction

**Question:** APT28 used social engineering to make the user execute code. What two User Execution techniques should Sunny monitor?

**Answer:** "Malicious File" and "Malicious Link"

**Explanation:**
After gaining initial access through spearphishing, APT28 relies on the victim to execute the payload themselves. The two primary methods are:

- **Malicious File (T1204.002):** The user is tricked into opening an infected attachment (Word document with macros, PDF exploit, etc.)
- **Malicious Link (T1204.001):** The user clicks a link that leads to a drive-by download or a credential harvesting page

Both techniques exploit human trust rather than technical vulnerabilities, making user awareness training a critical defensive control.

---

### Q4 : Execution via Scripting Interpreters

**Question:** Which scripting interpreters should Sunny monitor for successful execution?

**Answer:** "PowerShell" and "Windows Command Shell"

**Explanation:**
Once the user executes the initial payload, APT28 uses built-in Windows scripting engines to run their malicious code, a technique known as **Living off the Land (LotL)**. By using tools already present on the system, they avoid dropping suspicious executables that antivirus might flag.

- **PowerShell (T1059.001):** Commonly used for downloading additional payloads, running obfuscated scripts, and interacting with the Windows API
- **Windows Command Shell (T1059.003):** Used for executing system commands, running batch scripts, and chaining commands for reconnaissance

**Detection tip for Sunny:** Monitor for PowerShell processes spawned by Office applications ("winword.exe" → "powershell.exe"), and look for base64-encoded command-line arguments, which are a strong indicator of obfuscation.

---

### Q5 : Persistence via Registry

**Question:** Sunny found obfuscated scripts modifying the registry for persistence. Which registry keys should she monitor?

**Answer:** "Registry Run Keys"

**Explanation:**
APT28 modifies **Registry Run Keys** to ensure their malware survives reboots. These keys instruct Windows to automatically execute specified programs every time a user logs in. The most commonly abused keys are:

```
HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run
HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Run
```

- **MITRE Reference:** T1547.001 - Boot or Logon Autostart Execution: Registry Run Keys / Startup Folder

**Detection tip for Sunny:** Use Sysmon Event ID 13 (Registry value set) to detect modifications to these keys. Alert on any writes by unexpected processes (PowerShell or cmd.exe writing to Run keys).

---

### Q6 : Defense Evasion via LOLBins

**Question:** Which system binary should Sunny scrutinize for proxy execution?

**Answer:** "Rundll32"

**Explanation:**
**Rundll32.exe** is a legitimate Windows binary used to execute DLL files. APT28 abuses it to run malicious DLL payloads while appearing as a normal system process - a classic **Living off the Land Binary (LOLBin)** technique. Since Rundll32 is a signed Microsoft binary, it often bypasses application whitelisting and antivirus tools.

- **MITRE Reference:** T1218.011 - System Binary Proxy Execution: Rundll32

**Detection tip for Sunny:** Flag unusual Rundll32 executions - especially those loading DLLs from temp folders, AppData, or with network connections originating from the process.

---

### Q7 : Discovery via Network Sniffing

**Question:** Sunny found "tcpdump" on a compromised host. Which discovery technique might APT28 be using?

**Answer:** "Network Sniffing"

**Explanation:**
The presence of "tcpdump" - a legitimate network packet capture tool - on a compromised host strongly suggests APT28 was performing **Network Sniffing** to passively capture credentials, session tokens, and other sensitive data traveling across the network.

- **MITRE Reference:** T1040 - Network Sniffing

**Detection tip for Sunny:** Monitor for unexpected installation or execution of packet capture tools ("tcpdump", "Wireshark", "dumpcap"). Also alert on processes that open promiscuous mode network interfaces.

---

### Q8 : Lateral Movement via Remote Services

**Question:** Which remote services should Sunny monitor for APT28 lateral movement?

**Answer:** "SMB/Windows Admin Shares"

**Explanation:**
After establishing a foothold, APT28 moves laterally across the network by exploiting **SMB (Server Message Block)** and Windows administrative shares (C$, ADMIN$, IPC$). This allows them to copy malware to other machines and execute it remotely using tools like PsExec or WMI.

- **MITRE Reference:** T1021.002 - Remote Services: SMB/Windows Admin Shares

**Detection tip for Sunny:** Monitor for unusual SMB connections between workstations (lateral east-west traffic), unexpected access to "ADMIN$" shares, and the use of PsExec or similar remote execution tools.

---

### Q9 : Collection - Target Information Repository

**Question:** What information repository was APT28's likely target for stealing intellectual property?

**Answer:** "SharePoint"

**Explanation:**
APT28's primary objective at E-corp was **intellectual property theft**. **SharePoint** is a common enterprise collaboration and document management platform that often stores sensitive business documents, project files, and internal knowledge bases - making it a high-value target for espionage-motivated APT groups.

- **MITRE Reference:** T1213.002 - Data from Information Repositories: SharePoint

**Detection tip for Sunny:** Monitor SharePoint access logs for bulk downloads, access from unusual IP addresses or at unusual hours, and access to sensitive document libraries by accounts that don't normally access them.

---

### Q10 : Command & Control - Exfiltration Proxy

**Question:** APT28 couldn't reach their C2 directly. What two proxy techniques might they use for exfiltration?

**Answer:** "External Proxy" and "Multi-hop Proxy"

**Explanation:**
When direct C2 communication is blocked (by firewall rules or network monitoring), APT28 routes their traffic through proxies to disguise the true destination:

- **External Proxy (T1090.002):** Traffic is routed through an external server controlled by the attacker, masking the final C2 destination
- **Multi-hop Proxy (T1090.003):** Traffic hops through multiple intermediate servers (including compromised machines, Tor nodes, or VPNs), making attribution and blocking extremely difficult

**Detection tip for Sunny:** Look for repeated outbound connections to uncommon destinations, large data transfers to external IPs, encrypted traffic to non-standard ports, and connections chained through multiple hops.

---

## Full Attack Chain Summary

```
Reconnaissance       →  Spearphishing Link (recon + initial access)
Resource Development →  Compromise Email Accounts
Initial Access       →  Spearphishing Link
Execution            →  Malicious File / Malicious Link
                        PowerShell / Windows Command Shell
Persistence          →  Registry Run Keys
Defense Evasion      →  Rundll32 (proxy execution)
Discovery            →  Network Sniffing (tcpdump)
Lateral Movement     →  SMB / Windows Admin Shares
Collection           →  SharePoint (information repository)
Command & Control    →  External Proxy + Multi-hop Proxy
```

---

## MITRE ATT&CK Technique Reference

| # | Question Topic | Technique | MITRE ID |
|---|---------------|-----------|----------|
| 1 | Recon + Initial Access | Spearphishing Link | T1566.002 |
| 2 | Resource Development | Compromise Email Accounts | T1586.002 |
| 3 | User Execution | Malicious File + Malicious Link | T1204.002 / T1204.001 |
| 4 | Execution | PowerShell + Windows Command Shell | T1059.001 / T1059.003 |
| 5 | Persistence | Registry Run Keys | T1547.001 |
| 6 | Defense Evasion | Rundll32 | T1218.011 |
| 7 | Discovery | Network Sniffing | T1040 |
| 8 | Lateral Movement | SMB/Windows Admin Shares | T1021.002 |
| 9 | Collection | SharePoint | T1213.002 |
| 10 | C2 / Exfiltration | External Proxy + Multi-hop Proxy | T1090.002 / T1090.003 |

---

## Key Takeaways

- **APT28** is a sophisticated, state-sponsored threat actor that uses a wide range of TTPs across the full attack lifecycle.
- **Living off the Land** techniques (Rundll32, PowerShell, cmd.exe) are central to their evasion strategy - defenders must monitor native Windows tools, not just third-party software.
- **Spearphishing** remains one of the most effective initial access methods because it exploits human trust, not just technical vulnerabilities.
- **MITRE ATT&CK Navigator** is an essential tool for visualizing an APT group's known techniques and planning detection coverage.
- Mapping findings to MITRE ATT&CK IDs gives analysts a **shared language** for communicating threats and building detection rules.

---
