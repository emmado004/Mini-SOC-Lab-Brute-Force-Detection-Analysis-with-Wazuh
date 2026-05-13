# SOC Lab Setup
## Virtual Machine Setup
1. Install VMware Workstation Pro and create three VMs:
   - Ubuntu Server (Wazuh manager, SSH-Port 22, Self-Monitoring): SIEM Manager, victim
   - Windows 8 (RDP-Port 3389, Wazuh Agent): Victim
   - Kali Linux (Hydra): Attacker
2. Configure network adapters for each VM:
   - Adapter 1: NAT for internet access-VMnet8
## Install Wazuh manager
### Ubuntu Server
- Install the Wazuh (quickstart): 
curl -sO https://packages.wazuh.com/4.13/wazuh-install.sh && sudo bash wazuh-install.sh -a
- Access the dashboard via https://192.168.239.164 - the IP Address on your host machine.
- The Ubuntu Server (Manager) is monitored using Wazuh’s built-in self-monitoring feature. No additional agent installation was required, demonstrating the versatility of the Wazuh core in protecting the central SIEM infrastructure.
- Enable SSH: sudo apt install openssh-server -y
- In /var/ossec/etc/ossec.conf check if it has:
<img width="892" height="122" alt="image" src="https://github.com/user-attachments/assets/ca6ac29a-9d88-4342-b942-dab96fa1be44" />

- If don't have, add into then save and restart: sudo systemctl restart wazuh-manager
## Install Wazuh agent on Client
### Windows 8 (RDP Victim)
- On Dashboard Wazuh manager, choose Agents management -> Deploy new agent
- Configuration
  <img width="2318" height="1274" alt="image" src="https://github.com/user-attachments/assets/0d3127b9-42ac-4b7f-a236-ed9fe7e3c730" />

- Run PowerShell as Administrator on windows 8, run command line in 4 and 5
- Enable Audit Policy (Crucial):
   + Open secpol.msc -> Local Policies -> Audit Policy.
   + Set Audit logon events and Audit account logon events to Success & Failure.
<img width="495" height="271" alt="image" src="https://github.com/user-attachments/assets/b6a5abdb-2a26-4f01-b49d-ae685404297e" />

- Enable RDP: Go to Control Panel -> System and Security -> System -> Remote Settings -> Allow remote connections to this computer.
  <img width="474" height="251" alt="image" src="https://github.com/user-attachments/assets/07e646fa-4a94-4562-8111-63f8dc3de0ba" />
## Install Kali Linux
- Use built-in tools to perform controlled attacks such as brute-force attempts.
