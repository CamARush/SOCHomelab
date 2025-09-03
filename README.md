Introduction: I am creating a walkthrough on how to create a SOC Homelab with Proxmox. I plan to deploy 1 Linux VM for attacks and 2 Windows VMs. 1 Windows VM will be the target of the attacks and 1 will be used to monitor the victim endpoint, replicating a realistic SOC environment. I then plan to utilize Sysmon for Endpoint Monitorin, Suricata for NIDS, Wazuh for EDR, and ELK and Splunk for SIEM since I want more experience with both.

Notes: If you don't have an old computer you can use as a server, you can do this with VMWare. Just skip the ProxMox portion of this walkthrough.


Installing Proxmox: I won't go into too much detail on installing proxmox, as you can read the official documentation. 

Install Proxmox Virtual Environment here: https://www.proxmox.com/en/
Read the installation documetnation here: https://pve.proxmox.com/pve-docs/chapter-pve-installation.html

Once Proxmox is installed, you can now access the Proxmox web GUI. Here we can start deploying VMs. 

<img width="2552" height="1063" alt="image" src="https://github.com/user-attachments/assets/d63ff8f7-252b-423e-b310-53465f9475dd" />

I will prep my 2 HDDs for VM deployement by navigating to 'Disks' and wiping the drives.

<img width="1192" height="211" alt="image" src="https://github.com/user-attachments/assets/f3733878-f3c5-49cd-8b5f-b29e1844f7c0" />

Then 'Initialize Disk with GPT'

<img width="1187" height="211" alt="image" src="https://github.com/user-attachments/assets/8abc9462-9ede-41e5-94a3-e3f6217dc77d" />

Then we can navigate to 'Directory' to create the ext4 storage pools for VM deployment. 

<img width="1097" height="715" alt="image" src="https://github.com/user-attachments/assets/f5e9c946-13e7-49e5-8e05-8ad7d2fcae1b" />

I named named mine HDD-1 and HDD-2.

<img width="1405" height="690" alt="image" src="https://github.com/user-attachments/assets/16ef7a69-322b-4354-9d0d-0e9553c2eb83" />
