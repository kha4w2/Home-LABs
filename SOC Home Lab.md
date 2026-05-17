# SOC Home Lab
## Building a Realistic Detection & Prevention Pipeline

---

## 1. Network Topology & IP Addressing

<img width="1125" height="604" alt="image" src="https://github.com/user-attachments/assets/824b77f7-453c-4e81-a397-87ab325e47ac" />


The lab environment is segmented into three logical zones: Attacker Network, Internal Network, and Monitoring Network. All traffic between the attacker and victim is forced through an inline IPS to ensure inspection and control.

| Component | IP Address | Zone | Role |
|-----------|-----------|------|------|
| Kali Linux (eth0) | 192.168.214.128 | Attacker Network | Primary attack(Nmap, Hydra, DoS, SSH Brute Force, Vulnerability Scanning (Nessus, Nikto)) |
| Kali Linux (eth1) | 192.168.1.21 | Monitoring Network | Snort IDS sensor (mirror interface) |
| Windows 11 (Adapter 1) | 192.168.1.40 | Internal Network | Monitored victim (Wazuh Agent) |
| Windows 11 (Adapter 2) | 192.168.10.1 | Internal Network | Target interface for attacks |
| Ubuntu IPS (ens38) | 192.168.214.133 | Boundary | Connected to the attacker network |
| Ubuntu IPS (ens37) | 192.168.10.133 | Boundary | Connected to the victim network |
| Ubuntu IPS (Gateway) | 192.168.10.133 | Boundary | Default gateway for victim |
| Wazuh Server (Ubuntu) | 192.168.1.26 | Monitoring Network | SIEM (log collection & analysis) |

---

## 2. Introduction

This project demonstrates the design and implementation of a fully functional SOC Home Lab environment that simulates real-world enterprise security operations. The lab integrates multiple security layers including SIEM, IDS, and IPS, enabling both detection and prevention of cyber threats.

The architecture is built using virtualized systems and segmented networks to emulate attacker, victim, and monitoring zones. Tools such as Wazuh, Snort, and Kali Linux are combined to provide full visibility, alerting, and active response capabilities.

This lab transitions from passive monitoring (IDS) to active defense (IPS), validating detection accuracy and prevention effectiveness through real attack simulations.

---

## 3. Objectives

The main objectives of this SOC lab are:

- Design a segmented and controlled lab environment simulating real SOC architecture
- Deploy and configure a SIEM solution using Wazuh
- Implement network-based detection using Snort in IDS mode
- Transition Snort into inline IPS mode for active threat prevention
- Integrate IDS/IPS logs into SIEM for centralized monitoring and correlation
- Simulate real-world attacks (Reconnaissance, DoS, Brute Force, Vulnerability Scanning)
- Validate detection accuracy and prevention efficiency before and after IPS enforcement
- Build hands-on experience in SOC operations, threat detection, and incident analysis

---

## 4. Tools Used

| Category | Tool | Purpose |
|----------|------|---------|
| SIEM | Wazuh | Log collection, correlation, and alerting |
| IDS/IPS | Snort | Network intrusion detection & prevention |
| Attacking Machine | Kali Linux | Attack simulation (Nmap, Hydra, etc.) |
| Target System | Windows 11 | Victim machine |
| IPS System | Ubuntu | Inline IPS deployment |
| SIEM Server | Ubuntu | Hosts Wazuh Manager |
| Virtualization | VMware Workstation | Lab environment |

---

## 5. Step by Step

### ➤ Step 1: Initial Setup, Wazuh SIEM Deployment, and Endpoint (victim) Integration

<img width="1150" height="341" alt="image" src="https://github.com/user-attachments/assets/5e0ba649-1d73-45a1-a293-54437d8f18d3" />


Figure 1: Updating and Verifying Ubuntu Package Repository Status

<img width="1138" height="115" alt="image" src="https://github.com/user-attachments/assets/b64545d1-bd5d-4a88-895e-0be7bf8bf2f7" />

Figure 2: Downloading the Wazuh Installation Script Using curl

