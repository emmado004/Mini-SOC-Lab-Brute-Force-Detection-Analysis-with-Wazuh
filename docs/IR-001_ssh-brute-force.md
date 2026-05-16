# IR-001 | SSH Brute Force Attack on Ubuntu Server

| Field | Details |
|-------|---------|
| **Date** | 2025-XX-XX |
| **Time** | 15:16 (UTC+7) |
| **Severity** | Critical |
| **Status** | Confirmed Compromise — Simulated Lab Environment |
| **Analyst** | [Your Name] |
| **Assets Affected** | Ubuntu Server 192.168.239.164 (SSH service + Wazuh Manager) |

---

## Executive Summary

At 15:16, Wazuh SIEM detected and confirmed a brute-force attack against the
SSH service on Ubuntu Server (192.168.239.164). The attacker, operating from
Kali Linux, used Hydra to repeatedly attempt authentication against the
`administrator` account using a password wordlist.

Wazuh escalated alert severity across three tiers — from individual login
failures (Rule 5503) to pattern recognition (Rules 5551, 5763 at Level 10),
and finally to Rule 40112 (Level 12), which automatically correlated the full
failure chain with a single successful login to confirm a completed brute-force
attack. A PAM session was subsequently opened (Rule 5501), confirming the
attacker had gained shell access.

This incident is classified as Critical because the compromised server is also
the Wazuh Manager — an attacker with access to this host could tamper with
logs, disable agents, or establish persistence undetected.

---

## Attack Timeline

| Time | Rule ID | Level | Event | Significance |
|------|---------|-------|-------|--------------|
| 15:53 | 5503 | — | PAM: User login failed | Hydra begins iterating wordlist |
| 15:53 | 5551 | 10 | PAM: Multiple failed logins in a small period | Wazuh detects abnormal login frequency |
| 15:53 | 5760 | — | sshd: Authentication failed | sshd confirms each failed attempt |
| 15:53 | 5763 | 10 | sshd: Brute force trying to get access | Wazuh identifies deliberate brute-force pattern |
| 15:53 | **40112** | **12** | **Multiple auth failures followed by a success** | **Brute-force confirmed successful — compromise verified** |
| 15:53 | 5501 | — | PAM: Login session opened | Attacker gains shell access on Ubuntu Server |
| 15:53 | 5502 | — | PAM: Login session closed | SSH session terminated |

> The entire chain occurred within a single minute at 15:53. Compressed
> timestamps are a characteristic of automated tooling (Hydra) and serve
> as a primary IOC distinguishing this from a legitimate forgotten-password
> scenario.

---

## Detection Details

**Detection source:** Wazuh SIEM — Threat Hunting dashboard + MITRE ATT&CK dashboard

### Alert escalation — three-tier correlation

**Tier 1 — Individual event (Rule 5503, 5760):**
PAM and sshd log each failed authentication independently. At this stage,
the events are ambiguous — a user mistyping a password produces identical
alerts. No escalation action is taken at this tier.

**Tier 2 — Pattern detection (Rules 5551, 5763 — Level 10):**
Wazuh's correlation engine detects abnormal frequency and escalates. Rule
5763 (*sshd: brute force trying to get access*) is a dedicated brute-force
rule — at this point, Wazuh has distinguished deliberate attack behavior
from accidental login failures. A Level 10 alert warrants immediate
investigation.

**Tier 3 — Compromise confirmation (Rule 40112 — Level 12):**
Rule 40112 is the definitive alert. Wazuh correlates the full chain of
failures followed by one success and concludes the brute-force attack
succeeded. This is the highest-severity alert in this incident and the
primary escalation trigger for a SOC Tier 1 analyst.

![Wazuh Threat Hunting - SSH](../wazuh-ssh-threat-hunting-figure1.md)
*Figure 1: Alert chain at 15:53 — escalation from Rule 5503 to Rule 40112*

![Wazuh MITRE ATT&CK - SSH](../evidence/wazuh-ssh-mitre.png)
*Figure 2: MITRE ATT&CK mapping on Wazuh dashboard*

---

## Indicators of Compromise (IOC)

