# Challenges & Lessons Learned

## Challenges

**Hydra RDP requires `-t 1`**
RDP drops connections under multi-thread load. Single-thread is mandatory
for stable execution — learned through failed attempts, not documentation.

**No auto-correlation rule for RDP success**
Wazuh's Rule 40112 auto-confirms SSH compromise. No equivalent exists for
RDP — the analyst must manually read the pattern: repeated 60122 → 60106 + 67028. Detection strength varies by platform.

**Wazuh MITRE labels need contextual validation**
Rule 67028 was mapped to T1484 (Domain Policy Modification) and Rule 5760
to T1021.004 (Lateral Movement) — neither matched what actually occurred.
Tool-generated labels are a starting point, not a verdict.

## Lessons Learned

**Correlation over individual alerts**
No single alert tells the full story. The chain matters:
5503 → 5551 → 5763 → 40112 is more meaningful than any rule alone.

**Severity escalation is the action trigger**
Level 10+ on authentication events should fire an automated notification.
Waiting for manual review in production introduces unacceptable delay.

**Never expose the SIEM server as an attack target**
Ubuntu Server was both Wazuh Manager and SSH target. In production, an
attacker who owns the SIEM can delete logs and operate undetected.
Monitoring infrastructure must be isolated from monitored assets.