<img width="1121" height="402" alt="image" src="https://github.com/user-attachments/assets/428b72cf-0d19-406c-8692-0e2728277436" />

Figure 3: Executing the Wazuh Installation Script (Indexer Initialization Phase)

<img width="1108" height="378" alt="image" src="https://github.com/user-attachments/assets/c687ea40-982f-4b73-af30-30165362bf31" />

Figure 4: Completing Wazuh Manager and Dashboard Installation

<img width="1130" height="252" alt="image" src="https://github.com/user-attachments/assets/593b3477-91c4-47ef-afaf-af6124c45757" />

Figure 5: Wazuh Dashboard Login Page

<img width="1130" height="343" alt="image" src="https://github.com/user-attachments/assets/ef89f075-d612-4148-bd69-4756040d7be7" />


Figure 6: Wazuh Dashboard Health Check

<img width="1147" height="449" alt="image" src="https://github.com/user-attachments/assets/7eb49139-259f-45c3-a140-0119e1360202" />

Figure 7: Successful Ping from Windows 11 to Wazuh Server

<img width="1055" height="278" alt="image" src="https://github.com/user-attachments/assets/61280c42-67ea-4ecf-851a-0b63bbb767bc" />

Figure 8: Wazuh Dashboard – Agents Overview (No Agents Connected)

<img width="1082" height="334" alt="image" src="https://github.com/user-attachments/assets/d2f417cc-5b10-418e-b2c9-81305e026a9d" />


Figure 9: Wazuh Agent Deployment Page (Configuration Settings)

<img width="1080" height="290" alt="image" src="https://github.com/user-attachments/assets/07d8982b-9b33-47f1-8c7c-05a6014f56a9" />

Figure 10: Wazuh Agent Installation Command via PowerShell

<img width="1128" height="97" alt="image" src="https://github.com/user-attachments/assets/fca5bf6a-308f-459a-8033-d83ffeef38b7" />


Figure 11: Downloading and Installing Wazuh Agent on Windows

<img width="1128" height="98" alt="image" src="https://github.com/user-attachments/assets/e20d49ce-ac80-4e07-a292-4360e15f7c69" />


Figure 12: Successful Download of Wazuh Agent Package

<img width="1125" height="129" alt="image" src="https://github.com/user-attachments/assets/48bc5b40-6ba6-4eba-80af-8f41d3771832" />


Figure 13: Wazuh Agent Service Status (Already Running)

<img width="1127" height="326" alt="image" src="https://github.com/user-attachments/assets/a16e9dd0-ab1c-42af-b21b-cd8da5bc0c89" />


Figure 14: Wazuh Dashboard – Active Agent Successfully Connected

<img width="720" height="384" alt="image" src="https://github.com/user-attachments/assets/c8d156c5-8f30-496b-b3db-ff2889080e75" />


Figure 15: Wazuh Dashboard – Event Logs from Connected Windows 11 Agent (Log Visibility in Discover View)

<img width="1108" height="525" alt="image" src="https://github.com/user-attachments/assets/b557309a-546d-405f-9834-fa4a1da674ce" />

Figure 16: VMware Virtual Network Editor Configuration (VMnet0, VMnet1, VMnet8)

<img width="700" height="137" alt="image" src="https://github.com/user-attachments/assets/fa052604-f718-4f37-a956-7f03e57bb7d5" />

Figure 17: Kali Linux Virtual Machine Network Adapter Settings (Multi-NIC Configuration)

<img width="1125" height="412" alt="image" src="https://github.com/user-attachments/assets/f6486b24-57dd-424d-a205-bee44c03b44b" />

Figure 18: Verifying Assigned IP Addresses on Kali Linux (ip a Output)

<img width="720" height="390" alt="image" src="https://github.com/user-attachments/assets/7a4f7f74-1ef8-4026-966f-9f5013fd9677" />

Figure 19: Routing Table and Interface Status Verification on Kali Linux

**Explanation:**

