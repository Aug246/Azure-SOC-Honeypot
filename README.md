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
![Cloud Honeynet / SOC](https://i.imgur.com/RdPR7op.png)

- This machine is discoverable on the internet and malicious actors can attempt to log into the computer
- There is essentially no Network Security Group because I created an inbound security rule that allows all traffic into the VM
- All of the traffic including failed log in attempts are forwared to Log Analytics Workspace (LAW) as logs with EventID 4625
- Sentinel uses the LAW and a custom watchlist which can be found [here](https://drive.google.com/file/d/13EfjM_4BohrmaxqXZLB5VUBIz2sv9Siz/view) to filter through the logs with a custom query to find attackers and their locations.
```KQL
    let GeoIPDB_FULL = _GetWatchlist("geoip");
    let WindowsEvents = SecurityEvent;
    WindowsEvents | where EventID == 4625
    | order by TimeGenerated desc
    | evaluate ipv4_lookup(GeoIPDB_FULL, IpAddress, network)
    | summarize FailureCount = count() by IpAddress, latitude, longitude, cityname, countryname
    | project FailureCount, AttackerIp = IpAddress, latitude, longitude, city = cityname, country = countryname,
    friendly_location = strcat(cityname, " (", countryname, ")");
```
- This query is then used in a Sentinel workbook to create a map of the attack in the form of json text
```JSON
{
	"type": 3,
	"content": {
	"version": "KqlItem/1.0",
	"query": "let GeoIPDB_FULL = _GetWatchlist(\"geoip\");\nlet WindowsEvents = SecurityEvent;\nWindowsEvents | where EventID == 4625\n| order by TimeGenerated desc\n| evaluate ipv4_lookup(GeoIPDB_FULL, IpAddress, network)\n| summarize FailureCount = count() by IpAddress, latitude, longitude, cityname, countryname\n| project FailureCount, AttackerIp = IpAddress, latitude, longitude, city = cityname, country = countryname,\nfriendly_location = strcat(cityname, \" (\", countryname, \")\");",
	"size": 3,
	"timeContext": {
		"durationMs": 2592000000
	},
	"queryType": 0,
	"resourceType": "microsoft.operationalinsights/workspaces",
	"visualization": "map",
	"mapSettings": {
		"locInfo": "LatLong",
		"locInfoColumn": "countryname",
		"latitude": "latitude",
		"longitude": "longitude",
		"sizeSettings": "FailureCount",
		"sizeAggregation": "Sum",
		"opacity": 0.8,
		"labelSettings": "friendly_location",
		"legendMetric": "FailureCount",
		"legendAggregation": "Sum",
		"itemColorSettings": {
		"nodeColorField": "FailureCount",
		"colorAggregation": "Sum",
		"type": "heatmap",
		"heatmapPalette": "greenRed"
		}
	}
	},
	"name": "query - 0"
}
```
## Attack Maps & Findings

<img width="2900" height="1624" alt="image" src="https://github.com/user-attachments/assets/b7578232-ea8e-46b3-a4af-28785f8f6160" />



- **World Map of Attacks** - Displays source IP geolocations
- **Heatmaps** - Show attack frequency and intensity
- **Failed Logins (Event ID 4625)** - Highlight brute-force attempts

### Observation: 
- One attacker attempted **3.11k** times log into the exposed VM
- IP geolocation confirmed global scanning activity.

## Conclusion
To conclude this project, I built a vulnerable cloud-based honeypot using Microsft Azure. Microsoft Sentinel was then to capture and analyze attacker behaviour and display it on a custom Sentinel SIEM Dashboard to track real world attacks. If the virtual honey pot were left to be running for days it would detect attackers from all over the globe.

Someways that I can go about improving this system would be incorporate a honeynet, so that more attackers and drawn in. Also automated incident response along with real-time alterting for SOC workflows would better imitate a real life scenario. 

