# Azure-SOC-Honeypot
## Introduction

This project demonstrates the deployment of a honeypot-based Security Operations Center (SOC) in Microsoft Azure to capture, analyze, and visualize real-world cyberattacks. By intentionally exposing a vulnerable virtual machine (VM) to the internet, I collected logs of malicious activity, integrated them into Microsoft Sentinel, and built dashboards to monitor and understand attacker behavior.

The project highlights my ability to:
- Design and configure cloud-based security environments.
- Collect, analyze, and correlate security event data.
- Use visualization tools to track adversary behavior in real time.

## Objective

The main objective of this projects where

1. Deploy a honeypot environment in Azure to attract real-world attackers.
2. Capture and centralize logs from a vulnerable VM into a log analytics workspace.
3. Correlate and enrich log data with external watchlists (geolocation/IPs).
4. Build visual dashboards and attack maps in Sentinel to showcase attack activity.

## Tools and Regulations Employed

### Technologies & Tools
- Microsoft Azure (Resource Groups, Virtual Networks, Virtual Machines)
- Microsoft Sentinel (SIEM)
- Log Analytics Workspace
- Windows Event Viewer
- KQL (Kusto Query Language)
- Geolocation IP watchlists

### Regulations & Security Frameworks Referenced
- Logging and monitoring principles from **NIST CSF** and **ISO 27001**.
- Incident detection and analysis best practices aligned with **MITRE ATT&CK**.

### Azure Components
Resource Group – Organized project assets.
- Virtual Network (VNet) – Network for honeypot VM.
- Virtual Machine (VM) – Windows VM configured as the honeypot.
- Network Security Group (NSG) – Modified rules to allow all inbound traffic.
- Log Analytics Workspace – Centralized log repository.
- Microsoft Sentinel – SIEM for log ingestion, correlation, and visualization.

## Methodology
### 1. Environment Setup

- Created an Azure Resource Group and Virtual Network.
- Deployed a Windows Virtual Machine.
- Disabled default protections (Windows Firewall + restrictive NSG rules) to ensure it acted as an effective honeypot.
  
### 2. Logging & Monitoring
- Used Windows Event Viewer to observe failed login attempts (Event ID: 4625).
- Created a Log Analytics Workspace.
- Connected the VM to the workspace via the Azure Monitoring Agent.
  
### 3. Sentinel Integration
- Deployed Microsoft Sentinel and linked it to the Log Analytics Workspace.
- Installed the Windows Security Events Connector to collect authentication logs.Built a custom watchlist with geolocation data tied to IP addresses.

### 4. Visualization
- Created KQL queries to filter and enrich failed login data.
- Developed Sentinel workbooks to display attack heat maps and logon attempt dashboards.

## SOC Architechture
![Cloud Honeynet / SOC](https://imgur.com/a/soc-hJnR58l)

            ┌──────────────────────────┐
            │        Internet           │
            └─────────────┬────────────┘
                          │
                  Inbound Attacks
                          │
            ┌─────────────▼────────────┐
            │      Honeypot VM         │
            │  (Windows, no firewall)  │
            └─────────────┬────────────┘
                          │
                   Security Logs
                          │
            ┌─────────────▼────────────┐
            │ Log Analytics Workspace  │
            └─────────────┬────────────┘
                          │
                  Data Ingestion
                          │
            ┌─────────────▼────────────┐
            │    Microsoft Sentinel    │
            │ Dashboards & Analytics   │
            └──────────────────────────┘

## Attack Maps & Findings

![Attack Map](https://imgur.com/a/attack-map-5f70cU1)

- **World Map of Attacks** - Displays source IP geolocations
- **Heatmaps** - Show attack frequency and intensity
- **Failed Logins (Event ID 4625)** - Highlight brute-force attempts

### Observation: 
- One attacker attempted **3.11k** times log into the exposed VM
- IP geolocation confirmed global scanning activity.

## Conclusion
To conclude this project, I built a vulnerable cloud-based honeypot using Microsft Azure. Microsoft Sentinel was then to capture and analyze attacker behaviour and display it on a custom Sentinel SIEM Dashboard to track real world attacks. If the virtual honey pot were left to be running for days it would detect attackers from all over the globe.

Someways that I can go about improving this system would be incorporate a honeynet, so that more attackers and drawn in. Also automated incident response along with real-time alterting for SOC workflows would better imitate a real life scenario. 

