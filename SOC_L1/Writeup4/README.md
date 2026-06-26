# Snapped Phish-ing Line

---

## Scenario

As a member of the IT department at **SwiftSpend Financial**, multiple employees across different departments reported receiving a suspicious email. Several recipients noted unusual characteristics in the message, and some had already submitted their credentials and lost access to their accounts.

The incident was escalated for investigation. The objective: analyze the available evidence, determine the scope of the attack, and uncover how the adversary operated from the initial phishing email all the way to the credential-harvesting infrastructure behind it.

---

## Investigation & Findings

### 1. Email Header Analysis

The first step was identifying the target and the sender of the phishing email.

Opening the reported message in Thunderbird revealed the header details:

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Writeup4/Capturexx11.PNG?raw=true">
</div>

**Findings:**
- **Recipient:** William McClean — targeted with the "Quote for Services Rendered" themed email
- **Sender address used by the adversary:** `Accounts.Payable@groupmarketingonline.icu`

> 💡 **Analyst note:** The sender domain `groupmarketingonline.icu` is a strong red flag on its own — `.icu` is a low-cost, loosely-regulated TLD frequently abused in phishing campaigns because of minimal registration requirements. The display name "Group Marketing Online Accounts Payable" is also designed to look like a legitimate finance/vendor contact, playing on the invoice/quote pretext.

---

## 2. HTML Attachment Analysis

The phishing email carried a malicious HTML attachment. Inspecting the file's source revealed an embedded redirect link:

```html
<p>If you are not redirected automatically, please click <a href="http://kennaroads.buzz/data/Update365/office365/40e7baa2f826a57fcf04e5202520f8bd/?email=derick.marshall@swiftspend.finance#error">here</a>.</p>
```

**Findings:**
- **Root domain of the redirection URL:** `kennaroads.buzz`
- Opening the HTML file in a browser redirects the victim to a page that closely mimics the **Microsoft 365 login page**
- **Impersonated company:** Microsoft

**Q&A:**

| # | Question | Answer |
|---|----------|--------|
| 1 | Which individual received the "Quote for Services Rendered" email? | William McClean |
| 2 | Adversary's sending email address | `Accounts.Payable@groupmarketingonline.icu` |
| 3 | Root domain of the redirection URL | `kennaroads.buzz` |
| 4 | Company impersonated by the login page | Microsoft |

> 💡 **Analyst note:** The URL path itself is notable — `/40e7baa2f826a57fcf04e5202520f8bd/?email=victim@domain` — this is a common phishing kit pattern where a unique hash-like token per victim is used to track campaign delivery, and the victim's email is **pre-filled via URL parameter** to make the fake login page feel personalized and more convincing.

---

## 3. Phishing Kit Discovery

Enumerating the `kennaroads.buzz/data/` directory revealed open directory listing was enabled on the attacker's server — a common OPSEC failure in amateur phishing infrastructure:

```
Index of /data
├── Update365.zip       2023-04-16 14:19   394K
└── Update365/           2023-05-03 15:56   -
```

**Finding:**
- **Archive file name:** `Update365.zip`

This archive is the full phishing kit — the source code, assets, and credential-harvesting scripts powering the fake Microsoft login page.

---

## 4. File Hashing & Threat Intelligence

### SHA256 Hash Generation

Using **CyberChef** (pre-loaded on the VM under the `tools` folder), the archive was loaded into the input field and processed with the **SHA2** recipe (hash size: 256):

```
SHA256: ba3c15267393419eb08c7b2652b8b6b39b406ef300ae8a18fee4d16b19ac9686
```

### VirusTotal Lookup

Submitting the hash to **VirusTotal** (run from the local analysis machine, since the VM itself has no internet access) confirmed:

- **First flagged as malicious:** `2020-06-25`
- **Additional threat category (beyond Phishing):** the archive was also classified as containing **malware**

> 💡 **Analyst note:** Cross-referencing file hashes against VirusTotal is one of the fastest ways to confirm whether infrastructure or a payload has been seen before. The fact this kit was first flagged back in mid-2020 suggests it's a recycled/reused phishing kit template rather than a custom-built one — these "phishing-kit-as-a-template" packages circulate widely among less sophisticated threat actors.

---

## 5. Phishing Kit Extraction & Analysis

Extracting `Update365.zip` and navigating to `/data/Update365` revealed the full kit structure, including a `log.txt` file — the credential harvesting log containing victim data.

**Finding:**
- The archive contained the phishing kit's PHP backend, the cloned Microsoft login assets, and the `log.txt` capture file used to store submitted credentials.

The file responsible for handling form submissions was identified at:

```
Update365/office365/Validation/submit.php
```

This script processes the victim's submitted credentials and forwards them to the adversary via email.

---

## 6. Victim Data Analysis

With `log.txt` retrieved (via `wget`), the captured credential submissions were parsed to determine the scope of the attack.

### Identifying Repeated Submissions

```bash
grep -i Email log.txt | sort | uniq -c
```

This revealed that one email address appeared **twice** in the log — indicating the same victim submitted the phishing form more than once (likely due to a failed first attempt or a multi-step credential validation page common in these kits):

**Finding — Email submitted more than once:**
```
michael.ascot@swiftspend.finance
```

### Confirming Password Reuse

```bash
grep -i Email log.txt -A 1
```

This pulled the email **and the following line** (the captured password) for each entry, confirming that **the same password was submitted both times** by this victim — ruling out the possibility that it was two different people sharing a similar email, and confirming genuine credential exposure for this user.

---

## 7. Tracing the Adversary's Infrastructure

