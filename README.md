# Wazuh SIEM Lab - Security Monitoring & Threat Detection

## Overview
This project demonstrates practical experience with Wazuh, an open-source Security Information and Event Management (SIEM) platform. The lab environment consists of a centralized Wazuh manager deployed on Ubuntu VM with a Windows agent for real-time security monitoring and threat detection.

## Table of Contents
- [Lab Architecture](#lab-architecture)
- [Technologies Used](#technologies-used)
- [Prerequisites](#prerequisites)
- [Installation & Setup](#installation--setup)
- [Features Implemented](#features-implemented)
- [Use Cases & Detection Rules](#use-cases--detection-rules)
- [Screenshots](#screenshots)
- [Lessons Learned](#lessons-learned)
- [Future Improvements](#future-improvements)
- [Resources](#resources)

## Lab Architecture

```
┌─────────────────────────────────────┐
│      Wazuh Manager (Ubuntu VM)      │
│  - Log Aggregation                  │
│  - Event Correlation                │
│  - Alert Generation                 │
│  - Dashboard & Visualization        │
└──────────────┬──────────────────────┘
               │
               │ Agent Communication
               │ (Port 1514/1515)
               │
┌──────────────▼──────────────────────┐
│     Windows Agent (Endpoint)        │
│  - Log Collection                   │
│  - File Integrity Monitoring        │
│  - Security Event Forwarding        │
└─────────────────────────────────────┘
```

**Components:**
- **Wazuh Manager:** Ubuntu VM serving as the central management server
- **Wazuh Agent:** Windows machine configured for endpoint monitoring
- **Wazuh Dashboard:** Web-based interface for visualization and analysis

## Technologies Used
- **Wazuh** - Open-source SIEM and XDR platform
- **Ubuntu 20.04/22.04 LTS** - Manager server OS
- **Windows 10/11** - Agent endpoint
- **Elasticsearch** - Log storage and indexing
- **Kibana/OpenSearch** - Data visualization
- **VirtualBox/VMware** - Virtualization platform

## Prerequisites
- Ubuntu VM with minimum 4GB RAM and 50GB disk space
- Windows machine (physical or VM)
- Network connectivity between manager and agent
- Basic knowledge of Linux and Windows administration

## Installation & Setup

### Step 1: Wazuh Manager Installation (Ubuntu)

```bash
# Update system packages
sudo apt-get update && sudo apt-get upgrade -y

# Install required dependencies
sudo apt-get install curl apt-transport-https lsb-release gnupg -y

# Add Wazuh repository
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import && chmod 644 /usr/share/keyrings/wazuh.gpg

echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | tee -a /etc/apt/sources.list.d/wazuh.list

# Update package information
sudo apt-get update

# Install Wazuh manager
sudo apt-get install wazuh-manager -y

# Check Wazuh manager status
sudo systemctl status wazuh-manager
```

### Step 2: Install Wazuh Indexer

```bash
# Install Wazuh indexer
sudo apt-get install wazuh-indexer -y

# Start and enable the service
sudo systemctl daemon-reload
sudo systemctl enable wazuh-indexer
sudo systemctl start wazuh-indexer
```

### Step 3: Install Wazuh Dashboard

```bash
# Install Wazuh dashboard
sudo apt-get install wazuh-dashboard -y

# Start and enable the service
sudo systemctl daemon-reload
sudo systemctl enable wazuh-dashboard
sudo systemctl start wazuh-dashboard
```

### Step 4: Windows Agent Deployment

1. Download the Wazuh agent installer for Windows from the Wazuh manager
2. Run the installer with manager IP configuration:
```cmd
wazuh-agent-4.x.x-1.msi /q WAZUH_MANAGER="<MANAGER_IP>" WAZUH_REGISTRATION_SERVER="<MANAGER_IP>"
```
3. Start the Wazuh agent service:
```cmd
NET START WazuhSvc
```

### Step 5: Verify Agent Connection

```bash
# On the Wazuh manager, list connected agents
sudo /var/ossec/bin/agent_control -l
```

## Features Implemented

### 1. File Integrity Monitoring (FIM)
- Monitored critical Windows directories for unauthorized changes
- Configured real-time alerts for file modifications
- Tracked registry changes

### 2. Security Event Monitoring
- Windows Security Event Log analysis
- Failed login attempt detection
- User account modifications
- Privilege escalation attempts

### 3. Vulnerability Detection
- Automated vulnerability scanning
- CVE correlation and reporting
- Patch management tracking

### 4. Custom Detection Rules
- Brute force attack detection
- Malware execution alerts
- Suspicious PowerShell activity
- Unauthorized software installation

### 5. Dashboard & Visualization
- Security event overview dashboard
- Agent health monitoring
- Alert trending and statistics
- Compliance reporting (PCI DSS, GDPR, HIPAA)

## Use Cases & Detection Rules

### Use Case 1: Brute Force Detection
**Scenario:** Detect multiple failed login attempts on Windows agent

**Rule Configuration:**
```xml
<rule id="100001" level="10">
  <if_sid>60122</if_sid>
  <description>Multiple failed login attempts detected</description>
  <frequency>5</frequency>
  <timeframe>300</timeframe>
</rule>
```

### Use Case 2: File Integrity Monitoring
**Scenario:** Monitor critical system files for modifications

**Configuration in ossec.conf:**
```xml
<syscheck>
  <directories check_all="yes">C:\Windows\System32</directories>
  <directories check_all="yes">C:\Program Files</directories>
  <alert_new_files>yes</alert_new_files>
</syscheck>
```

### Use Case 3: Malware Detection
**Scenario:** Detect malware execution through Windows Defender logs

**Log Source:** Microsoft-Windows-Windows Defender/Operational

## Screenshots

### Wazuh Dashboard Overview
![Dashboard Screenshot](screenshots/dashboard.png)
*Main security monitoring dashboard showing real-time events*

### Agent Management
![Agent Management](screenshots/agents.png)
*Connected Windows agent status and health*

### Security Alerts
![Security Alerts](screenshots/alerts.png)
*Real-time security alerts and threat detection*

### File Integrity Monitoring
![FIM Alerts](screenshots/fim.png)
*File integrity monitoring events*

## Lessons Learned

1. **Agent-Manager Communication:** Understanding firewall rules and network configuration is critical for proper agent enrollment
2. **Log Parsing:** Different log sources require proper decoder configuration for accurate event correlation
3. **Alert Tuning:** Initial rule sets generate many false positives; tuning is essential for effective monitoring
4. **Resource Management:** SIEM platforms require adequate resources; proper sizing prevents performance issues
5. **Real-world Application:** Hands-on experience with log analysis reinforces theoretical security concepts

## Future Improvements

- [ ] Add additional agents (Linux, macOS)
- [ ] Integrate threat intelligence feeds (MISP, AlienVault OTX)
- [ ] Implement active response automation
- [ ] Create custom compliance reports
- [ ] Set up email/Slack alert notifications
- [ ] Deploy in cloud environment (AWS/Azure)
- [ ] Integrate with SOAR platform for automated incident response
- [ ] Add honeypot integration for threat detection

## Troubleshooting

### Agent Not Connecting
```bash
# Check manager firewall
sudo ufw status
sudo ufw allow 1514/tcp
sudo ufw allow 1515/tcp

# Verify manager is listening
sudo netstat -tuln | grep 1514
```

### Dashboard Not Accessible
```bash
# Check dashboard status
sudo systemctl status wazuh-dashboard

# Check logs
sudo tail -f /var/log/wazuh-dashboard/wazuh-dashboard.log
```

## Resources

- [Official Wazuh Documentation](https://documentation.wazuh.com/)
- [Wazuh GitHub Repository](https://github.com/wazuh/wazuh)
- [Wazuh Community Forums](https://groups.google.com/g/wazuh)
- [OSSEC Rules Documentation](https://www.ossec.net/docs/)

## Author

**Your Name**
- GitHub: [@yourusername](https://github.com/yourusername)
- LinkedIn: [Your LinkedIn](https://linkedin.com/in/yourprofile)

## License

This project is for educational purposes only.

## Acknowledgments

- Wazuh team for developing an excellent open-source SIEM platform
- Security community for sharing detection rules and use cases

---

⭐ If you found this project helpful, please consider giving it a star!
