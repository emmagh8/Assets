# Snapped Phish-ing Line

---

## Scenario

As a member of the IT department at **SwiftSpend Financial**, multiple employees across different departments reported receiving a suspicious email. Several recipients noted unusual characteristics in the message, and some had already submitted their credentials and lost access to their accounts.

The incident was escalated for investigation. The objective: analyze the available evidence, determine the scope of the attack, and uncover how the adversary operated from the initial phishing email all the way to the credential-harvesting infrastructure behind it.

---

## Investigation & Findings

### 1. Recipient of the "Quote for Services Rendered" Email

**Answer:** William McClean

Identified by reviewing the email headers in the "phish-emails" folder on the VM desktop. The **To** field of the relevant email reads:

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Writeup4/Capturexx11.PNG?raw=true">
</div>


---
### 2. Adversary's Sending Email Address

**Answer:** Accounts.Payable@groupmarketingonline.icu

Extracted from the **From** header of the same email:

The sender domain "groupmarketingonline.icu" is an immediate red flag. The ".icu" TLD is a low-cost, loosely-regulated top-level domain frequently abused in phishing campaigns due to minimal registration requirements and cheap bulk pricing. The display name "Group Marketing Online Accounts Payable" is crafted to look like a legitimate finance contact, reinforcing the invoice pretext. Display names are trivial to spoof and should never be trusted without verifying the underlying address and sending infrastructure.

---

### 3. Root Domain of the Redirection URL

**Method:** Opened the HTML attachment sent to **Zoe Duncan** in a text editor to inspect the source. The file contained an embedded redirect link:

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Writeup4/Capturexx12.PNG?raw=true">
</div>

**Answer:** kennaroads.buz

The URL structure reveals more than just a domain — the path "/40e7baa2f826a57fcf04e5202520f8bd/?email=victim@domain" follows a classic phishing kit pattern. The hash-like token is a unique per-victim campaign tracker, and the "?email=" parameter **pre-fills the victim's email address** on the fake login page to make it feel convincingly personalized. This increases the likelihood that the victim will complete the credential submission without suspicion.

---

### 4. Company Impersonated by the Login Page

**Method:** Opened the HTML attachment directly in the VM's web browser, which triggered the redirect to "kennaroads.buzz" and rendered the spoofed login page.

**Answer:** Microsoft

The page closely mimicked the Microsoft 365 login interface using matching logos, colors, and layout to deceive victims into entering their corporate Microsoft credentials.

---

### 5. Name of the Phishing Kit Archive

**Method:** Navigated to the `/data` directory on `kennaroads.buzz`. The attacker had left **open directory listing** enabled on their server — a significant OPSEC failure that exposed the full contents:

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Writeup4/Capturexx13.PNG?raw=true">
</div>

**Answer:** Update365.zip

---

### 6. SHA256 Hash of the Archive

**Method:** I downloaded "Update365.zip" to the VM using "wget", then i loaded the file into **CyberChef** . Applied the **SHA2** recipe with hash size set to **256**.


**Answer:** "ba3c15267393419eb08c7b2652b8b6b39b406ef300ae8a18fee4d16b19ac9686"

---

### 7. Additional Threat Category (VirusTotal)

**Method:** I submitted the SHA256 hash from the previous step to VirusTotal

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Writeup4/Screenshot_2026-06-27_10_10_19.png?raw=true">
</div>

**Answer:** Trojan

---

### 8. Number of Files in the Archive

**Method:** I reviewed the VirusTotal Details page for the phishing kit . The Details tab lists the archive's file count.

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Writeup4/Screenshot_2026-06-27_10_10_10.png?raw=true">
</div>

**Answer:** 49

---

### 9. Email Address Submitted More Than Once

**Method:** I navigated to "/data/Update365/" on "kennaroads.buzz" and identified "log.txt"  the kit's **live credential capture log**. I D
downloaded the file :

```
wget http://kennaroads.buzz/data/Update365/log.txt
```

Then i parsed the log to count email submissions:

```
grep -i Email log.txt | sort | uniq -c
```

This revealed that one address appeared **twice** in the log. To confirm it was the same password submitted both times (ruling out two separate users):

```
grep -i Email log.txt -A 1
```

**Answer:** michael.ascot@swiftspend.finance

 The fact that the same email and the same password were submitted twice is significant it likely means the phishing page used a **fake "incorrect password" error** on the first submission to prompt the victim to re-enter their credentials, a common technique to ensure the captured credentials are correct before accepting them. This confirms genuine credential exposure and flags this account for an immediate forced password reset.

---

### 10. Adversary's Credential Collection Email Address

**Method:** Extracted the phishing kit archive and located "Update365/office365/Validation/submit.php" the PHP script responsible for handling credential form submissions. Reviewed the script's email delivery logic:

```
Update365/office365/Validation/submit.php
```

The "$to" variable in the submit handler pointed to:

**Answer:** m3npat@yandex.com

---

## Indicators of Compromise (IOCs)

| Type | Value |
|------|-------|
| Phishing sender address | "Accounts.Payable@groupmarketingonline.icu" |
| Phishing sender domain | "groupmarketingonline.icu" |
| Malicious redirect domain | "kennaroads.buzz" |
| Phishing kit archive | "Update365.zip" |
| Archive SHA256 | "ba3c15267393419eb08c7b2652b8b6b39b406ef300ae8a18fee4d16b19ac9686" |
| Credential drop email  | "m3npat@yandex.com" |

---

**Recommendations:**
- Force immediate **password reset** for all employees whose addresses appear in "log.txt", prioritizing "michael.ascot@swiftspend.finance"
- Block sender domain "groupmarketingonline.icu" at the email gateway
- Block "kennaroads.buzz" at the DNS/web proxy layer
  
---

## Key Takeaways

- **A single phishing report can unravel an entire campaign.** By tracing the redirect URL to the attacker's server and finding open directory listing, one employee's report gave full visibility into the kit, victim logs, and adversary infrastructure.
- **Open directory listing is an analyst's best friend and an attacker's worst mistake.** Always enumerate exposed directories on attacker infrastructure the evidence yield can be significant.
- **Two adversary email addresses in a single kit is a kit-reuse signal.** When you find addresses left over from a previous operator, it points to commodity phishing kit marketplaces and may help link campaigns across incidents.
- **The "wrong password" loop is a deliberate technique, not a bug.** Repeated credential submissions from the same victim indicate the page intentionally induced re-entry to validate credentials accounts that appear twice in a log are confirmed compromised, not just noise.
---
