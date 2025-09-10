# **Creating a SOC homelab**

### Introduction: 
Here are the steps I took to create a SOC Homelab with Proxmox. I plan to deploy 1 Kali Linux VM for attacks, 1 Windows VM to target, and 1 Ubuntu Server VM for monitoring. I plan to utilize Sysmon for endpoint logs, Suricata for IDS, Wazuh for XDR, and Splunk for SIEM. Sysmon, Suricata, and Wazuh logs will be injested into Wazuh, which will then be imported into Splunk. If it all works out, i'll be able to access Wazuh and Splunk through their web dashboards for log analysis.

### Topology: 
<img width="1279" height="999" alt="SOC-Homelab-Topology" src="https://github.com/user-attachments/assets/70976204-e28b-4eff-9513-7cc1156bffe4" />

## Step 1: Setting up Virtual Machines

1.1 Installing Proxmox: 

Install Proxmox Virtual Environment here: https://www.proxmox.com/en/
The official documentation was helpful: https://pve.proxmox.com/pve-docs/

Once Proxmox is installed, you can use the CLI or access the Proxmox web GUI to start deploying VMs. I'm going to use the web GUI. 

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

## Step 2. Installing Tools onto SOC-Server

2.1 Installing Wazuh

https://documentation.wazuh.com/current/quickstart.html

1. Download and run the Wazuh installation assistant by using the command: wget https://packages.wazuh.com/4.12/wazuh-install.sh && sudo bash ./wazuh-install.sh -a

Once the installation is complete, it should create 4 services: filebeat, wazuh-indexer, wazuh-manager, and wazuh-dashboard.
I can check the status of the services by using the command: systemctl status <name-of-service>
If the service isn't set to start on boot, we can do this by using the command: systemctl enable <name-of-service>

If the installation was successfull, it will start up the Wazuh dashboard and give us generated credentials. 
<img width="801" height="34" alt="image" src="https://github.com/user-attachments/assets/b8dfe54b-fd1b-4b43-9b39-6c6e8b340977" />
<img width="1757" height="1221" alt="image" src="https://github.com/user-attachments/assets/9248c362-3422-4246-9df0-bb990d805b29" />

I can already see Wazuh logs from the SOC-Server populating the Wazuh dashboard. Now I need to connect the Windows-Endpoint agent to the Wazuh server so we can see those logs as well.
<img width="1757" height="1242" alt="image" src="https://github.com/user-attachments/assets/d789a61a-c356-4a8f-83bc-aea3b0c40ef5" />

2.2 Installing Splunk

Install Splunk Enterprise from the official website: https://www.splunk.com/en_us/download/splunk-enterprise.html?locale=en_us
Follow the official documentation here to install Splunk: https://help.splunk.com/en/splunk-enterprise/get-started/install-and-upgrade/10.0/install-splunk-enterprise-on-linux-or-macos/install-on-linux
Follow the official documentation here to start Splunk: https://help.splunk.com/en/splunk-enterprise/get-started/install-and-upgrade/10.0/start-using-splunk-enterprise/start-splunk-enterprise-for-the-first-time#id_73f04e49_0810_4340_891c_e7573f05ef03__Start_Splunk_Enterprise_for_the_first_time

Like Wazuh, it creates the splunk service which we can use the same commands to check the status and ensure it starts on boot.

Once Splunk has successfully started, I can access the Splunk dashboard and use the credentials I created when starting it for the first time. 

<img width="1770" height="503" alt="image" src="https://github.com/user-attachments/assets/f89233cc-7eca-4094-85e2-2d10d45944c0" />

## Step 3: Installing and Configuring Tools on Windows-Endpoint

3.1 Install Wazuh Agent.

Following this documentation, I installed the Wazuh Agent: https://documentation.wazuh.com/current/installation-guide/wazuh-agent/index.html

I then went back to the Wazuh dashboard and followed the instructions to deploy a Wazuh Agent on our Windows-Endpoint VM.

