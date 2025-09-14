<a id="top"></a>
# Creating my SOC Homelab

## Table of Contents
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
7. [Future Topology Upgrades/Plans](#future-upgrades)
8. [Conclusion](#conclusion)  

---

<a id="introduction"></a>
## Introduction
Here are the steps I took to create my SOC Homelab with Proxmox. I plan to deploy 1 Kali Linux VM for attacks, 1 Windows VM to target, and 1 Ubuntu Server VM for monitoring. I plan to utilize Sysmon for endpoint logs, Suricata for IDS, Wazuh for XDR, and Splunk for SIEM. Sysmon, Suricata, and Wazuh logs will be ingested into Wazuh, which will then be imported into Splunk. If it all works out, I'll be able to access Wazuh and Splunk through their web dashboards for log analysis.

---

<a id="topology"></a>
## Topology
<img width="1279" height="999" alt="SOC-Homelab-Topology" src="https://github.com/user-attachments/assets/70976204-e28b-4eff-9513-7cc1156bffe4" />

---

<a id="step-1-setting-up-virtual-machines"></a>
## Step 1: Setting up Virtual Machines

<a id="installing-proxmox"></a>
### 1.1 Installing Proxmox
- Install Proxmox Virtual Environment: https://www.proxmox.com/en/  
- Official documentation: https://pve.proxmox.com/pve-docs/

Once Proxmox is installed, you can use the CLI or access the Proxmox web GUI to start deploying VMs. I'm going to use the web GUI.  

<img width="2552" height="1063" alt="image" src="https://github.com/user-attachments/assets/d63ff8f7-252b-423e-b310-53465f9475dd" />

<a id="deploying-virtual-machines"></a>
### 1.2 Deploying Virtual Machines
I need to start by downloading my Linux and Windows ISOs. I'm going to choose Kali Linux for my Linux distro. It has a lot of pentesting tools built-in that I may want to try in the future. I'll choose Windows 11 Enterprise Evaluation for the target endpoint and Ubuntu Server for the system I plan to monitor the endpoint from.

**Note:** Windows 11 Enterprise Evaluation can only be used for 90 days, but the license can be extended using the `slmgr /rearm` command.  

- Kali Linux: https://www.kali.org/get-kali/#kali-platforms  
- Windows 11 Enterprise Evaluation: https://www.microsoft.com/en-us/evalcenter/evaluate-windows-11-enterprise  
- Ubuntu Server: https://ubuntu.com/download/server  

I plan to deploy my VMs on HDD-1, so I selected it and uploaded my ISOs to it.  

<img width="814" height="275" alt="image" src="https://github.com/user-attachments/assets/34a318b5-8442-4fa7-9094-97a3f9e4fa88" /> <br>

After they're done uploading, they're available to be installed on a VM.  

<img width="1460" height="909" alt="image" src="https://github.com/user-attachments/assets/2c84d353-1702-47af-9d81-85ef7231a75c" /> <br>

I named my VM `Kali-Linux` and left the default settings, as they seemed to be good for my application. I went through the installation process and now my VM is up and running.  

<img width="1267" height="800" alt="image" src="https://github.com/user-attachments/assets/7c192389-c5a5-4f3e-a506-ae730162039a" /> <br>

Now I'm going to repeat this process for the rest of the VMs.  

<img width="293" height="111" alt="image" src="https://github.com/user-attachments/assets/b92abadf-96d7-4529-87b2-ca9b2907a298" />

---

<a id="step-2-installing-tools-on-soc-server"></a>
## Step 2: Installing Tools on SOC-Server

<a id="installing-wazuh"></a>
### 2.1 Installing Wazuh
Official Quickstart Documentation: https://documentation.wazuh.com/current/quickstart.html  

```bash
wget https://packages.wazuh.com/4.12/wazuh-install.sh && sudo bash ./wazuh-install.sh -a
```

Once the installation is complete, it should create **4 services**:  
- `filebeat`  
- `wazuh-indexer`  
- `wazuh-manager`  
- `wazuh-dashboard`  

I can check the service status using the command:  
```bash
systemctl status <service-name>
```

I can enable service on boot using the command:  
```bash
systemctl enable <service-name>
```

If the installation was successful, it will start up the Wazuh dashboard and generate credentials I can log-in with.  

<img width="1757" height="1221" alt="image" src="https://github.com/user-attachments/assets/9248c362-3422-4246-9df0-bb990d805b29" /> <br>

I can already see Wazuh logs from the SOC-Server populating the dashboard. Now I need to connect the Windows-Endpoint agent to Wazuh.  

<img width="1757" height="1242" alt="image" src="https://github.com/user-attachments/assets/d789a61a-c356-4a8f-83bc-aea3b0c40ef5" />

<a id="installing-splunk"></a>
### 2.2 Installing Splunk
- Download Splunk Enterprise: https://www.splunk.com/en_us/download/splunk-enterprise.html?locale=en_us  
- Install on Linux Documentation: https://help.splunk.com/en/splunk-enterprise/get-started/install-and-upgrade/10.0/install-splunk-enterprise-on-linux-or-macos/install-on-linux  
- Start Splunk Documentation: https://help.splunk.com/en/splunk-enterprise/get-started/install-and-upgrade/10.0/start-using-splunk-enterprise/start-splunk-enterprise-for-the-first-time  

It also creates the `splunk` service where I can check the status and enable on boot with `systemctl`.

Once Splunk is started, I can access the Splunk dashboard using the credentials I created during the startup process.  

<img width="1770" height="503" alt="image" src="https://github.com/user-attachments/assets/f89233cc-7eca-4094-85e2-2d10d45944c0" />

---

<a id="step-3-installing-configuring-tools-on-windows-endpoint"></a>
## Step 3: Installing & Configuring Tools on Windows-Endpoint

<a id="wazuh-agent"></a>
### 3.1 Install Wazuh Agent
- Documentation: https://documentation.wazuh.com/current/installation-guide/wazuh-agent/index.html  

I followed the instructions in the Wazuh dashboard to deploy the agent.  

<img width="559" height="293" alt="image" src="https://github.com/user-attachments/assets/f7c51925-9852-46b9-a750-17340bcf03b8" />

<img width="1633" height="530" alt="image" src="https://github.com/user-attachments/assets/05b807dc-7bcf-4151-9be6-30e2306ff363" />

<img width="1633" height="951" alt="image" src="https://github.com/user-attachments/assets/6138a291-9de9-42d9-8b33-ca8f9abc2c08" />

<a id="sysmon"></a>
### 3.2 Installing & Configuring Sysmon
1. Download Sysmon: https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon  
2. Download a Sysmon configuration file.
   - I chose this one: https://github.com/SwiftOnSecurity/sysmon-config  
4. Start Sysmon using the command: `.\sysmon64.exe -accepteula -i c:\windows\sysmonconfig-export.xml`
5. Verify Sysmon is installed correctly using the command `Get-Process sysmon64`
   - I can also verify that it's installed by checking the Event Viewer path: `Application and Services Logs > Windows > Sysmon > Operational` 

<img width="912" height="438" alt="image" src="https://github.com/user-attachments/assets/c8edbe43-3e81-4929-b5a3-1b392c7bea11" /> 

6. Add Sysmon logs to Wazuh.
   - Documentation: https://wazuh.com/blog/using-wazuh-to-monitor-sysmon-events/
   - Add the following code to `ossec.conf` to tell Wazuh to monitor Sysmon logs:  
```xml
<localfile>
  <location>Microsoft-Windows-Sysmon/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>
```
<br>
7. Test Wazuh integration.
<img width="1234" height="129" alt="image" src="https://github.com/user-attachments/assets/4c28d775-06e1-46a0-8059-78996a6b73c1" />
<img width="564" height="39" alt="image" src="https://github.com/user-attachments/assets/c56416dc-8d57-4276-82cb-3d0bd5a7aedc" />

<a id="suricata"></a>
### 3.3 Installing & Configuring Suricata
1. Download Suricata: https://suricata.io/download/  
2. Install npcap (dependency): https://npcap.com/#download  
3. Download the Emerging Threats Ruleset: https://rules.emergingthreats.net/open/suricata-7.0.3/  
4. Place rules in: `/Program Files/Suricata/rules`  
5. Ensure `suricata.yaml` points to new rules.  
6. Add Suricata logs to Wazuh.
   - Documentation: https://documentation.wazuh.com/current/proof-of-concept-guide/integrate-network-ids-suricata.html
   - Add the following code to `ossec.conf` to tell Wazuh to monitor Suricata logs.
```xml
<localfile>
  <log_format>json</log_format>
  <location>C:\Program Files\Suricata\log\eve.json</location>
</localfile>
```
7. Start Suricata using the command `suricata.exe -i <windows-endpoint-ip>`
8. Test Suricata IDS and Wazuh integration.
   - I used the command: `curl http://testmyids.com`

<img width="1963" height="124" alt="image" src="https://github.com/user-attachments/assets/3ec64602-cc01-4455-889d-013152818095" />
<img width="755" height="168" alt="image" src="https://github.com/user-attachments/assets/98441b25-a138-4176-8024-7bab4095b41e" />

---

<a id="step-4-configuring-splunk"></a>
## Step 4: Configuring Splunk
Now that all monitoring tools are set up, I need to forward logs from Wazuh to Splunk.  

Official Documentation: https://documentation.wazuh.com/current/integrations-guide/splunk/index.html#server-integration-using-splunkforwarder  

I chose **Wazuh Server integration using the Splunk Forwarder**.  

<img width="1625" height="1195" alt="image" src="https://github.com/user-attachments/assets/755c2e50-ef91-44ce-8b74-d770934ae0d1" />

---

<a id="future-upgrades"></a>
## Future Upgrades/Plans
1. Set up a virtualized pfSense server to isolate the SOC-Homelab from my home network.  
2. Set up Suricata on a dedicated VM so it can function as a NIDS.  
3. Configure Suricata to capture network traffic and store packet captures on an SMB share.  
4. Install tShark on my SOC-Server to analyze captured packets.
5. Set up an Active Directory domain controller.  
6. Try more sophisticated and a wider arrange of attacks.
7. Spend more time in Wazuh configuring alerts to prevent false positives. 

## Future Topology
<img width="1842" height="940" alt="image" src="https://github.com/user-attachments/assets/65928f54-eaee-4208-b7f7-066d1bc8e797" />

---

<a id="conclusion"></a>
## Conclusion
Building this homelab was a valuable learning experience with many unexpected variables. It provided a deep understanding of security infrastructure including virtualization, networking, and the Linux CLI. While my initial goal was to gain practical skills for a SOC Analyst role, I also gained insight into the security architecutre requried to build a SOC. This homelab will now serve as platform for me to continuously practice Blue and Red team workflows, providing invaluable experience for my cybersecurity career. 

<a href="#top">Back to top</a>
