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
## Attack Scenarios
| # | Scenario | Target | Tool | MITRE Technique | Key Alert | Severity |
|---|----------|--------|------|-----------------|-----------|----------|
| 01 | SSH Brute Force | Ubuntu :22 | Hydra | [T1110.001](https://attack.mitre.org/techniques/T1110/001/) · [T1110](https://attack.mitre.org/techniques/T1110/) · [T1021.004](https://attack.mitre.org/techniques/T1021/004/) · [T1078](https://attack.mitre.org/techniques/T1078/) | Rule 40112 (Level 12) | 🔴 Critical |
| 02 | RDP Brute Force | Win8 :3389 | Hydra | [T1531](https://attack.mitre.org/techniques/T1531/) · [T1484](https://attack.mitre.org/techniques/T1484/) · [T1078](https://attack.mitre.org/techniques/T1078/) | Rule 67028 + 60106 | 🟠 High |

## Key findings
1. **Wazuh Correlation Engine Confirms Automatic Compromise** — Rule 40112 (Level 12) correlates a long sequence of failed attempts followed by a single success, automatically concluding a "brute-force succeeded" event without requiring the analyst to manually parse individual logs.
2. **SSH Attack Severity Exceeds RDP in this Environment** — Since the Ubuntu Server acts as both the target and the Wazuh Manager, a successful compromise grants the attacker the ability to purge logs and disable the entire monitoring infrastructure.
3. **Compressed Timestamps as a Primary IOC** — The entire sequence from the start of the brute-force attack to the successful login occurred in under one minute. This rapid succession is a classic Indicator of Compromise (IOC) signaling an automated tool (Hydra) rather than manual human activity.
4. **Context-Driven MITRE Label Interpretation** — While Wazuh assigns T1021.004 (SSH Lateral Movement) to Rule 5760, in this specific lab, SSH served as the Initial Access vector. This highlights that tool-generated outputs must always be interpreted within the specific context of the incident.
<!--1. **Wazuh correlation engine xác nhận compromise tự động** — Rule 40112
   (Level 12) correlate toàn bộ chuỗi thất bại + 1 lần thành công và kết
   luận "brute-force succeeded" mà không cần analyst đọc từng log riêng lẻ.

2. **SSH attack nguy hiểm hơn RDP trong lab này** — Ubuntu Server vừa là
   attack target vừa là Wazuh manager. Attacker kiểm soát server này đồng
   nghĩa với khả năng xóa log và can thiệp vào toàn bộ hệ thống monitoring.

3. **Timestamp nén ngắn là IOC đầu tiên cần kiểm tra** — toàn bộ chuỗi
   brute-force đến logon thành công diễn ra trong dưới 1 phút, đặc trưng
   của automated attack tool (Hydra), không phải manual login.

4. **MITRE label cần đọc trong context** — Wazuh gán T1021.004 (SSH Lateral
   Movement) cho rule 5760, nhưng trong lab này SSH là initial access vector.
   Đây là nhắc nhở rằng tool output cần được đọc trong context của từng
   incident cụ thể. -->
## Project Structure
Explore documentation in the /docs folder for setup, architecture, and lessons learned.