This step establishes the foundational SOC lab environment by preparing the Ubuntu system, deploying the Wazuh SIEM platform (192.168.1.26), and integrating a Windows 11 endpoint (192.168.1.40) as a monitored agent. The process begins with system updates, followed by automated installation of the full Wazuh stack, including the Manager, Indexer, and Dashboard. After verifying SIEM availability through the web interface, network connectivity between the SIEM and the endpoint is confirmed. The Wazuh agent is then deployed on the Windows machine, enabling centralized log collection and visibility within the SIEM. Additionally, Kali Linux is configured with multiple network interfaces (eth0: 192.168.214.128, eth1: 192.168.1.21) to support multi-network interaction, forming the basis for attack simulation and monitoring. This step ensures that the SIEM is fully operational, endpoints are successfully integrated, and the lab network is properly structured for subsequent security testing and analysis.

**Commands Used:**

```bash
# Update and upgrade Ubuntu system (Ensures system packages are up-to-date)
sudo apt update
sudo apt upgrade

# Download Wazuh installation script. (Download the official Wazuh installer)
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh

# Install Wazuh SIEM (All-in-One) (Deploys Wazuh Manager, Indexer, and Dashboard)
sudo bash wazuh-install.sh -a

# Access Wazuh Dashboard (Web interface for SIEM monitoring)
https://192.168.1.26:443

# Verify connectivity from Windows victim (Confirms network communication with SIEM)
ping 192.168.1.26

# Install Wazuh Agent on Windows (PowerShell) (Installs and starts Wazuh agent)
Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.14.4-1.msi -OutFile $env:tmp\wazuh-agent.msi
msiexec.exe /i $env:tmp\wazuh-agent.msi /q WAZUH_MANAGER='192.168.1.26' WAZUH_AGENT_GROUP='default' WAZUH_AGENT_NAME='Windows11'
NET START Wazuh

# Verify network configuration on Kali (Validates interfaces and connectivity)
ip a
ip route
nmcli device status
sudo ethtool eth0 | grep "Link detected"
sudo ethtool eth1 | grep "Link detected"
sudo ethtool eth2 | grep "Link detected"
```

---

### ➤ Step 2: Install and Validate Snort IDS on Kali Linux

<img width="712" height="285" alt="image" src="https://github.com/user-attachments/assets/02d6b119-d61f-4182-8abd-ca70af37efc2" />

Figure 20: Updating Kali Linux Package Repository and Upgrading System Packages

<img width="1121" height="511" alt="image" src="https://github.com/user-attachments/assets/c4406ece-95b9-4072-9bd5-748b0e5da200" />

Figure 21: Installing Snort IDS and Required Dependencies

<img width="715" height="244" alt="image" src="https://github.com/user-attachments/assets/50fd4d6e-d3b6-45f3-a3f9-f9643467b990" />

Figure 22: Verifying Snort Installation Version and Configuration

<img width="709" height="256" alt="image" src="https://github.com/user-attachments/assets/fe040c07-0b0f-4ddc-b053-b7a6f439ea67" />


Figure 23: Inspecting Snort Configuration Directory (/etc/snort)

<img width="711" height="219" alt="image" src="https://github.com/user-attachments/assets/f6c86435-dc8a-4143-bb2a-c34ac7c682ae" />

![Figure 24: Reviewing Default Snort Rule Sets](s)

**Explanation:**

This step installs and validates the Snort IDS on the Kali Linux machine to enable network traffic monitoring and threat detection. The system is prepared and Snort is installed with its required dependencies and rule sets. Installation is verified to ensure proper functionality, and configuration files along with detection rules are confirmed to be in place. This ensures Snort is correctly configured and ready to operate as an IDS within the SOC lab environment.

**Commands Used:**

```bash
sudo apt update
sudo apt upgrade
sudo apt install snort -y
snort -v
ls -all /etc/snort/
ls -all /etc/snort/rules/
```

---

### ➤ Step 3: Configure Snort IDS (snort.lua Customization)

<img width="1125" height="624" alt="image" src="https://github.com/user-attachments/assets/7e78cf2c-8486-46e8-bfdb-6432f86e8e1b" />


