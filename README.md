Topology: 
<img width="1279" height="999" alt="SOC-Homelab-Topology" src="https://github.com/user-attachments/assets/70976204-e28b-4eff-9513-7cc1156bffe4" />

Introduction: 
Here are the steps I took to create a SOC Homelab with Proxmox. I plan to deploy 1 Kali Linux VM for attacks, 1 Windows VM to target, and 1 Ubuntu Server VM for monitoring. I plan to utilize Sysmon for endpoint logs, Suricata for IDS, Wazuh for EDR, and Splunk for SIEM. Sysmon, Suricata, and Wazuh logs will be injested into Wazuh, which will then be imported into Splunk. If it all works out, i'll be able to access Wazuh and Splunk through their web dashboards for log analysis.

Step 1: Setting up Virtual Machines

1.1 Installing Proxmox: 

Install Proxmox Virtual Environment here: https://www.proxmox.com/en/
The official documentation was helpful: https://pve.proxmox.com/pve-docs/

Once Proxmox is installed, you can now access the Proxmox web GUI. Here we can start deploying VMs. 

<img width="2552" height="1063" alt="image" src="https://github.com/user-attachments/assets/d63ff8f7-252b-423e-b310-53465f9475dd" />

1.2 Deploying Virtual Machines:

I need to start by downloading my Linux and Windows ISOs. I'm going to choose Kali Linux for my Linux distro, as it has a lot of pentesting tools built-in that I may want to try in the future. I'll choose Windows 11 Enterprise Evaluation for the target endpoint and Ubuntu Server for the system I plan to monitor the endpoint from.

Note: Windows 11 Enterprise Evaluation can only be used for 90 days, but the license can be extended using the the "slmgr /rearm" command. 

Kali Linux: https://www.kali.org/get-kali/#kali-platforms
Windows 11 Enterprise Evaluation: https://www.microsoft.com/en-us/evalcenter/evaluate-windows-11-enterprise
Ubuntu Server: https://ubuntu.com/download/server

I plan to deploy my VMs on HDD-1, so I selected it and and uploaded my ISOs to it.

<img width="814" height="275" alt="image" src="https://github.com/user-attachments/assets/34a318b5-8442-4fa7-9094-97a3f9e4fa88" />

After they're done uploading, they're available to be installed on a VM.

<img width="1460" height="909" alt="image" src="https://github.com/user-attachments/assets/2c84d353-1702-47af-9d81-85ef7231a75c" />

I named my VM 'Kali-Linux' and left the default settings, as they seemed to be good for my application. I went through the installation process and now my VM is up and running.

<img width="1267" height="800" alt="image" src="https://github.com/user-attachments/assets/7c192389-c5a5-4f3e-a506-ae730162039a" />

Now i'm going to repeat this process for the rest of the VMs.

<img width="293" height="111" alt="image" src="https://github.com/user-attachments/assets/b92abadf-96d7-4529-87b2-ca9b2907a298" />

Step 2: Installing Tools onto Windows-Endpoint

2.1 Installing Sysmon 

1. Download Sysmon on the target VM here: https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon
2. Download a Sysmon configuration file here: https://github.com/SwiftOnSecurity/sysmon-config

Install Sysmon using the command: .\sysmon64.exe -accepteula -i c:\windows\sysmonconfig-export.xml

We can verify it's installed correctly by using the command: Get-Process sysmon64

<img width="689" height="128" alt="image" src="https://github.com/user-attachments/assets/b6904305-34d8-454a-afe6-7f94cc5760f0" />

You can also see Sysmon logs starting to populate in Event Viewer by navigating to: Application and Services Logs > Windows > Sysmon > Operational

<img width="912" height="438" alt="image" src="https://github.com/user-attachments/assets/c8edbe43-3e81-4929-b5a3-1b392c7bea11" />

2.2 Install Suricata

1. Download Suricata here: https://suricata.io/download/
2. Download npcap as it's a dependency for Suricata: https://npcap.com/#download
3. Download a pre-configured ruleset for emerging threats here: https://rules.emergingthreats.net/open/suricata-7.0.3/

Note: I will hold off on configuring Suricata until later. 

<img width="1257" height="720" alt="image" src="https://github.com/user-attachments/assets/cd0acacb-df8d-4705-99e3-6258d5243281" />

2.3 Install Wazuh Agent.

Following this documentation, I installed the Wazuh Agent: https://documentation.wazuh.com/current/installation-guide/wazuh-agent/wazuh-agent-package-windows.html

<img width="339" height="298" alt="image" src="https://github.com/user-attachments/assets/1231ba73-6bf2-450f-a8be-cb3bc81820b8" />


3. Installing Tools onto SOC-Server

Now that all of our monitoring tools are downloaded on the Windows-Endpoint, I can install Wazuh Server and Splunk on the SOC-Server.

3.1 Installing Wazuh Server

https://documentation.wazuh.com/current/quickstart.html

1. Download and run the Wazuh installation assistant by using the command: wget https://packages.wazuh.com/4.12/wazuh-install.sh && sudo bash ./wazuh-install.sh -a

Once the installation is complete, it will start up the Wazuh dashboard and give us generated credentials. 
<img width="801" height="34" alt="image" src="https://github.com/user-attachments/assets/b8dfe54b-fd1b-4b43-9b39-6c6e8b340977" />

We can use them to log into the Wazuh dashboard.
<img width="1757" height="1221" alt="image" src="https://github.com/user-attachments/assets/9248c362-3422-4246-9df0-bb990d805b29" />

We can already see Wazuh logs from the SOC-Server populating the Wazuh dashboard. Now we need to connect the Windows-Endpoint agent to the Wazuh server so we can see those logs as well.
<img width="1757" height="1242" alt="image" src="https://github.com/user-attachments/assets/d789a61a-c356-4a8f-83bc-aea3b0c40ef5" />

3.3 Installing Splunk

Install Splunk Enterprise from the official website: https://www.splunk.com/en_us/download/splunk-enterprise.html?locale=en_us
Follow the official documentation here to install Splunk: https://help.splunk.com/en/splunk-enterprise/get-started/install-and-upgrade/10.0/install-splunk-enterprise-on-linux-or-macos/install-on-linux
Follow the official documentation here to start Splunk: https://help.splunk.com/en/splunk-enterprise/get-started/install-and-upgrade/10.0/start-using-splunk-enterprise/start-splunk-enterprise-for-the-first-time#id_73f04e49_0810_4340_891c_e7573f05ef03__Start_Splunk_Enterprise_for_the_first_time

Once Splunk has successfully started, we can access the Splunk dashboard. 

<img width="1770" height="503" alt="image" src="https://github.com/user-attachments/assets/f89233cc-7eca-4094-85e2-2d10d45944c0" />

Future Plans:
1. Set up a virtualized pfSense server to isolate the SOC-Homelab from my home network.
2. Set up Suricata on a dedicated VM so it can function as a NIDS.
3. Configure Suricata to capture network traffic and configure a NFS share so Suricata can store packet captures.
4. Install tShark on my SOC-Server, so I analyze the packet captures.
5. Try different attacks like SSL stripping. 

Topology:
<img width="1842" height="940" alt="image" src="https://github.com/user-attachments/assets/65928f54-eaee-4208-b7f7-066d1bc8e797" />

Conclusion: 

