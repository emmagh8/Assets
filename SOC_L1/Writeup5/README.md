# Phishing Unfolding 

**Difficulty:** Medium  
**Platform:** TryHackMe SOC Simulator  
**SIEM Tool Used:** Splunk  
**Sysmon:** data source for process telemetry

---

## Overview

This scenario simulates a real-world phishing attack progressing through a corporate environment. As a SOC analyst, I was assigned 34 alerts generated across email, process, and execution data sources. My task was to triage each alert, classify it as a **True Positive**  or **False Positive**, investigate suspicious events using Splunk, and escalate confirmed threats with detailed case reports.

---

## Alert Analysis

### TRUE POSITIVES

---

#### Alert 1: Suspicious Inbound Phishing Email

**Event Details (from Splunk):**

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Writeup5/Capture4(2).PNG?raw=true">
</div>

**Analysis:**  
The email was sent from a suspicious external domain (".xyz" TLD), with no attachment but highly persuasive, unsolicited content using classic social engineering language. The subject line and body both contain exaggerated promises designed to manipulate the recipient into clicking a link.

**Classification:** 

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Writeup5/Capture5(2).PNG?raw=true">
</div>

---

#### Alert 2: Suspicious Parent-Child Relationship  #1028

**Rule:** Suspicious Parent Child Relationship  
**Type:** Process
**Severity:** High  

**Event Details (from Splunk):**

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Writeup5/Capture8(1).PNG?raw=true">
</div>

**Analysis:**  
PowerShell spawning "nslookup.exe" with a long Base64-like encoded subdomain querying "haz4rdw4re.io" is a strong indicator of DNS tunneling or C2  beacon activity. The working directory in the user's Downloads folder, combined with the suspicious domain, confirms malicious intent.

**Classification:** 

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Writeup5/Capture9(3).PNG?raw=true">
</div>


---

#### Alert 3: Network Drive Mapped to Local Drive #1022

**Event Details (from Splunk):**

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Writeup5/Capture13(1).PNG?raw=true">
</div>


**Analysis:**  
PowerShell executed "net.exe" to map a network share ("\\FILESRV-01\SSF-FinancialRecords") to drive "Z:". This targets a sensitive financial records share. The mapping was then deleted (anti-forensic cleanup). The entire operation originates from a non-standard working directory (Downloads), strongly suggesting scripted, malicious activity.

**Classification:**  True Positive  

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Writeup5/Capture14(1).PNG?raw=true">
</div>


---

#### Alert 4: Suspicious Parent-Child Relationship (Robocopy)  #1023

**Event Details (from Splunk):**

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Writeup5/Capture16(1).PNG?raw=true">
</div>

**Analysis:**  
Immediately after mapping the financial records share to "Z:\", "Robocopy.exe" was used to recursively copy ("/E") all contents of the network drive into a local folder named **"exfiltration"**. This is a textbook **data staging for exfiltration** technique.

**Classification:** 

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Writeup5/Capture17(1).PNG?raw=true">
</div>


---

### FALSE POSITIVES

---

#### FP 1: TrustedInstaller.exe

**Event Details:**

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Writeup5/Capture1(1).PNG?raw=true">
</div>

**Analysis:**  
"TrustedInstaller.exe" is a legitimate Windows component responsible for installing Windows updates and protected system files. It is running from its expected path ("C:\Windows\servicing\") and launched by "services.exe", which is the standard parent process. No anomalies detected.

**Classification:** 
<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Writeup5/Capture.PNG?raw=true">
</div>

---

#### FP 2: taskhostw.exe

**Event Details:**

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Writeup5/Capture2(1).PNG?raw=true">
</div>

**Analysis:**  
"taskhostw.exe" is a valid Windows host process for task management. Running from "C:\Windows\system32\" and launched by "svchost.exe" is completely normal behavior. The "KEYROAMING" argument is associated with credential roaming tasks in Windows.

**Classification:** 

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Writeup5/Captur3.PNG?raw=true">
</div>
---

#### FP 3: WUDFHost.exe


**Event Details:**

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Writeup5/Capture6(2).PNG?raw=true">
</div>


**Analysis:**  

"WUDFHost.exe" (Windows User-Mode Driver Framework Host) is a legitimate Windows system process for managing user-mode device drivers. The lengthy command line with GUIDs represents standard UMDF port communication parameters. Running from "system32" under "services.exe" is expected behavior.

**Classification:** 

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Writeup5/Captur3.PNG?raw=true">
</div>


---

## Performance Results

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Writeup5/Capture25.PNG?raw=true">
</div>

False positive rate was 58% — several phishing alerts (IDs 1000, 1003, 1013, 1014, 1017, 1018) were incorrectly classified. These likely involved legitimate marketing emails that share surface-level characteristics with phishing but lack the full attack chain indicators.

---

## Key Takeaways

1. **Context matters in phishing triage:** not every email from an external domain is malicious. Look for combinations of suspicious TLD + social engineering language + urgency cues before classifying as TP.

2. **Process chain analysis is critical:** the entire attack on win-3450 was rooted in a single "powershell.exe" process (PID 3728). Pivoting on parent PIDs in Splunk/Sysmon revealed the full kill chain.

3. **Naming conventions reveal intent:** the attacker explicitly named a staging folder "exfiltration." Adversaries are not always subtle; anomalous directory names are high-value indicators.

4. **DNS tunneling detection:** long encoded subdomains queried via "nslookup.exe" spawned by "powershell.exe" is a red flag. Always check the full command line, not just the process name.

5. **Anti-forensic behavior:** the immediate deletion of the mapped network drive ("net use Z: /delete") after copying data is a sign of a skilled attacker attempting to cover their tracks.

6. **Legitimate Windows processes can generate false alerts:** "TrustedInstaller.exe", "taskhostw.exe", and "WUDFHost.exe" are all valid system processes. Always verify the parent process, execution path, and working directory before escalating.

---