### Where Credentials Were Sent

Reviewing `submit.php`, the script sends captured credentials via email to:

```
m3npat@yandex.com
```

**Finding — Adversary's credential collection address:** `m3npat@yandex.com`

### Finding a Secondary Adversary Address

Searching the kit source files for additional mail destinations:

```bash
grep -ri "gmail.com" .
```

This revealed the same Gmail address appearing across **3 separate files** within the kit, assigned to a `$to` variable used for email delivery:

```
jamestanner2299@gmail.com
```

> 💡 **Analyst note:** This is a particularly interesting finding. The presence of *two* separate collection addresses — a Yandex address actively used in the live `submit.php`, and a Gmail address hardcoded in multiple template files — suggests this kit was **originally built/sold by one threat actor (Gmail address) and then repurposed by another (Yandex address)** who modified the active delivery destination but didn't fully scrub the kit of the original author's hardcoded test/contact address. This kind of artifact is common in commodity phishing-kit marketplaces, where kits are resold or shared with minimal cleanup.

---

## 8. Hidden Flag Discovery

While enumerating the `/data/Update365/office365/` directory structure, it became apparent the kit used an auto-generated, per-victim hash-based subdirectory for the live phishing page (as seen in the redirect URL in Section 2).

Since no directory fuzzing tools (`gobuster`, `dirb`, `dirsearch`, `ffuf`) were available on the VM, manual enumeration was used. A guess of a common filename — `flag.txt` — at the kit's base path succeeded:

```
http://kennaroads.buzz/data/Update365/office365/flag.txt
```

The file contained an encoded string. Loading it into **CyberChef** and applying:

1. **Reverse** (the string was stored backwards)
2. **From Base64**

...decoded the hidden flag:

```
THM{pL4y_w1Th_tH3_URL}
```

> 💡 **Analyst note:** This step is a good reminder that attacker infrastructure often contains leftover debug/test artifacts (`flag.txt` here standing in for what might realistically be a leftover dev file). In real investigations, always enumerate exposed directories beyond just the obvious phishing page — misconfigured open directories frequently leak additional evidence.

---

## 9. Findings Summary

| # | Question | Answer |
|---|----------|--------|
| 1 | Individual who received the "Quote for Services Rendered" email | William McClean |
| 2 | Adversary's sending email address | `Accounts.Payable@groupmarketingonline.icu` |
| 3 | Root domain of the redirection URL | `kennaroads.buzz` |
| 4 | Company impersonated by the login page | Microsoft |
| 5 | Name of the archive file | `Update365.zip` |
| 6 | SHA256 hash of the archive | `ba3c15267393419eb08c7b2652b8b6b39b406ef300ae8a18fee4d16b19ac9686` |
| 7 | Additional threat category (besides Phishing) | Malware |
| 8 | Email address submitted more than once | `michael.ascot@swiftspend.finance` |
| 9 | Adversary's credential collection email | `m3npat@yandex.com` |
| 10 | Secondary adversary email found in kit source | `jamestanner2299@gmail.com` |
| 11 | Hidden flag (CyberChef decoded) | `THM{pL4y_w1Th_tH3_URL}` |

> ⚠️ *Consider redacting the flag value if this room is part of a paid/restricted TryHackMe path — some communities discourage publishing flags directly. Spoiler tags or a "request access" note are good alternatives.*

---

## 10. Indicators of Compromise (IOCs)

| Type | Value |
|------|-------|
| Phishing sender address | `Accounts.Payable@groupmarketingonline.icu` |
| Phishing sender domain | `groupmarketingonline.icu` |
| Malicious redirect domain | `kennaroads.buzz` |
| Phishing kit archive | `Update365.zip` |
| Archive SHA256 | `ba3c15267393419eb08c7b2652b8b6b39b406ef300ae8a18fee4d16b19ac9686` |
| Credential drop email (active) | `m3npat@yandex.com` |
| Credential drop email (secondary/legacy) | `jamestanner2299@gmail.com` |
| Confirmed compromised victim | `michael.ascot@swiftspend.finance` |
| Targeted employee | `william.mcclean@swiftspend.finance` |
| Targeted employee (referenced in URL) | `derick.marshall@swiftspend.finance` |
| First seen malicious (VirusTotal) | 2020-06-25 |

---

## 11. Lessons Learned

- **Open directory listings are a goldmine for analysts.** The attacker's misconfigured server allowed full enumeration of the phishing kit, victim logs, and source code — turning a single reported email into a full campaign takedown of evidence.
- **Hardcoded artifacts reveal attacker history.** The two separate collection email addresses (Gmail + Yandex) hinted at the kit's provenance — likely originally built or distributed by one actor and repurposed by another.
- **Password reuse across submission attempts confirms real compromise**, not just a logging artifact — this distinction matters when scoping an incident and deciding which accounts need forced resets.
- **Always check for stray files beyond the obvious phishing page.** The `flag.txt` discovery reinforces that attacker-hosted directories often contain more than the rendered phishing content.
- **`.icu` and similar cheap/loosely-regulated TLDs are a recurring pattern** in phishing infrastructure — useful as a triage heuristic, not a sole detection rule.

---

## 🧰 Tools Used

| Tool | Purpose |
|------|---------|
| Mozilla Thunderbird | Viewing email headers and source |
| CyberChef | SHA256 hashing, Base64 decoding, string reversal |
| VirusTotal | Hash reputation and first-seen timestamp lookup |
| `wget` | Downloading kit files and logs from the attacker's open directory |
| `grep` / `sort` / `uniq` | Parsing and analyzing victim credential logs |

---
