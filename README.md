Introduction: Here are the steps I took to create a SOC Homelab with Proxmox. I plan to deploy 1 Linux VM for attacks and 2 Windows VMs. 1 Windows VM will be the target and 1 will be used to monitor the target endpoint, replicating a realistic SOC environment. I then plan to utilize Sysmon for Endpoint Monitoring, Suricata for NIDS, Wazuh for EDR, and ELK and Splunk for SIEM since I want more experience with both tools.

Step 1: Setting up Virtual Machines

1.1 Installing Proxmox: 

Install Proxmox Virtual Environment here: https://www.proxmox.com/en/
The official documentation was helpful: https://pve.proxmox.com/pve-docs/

Once Proxmox is installed, you can now access the Proxmox web GUI. Here we can start deploying VMs. 

<img width="2552" height="1063" alt="image" src="https://github.com/user-attachments/assets/d63ff8f7-252b-423e-b310-53465f9475dd" />

I will prep my 2 HDDs for VM deployement by navigating to 'Disks' and wiping the drives. Note: If you only have a single boot drive, your drive should already be prepped for VM deployement. 

<img width="1192" height="211" alt="image" src="https://github.com/user-attachments/assets/f3733878-f3c5-49cd-8b5f-b29e1844f7c0" />

Then 'Initialize Disk with GPT'

<img width="1187" height="211" alt="image" src="https://github.com/user-attachments/assets/8abc9462-9ede-41e5-94a3-e3f6217dc77d" />

Then we can navigate to 'Directory' to create the ext4 storage pools for VM deployment. Note: I've chosen ext4 isntead of xfs due to it being more resilient to data corruption after sudden power loss. 

<img width="1097" height="715" alt="image" src="https://github.com/user-attachments/assets/f5e9c946-13e7-49e5-8e05-8ad7d2fcae1b" />

I named named mine HDD-1 and HDD-2.

<img width="1405" height="690" alt="image" src="https://github.com/user-attachments/assets/16ef7a69-322b-4354-9d0d-0e9553c2eb83" />

1.2 Deploying Virtual Machines:

I need to start by downloading my Linux and Windows ISOs. I'm going to choose Kali Linux for my Linux distro, as it has a lot of pentesting tools built-in that I may want to try in the future. I'll choose Windows Enterprise Evaluation, which gives me free reign to use Windows Enterprise for 90 days. Note: The license can be extended using the the "slmgr /rearm" command. 

Kali Linux: https://www.kali.org/get-kali/#kali-platforms
Windows 11 Enterprise Evaluation: https://www.microsoft.com/en-us/evalcenter/evaluate-windows-11-enterprise

Now I selected HDD-1 and uploaded my Kali Linux ISO into Proxmox.

<img width="849" height="743" alt="image" src="https://github.com/user-attachments/assets/45c48f8d-47b2-4f8f-b7b5-d235be086510" />

After it's done uploading, it's available to be used in a VM.

<img width="1274" height="311" alt="image" src="https://github.com/user-attachments/assets/c540d496-8bd1-44b0-a6ef-fa618bcbfe7e" />

I named my VM 'Kali-Linux' and left the default settings, as they seemed to be good for my application. Now I have my first VM. 

<img width="294" height="198" alt="image" src="https://github.com/user-attachments/assets/f9575c49-6f0e-43ac-9c03-42295475a500" />

After starting up the VM and going through the installation process, my Kali VM is up and running.

<img width="1267" height="800" alt="image" src="https://github.com/user-attachments/assets/7c192389-c5a5-4f3e-a506-ae730162039a" />

Now i'm going to repeat this process for 2 Windows VMs.

<img width="303" height="241" alt="image" src="https://github.com/user-attachments/assets/1304e719-643a-4e9e-b5e8-e5e9fa4875c1" />

Step 2: Installing Tools

2.1 Installing Sysmon 

1. Download Sysmon on the target VM here: https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon
2. Download a Sysmon configuration file here: https://github.com/SwiftOnSecurity/sysmon-config

Install Sysmon using the command: .\sysmon64.exe -accepteula -i c:\windows\sysmonconfig-export.xml

We can verify it's installed correctly by using the command: Get-Process sysmon64

You can also see Sysmon logs starting to populate in Event Viewer by navigating to: Application and Services Logs > Windows > Sysmon > Operational

<img width="912" height="438" alt="image" src="https://github.com/user-attachments/assets/c8edbe43-3e81-4929-b5a3-1b392c7bea11" />




Creating an isolated network:
