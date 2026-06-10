# Core SOC Solutions

## Overview

The purpose of this section was to understand the core security solutions used by SOC teams. I learned the role of EDR, SIEM, and SOAR, how they work together, and how SOC analysts use them to detect, investigate, and respond to security incidents.

---

## Introduction to EDR

EDR (Endpoint Detection and Response) is a security solution designed to monitor, detect, and respond to threats at the endpoint level.

An endpoint can be:

- Laptop
- Desktop
- Server
- Mobile device

The name EDR can be understood through its three components:

### Endpoint

Endpoints are the devices where users interact with company resources and data.

### Detection

Unlike traditional antivirus solutions that mainly rely on signatures, EDR focuses on monitoring suspicious behaviors and activities in real time.

### Response

Once a threat is detected, the EDR allows security teams to take actions such as:

- Isolating a device from the network
- Killing a malicious process
- Containing the threat before it spreads

---

## EDR vs Antivirus

Why do we need EDR when we have AV !
Both Antivirus (AV) and EDR aim to protect endpoints, However they provide different levels of visibility and protection, AV detecd some basic threats but to detect advanced threats use EDR is the solution.
One important concept I learned is that if a suspicious file is detected on one endpoint, an EDR can quickly identify whether the same file exists on other endpoints in the organization.

---

## How EDR Works

An EDR solution typically consists of two main components:

### EDR Agents

Agents or sensors are installed on endpoints.

Their responsibilities include:

- Monitoring activities
- Collecting endpoint telemetry
- Sending data to the central console

The agents act as the eyes and ears of the EDR.

### EDR Console

The EDR console receives data from all agents and acts as the brain of the system.

It:

- Correlates collected events
- Uses threat intelligence
- Applies analytics and machine learning
- Detects suspicious activity

---

## Introduction to SIEM

SIEM stands for Security Information and Event Management.

A SIEM is a security solution that:

- Collects logs from multiple sources
- Normalizes log formats
- Correlates events
- Applies detection rules
- Generates alerts

The SIEM acts as a centralized platform where SOC analysts can monitor and investigate security events.

---

## Log Sources

Every system generates logs whenever an activity occurs.

Common log sources include:

### Windows Systems

Windows records events through Event Viewer and assigns Event IDs to different activities.

### Linux Systems

Common Linux log locations include:

- `/var/log/auth.log`
- `/var/log/secure`
- `/var/log/cron`
- `/var/log/httpd`
- `/var/log/kern`

### Web Servers

Web servers generate logs containing:

- Requests
- Responses
- Errors
- Access information

Monitoring these logs helps identify web attacks and suspicious activity.

---

## Log Ingestion Methods

SIEM platforms support multiple methods for collecting logs.

### Agent / Forwarder

A lightweight agent is installed on endpoints and forwards logs to the SIEM.

**Example:**
- Splunk Universal Forwarder

### Syslog

A common protocol used to send logs from servers and network devices to a centralized location.

### Manual Upload

Some SIEM platforms allow analysts to upload offline log files for analysis.

### Port Forwarding

Systems can be configured to send logs directly to a listening SIEM port.

---

## Detection Rules and Alert Generation

SIEM solutions generate alerts through detection rules.

These rules are based on logical conditions.

Examples:

- Multiple failed login attempts
- Successful login after several failed attempts
- USB device insertion
- Large outbound data transfers

### Example Detection Rule

Windows generates Event ID 104 whenever event logs are cleared.

A detection rule can be:

"If Log Source = WinEventLog and Event ID = 104 → Generate "Event Log Cleared" alert"

This helps detect attackers attempting to remove evidence of their actions.

---

## Splunk Basics

In this section, I explored the basic Splunk architecture and components.

### Splunk Forwarder

The Forwarder is installed on endpoints and is responsible for collecting and forwarding logs.

### Splunk Indexer

The Indexer:

- Receives data
- Parses logs
- Normalizes fields
- Stores indexed data

This makes searching and analysis much easier.

### Search Head

The Search Head is where analysts perform searches using SPL (Search Processing Language).

When a search is executed:

1. The request is sent to the Indexer.
2. Matching events are retrieved.
3. Results are presented to the analyst.

---

## Splunk Lab Experience
<div align="center">
  <p>Inside the SOC: The Incident Response Pipeline.</p>
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Solutions/Capture1.PNG?raw=true">
</div>

During the lab, I:

- Uploaded log files into Splunk
- Explored the Splunk interface
- Practiced using SPL queries
- Investigated alerts
- Assigned verdicts based on findings
<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Solutions/Capture2.PNG?raw=true">
</div>
<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Solutions/Capture3.PNG?raw=true">
</div>

This helped me understand how analysts use Splunk to investigate security events and make triage decisions.

---

## Elastic Stack (ELK)

Elastic Stack is an open-source platform that provides capabilities similar to Splunk.

Its main purpose is to:

- Collect data
- Process logs
- Store information
- Visualize security events

### Core Components

#### Elasticsearch