Figure 25: Editing Snort Configuration File (snort.lua – HOME_NET Definition)

<img width="720" height="174" alt="image" src="https://github.com/user-attachments/assets/c05f0840-1211-4ab2-a790-f6ce0303d709" />

Figure 26: Adding Local Rule Path in Snort Configuration

<img width="720" height="222" alt="image" src="https://github.com/user-attachments/assets/cae58b6c-1f82-4f67-968a-6ef3eea284b7" />

Figure 27: Configuring Snort Alert Output (alert_fast and alert_json Modes)

**Commands Used:**

```bash
sudo nano /etc/snort/snort.lua
```

**Explanation:**

This step configures Snort alert output formats to support both real-time monitoring and SIEM integration. The alert_fast mode is enabled for quick, human-readable alerts during live analysis and debugging, while alert_json is configured to generate structured logs suitable for ingestion by the Wazuh SIEM. This ensures efficient alert visibility and seamless log correlation within the SOC environment.

---

### ➤ Step 4: Create Custom Detection Rules and Validate Snort Alerts

<img width="1151" height="417" alt="image" src="https://github.com/user-attachments/assets/6244d7a9-cfd4-4ceb-bad6-25f2e91bcf5f" />


Figure 28: Adding Snort Local Rules File (local.rules)

<img width="737" height="282" alt="image" src="https://github.com/user-attachments/assets/850efd5a-8aad-438a-a4c6-0c0650d4538e" />

Figure 29: Validating Snort Configuration (Test Mode)

<img width="720" height="96" alt="image" src="https://github.com/user-attachments/assets/f50e4df1-fcca-4692-9648-2bff890ff36e" />

Figure 30: Running Snort in IDS Mode (alert_fast Output)

<img width="720" height="310" alt="image" src="https://github.com/user-attachments/assets/d03db577-e395-4a44-a008-bb6b584d3c4d" />

Figure 31: Real-Time Snort Alerts During Nmap SYN Scan and ICMP Traffic Simulation

<img width="720" height="602" alt="image" src="https://github.com/user-attachments/assets/eab3d764-625c-4428-8dcd-f310d406c33c" />

Figure 32: Real-Time Snort Alerts in JSON Format (SYN Scan, ICMP Ping, and ICMP Flood Detection)

**Commands Used:**

```bash
sudo nano /etc/snort/rules/local.rules
sudo snort -T -c /etc/snort/snort.lua -i eth1
sudo mkdir -p /var/log/snort
sudo snort -c /etc/snort/snort.lua -i eth1 -l /var/log/snort
nmap -sS 192.168.1.40
ping -i 0.5 192.168.1.40
sudo cat /var/log/snort/alert_fast.txt
sudo cat /var/log/snort/alert_json.txt | jq
```

**Explanation:**

This step involves creating custom Snort detection rules and validating their effectiveness through simulated attack scenarios. Custom rules are defined to detect ICMP ping, ICMP flood (DoS), and TCP SYN scan activities, enabling tailored threat detection within the lab environment. The configuration is tested to ensure all rules are correctly loaded without errors, followed by running Snort in IDS mode for real-time monitoring. Attack simulations using Nmap and ICMP traffic successfully trigger the defined rules, generating alerts in both alert_fast and alert_json formats. The results confirm accurate detection, proper logging, and readiness for analysis and integration with the SIEM, validating Snort's functionality as an effective IDS in the SOC lab.

---

### ➤ Step 5: Integration of Snort IDS with Wazuh SIEM and End-to-End Alert Validation


<img width="1135" height="453" alt="image" src="https://github.com/user-attachments/assets/645a3d82-9b1b-4040-bb43-12069b7b493f" />


Figure 33: Wazuh Dashboard – Linux Agent Deployment Configuration (DEB Package Selection)


<img width="1131" height="351" alt="image" src="https://github.com/user-attachments/assets/74af8369-7b29-43a3-b924-c09266705837" />


Figure 34: Generated Wazuh Agent Installation Command for Kali Linux

