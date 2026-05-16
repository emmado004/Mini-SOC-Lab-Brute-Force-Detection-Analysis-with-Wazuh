# Recommendations

## Immediate

| Asset | Action |
|-------|--------|
| Ubuntu Server | Disable SSH password auth — key-based only |
| Ubuntu Server | Install Fail2ban — block IP after 5 failures |
| Windows 8 | Enable Account Lockout Policy (threshold: 5 attempts) |
| Both | Restrict SSH/RDP to trusted IPs only |

## Detection

| Action | Purpose |
|--------|---------|
| Alert on Wazuh Level 10+ | Current default requires manual review — too slow |
| Custom RDP correlation rule | Equivalent to Rule 40112 for Windows incidents |
| Add Sysmon to Windows 8 | Richer telemetry beyond Windows Event Log |

## Architecture

| Action | Purpose |
|--------|---------|
| Isolate Wazuh to a dedicated VLAN | SIEM must never be reachable as an attack target |
| Require VPN before SSH/RDP | Removes both services from direct exposure |
