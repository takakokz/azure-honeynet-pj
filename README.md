# Building a SOC + Honeynet in Azure (Live Traffic)
![Smaller_Overview](https://github.com/takakokz/azure-honeynet-pj/assets/13964231/a1a51dab-73e6-4b30-8a66-45467779e19d)

## Overview

In this project, I build a mini honeynet in Azure and ingest log sources from various resources into a Log Analytics workspace, which is then used by Microsoft Sentinel to build attack maps, trigger alerts, and create incidents. I measured some security metrics in the insecure environment for 24 hours, apply some security controls to harden the environment, measure metrics for another 24 hours, then show the results below. 

- SecurityEvent (Windows Event Logs)
- Syslog (Linux Event Logs)
- SecurityAlert (Log Analytics Alerts Triggered)
- SecurityIncident (Incidents created by Sentinel)
- AzureNetworkAnalytics_CL (Malicious Flows allowed into our honeynet)

## Objective

Objectives of this project is to allow me to conduct:

- Consolidate logs generated from various sources into one SEIM
- Analyze attacks from all over the world in real time
- Investigate and learn attackers technique and procedure
- Configure firewall to create vulnerable and hardened environemnt
- Implement Next Generation Security (NGS)
- Apply configurations recommended by NIST800-31 and Microsoft Defender for Cloud

## Method
After honeynet is created and configured to ingest logs into a SEIM, I have left environments vulnerable to cyberattacks for 24 hours.
I have investigated critical attacks to learn attackers' techniques then configured security control recommended by built-in recommendations.
The honeynet is strengthened and number of logs with critical alert dramatically decreased after security hardening.

## Azure Technology Deployed and Related Standard
[Azure Virtual Network](https://learn.microsoft.com/en-us/azure/virtual-network/virtual-networks-overview)  
[Azure Resource Group](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/manage-resource-groups-portal)  
[Azure Key vault](https://azure.microsoft.com/en-us/products/key-vault)  
[Azure Blob](https://azure.microsoft.com/en-us/products/storage/blobs)  
[Network Security Group](https://learn.microsoft.com/en-us/azure/virtual-network/network-security-groups-overview)  
[Overview of Azure platform logs](https://learn.microsoft.com/en-us/azure/azure-monitor/essentials/platform-logs-overview)    
[Microsoft Sentinel](https://azure.microsoft.com/en-us/products/microsoft-sentinel)  
[Microsoft Sentinel Watchlist](https://learn.microsoft.com/en-us/azure/sentinel/watchlists))  
[Microsoft Defender for Cloud](https://learn.microsoft.com/en-us/azure/defender-for-cloud/defender-for-cloud-introduction)  
[Write SQL Server Audit events to the Security log]([https://azure.microsoft.com/en-us/products/key-vault](https://learn.microsoft.com/en-us/sql/relational-databases/security/auditing/write-sql-server-audit-events-to-the-security-log?view=sql-server-ver16))  
Virtual Machine (2 Windows, 1 Linux)
NIST800-31

## Architecture Before Hardening / Security Controls
![Before Hardening](https://github.com/takakokz/azure-honeynet-pj/assets/13964231/6a2fdcfc-3e83-4aa2-b97f-2e378b924eb6)  
This diagram shows Honeynet which is vulnerable to attacks from internet.  
I created a Windows VM and a Linux VM then added rules to open all ports and traffics to both of their Network Security Groups. The firewall was diabled on Windows VM to permit all traffics as well.  

After leaving this vulnerable honeynet run for 24 hours, numbers of logs are generated by attackers. I also manually created logs using an another VM for some types of attacks could not be generated from attackers.

## Architecture After Hardening / Security Controls
![Architecture Diagram](https://github.com/takakokz/azure-honeynet-pj/assets/13964231/f727efca-def1-4c76-964a-0c5e0f2fca3d)  

The architecture of the mini honeynet in Azure consists of the following components:

- Virtual Network (VNet)
- Network Security Group (NSG)
- Virtual Machines (2 windows, 1 linux)
- Log Analytics Workspace
- Azure Key Vault
- Azure Storage Account
- Microsoft Sentinel

For the "BEFORE" metrics, all resources were originally deployed, exposed to the internet. The Virtual Machines had both their Network Security Groups and built-in firewalls wide open, and all other resources are deployed with public endpoints visible to the Internet; aka, no use for Private Endpoints.

For the "AFTER" metrics, Network Security Groups were hardened by blocking ALL traffic with the exception of my admin workstation, and all other resources were protected by their built-in firewalls as well as Private Endpoint

## Attack Maps Before Hardening / Security Controls
![NSG Allowed Inbound Malicious Flows](https://i.imgur.com/1qvswSX.png)<br>
![Linux Syslog Auth Failures](https://i.imgur.com/G1YgZt6.png)<br>
![Windows RDP/SMB Auth Failures](https://i.imgur.com/ESr9Dlv.png)<br>

## Metrics Before Hardening / Security Controls

The following table shows the metrics we measured in our insecure environment for 24 hours:
Start Time 2023-03-15 17:04:29
Stop Time 2023-03-16 17:04:29

| Metric                   | Count
| ------------------------ | -----
| SecurityEvent            | 19470
| Syslog                   | 3028
| SecurityAlert            | 10
| SecurityIncident         | 348
| AzureNetworkAnalytics_CL | 843

## Attack Maps Before Hardening / Security Controls

```All map queries actually returned no results due to no instances of malicious activity for the 24 hour period after hardening.```

## Metrics After Hardening / Security Controls

The following table shows the metrics we measured in our environment for another 24 hours, but after we have applied security controls:
Start Time 2023-03-18 15:37
Stop Time	2023-03-19 15:37

| Metric                   | Count
| ------------------------ | -----
| SecurityEvent            | 8778
| Syslog                   | 25
| SecurityAlert            | 0
| SecurityIncident         | 0
| AzureNetworkAnalytics_CL | 0

## Incident / Alert Setting
- <b>Brute Force Success Windows</b>  
If a person with same IP address failed login to Windows VM more than 5 times and a seccessful login history in past one hour, alert as an incident 
Come from Security Event Log  
Event ID = 4625 (Fail) and 4624 (Success)  
![SecurityEvent](https://github.com/takakokz/azure-honeynet-pj/assets/13964231/8118485c-1501-4708-81c7-344e4ee4d5a6)  
Someone in LA, California successfully login to Windows VM 5 times after several failed attempts  

- <b>Brute Force Attempt MS SQL Server</b>  
If a person with same IP address failed login to MS SQL server more than 10 times, alert as an incident.    
Come from Application Log  
Event ID = 18456 (Failed Login)  
![SQL Bruce Force](https://github.com/takakokz/azure-honeynet-pj/assets/13964231/a1ebf00e-01c2-46ee-bc19-651f0ed552db)  
Someone in Africa tried to login to MS SQL server for 6 times

- <b>CUSTOM: Malware Detected</b>  
If file with Malware is placed on the computer, alert as an insident  
![Malware](https://github.com/takakokz/azure-honeynet-pj/assets/13964231/d6a2a05e-145a-4f88-94ca-9c15b91e3382)  


Table name = Event  
EventLog == "Microsoft-Windows-Windows Defender/Operational"  
EventID == "1116" or EventID == "1117"

## Conclusion

In this project, a mini honeynet was constructed in Microsoft Azure and log sources were integrated into a Log Analytics workspace. Microsoft Sentinel was employed to trigger alerts and create incidents based on the ingested logs. Additionally, metrics were measured in the insecure environment before security controls were applied, and then again after implementing security measures. It is noteworthy that the number of security events and incidents were drastically reduced after the security controls were applied, demonstrating their effectiveness.

It is worth noting that if the resources within the network were heavily utilized by regular users, it is likely that more security events and alerts may have been generated within the 24-hour period following the implementation of the security controls.