<img width="1133" height="459" alt="image" src="https://github.com/user-attachments/assets/23eef3a7-309f-4de6-83d6-d9a273f2a3d0" />

Figure 35: Installing and Starting Wazuh Agent on Kali Linux (Snort Node)

<img width="1133" height="345" alt="image" src="https://github.com/user-attachments/assets/9369fbee-6131-4ef1-868a-aa361745ec33" />

Figure 36: Wazuh Dashboard – Kali Agent Successfully Connected (Snort_IDS_IPS Active)

<img width="1133" height="522" alt="image" src="https://github.com/user-attachments/assets/ec453059-0142-4c8e-8b1e-c2858eaad4bd" />

Figure 37: Wazuh Discover View – Incoming Logs from Snort Agent (Pre-Parsing Stage)

<img width="1136" height="129" alt="image" src="https://github.com/user-attachments/assets/5abbeade-822f-4d3e-bbcc-964421cc90b6" />

Figure 38: Editing Wazuh Agent Configuration File (ossec.conf) on Kali Linux

<img width="1138" height="417" alt="image" src="https://github.com/user-attachments/assets/ffb5c9ce-0569-4fa7-a2bf-747687c41d05" />

Figure 39: Configuring Client Buffer and Agent Enrollment Settings

<img width="1125" height="397" alt="image" src="https://github.com/user-attachments/assets/6aef57fd-a494-4c4a-a79e-0b16e0ca0794" />

Figure 40: Adding Snort JSON Log Source to Wazuh Agent Configuration

<img width="1125" height="406" alt="image" src="https://github.com/user-attachments/assets/ad009345-a323-4189-9812-10de750000cb" />

Figure 41: Adjusting Log File Permissions and Restarting Wazuh Agent Service

<img width="1133" height="70" alt="image" src="https://github.com/user-attachments/assets/4d083783-c519-4cab-ae79-c6a19a9a8548" />

Figure 42: Editing Wazuh Manager Configuration File (ossec.conf) on Ubuntu Server

<img width="1125" height="332" alt="image" src="https://github.com/user-attachments/assets/a9519e00-823c-4f79-9473-8e8ab3c267bb" />

Figure 43: Enabling JSON Output Logging on Wazuh Manager

<img width="1125" height="369" alt="image" src="https://github.com/user-attachments/assets/fc78e4e9-eb7d-47ae-8a63-27c673ec2650" />

Figure 44: Adding Additional Log Sources on Wazuh Manager

<img width="1125" height="52" alt="image" src="https://github.com/user-attachments/assets/1f8eab78-9f34-4948-af7c-ee8852b2733b" />

Figure 45: Editing Local Rules File on Wazuh Manager (local_rules.xml)

<img width="1125" height="426" alt="image" src="https://github.com/user-attachments/assets/b4038b2a-9378-424a-b825-78fab03be66f" />

Figure 46: Creating Custom Wazuh Rules for Snort Alert Correlation (SYN Scan Detection)

<img width="1124" height="514" alt="image" src="https://github.com/user-attachments/assets/d33f5d67-0c66-424b-80a0-c58a9d5dad76" />

Figure 47: Validating Wazuh Rules Using wazuh-logtest Tool

<img width="1124" height="579" alt="image" src="https://github.com/user-attachments/assets/218fb2cd-a8ab-4995-82c7-49a39ec8f8aa" />

Figure 48: Modifying Filebeat Ingest Pipeline to Handle Timestamp Parsing Issue

<img width="1125" height="219" alt="image" src="https://github.com/user-attachments/assets/34744c37-99bb-4148-bb98-2b6eb7cca0a3" />

Figure 49: Reloading Filebeat Pipelines and Restarting Wazuh Stack Services

<img width="1125" height="553" alt="image" src="https://github.com/user-attachments/assets/6cc4948b-3c4a-4d1e-a4e7-480b1168d290" />

Figure 50: Real-Time Attack Simulation (Nmap SYN Scan) and Snort Alert Generation

<img width="1125" height="547" alt="image" src="https://github.com/user-attachments/assets/c27bcb83-da04-4b8b-9f99-36a070a43c7e" />

