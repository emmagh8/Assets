# Cyber Defence Frameworks

## Overview

In this section, I learned several frameworks used by SOC analysts to understand attacker behavior, investigate incidents, and improve detection strategies.

The main frameworks covered were:

- Pyramid of Pain
- Cyber Kill Chain
- Threat Modelling
- Unified Kill Chain
- MITRE ATT&CK
- Cyber Analytics Repository (CAR)
- D3FEND

Although each framework has a different purpose, they all help defenders understand how attackers operate and how security teams can detect or disrupt their activities.

---

## Pyramid of Pain

The Pyramid of Pain was created by David J. Bianco and explains how difficult it is for attackers to change different indicators when defenders detect them.

The higher we move in the pyramid, the more impact our detection has on the attacker.

| Level | Example | Difficulty for Attacker |
|---------|---------|---------|
| Hash Values | SHA256 | $\color{#55FF00}{\text{Very Low}}$ |
| IP Addresses | C2 Server IP | $\color{#00FF55}{\text{Low}}$ |
| Domain Names | Malicious Domain | $\color{#FFFF00}{\text{Medium}}$ |
| Network / Host Artifacts | Registry Keys, File Paths | $\color{#ffa500}{\text{High}}$ |
| Tools | Mimikatz, Malware Families | $\color{#FF2A00}{\text{Very High}}$  |
| TTPs | Credential Dumping, Lateral Movement | $\color{#FF0000}{\text{Extreme}}$ |

---

## Cyber Kill Chain

The Cyber Kill Chain was developed by Lockheed Martin to describe how attacks progress from the initial planning stage to achieving the attacker's objectives.

### Attack Stages

1. Reconnaissance
2. Weaponization
3. Delivery
4. Exploitation
5. Installation
6. Command & Control
7. Actions on Objectives

### Example

A phishing email attack could follow these stages:

- Reconnaissance → Collect employee emails
- Weaponization → Create malicious document
- Delivery → Send phishing email
- Exploitation → Victim opens attachment
- Installation → Malware installed
- Command & Control → Malware contacts attacker
- Actions on Objectives → Data theft

The Kill Chain helps SOC analysts understand where an attack is occurring and where defensive controls can stop it.

---

## Threat Modelling

Threat modelling is the process of identifying threats, weaknesses, and possible attack paths before an attack happens.

The goal is not only to detect attacks but also to reduce risk in advance.

### Basic Process

1. Identify assets
2. Identify threats
3. Assess risks
4. Define mitigations
5. Improve security controls

---

## Unified Kill Chain

Unified Kill Chain extends the traditional Cyber Kill Chain by focusing on what attackers do after gaining access to a network.

### Main Phases

#### Initial Foothold

- Reconnaissance
- Social Engineering
- Exploitation
- Persistence

#### Network Propagation

- Discovery
- Credential Access
- Privilege Escalation
- Lateral Movement
- Pivoting

#### Actions on Objectives

- Collection
- Exfiltration
- Impact

The traditional Kill Chain mainly focuses on the initial compromise.

Unified Kill Chain provides a broader view of attacker behavior inside the network and helps analysts understand how attacks spread.

---

## MITRE ATT&CK

MITRE ATT&CK is a knowledge base of attacker tactics and techniques based on real-world observations.

It helps defenders understand how attackers operate.

### Key Concepts

#### Tactics

The attacker's goal.

Example:

- Credential Access
- Persistence
- Lateral Movement

#### Techniques

The method used to achieve the goal.

Example:

- Brute Force
- Pass the Hash
- PowerShell

MITRE ATT&CK provides a common language for security teams and helps map alerts to known attacker techniques.

---

## Cyber Analytics Repository (CAR)

CAR was developed by MITRE to provide detection analytics that can be mapped to ATT&CK techniques.

If ATT&CK explains attacker behavior, CAR helps defenders build detections for that behavior.

---

## D3FEND

D3FEND = Detection, Denial, and Disruption Framework Empowering Network Defense.
D3FEND is a framework focused on defensive techniques.

While ATT&CK describes offensive activities, D3FEND describes defensive countermeasures.

| ATT&CK | D3FEND |
|---------|---------|
| Attacker View | Defender View |
| Attack Techniques | Defensive Techniques |
| Offensive Actions | Defensive Controls |

D3FEND helps defenders understand which security controls can be used to counter specific attacker techniques.

---

# Key Takeaways

- The Pyramid of Pain shows which detections create the most pressure on attackers.
- Cyber Kill Chain explains how attacks progress from start to finish.
- Threat Modelling helps identify risks before attacks occur.
- Unified Kill Chain provides deeper visibility into attacker activity after compromise.
- MITRE ATT&CK helps analysts understand and classify attacker behavior.
- CAR supports building detections based on ATT&CK techniques.
- D3FEND focuses on defensive strategies and countermeasures.

This section helped me understand how SOC analysts use different frameworks to detect threats, investigate incidents, and improve security operations.
