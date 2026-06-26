# The Greenholt Phish


## Scenario

A sales executive at **Greenholt PLC** reported a suspicious email received from a known customer. The message raised several red flags:

- A generic greeting (not personalized like the customer's usual emails)
- An unexpected request related to a money transfer
- An unsolicited attachment

According to the employee, this did not match the customer's normal communication style. The email was escalated to the **SOC (Security Operations Center)** for investigation.

**Goal:** Analyze the email sample to determine whether it is legitimate or a phishing attempt.

---

## Email Overview

The reported email appeared as follows:

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Writeup3/Capture1.PNG?raw=true">
</div>

The email claimed that funds had been transferred via SWIFT and that a payment receipt was attached a classic **Business Email Compromise (BEC)** / invoice fraud pretext designed to lend legitimacy to a malicious attachment.

---

## Investigation & Findings

### 1. Transfer Reference Number

**Answer:** 09674321

This number is referenced again in the attachment filename, reinforcing the social engineering narrative.

---

### 2. Display Name of the Sender

**Answer:** Mr. James Jackson

Visible in the email header's **From** field. A display name is purely cosmetic and trivial for an attacker to spoof it should never be trusted on its own.

---

### 3. Sender's Email Address

**Answer:** info@mutawamarine.com

Found beneath the display name in the **From** header. This is the actual originating address behind "Mr. James Jackson."

---

### 4. Reply-To Address

**Answer:** info.mutawamarine@mail.com

Extracted from the **Reply-To** header. This is a major red flag the Reply-To domain "mail.com" does not match the From domain "mutawamarine.com". This mismatch is a strong phishing indicator: any reply from the victim would be redirected to an entirely different mailbox controlled by the attacker, not the legitimate sender's domain.

---

### 5. Originating IP Address

I navigated to **View > Message Source** to examine the raw email headers.

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Writeup3/Capture4(1).PNG?raw=true">
</div>

The bottom-most entry is closest to the original sender, while headers added by intermediate relays sit above it.

**Answer:** 192.119.71.157

---

### 6. Owner of the Originating IP

Using **IPinfo** to look up 192.119.71.157 :

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Writeup3/Capture5(1).PNG?raw=true">
</div>

**Answer:** Hostwinds 

This is a significant red flag. The IP belongs to a **commercial VPS hosting provider**, not a legitimate corporate mail server. Legitimate business correspondence from a known customer should originate from their organization's actual mail infrastructure not a rented server, which is commonly used by attackers to send phishing emails cheaply and anonymously.

---

### 7 & 8. SPF and DMARC Records

**Method:** Used the email header (extracted from Thunderbird's message source) and pasted it into **MXToolbox's Email Header Analyzer**.

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Writeup3/Capture7(2).PNG?raw=true">
</div>

---

### 9. Attachment Filename

**Answer:** SWT_#09674321____PDF__.CAB

Found in the message source under the Content-Disposition header:


**Red flag:** The filename is deliberately crafted to *look* like a PDF ("____PDF__") while actually being a ".CAB" (cabinet/archive) file a classic technique to trick users into trusting and opening the attachment.

---

### 10. SHA256 Hash of the Attachment

After saving the attachment locally, I generated its hash:


<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Writeup3/Capture9(2).PNG?raw=true">
</div>

**Answer:** "2e91c533615a9bb8929ac4bb76707b2444597ce063d84a4b33525e25074fff3f"

---

### 11. Attachment File Size

Using the SHA256 hash to search **VirusTotal**:

<div align="center">
  <img src="
https://github.com/emmagh8/Assets/blob/main/SOC_L1/Writeup3/Capture10(2).PNG?raw=true">
</div>

**Answer:** 400.26 KB

This confirms the file is actively recognized as malicious by the majority of antivirus engines on VirusTotal a near-conclusive indicator that this email is a phishing/malware delivery attempt rather than a legitimate business communication.

---

### 12. Actual File Type of the Attachment

Despite the ".CAB" extension and the misleading "PDF" text in the filename, VirusTotal's file type tagging identified the true container format as RAR (a compressed archive, not a PDF document at all).

**Answer:** RAR

This is a deliberate **extension spoofing / file masquerading** technique disguising an executable-bearing archive as an innocuous document to bypass both user suspicion and basic file-type filtering.

---

**Recommendation:**

- Block sender domain "mutawamarine.com" and IP "192.119.71.157" at the email gateway
- Add the file hash to the organization's blocklist/EDR

---

## Key Takeaways

- **Display names mean nothing** — always verify the actual sending address and originating IP.
- A mismatch between the **From domain** and **Reply-To domain** is a strong phishing indicator.
- **SPF/DMARC analysis** can technically prove an email was spoofed, even when it looks legitimate at first glance.
- Attackers frequently use **VPS/hosting providers** instead of legitimate corporate mail servers checking IP ownership is a quick, high-value triage step.
- File extensions and names can be **easily disguised**; always verify the true file type using a hash lookup or magic-byte analysis (e.g., VirusTotal, "file" command) rather than trusting the filename.