| Type | Value | Description |
|------|-------|-------------|
| IP (attacker) | 192.168.239.x | Kali Linux — brute-force source |
| IP (victim) | 192.168.239.164 | Ubuntu Server — SSH target + Wazuh Manager |
| Port | 22 | SSH service |
| Protocol | SSH | Secure Shell |
| Account targeted | administrator | Fixed username from Hydra `-l` flag |
| Tool identified | Hydra | Inferred from attack pattern and timing |
| Defining alert | Rule 40112 (Level 12) | Wazuh-confirmed brute-force success |

---

## Root Cause Analysis

**1. Password authentication enabled on SSH:**
The SSH service accepts password-based login with no attempt limit, allowing
Hydra to submit unlimited authentication requests without being blocked.

**2. Weak password on `administrator` account:**
Hydra successfully recovered the password using a standard wordlist, indicating
the password lacked sufficient complexity or length.

**3. No rate limiting or automatic blocking:**
The absence of Fail2ban or equivalent tooling means there was no mechanism
to detect and block the attacker's IP after repeated failures.

**4. Privileged account exposed via SSH:**
The `administrator` account has direct SSH access. In a hardened environment,
privileged accounts should not be directly reachable via remote login services.

---

## Recommendations

### Immediate actions
1. **Disable password authentication** — enforce SSH key-based authentication only:
```bash
   # /etc/ssh/sshd_config
   PasswordAuthentication no
   PubkeyAuthentication yes
```
2. **Install and configure Fail2ban** — auto-block IPs after 5 failed attempts:
```bash
   apt install fail2ban
   # /etc/fail2ban/jail.local
   # maxretry = 5 | bantime = 3600 | findtime = 300
```

### Long-term hardening
3. **Disable direct root/administrator SSH login:**
   Set `PermitRootLogin no` in `/etc/ssh/sshd_config`
4. **Restrict SSH access by IP:** Use `AllowUsers` or firewall rules to
   limit SSH connections to trusted IP ranges only
5. **Isolate the Wazuh Manager from the attack surface:** In production,
   the SIEM server should not be reachable via the same network segment
   as monitored hosts

---

## MITRE ATT&CK Mapping

| Tactic | Technique | ID | Rule | Evidence |
|--------|-----------|-----|------|----------|
| Credential Access | Brute Force: Password Guessing | T1110.001 | 5503, 5760 | Repeated PAM/sshd login failures |
| Credential Access | Brute Force | T1110 | 5551, 5763, 40112 | Frequency pattern + confirmed success |
| Lateral Movement | Remote Services: SSH | T1021.004 | 5760 | SSH used as the access vector |
| Initial Access | Valid Accounts | T1078 | 5501, 40112 | Session opened with cracked credentials |
| Persistence | Valid Accounts | T1078 | 40112 | Attacker holds valid credentials post-attack |
| Defense Evasion | Valid Accounts | T1078 | 40112 | Legitimate credentials make traffic appear normal |

---

## Analyst Notes

**On Rule 40112:** This rule demonstrates a key advantage of Wazuh's
correlation engine over simpler log aggregators. Rather than surfacing
individual events, it synthesizes the full attack sequence into a single
high-confidence verdict. In a production SOC, this rule should trigger
an automated notification (email or Slack) to ensure no analyst misses it.

**On T1021.004 (SSH — Lateral Movement):** Wazuh assigns Lateral Movement
as a tactic for Rule 5760. In this lab context, SSH is the initial access
vector — the attacker is entering the environment for the first time, not
moving between already-compromised hosts. This illustrates an important
analyst habit: MITRE labels provided by tooling should always be
interpreted within the specific context of the incident, not applied
literally.

---

## References
- [MITRE T1110.001 — Brute Force: Password Guessing](https://attack.mitre.org/techniques/T1110/001/)
- [MITRE T1021.004 — Remote Services: SSH](https://attack.mitre.org/techniques/T1021/004/)
- [MITRE T1078 — Valid Accounts](https://attack.mitre.org/techniques/T1078/)
- [Wazuh Rules Documentation](https://documentation.wazuh.com/current/user-manual/ruleset/rules.html)
- [Fail2ban Setup Guide](https://www.fail2ban.org/wiki/index.php/MANUAL_0_8)
