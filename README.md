
# Sentinel RDP ThreatTracker 

 
Overview 

The Sentinel RDP ThreatTracker project demonstrates the implementation of a Security Information and Event Management (SIEM) system using Microsoft Sentinel in conjunction with Azure services. The goal of this project was to monitor, analyze, and visualize failed Remote Desktop Protocol (RDP) login attempts, leveraging advanced tools for log collection, enrichment, and visualization. 

This setup highlights key cybersecurity practices such as deploying honeypots, configuring logging pipelines, enriching log data with geolocation, and creating custom dashboards for threat tracking. Below is an outline of the steps and components implemented to bring this system to life. 

 

 

 

 
## 1. Setting Up a Virtual Machine
Created a virtual machine (VM) in Azure as the foundation for this project: 

Assigned it to the Honeypot-RG resource group under Azure Subscription 1. 

Named the VM honeypot-vm and deployed it to the Australia Central region (chose this region because most of the others were not available). 

Used the Windows 10 Pro, Version 22 H2 Gen 1 image with a Standard D2s_v3 configuration (2 vcpus, 8 GiB memory). 

Configured RDP (3389) as the only allowed inbound port. 

For security, I created a custom network security group and added a rule (DANGER_ANY_IN) to allow traffic for testing purposes. Once reviewed, I deployed the VM.
## 2. Creating a Log Analytics Workspace 
Created a Log Analytics Workspace named law-honeypot within the same resource group and set its region to Australia Central. This workspace would later be connected to the VM for log collection. 
## 3. Configuring Microsoft Defender for Cloud
In Microsoft Defender for Cloud, I enabled foundational Cloud Security Posture Management (CSPM) and server protection while turning off SQL Server monitoring to simplify the setup. For data collection, I selected All Events, ensuring that comprehensive logs were gathered. 
## 4. Connecting the VM to Log Analytics Workspace 
Once the workspace and VM were ready, I connected them. This involved linking the honeypot-vm to law-honeypot through the Azure portal. This step allowed the VM's logs to flow directly into the workspace.
## 5. Setting Up Microsoft Sentinel
With the workspace configured, I added Microsoft Sentinel to law-honeypot. This marked the beginning of setting up advanced monitoring capabilities. 
## 5. Setting Up Microsoft Sentinel
With the workspace configured, I added Microsoft Sentinel to law-honeypot. This marked the beginning of setting up advanced monitoring capabilities. 
## 6. Configuring the VM Environment 
To prepare the VM for generating and capturing relevant data: 

I logged in using Remote Desktop and accessed the Event Viewer under Windows Logs > Security. 

Used ipgeolocation.io to get an API key for log enrichment. 

![App Screenshot](https://i.imgur.com/xQHYbe4.png)

Pinned the VM’s IP and ran ping -t commands to simulate failed connection attempts. 

Disabled all firewall profiles (Domain, Private, Public) to allow unrestricted access for testing. 
## 7. Creating Custom_Security_Log_Exporter.ps1 
This PowerShell script, Custom_Security_Log_Exporter.ps1, is designed to monitor and log failed Remote Desktop Protocol (RDP) login attempts on a Windows system. It uses the Windows Event Viewer to filter and extract specific security events, enriches the data with geolocation information, and writes the results to a custom log file. 

Here's an explanation of its functionality:

Setup and Configuration: 

The script requires an API key from ipgeolocation.io to retrieve geolocation data based on IP addresses. 

A log file named failed_rdp.log is created in the C:\ProgramData directory if it does not already exist. Sample log entries are generated initially to assist in training the Log Analytics workspace for field extraction. 

Event Filtering: 

It uses an XML filter to search the Windows Event Viewer for Event ID 4625, which corresponds to failed login attempts. These events are categorized under the Security log. 

Infinite Monitoring Loop: 

The script continuously runs, checking for new failed login events in the Event Viewer. It uses an infinite loop with a short delay between iterations to prevent excessive system resource usage. 

Processing Events: 

For each event, the script extracts relevant details, including: 

Timestamp of the event. 

Destination host (the system where the failed login attempt occurred). 

Username used in the failed login attempt. 

Source IP address. 

It ensures that duplicate entries are not written to the log file by checking if an event with the same timestamp already exists. 

Geolocation Enrichment: 

The script queries the ipgeolocation.io API to retrieve the geographical location (latitude, longitude, state/province, and country) of the source IP address involved in the failed login attempt. 

To avoid rate-limiting by the API, the script pauses briefly before making each API request. 

Log Creation and Writing: 

If a valid geolocation response is obtained, the script formats the event data into a string and appends it to the custom log file. Each log entry includes: 

Latitude and longitude. 

Destination host. 

Username. 

Source IP address and host. 

State/province and country. 

A label combining the country name and source IP. 

The timestamp of the event. 

If the event is already logged, the script skips it to avoid redundancy. 

Sample Logs and Training: 

To train Log Analytics or other tools, the script initially writes fake sample records to the log file. These sample logs use a placeholder destination host (samplehost) and can be filtered out when analyzing the data. 

![App Screenshot](https://i.imgur.com/3K2s60s.png)


## 8. Creating a Custom Log in Log Analytics Workspace  

To process the generated logs: 

Created a custom log in law-honeypot using the uploaded failure_rdp.log file. 

Configured the collection path as C:\ProgramData|failure_rdp.log. 

Named the log details FAILURE_RDP_GEO. 

This setup allowed the workspace to process and categorize data from the log file effectively.



## 9. Visualizing Data with Sentinel Workbooks 
Customized a Sentinel Workbook for visualizing activity: 

Removed pre-existing charts and added a custom query that produces a summarized table where each row represents a unique combination of: 

Event details (timestamp, label, source/destination hosts, username) 

Geographic information (country, state, coordinates) 

For each combination, the number of events (event_count) is displayed. 

![App Screenshot](https://i.imgur.com/wZSOLqO.png)

Adjusted the query to reference FAILURE_RDP_GEO, matching the custom log I had created. 

Configured the visualization to display a map of events, changing the metric label to display country names. 

![App Screenshot](https://i.imgur.com/9TeItcn.png)
