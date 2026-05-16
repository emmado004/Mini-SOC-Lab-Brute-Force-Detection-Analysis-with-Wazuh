# IR-002 | RDP Brute Force Attack on Windows 8

| Field | Details |
|-------|---------|
| **Date** | 2025-05-14 |
| **Time** | 07:25 (UTC+7) |
| **Severity** | High |
| **Status** | Confirmed Compromise — Simulated Lab Environment |
| **Analyst** | [Your Name] |
| **Assets Affected** | Windows 8 192.168.239.163 (RDP service) |

---

## Executive Summary

At 07:25, Wazuh SIEM detected a brute-force attack against the RDP service
(port 3389) on Windows 8 (192.168.239.163). The attacker, operating from
Kali Linux, used Hydra with a single-threaded configuration to iterate
through a password wordlist targeting the `win8` account.

The attack resulted in a successful login, confirmed by Rule 60106 (Windows
Logon Success) and Rule 67028 (Special Privileges Assigned to New Logon).
The appearance of Rule 67028 is particularly significant — it indicates the
compromised account holds elevated privileges, meaning the attacker gained
access with administrative-level rights.

Unlike IR-001, Wazuh does not have a dedicated correlation rule for RDP
brute-force success equivalent to Rule 40112. Confirming the compromise
required the analyst to manually identify the pattern of repeated Rule 60122
failures followed by Rule 60106 and 67028 within the same timestamp window.

---

## Attack Timeline

| Time | Rule ID | Event | Significance |
|------|---------|-------|--------------|
| 07:25 | 60104 | Windows Audit Failure | First failed authentication attempt logged |
| 07:25 | 60122 | Logon Failure — unknown user or bad password | Hydra iterating through wordlist |
| 07:25 | 60602 | Windows Application Error | System stress side-effect of brute-force load |
| 07:25 | **67028** | **Special privileges assigned to new logon** | **Privileged account login confirmed** |
| 07:25 | 60106 | Windows Logon Success | Attacker authenticates successfully via RDP |
| 07:25 | 60137 | Windows User Logoff | RDP session terminated |

> The entire sequence occurred at 07:25. The compressed timestamp window
> is characteristic of automated brute-force tooling. Notably, this
> incident lacks a single high-level correlation rule like Rule 40112
> in IR-001 — the analyst must recognize the pattern manually:
> multiple 60122 → single 60106 + 67028 = confirmed brute-force success.

---

## Detection Details

**Detection source:** Wazuh SIEM — Threat Hunting dashboard + MITRE ATT&CK dashboard

### Reading the alert chain

**Step 1 — Identify the pattern:**
Multiple occurrences of Rule 60122 (*Logon Failure*) originating from the
same source IP within a narrow time window are the primary indicator of
brute-force activity. A legitimate user mistyping a password produces one
or two failures — not a high-frequency burst at 07:25.

**Step 2 — Confirm the compromise:**
Rule 67028 (*Special Privileges Assigned to New Logon*) is the most
important alert in this chain. This rule fires when Windows assigns elevated
privileges (such as SeDebugPrivilege or SeTcbPrivilege) to a newly
authenticated session — a behavior that only occurs when a privileged
account successfully logs in. Its presence immediately after the Rule 60122
chain confirms the attacker cracked a privileged account.

**Step 3 — Corroborate with supporting alerts:**
Rule 60106 (*Logon Success*) and Rule 60137 (*Logoff*) complete the
picture. Rule 60602 (*Application Error*) is a correlated side-effect —
not a direct attack indicator, but its co-occurrence with the attack
chain supports the finding.

![Wazuh Threat Hunting - RDP](picture/wazuh-rdp-threat-hunting-figure1.md)
*Figure 1: Alert chain at 07:25 — Rules 60122 followed by 67028 and 60106*

![Wazuh MITRE ATT&CK - RDP](picture/wazuh-rdp-threat-hunting-figure2.md)
*Figure 2: MITRE ATT&CK mapping on Wazuh dashboard*

---

## Indicators of Compromise (IOC)

| Type | Value | Description |
|------|-------|-------------|
| IP (attacker) | 192.168.239.x | Kali Linux — brute-force source |
| IP (victim) | 192.168.239.163 | Windows 8 — RDP target |
| Port | 3389 | RDP service |
| Protocol | RDP | Remote Desktop Protocol |
| Account targeted | win8 | Fixed username from Hydra `-l` flag |
| Tool identified | Hydra | Single-threaded (`-t 1`), inferred from pattern |
| Key alert | Rule 67028 + 60106 | Privileged logon confirmed after failure chain |