Figure 51: Wazuh Discover View – Confirmed Detection of SYN Scan Alerts (Rule ID: 100201)

**Explanation:**

In this step, Snort IDS is fully integrated with Wazuh SIEM by deploying the Wazuh agent on the Kali machine and configuring it to monitor Snort JSON alert logs. The agent is tuned using buffer optimization and proper file permissions to ensure reliable log collection. On the Wazuh Manager side, JSON logging is enabled and custom detection rules are created to parse and classify Snort alerts, specifically identifying SYN scan attacks with high severity. The configuration is validated using wazuh-logtest, and a parsing issue related to timestamps is resolved by modifying the Filebeat ingest pipeline. After reloading the pipeline and restarting the SIEM stack, Snort is executed and an attack is simulated using Nmap. The generated alerts are successfully ingested, parsed, indexed, and visualized in Wazuh, confirming complete end-to-end integration and accurate threat detection within the SOC environment.

**Commands Used:**

```bash
# Install Wazuh Agent on Kali
sudo wget https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.14.4-1_amd64.deb
sudo WAZUH_MANAGER='192.168.1.26' WAZUH_AGENT_GROUP='default' WAZUH_AGENT_NAME='Snort_IDS_IPS' dpkg -i ./wazuh-agent_4.14.4-1_amd64.deb

# Start and Enable Agent
sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
sudo systemctl status wazuh-agent

# Edit Agent Configuration (Kali)
sudo nano /var/ossec/etc/ossec.conf

# Adjust Permissions for Snort Logs
sudo chmod 755 /var/log/snort
sudo chmod 644 /var/log/snort/alert_json.txt

# Restart Agent
sudo systemctl restart wazuh-agent
sudo systemctl status wazuh-agent

# Edit Wazuh Manager Configuration (Ubuntu)
sudo nano /var/ossec/etc/ossec.conf

# Edit Local Rules
sudo nano /var/ossec/etc/rules/local_rules.xml

# Validate Rules
sudo /var/ossec/bin/wazuh-logtest

# Edit Filebeat Pipeline
sudo nano /usr/share/filebeat/module/wazuh/alerts/ingest/pipeline.json

# Reload Pipelines and Restart Services
sudo filebeat setup --pipelines --modules wazuh
sudo systemctl restart filebeat
sudo systemctl restart wazuh-manager
sudo systemctl restart wazuh-indexer
sudo systemctl restart wazuh-dashboard

# Run Snort IDS
sudo snort -c /etc/snort/snort.lua -i eth1 -l /var/log/snort &

# Simulate Attack
nmap -sS 192.168.1.40

# Verify Alerts in Elasticsearch
curl -k -u admin:PASSWORD "https://localhost:9200/wazuh-alerts-4.x-*/_search?q=rule.id:100201&pretty"
```

---

### ➤ Step 6: Deploying Snort as an Inline IPS Using NFQUEUE for Active Traffic Prevention

<img width="1130" height="1344" alt="image" src="https://github.com/user-attachments/assets/0ba9ecca-7dff-47a2-b538-af3395da88ba" />


Figure 52: Virtual Machine Network Adapter Configuration for Inline IPS Deployment


<img width="1125" height="549" alt="image" src="https://github.com/user-attachments/assets/5dee015e-f94c-4d69-a454-0844bc697fb8" />


Figure 53: System Preparation and Enabling IP Forwarding for Packet Routing

<img width="1125" height="839" alt="image" src="https://github.com/user-attachments/assets/9c824c23-613f-456d-9a43-01fe2572c85c" />


Figure 54: Identifying Victim Network Interface and Interface Index on Windows

![Figure 55: Configuring Default Gateway on Victim to Route Traffic via IPS](s)

![Figure 56: Configuring iptables to Redirect Traffic into NFQUEUE for Inline Inspection](s)

![Figure 57: Assigning Static IP Addresses and Disabling DHCP on IPS Interfaces](s)

![Figure 58: Verifying Network Interfaces and Routing Table on IPS Machine](s)