Elasticsearch is a search and analytics engine responsible for:

- Storing data
- Indexing logs
- Performing fast searches
- Supporting threat hunting and investigations

#### Logstash

Logstash is a data processing pipeline that:

- Collects logs from multiple sources
- Filters and transforms data
- Normalizes log formats
- Sends data to Elasticsearch

A typical Logstash configuration contains:

- Input
- Filter
- Output

#### Beats

Beats are lightweight agents used to collect and ship data from endpoints to Elasticsearch.

Common examples include:

- Filebeat
- Winlogbeat
- Auditbeat
- Packetbeat

#### Kibana

Kibana is the visualization layer of the Elastic Stack.

SOC analysts use Kibana to:

- Search events
- Investigate alerts
- Build dashboards
- Monitor security activity

---

## Splunk vs Elastic Stack

Both platforms are widely used in SOC environments.

| Feature | Splunk | Elastic Stack |
|----------|---------|--------------|
| Licensing | Commercial | Open Source |
| Log Collection | Forwarders | Beats |
| Search Engine | Splunk Search | Elasticsearch |
| Visualization | Splunk Dashboards | Kibana |
| Complexity | Easier to start | More flexible but requires more setup |

Both solutions help SOC analysts collect, search, investigate, and visualize security events.

---

## Introduction to SOAR

SOAR stands for:

**Security Orchestration, Automation, and Response**

SOAR platforms help SOC teams manage investigations and response actions through a centralized interface.

Instead of manually switching between:

- SIEM
- EDR
- Firewall
- Threat Intelligence Platforms
- Ticketing Systems

SOC analysts can work from a single platform.

SOAR also provides:

- Case management
- Ticketing
- Incident tracking
- Automated response workflows

---

## Why Is It Called SOAR?

### Security Orchestration

Orchestration connects multiple security tools together and coordinates actions between them.

It uses predefined workflows called **Playbooks**.

### Example Playbook

1. Receive alert from SIEM
2. Check user activity history
3. Verify IP reputation
4. Search for successful logins
5. Escalate the case if required

This allows analysts to follow a consistent investigation process.

---

### Security Automation

Automation reduces repetitive manual work by executing predefined actions automatically.

### Example

When a VPN brute-force alert is generated:

1. Query SIEM automatically
2. Check threat intelligence feeds
3. Review user activity
4. Disable compromised account
5. Create an incident ticket

Without automation, analysts would need to perform each step manually.

---

### Security Response

Response focuses on containing and mitigating threats.

SOAR platforms can perform actions such as:

- Blocking malicious IP addresses
- Disabling user accounts
- Creating incident tickets
- Isolating infected hosts
- Triggering containment procedures

This helps SOC teams respond faster to security incidents.

---

## How EDR, SIEM, and SOAR Work Together

The three technologies complement each other during security operations.
<div align="center">
  <p>Inside the SOC: The Incident Response Pipeline.</p>
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Solutions/Alert.png?raw=true">
</div>


### Example Workflow

1. A user downloads a malicious file.
2. The EDR agent detects suspicious behavior.
3. Events are sent to the SIEM.
4. The SIEM correlates logs and generates an alert.
5. The analyst investigates the alert.
6. SOAR automatically blocks the threat and opens an incident case.

---

## SOAR Lab Experience

<div align="center">
  <img src="https://github.com/emmagh8/Assets/blob/main/SOC_L1/Solutions/CaptureSOAR.PNG?raw=true">
</div>

During the lab, I:
1. Launched the interactive split-screen SOAR workflow simulation.
2. Set up a Case Ticket to document and track the security incident
3. Configured Threat Intelligence (TI) Feeds to pull in relevant threat data
4. Performed Incident Data Extraction to parse and isolate key indicators
5. Ran Reputation Checks against extracted IOCs to assess their threat level
6. Defined a Course of Action to respond based on analysis results
7. Executed the full workflow by running it and verifying a smooth flowchart transition

This helped me understand how SOC analysts use SOAR platforms to automate and orchestrate threat intelligence workflows, reducing manual effort and enabling faster, more consistent incident response decisions.

## Lab Experience

During this section, I explored:

- EDR concepts and architecture
- SIEM fundamentals
- Splunk architecture
- Elastic Stack components
- SOAR workflows and automation


This helped me understand how SOC teams use different security solutions together to detect and respond to threats.

---

## Key Takeaways

- EDR provides endpoint visibility, detection, and response capabilities.
- SIEM centralizes logs and generates alerts through correlation and detection rules.
- Splunk uses Forwarders, Indexers, and Search Heads to process and analyze data.
- Elastic Stack provides an open-source alternative for log management and security monitoring.
- SOAR integrates multiple security tools into a single workflow.
- Automation reduces repetitive tasks and accelerates investigations.
- Orchestration connects security tools and standardizes response processes.
- Understanding how EDR, SIEM, and SOAR work together is essential for SOC operations.
- Hands-on Splunk practice helped me understand how alerts are investigated and classified in a SOC environment.
