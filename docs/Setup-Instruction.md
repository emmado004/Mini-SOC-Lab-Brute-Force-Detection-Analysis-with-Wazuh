# SOC Lab Setup
## Virtual Machine Setup
1. Install VMware Workstation Pro and create three VMs:
   - Ubuntu Server (Wazuh manager, SSH-Port 22, Self-Monitoring): SIEM Manager, victim
   - Windows 8 (RDP-Port 3389, Wazuh Agent): Victim
   - Kali Linux (Hydra, Nmap): Attacker
2. Configure network adapters for each VM:
   - Adapter 1: NAT for internet access-VMnet8
## Install Wazuh manager
### Ubuntu Server
- Install the Wazuh (quickstart): 
curl -sO https://packages.wazuh.com/4.13/wazuh-install.sh && sudo bash wazuh-install.sh -a
- Access the dashboard via https://192.168.239.164 - the IP Address on your host machine.
## Install Wazuh agent on Client
### Windows 8 (RDP Victim)
- On Dashboard Wazuh manager, choose Agents management
