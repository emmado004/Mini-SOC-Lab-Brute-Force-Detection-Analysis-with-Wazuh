# Mini-SOC-Lab-Brute-Force-Detection-Analysis-with-Wazuh
SOC Lab: Brute Force Detection with Wazuh SIEM. Configured Windows/Linux endpoints to monitor RDP &amp; SSH attacks. Simulated threats via Kali Linux, analyzed security events (PCI DSS, MITRE), and implemented logging hardening. Focused on Incident Response and security monitoring for Tier 1 Analyst.
## Objectives:
- Collect logs using Wazuh agents
- Detect brute force behavior
- Investigate alerts through Wazuh dashboard
- Produce incident reports similar to SOC Tier 1 workflow
## This lab includes:
- Ubuntu Server running Wazuh Manager
  + OS: Ubuntu Live Server 24.04.4
  + IP: 192.168.239.164/24 (VMnet8-NAT)
  + Gateway: 192.168.239.2 (VMnet8-NAT)
  + DNS: 8.8.8.8
- Windows 8 victim machine
  + OS: Windows 8.1 Pro
  + IP: 192.168.239.163/24 (VMnet8-NAT)
  + Gateway: 192.168.239.2 (VMnet8-NAT)
  + DNS: 8.8.8.8
- Kali Linux attacker machine
  + OS: Linux
  + IP: 192.168.239.128/24 (VMnet8-NAT)
## Simulated attacks
- SSH brute force attack against Ubuntu Server
- RDP brute force attack against Windows 8
<img width="303" height="130" alt="arch" src="https://github.com/user-attachments/assets/c1dbecf5-b56f-4146-8730-8d725d044640" />

## Technologies Used
- Wazuh SIEM v4.13.1
- Ubuntu Server
- Windows 8
- Kali Linux
- Hydra
- SSH
- RDP
- Linux authentication logs
- Windows Event Logs
## Project Structure
Explore documentation in the /docs folder for setup, architecture, and lessons learned.
