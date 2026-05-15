# Attack Simulation Section
This document provides a detailed description of the attack commands executed within the lab, the tools utilized, and the technical objectives for each step. All activities were conducted in an isolated lab environment (VMware NAT/Internal Network).
## Environment
| VM | IP | OS | Role |
|----|----|----|------|
| Kali Linux | 192.168.239.x | Kali 2024 | Attacker |
| Ubuntu Server | 192.168.239.164 | Ubuntu 22.04 | SSH target + Wazuh SIEM |
| Windows 8 | 192.168.239.163 | Windows 8.1 | RDP victim |

## Scenario 01 — SSH Brute Force into Ubuntu Server
### Technical Objective
To verify Wazuh's capability to detect SSH brute-force attacks and automatically escalate severity via its correlation engine (transitioning from individual events to a confirmed compromise).
### Tools
- Hydra — A high-speed network logon cracker, pre-installed on Kali Linux.
- password.txt — A custom wordlist containing the correct password at the end of the list.
### Execution command
```bash
hydra -l administrator -P password.txt ssh://192.168.239.164
```
| Parameter | Value | Description |
|---------|---------|---------|
| `-l` | `administrator` | Static Username — targeting a specific account. |
| `-P` | `password.txt` | Wordlist — testing each line sequentially. |
| `ssh://` | `192.168.239.164` | Protocol and target IP |

> The `-t` (thread) parameter is not used here to allow Hydra to run with default settings, which is sufficient to trigger Wazuh without crashing the SSH service.
<img width="897" height="326" alt="image" src="https://github.com/user-attachments/assets/01763d0c-7698-42b1-8fa5-062ad45b4e0c" />