![Figure 59: Configuring Attacker Routing to Forward Traffic via IPS](s)

![Figure 60: Verifying Snort Installation and Version](s)

![Figure 61: Configuring HOME_NET and EXTERNAL_NET in Snort](s)

![Figure 62: Configuring DAQ Module for NFQUEUE Inline Mode](s)

![Figure 63: Enabling IPS Mode and Loading Custom Detection Rules](s)

![Figure 64: Configuring Snort Output Modules (alert_fast and alert_json)](s)

![Figure 65: Creating Log Directory and Setting Proper Permissions](s)

![Figure 66: Creating Custom Snort Rules for Attack Detection and Prevention](s)

![Figure 67: Running Snort in Inline IPS Mode with NFQUEUE](s)

![Figure 68: Validating Traffic Blocking from Attacker (ICMP Failure)](s)

![Figure 69: Verifying Snort Alerts Generated from Blocked Traffic](s)

**Explanation:**

In this step, a dedicated Ubuntu machine is deployed as an inline Intrusion Prevention System (IPS) using Snort with NFQUEUE integration. The network architecture is adjusted to force all traffic between the attacker (Kali Linux) and the victim (Windows 11) to pass through the IPS by modifying routing tables and default gateways. IP forwarding is enabled to allow packet traversal, while iptables rules are configured to redirect traffic into NFQUEUE for deep inspection. Snort is configured in inline mode with custom detection rules targeting common attack techniques such as ICMP flooding, SYN scans, and brute-force attempts. Once operational, Snort actively inspects and blocks malicious traffic in real time. The setup is validated by launching attack simulations, where traffic is successfully dropped and corresponding alerts are generated, confirming effective inline prevention and proper IPS functionality within the SOC lab environment. (an Ubuntu-based inline IPS (192.168.1.5) is deployed using Snort with NFQUEUE between the attacker (192.168.214.129 – eth0) and victim (192.168.10.1 – VMnet1) networks via interfaces ens38 (192.168.214.133) and ens37 (192.168.10.133), where routing and iptables are configured to force traffic through the IPS, enabling real-time inspection and active blocking of malicious traffic such as ICMP and SYN scans.)

**Commands Used:**

```bash
# Update system packages (Ensures the system is up-to-date before configuration)
sudo apt update && sudo apt upgrade

# Enable IP forwarding (Allows the system to route packets between interfaces)
sudo sysctl -w net.ipv4.ip_forward=1
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
cat /proc/sys/net/ipv4/ip_forward

# Verify NFQUEUE support (Confirms that Snort supports inline mode via NFQUEUE)
snort --daq-list | grep nfq

# Install iptables persistence (Ensures iptables rules persist after reboot)
sudo apt install iptables-persistent -y

# Configure iptables rules for inline IPS (Redirects forwarded traffic into NFQUEUE for inspection)
sudo iptables -F
sudo iptables -X
sudo iptables -F FORWARD
sudo iptables -I FORWARD -i ens37 -o ens38 -j NFQUEUE --queue-num 5 --queue-bypass
sudo iptables -I FORWARD -i ens38 -o ens37 -j NFQUEUE --queue-num 5 --queue-bypass
sudo iptables -L FORWARD -v -n
sudo netfilter-persistent save

# Configure network interfaces (Assign static IPs and verify routing configuration)
sudo nano /etc/netplan/00-installer-config.yaml
sudo netplan apply
ip a
ip route

# Configure attacker routing (Forces traffic to pass through the IPS)
sudo ip route add 192.168.10.0/24 via 192.168.214.133 dev eth0
ip route show

# Verify Snort installation (Displays Snort version and confirms installation)
snort -V

# Edit Snort configuration (Configure networks, DAQ, and IPS mode)
sudo nano /etc/snort/snort.lua

# Prepare logging directory (Creates log files and assigns proper permissions)
sudo mkdir -p /var/log/snort
sudo chmod 755 /var/log/snort
sudo touch /var/log/snort/alert_fast.txt
sudo touch /var/log/snort/alert_json.txt
sudo chmod 644 /var/log/snort/*

# Create custom rules (Adds detection and prevention rules)
sudo nano /etc/snort/rules/local.rules

# Run Snort in inline IPS mode (Starts Snort in inline mode)
sudo snort -Q --daq nfq --daq-mode inline -c /etc/snort/snort.lua -i 5 -l /var/log/snort -A alert_fast

# Test attack traffic (Simulates attack traffic)
ping -I eth0 192.168.10.1

# View alerts (Verifies detection and blocking)
sudo cat /var/log/snort/alert_fast.txt
```