---

## Root Cause Analysis

**1. RDP service exposed with no login attempt restriction:**
Port 3389 is accessible with no account lockout policy configured, allowing
Hydra to attempt unlimited password combinations without interruption.

**2. Weak password on the `win8` account:**
The password was successfully recovered via a standard wordlist, indicating
insufficient password complexity or length requirements were enforced.

**3. Privileged account used as RDP login:**
The `win8` account holds elevated privileges (evidenced by Rule 67028). In
a hardened environment, standard user accounts — not privileged ones —
should be used for remote desktop access where possible.

**4. RDP directly exposed on the internal network:**
The service is reachable without requiring a VPN or jump host, increasing
the attack surface for any threat actor with network access.

---

## Recommendations

### Immediate actions
1. **Enable Account Lockout Policy:** Lock the account after 5 failed
   attempts within 5 minutes via Group Policy:
   `Computer Configuration → Windows Settings → Security Settings
   → Account Policies → Account Lockout Policy`
2. **Restrict RDP access by IP:** Configure Windows Firewall to allow
   port 3389 only from trusted IP addresses

### Long-term hardening
3. **Enable Network Level Authentication (NLA):** Require authentication
   at the network layer before an RDP session is established — this
   significantly reduces the attack surface
4. **Require VPN before RDP:** Implement a VPN gateway so RDP is never
   directly reachable from untrusted networks
5. **Enable Multi-Factor Authentication for RDP:** Integrate MFA
   (e.g., Microsoft Authenticator or Duo) to prevent credential-only access
6. **Separate privileged and standard accounts:** Use dedicated, lower-privilege
   accounts for day-to-day RDP access; reserve privileged accounts for
   administrative tasks only

---

## MITRE ATT&CK Mapping

| Tactic | Technique | ID | Rule | Evidence |
|--------|-----------|-----|------|----------|
| Impact | Account Access Removal | T1531 | 60122 | Repeated logon failures targeting the account |
| Defense Evasion / Privilege Escalation | Domain Policy Modification | T1484 | 67028 | Special privileges assigned upon successful logon |
| Initial Access / Persistence / Defense Evasion | Valid Accounts | T1078 | 60106, 67028 | Attacker authenticated with cracked valid credentials |

---

## Comparison with IR-001 (SSH Brute Force)

| | IR-001 — SSH | IR-002 — RDP |
|--|---|---|
| Target | Ubuntu 192.168.239.164 | Windows 8 192.168.239.163 |
| Port | 22 | 3389 |
| Account | administrator | win8 |
| Wazuh auto-correlation | Yes — Rule 40112 | No — analyst reads pattern manually |
| Highest alert level | 12 | Not specified |
| Severity | Critical | High |
| Why more dangerous | Target is the Wazuh Manager | Privileged account compromised |

---

## Analyst Notes

**On Rule 67028 and T1484 (Domain Policy Modification):**
Wazuh maps Rule 67028 to MITRE T1484, which describes an attacker actively
modifying domain policies to escalate privileges. In this lab, Rule 67028
fired because Windows automatically assigns privileges to a privileged
account upon successful login — the attacker did not actively modify any
policy. This is standard Windows behavior following authentication.

This is a contextual false-positive in MITRE mapping: the event is real
and correctly detected, but the technique label does not accurately describe
what occurred. The correct interpretation is that T1078 (Valid Accounts)
drove the privilege escalation outcome — the attacker gained elevated access
by cracking a privileged account, not by manipulating policy objects.

Recognizing this distinction is an important analyst habit: tool-generated
MITRE labels provide a starting point, not a final verdict.

---

## References
- [MITRE T1531 — Account Access Removal](https://attack.mitre.org/techniques/T1531/)
- [MITRE T1484 — Domain Policy Modification](https://attack.mitre.org/techniques/T1484/)
- [MITRE T1078 — Valid Accounts](https://attack.mitre.org/techniques/T1078/)
- [Windows Event ID 4625 — Logon Failure](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4625)
- [Windows Event ID 4672 — Special Privileges](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4672)
- [Wazuh Rules Documentation](https://documentation.wazuh.com/current/user-manual/ruleset/rules.html)