<img width="559" height="293" alt="image" src="https://github.com/user-attachments/assets/f7c51925-9852-46b9-a750-17340bcf03b8" />

<img width="1633" height="530" alt="image" src="https://github.com/user-attachments/assets/05b807dc-7bcf-4151-9be6-30e2306ff363" />

<img width="1633" height="951" alt="image" src="https://github.com/user-attachments/assets/6138a291-9de9-42d9-8b33-ca8f9abc2c08" />

3.2 Installing & Configuring Sysmon 

1. Download Sysmon on the target VM here: https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon
2. Download a Sysmon configuration file here: https://github.com/SwiftOnSecurity/sysmon-config

Install Sysmon using the command: .\sysmon64.exe -accepteula -i c:\windows\sysmonconfig-export.xml

We can verify it's installed correctly by using the command: Get-Process sysmon64

<img width="689" height="128" alt="image" src="https://github.com/user-attachments/assets/b6904305-34d8-454a-afe6-7f94cc5760f0" />

You can also see Sysmon logs starting to populate in Event Viewer by navigating to: Application and Services Logs > Windows > Sysmon > Operational

<img width="912" height="438" alt="image" src="https://github.com/user-attachments/assets/c8edbe43-3e81-4929-b5a3-1b392c7bea11" />

Now I'll configure Wazuh to monitor the Sysmon logfiles by adding the following code to the Wazuh Agent ossec.conf file.

<localfile>
  <location>Microsoft-Windows-Sysmon/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>

Now I'll check the Wazuh dashbaord to see if the Sysmon logs are properly populating.

<img width="1234" height="129" alt="image" src="https://github.com/user-attachments/assets/4c28d775-06e1-46a0-8059-78996a6b73c1" />
<img width="564" height="39" alt="image" src="https://github.com/user-attachments/assets/c56416dc-8d57-4276-82cb-3d0bd5a7aedc" />

3.3 Installing & Configuring Suricata

1. Download Suricata here: https://suricata.io/download/
3. Download npcap as it's a dependency for Suricata: https://npcap.com/#download
4. Download a pre-configured ruleset for emerging threats here: https://rules.emergingthreats.net/open/suricata-7.0.3/
5. Copy the emerging threat rules and put them in the /Program Files/Suricata/rules folder.
6. Ensure the suricata.yaml file is using the new rules.
7. Configure Suricata log injestion into Wazuh by adding this code to the ossec.conf file.

  <localfile>
    <log_format>json</log_format>
    <location>C:\Program Files\Suricata\log\eve.json</location>
  </localfile>

8. Start Suricata using the command: suricata.exe -i <windows-endpoint-ip>

<img width="1089" height="308" alt="image" src="https://github.com/user-attachments/assets/83e4a5d0-0d5f-42ef-b2f4-82e8eb3e3249" />

Now I'm going to ensure that Suricata logs are going into Wazuh by triggering the IDS. I did this by using this command on the Windows-Endpoint: curl http://testmyids.com

Going to the Wazuh dashboard and refreshing shows the Suricata log. It's working! 

<img width="1963" height="124" alt="image" src="https://github.com/user-attachments/assets/3ec64602-cc01-4455-889d-013152818095" />
<img width="755" height="168" alt="image" src="https://github.com/user-attachments/assets/98441b25-a138-4176-8024-7bab4095b41e" />

## Step 4. Configuring Splunk

Now that i've set up all of the monitoring tools on the endpoint and configured Wazuh to collect them, I need a way to send those logs into Splunk. 

Official Documentation: https://documentation.wazuh.com/current/integrations-guide/splunk/index.html#server-integration-using-splunkforwarder

There are a few ways to do this, but I'll be choosing Wazuh server integration using the Splunk forwarder. 

<img width="1625" height="1195" alt="image" src="https://github.com/user-attachments/assets/755c2e50-ef91-44ce-8b74-d770934ae0d1" />

Step 5. Testing