---

### ➤ Step 7: Attack Simulation and IPS Effectiveness Validation

![Figure 70: Nmap SYN Scan Behavior Before and After Enabling Snort IPS](s)

![Figure 71: ICMP Flood (DoS Attack) Before and After IPS Enforcement](s)

![Figure 72: SSH Brute Force Attack Execution Using Hydra](s)

![Figure 73: SSH Brute Force Attack Results (Before vs After IPS)](s)

![Figure 74: Web Vulnerability Scanning Using Nikto (Before vs After IPS)](s)

![Figure 77: Nessus Scan Results Before Enabling Snort IPS](s)

![Figure 77: Nessus Scan Results After Enabling Snort IPS](s)

![Figure 77: Snort IPS Alerts During Attack Simulation](s)

**Explanation:**

This step evaluates the effectiveness of the deployed security architecture by simulating multiple real-world attack scenarios and comparing system behavior before and after enabling Snort in inline IPS mode.

Initially, reconnaissance and exploitation techniques such as SYN scanning, ICMP flooding, SSH brute force attacks, and vulnerability scanning using Nikto and Nessus are executed while the IPS is inactive. During this phase, all attacks successfully reach the target system, demonstrating normal network communication and full service exposure.

After enabling Snort IPS, a significant behavioral change is observed. Reconnaissance activities such as Nmap and Nessus scans fail to identify the target, ICMP flood traffic is completely dropped, and brute force attempts are disrupted due to connection limitations and filtering mechanisms. Additionally, web vulnerability scanning using Nikto is blocked, preventing further enumeration of the target system.

Simultaneously, Snort generates real-time alerts for each detected attack, including SYN scan and port scanning activities, providing full visibility into malicious behavior.

This confirms that the system has successfully transitioned from a passive detection model to an active prevention architecture, where threats are not only identified but also mitigated in real time. The results validate the effectiveness of Snort as an inline IPS within the SOC lab environment.

**Commands Used:**

```bash
# Nmap SYN Scan
nmap -sS -e eth0 192.168.10.1

# ICMP Flood (DoS Attack)
ping -i 0.05 -I eth0 192.168.10.1

# SSH Brute Force Attack using Hydra
hydra -l sshuser -P /usr/share/wordlists/rockyou.txt ssh://192.168.10.1 -t 4 -V -f

# Web Vulnerability Scanning using Nikto
nikto -h http://192.168.10.1

# Nessus Vulnerability Scanning (via Web Interface)
https://127.0.0.1:8834

# View Snort Alerts
sudo cat /var/log/snort/alert_fast.txt
```

---

## 6. Conclusion

This project successfully demonstrates the design and implementation of a fully integrated SOC Home Lab that combines monitoring, detection, and active prevention capabilities within a controlled virtual environment. The lab evolved from a basic SIEM deployment using Wazuh into a multi-layered security architecture incorporating both IDS and IPS functionalities through Snort. By enforcing traffic flow through an inline IPS, the environment effectively simulates real-world network defense strategies where inspection and control are centralized. Through multiple attack simulations using Kali Linux, the system demonstrated a clear transition from passive detection (log generation and alerting) to active prevention (real-time traffic blocking). Integration with the SIEM enabled centralized visibility, correlation, and analysis of security events across the entire environment. Overall, this lab validates the effectiveness of combining SIEM, IDS, and IPS technologies to build a realistic SOC workflow that covers the full lifecycle of cyber threat management:

**Detection → Analysis → Correlation → Response → Prevention**
