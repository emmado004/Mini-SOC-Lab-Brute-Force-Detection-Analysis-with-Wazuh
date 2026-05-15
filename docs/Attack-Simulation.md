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

### Triggered Wazuh Alerts
| Order | Rule ID | Level | Description |
|--------|---------|-------|-------|
| 1 | 5503 | 5 | PAM: User login failed |
| 2 | 5551 | 10 | PAM: Multiple failed logins in a small period |
| 3 | 5760 | 5 | sshd: Authentication failed |
| 4 | 5763 | 10 | sshd: Brute force trying to get access |
| 5 | **40112** | **12** | **Multiple auth failures followed by a success** ← key |
| 6 | 5501 | 3 | PAM: Login session opened |
| 7 | 5502 | 3 | PAM: Login session closed |

Rule 40112 (Level 12) is the highest severity in this chain. Wazuh does not merely record individual events; it automatically correlates the entire sequence of failures followed by a single success to conclude that the **"brute-force succeeded"** --> alert requires escalation to Tier 2

<img width="1847" height="785" alt="image" src="https://github.com/user-attachments/assets/15986d74-0d8f-45e5-bada-6bd67139f7ac" />

### MITRE mapping
| rule.mitre.id | Tactic | Trigger |
|---|---|---|
| T1110.001 | Credential Access | Rule 5503, 5760 — failed attempts |
| T1110 | Credential Access | Rule 5551, 5763, 40112 — pattern + success |
| T1021.004 | Lateral Movement | Rule 5760 — SSH as access vector |
| T1078 | Initial Access, Persistence | Rule 5501, 40112 — valid account used |

## Scenario 02 — RDP Brute Force against Windows 8
### Technical Objective
To verify that the Wazuh agent on Windows accurately records and transmits alerts to the SIEM during an RDP brute-force attack, specifically detecting privileged account logons following a series of failures.
### Tools
- **Hydra** — brute-force tool
- **password.txt** — Custom wordlist

### Execution Command

```bash
hydra -l win8 -P password.txt rdp://192.168.239.163 -t 1 -V
```

| Parameter | Value | Description |
|---------|---------|---------|
| `-l` | `win8` |Target Username: `win8` |
| `-P` | `password.txt` | Wordlist brute-force |
| `rdp://` | `192.168.239.163` | Protocol RDP, target Windows 8 |
| `-t 1` | 1 thread | Limits to 1 thread as RDP is unstable with multi-threading |
| `-V` | — | Verbose — displays every attempt for real-time tracking |

> `-t 1` is mandatory for RDP. Multi-threading often causes connection errors and unstable results—this is a specific characteristic of the RDP protocol.
### Wazuh alerts trigger được

| Order | Rule ID | Level | Description |
|--------|---------|-------|-------|
| 1 | 60104 | 5 | Windows Audit Failure |
| 2 | 60122 | 5 | Logon Failure — unknown user or bad password |
| 3 | 60602 | 9 | Windows Application Error (side effect) |
| 4 | **67028** | 3 | **Special privileges assigned to new logon** ← compromise confirmed |
| 5 | 60106 | 3 | Windows Logon Success |
| 6 | 60137 | 3 | Windows User Logoff |

### MITRE mapping
| Technique ID | Tactic | Trigger |
|---|---|---|
| T1531 | Impact | Rule 60122 — Logon failures |
| T1484 | Defense Evasion, Privilege Escalation | Rule 67028 — Special privileges |
| T1078 | Initial Access, Persistence, Defense Evasion | Rule 60106, 67028 — valid account used |