Future Upgrades: 
1. Set up a virtualized pfSense server to isolate the SOC-Homelab from my home network.
2. Set up Suricata on a dedicated VM so it can function as a NIDS.
3. Configure Suricata to capture network traffic and configure a SMB share where Suricata can store the packet captures.
4. Install tShark on my SOC-Server, so I analyze the packet captures stored in the SMB share.
5. Set up a Active Directory domain controller.

Future Topology:
<img width="1842" height="940" alt="image" src="https://github.com/user-attachments/assets/65928f54-eaee-4208-b7f7-066d1bc8e797" />

Conclusion: 


<a id="top"></a>
# 🛡️ Creating a SOC Homelab

## 📑 Table of Contents
1. [Introduction](#introduction)  
2. [Topology](#topology)  
3. [Step 1: Setting up Virtual Machines](#step-1-setting-up-virtual-machines)  
   - [1.1 Installing Proxmox](#installing-proxmox)  
   - [1.2 Deploying Virtual Machines](#deploying-virtual-machines)  
4. [Step 2: Installing Tools on SOC-Server](#step-2-installing-tools-on-soc-server)  
   - [2.1 Installing Wazuh](#installing-wazuh)  
   - [2.2 Installing Splunk](#installing-splunk)  
5. [Step 3: Installing & Configuring Tools on Windows-Endpoint](#step-3-installing-configuring-tools-on-windows-endpoint)  
   - [3.1 Wazuh Agent](#wazuh-agent)  
   - [3.2 Sysmon](#sysmon)  
   - [3.3 Suricata](#suricata)  
6. [Step 4: Configuring Splunk](#step-4-configuring-splunk)  
7. [Step 5: Testing](#step-5-testing)  
8. [Future Upgrades](#future-upgrades)  
9. [Conclusion](#conclusion)  

---

<a id="introduction"></a>
## 🔹 Introduction
Here are the steps I took to create a SOC Homelab with Proxmox. I plan to deploy 1 Kali Linux VM for attacks, 1 Windows VM to target, and 1 Ubuntu Server VM for monitoring. I plan to utilize Sysmon for endpoint logs, Suricata for IDS, Wazuh for XDR, and Splunk for SIEM. Sysmon, Suricata, and Wazuh logs will be ingested into Wazuh, which will then be imported into Splunk. If it all works out, I'll be able to access Wazuh and Splunk through their web dashboards for log analysis.

---

<a id="topology"></a>
## 🗺️ Topology
<img width="1279" height="999" alt="SOC-Homelab-Topology" src="https://github.com/user-attachments/assets/70976204-e28b-4eff-9513-7cc1156bffe4" />

---

<a id="step-1-setting-up-virtual-machines"></a>
## ⚙️ Step 1: Setting up Virtual Machines

<a id="installing-proxmox"></a>
### 1.1 Installing Proxmox
Install Proxmox Virtual Environment here: https://www.proxmox.com/en/  
The official documentation was helpful: https://pve.proxmox.com/pve-docs/

Once Proxmox is installed, you can use the CLI or access the Proxmox web GUI to start deploying VMs. I'm going to use the web GUI.  

<img width="2552" height="1063" alt="image" src="https://github.com/user-attachments/assets/d63ff8f7-252b-423e-b310-53465f9475dd" />

<a id="deploying-virtual-machines"></a>
### 1.2 Deploying Virtual Machines
I need to start by downloading my Linux and Windows ISOs. I'm going to choose Kali Linux for my Linux distro, as it has a lot of pentesting tools built-in that I may want to try in the future. I'll choose Windows 11 Enterprise Evaluation for the target endpoint and Ubuntu Server for the system I plan to monitor the endpoint from.

**Note:** Windows 11 Enterprise Evaluation can only be used for 90 days, but the license can be extended using the `slmgr /rearm` command.  

- Kali Linux: https://www.kali.org/get-kali/#kali-platforms  
- Windows 11 Enterprise Evaluation: https://www.microsoft.com/en-us/evalcenter/evaluate-windows-11-enterprise  
- Ubuntu Server: https://ubuntu.com/download/server  

I plan to deploy my VMs on HDD-1, so I selected it and uploaded my ISOs to it.  

<img width="814" height="275" alt="image" src="https://github.com/user-attachments/assets/34a318b5-8442-4fa7-9094-97a3f9e4fa88" />

After they're done uploading, they're available to be installed on a VM.  

<img width="1460" height="909" alt="image" src="https://github.com/user-attachments/assets/2c84d353-1702-47af-9d81-85ef7231a75c" />

I named my VM `Kali-Linux` and left the default settings, as they seemed to be good for my application. I went through the installation process and now my VM is up and running.  

<img width="1267" height="800" alt="image" src="https://github.com/user-attachments/assets/7c192389-c5a5-4f3e-a506-ae730162039a" />

Now I'm going to repeat this process for the rest of the VMs.  

<img width="293" height="111" alt="image" src="https://github.com/user-attachments/assets/b92abadf-96d7-4529-87b2-ca9b2907a298" />

---

<a id="step-2-installing-tools-on-soc-server"></a>
## ⚙️ Step 2: Installing Tools on SOC-Server

<a id="installing-wazuh"></a>
### 2.1 Installing Wazuh
Official Quickstart: https://documentation.wazuh.com/current/quickstart.html  

```bash
wget https://packages.wazuh.com/4.12/wazuh-install.sh && sudo bash ./wazuh-install.sh -a
```

Once the installation is complete, it should create **4 services**:  
- `filebeat`  
- `wazuh-indexer`  
- `wazuh-manager`  
- `wazuh-dashboard`  

Check service status:  
```bash
systemctl status <service-name>
```

Enable service on boot:  
```bash
systemctl enable <service-name>
```

If the installation was successful, it will start up the Wazuh dashboard and give us generated credentials.  

<img width="801" height="34" alt="image" src="https://github.com/user-attachments/assets/b8dfe54b-fd1b-4b43-9b39-6c6e8b340977" />
<img width="1757" height="1221" alt="image" src="https://github.com/user-attachments/assets/9248c362-3422-4246-9df0-bb990d805b29" />

I can already see Wazuh logs from the SOC-Server populating the dashboard. Now I need to connect the Windows-Endpoint agent to Wazuh.  

<img width="1757" height="1242" alt="image" src="https://github.com/user-attachments/assets/d789a61a-c356-4a8f-83bc-aea3b0c40ef5" />

<a id="installing-splunk"></a>
### 2.2 Installing Splunk
- Download Splunk Enterprise: https://www.splunk.com/en_us/download/splunk-enterprise.html?locale=en_us  
- Install on Linux/macOS: https://help.splunk.com/en/splunk-enterprise/get-started/install-and-upgrade/10.0/install-splunk-enterprise-on-linux-or-macos/install-on-linux  
- Start Splunk: https://help.splunk.com/en/splunk-enterprise/get-started/install-and-upgrade/10.0/start-using-splunk-enterprise/start-splunk-enterprise-for-the-first-time  

Like Wazuh, it creates the `splunk` service which can be enabled with `systemctl`.

Once Splunk is started, access the Splunk dashboard using the credentials you created.  

<img width="1770" height="503" alt="image" src="https://github.com/user-attachments/assets/f89233cc-7eca-4094-85e2-2d10d45944c0" />

---

<a id="step-3-installing-configuring-tools-on-windows-endpoint"></a>
## 🪟 Step 3: Installing & Configuring Tools on Windows-Endpoint

<a id="wazuh-agent"></a>
### 3.1 Install Wazuh Agent
Docs: https://documentation.wazuh.com/current/installation-guide/wazuh-agent/index.html  

Followed the instructions in the Wazuh dashboard to deploy the agent.  

<img width="559" height="293" alt="image" src="https://github.com/user-attachments/assets/f7c51925-9852-46b9-a750-17340bcf03b8" />

<img width="1633" height="530" alt="image" src="https://github.com/user-attachments/assets/05b807dc-7bcf-4151-9be6-30e2306ff363" />

<img width="1633" height="951" alt="image" src="https://github.com/user-attachments/assets/6138a291-9de9-42d9-8b33-ca8f9abc2c08" />

<a id="sysmon"></a>
### 3.2 Installing & Configuring Sysmon
1. Download Sysmon: https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon  
2. Sysmon Config File: https://github.com/SwiftOnSecurity/sysmon-config  

```powershell
.\sysmon64.exe -accepteula -i c:\windows\sysmonconfig-export.xml
Get-Process sysmon64
```

Sysmon logs appear in **Event Viewer**:  
`Application and Services Logs > Windows > Sysmon > Operational`  

Add to `ossec.conf` for Wazuh:  
```xml
<localfile>
  <location>Microsoft-Windows-Sysmon/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>
```

Check Wazuh dashboard for Sysmon logs.  

<img width="1234" height="129" alt="image" src="https://github.com/user-attachments/assets/4c28d775-06e1-46a0-8059-78996a6b73c1" />
<img width="564" height="39" alt="image" src="https://github.com/user-attachments/assets/c56416dc-8d57-4276-82cb-3d0bd5a7aedc" />

<a id="suricata"></a>
### 3.3 Installing & Configuring Suricata
1. Download Suricata: https://suricata.io/download/  
2. Install npcap (dependency): https://npcap.com/#download  
3. Download Emerging Threats Ruleset: https://rules.emergingthreats.net/open/suricata-7.0.3/  
4. Place rules in: `/Program Files/Suricata/rules`  
5. Ensure `suricata.yaml` points to new rules.  
6. Add Suricata logs to Wazuh (`ossec.conf`):  
```xml
<localfile>
  <log_format>json</log_format>
  <location>C:\Program Files\Suricata\log\eve.json</location>
</localfile>
```
7. Start Suricata:  
```powershell
suricata.exe -i <windows-endpoint-ip>
```

Test Suricata IDS:  
```powershell
curl http://testmyids.com
```

Wazuh dashboard now shows Suricata logs ✅  

<img width="1963" height="124" alt="image" src="https://github.com/user-attachments/assets/3ec64602-cc01-4455-889d-013152818095" />
<img width="755" height="168" alt="image" src="https://github.com/user-attachments/assets/98441b25-a138-4176-8024-7bab4095b41e" />

---

<a id="step-4-configuring-splunk"></a>
## 📡 Step 4: Configuring Splunk
Now that all monitoring tools are set up, I need to forward logs from Wazuh → Splunk.  

Official Docs: https://documentation.wazuh.com/current/integrations-guide/splunk/index.html#server-integration-using-splunkforwarder  

I chose **Wazuh Server integration using the Splunk Forwarder**.  

<img width="1625" height="1195" alt="image" src="https://github.com/user-attachments/assets/755c2e50-ef91-44ce-8b74-d770934ae0d1" />

---

<a id="step-5-testing"></a>
## 🧪 Step 5: Testing
*(Testing in progress...)*

---

<a id="future-upgrades"></a>
## 🚀 Future Upgrades
1. Set up a virtualized pfSense server to isolate the SOC-Homelab from my home network.  
2. Set up Suricata on a dedicated VM so it can function as a NIDS.  
3. Configure Suricata to capture network traffic and store packet captures on an SMB share.  
4. Install tShark on my SOC-Server to analyze captured packets.  
5. Set up an Active Directory domain controller.  

**Future Topology:**  
<img width="1842" height="940" alt="image" src="https://github.com/user-attachments/assets/65928f54-eaee-4208-b7f7-066d1bc8e797" />

---

<a id="conclusion"></a>
## 🏁 Conclusion
This lab provides a foundation for learning SOC tools and workflows, including endpoint monitoring, IDS, XDR, and SIEM. Future improvements will expand its realism and security posture.

<a href="#top">⬆️ Back to top</a>
