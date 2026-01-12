# Splunk Universal Forwarder Lab: Endpoint Log Ingestion

**Advanced SOC Program – Lab Documentation**

This lab demonstrates installing, configuring, and verifying **Splunk Universal Forwarders** on Windows and Linux (Kali) endpoints to forward logs into a Splunk Enterprise indexer.

- Windows: Security, System, and Application Event Logs
- Linux (Kali): System logs from `/var/log/*`

Author: David Beterib  
Program: Advanced SOC  
Date: January 2026

## Lab Synopsis

The lab successfully set up Splunk Enterprise on a Windows host as the indexer, installed Universal Forwarders on both Windows and Kali Linux endpoints, configured log monitoring, and verified ingestion via searches.

Key achievements:
- End-to-end log forwarding pipeline established
- Network connectivity and firewall rules validated
- Structured logs from both OS platforms visible in Splunk
- Troubleshooting confirmed: absence of failed login events (Event ID 4625) was due to no actual failed attempts occurring, not a configuration issue

## Lab Environment

- **Indexer**: Splunk Enterprise on Windows (IP: 192.168.0.168, receiving on TCP port 9997)
- **Windows Endpoint**: Local Windows machine with Universal Forwarder
  - Monitored channels: WinEventLog://Security, System, Application
  - Target index: `wineventlog`
- **Linux Endpoint**: Kali Linux VM (IP: 192.168.0.11, bridged networking)
  - Monitored path: `/var/log/*`
- Network: Local bridged/LAN setup
- Tools: Splunk Enterprise, Splunk Universal Forwarder (Windows & Linux), `nc` for connectivity testing, Event Viewer

## Step-by-Step Highlights

### 1. Splunk Enterprise & Web UI Access
- Started Splunk service on the local Windows machine
- Accessed the GUI at `http://127.0.0.1:8000`
- Used Search & Reporting app for verification searches (e.g., `index=wineventlog | head 20`)

### 2. Windows Universal Forwarder Setup
- Installed Splunk Universal Forwarder on Windows
- Created/edited `inputs.conf` in `C:\Program Files\SplunkUniversalForwarder\etc\system\local` to monitor:
  - WinEventLog://Security
  - WinEventLog://System
  - WinEventLog://Application
- Configured forwarding to indexer (192.168.0.168:9997)
- Created Windows Firewall inbound rule for TCP port 9997 ("Splunk 9997")
- Restarted SplunkForwarder service
- Verified: Successful events appeared in Splunk search

### 3. Linux (Kali) Universal Forwarder Setup
- Installed Universal Forwarder on Kali Linux VM
- Accepted license and set admin credentials
- Switched VM to bridged mode → IP 192.168.0.11
- Tested connectivity: `nc -zv 192.168.0.168 9997`
- Added forwarder: `splunk add forward-server 192.168.0.168:9997`
- Verified active forwarding: `splunk list forward-server`
- Configured monitoring of `/var/log` directory
- Verified: Logs from paths like `/var/log/Xorg.0.log`, `/var/log/lightdm`, `/var/log/apache2`, etc., ingested successfully

### 4. Log Ingestion Verification
- In Splunk Search:
  - Windows: `index=wineventlog` → saw Application/System/Security events (including successful logons – Event ID 4624)
  - Linux: `index=* host="kali" OR host="192.168.0.11"` → ~260 events from various `/var/log` sources
- Confirmed continuous forwarding from both forwarders

## Key Troubleshooting: Missing Failed Login Events (Event ID 4625)

No Event ID 4625 appeared in Splunk searches during testing.

**Root Cause** (not a Splunk or forwarder issue):
- Checked Windows Event Viewer → Security log: Only successful logons (4624) present; no failed authentication attempts recorded
- Windows only generates and logs failed logon events (4625) when actual failures occur (wrong password, lockouts, etc.)
- Since no failed attempts were made during the lab (locally or remotely), no 4625 events existed to forward
- Configuration was correct: inputs.conf enabled Security log monitoring, successful 4624 events confirmed the pipeline worked

**Conclusion**: This is expected behavior ("garbage in, garbage out"). To see 4625 events, intentionally generate failed logins (e.g., wrong password attempts), and they will appear in Splunk.

## Screenshots (Evidence)

All screenshots are included in the attached PDF file (`SPLUNK_IMPLEMENTATION.pdf`):

- Splunk Web UI and search results
- inputs.conf configuration
- Forwarder installation outputs
- Firewall rule setup
- Connectivity tests (`nc`)
- Splunk searches showing ingested Windows & Linux events
- Event Viewer confirming no 4625 events

(Refer to the PDF for full-resolution images and detailed context.)

## Key Learnings

- Full log collection pipeline: source generation → forwarder collection → network transmission → indexing → searching
- Configuration files: `inputs.conf` for data sources, outputs for forwarding
- Network essentials: Firewall rules (TCP 9997), connectivity testing
- Verification SPL queries: `index=`, `host=`, `source=`, `sourcetype=`
- Debugging mindset: Check the source first (Event Viewer/syslog) before assuming tool failure

## Future Enhancements (Ideas)

- Enable Sysmon on Windows for richer endpoint logging
- Create custom indexes and field extractions
- Set up basic Splunk alerts for security events
- Add modular inputs or scripted inputs for more sources

This lab serves as hands-on proof of concept for centralized log management in a SOC environment.

