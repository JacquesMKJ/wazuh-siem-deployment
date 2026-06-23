# 🛡️ Wazuh SIEM/XDR Deployment — Home Lab

> Deployment of a Wazuh security monitoring platform in a virtualized environment simulating an enterprise infrastructure.

---

## 📋 Project Description

This project involves deploying and configuring **Wazuh**, an open-source **SIEM/XDR** platform (Security Information and Event Management / Extended Detection and Response), in a virtualized lab environment using VirtualBox.

The goal is to simulate an enterprise deployment with:
- A **Wazuh Server** centralizing log collection and analysis
- A **Wazuh Agent** installed on a monitored endpoint
- **Real-time security alert detection**

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────┐
│                  Host-Only Network                  │
│                  192.168.xx.x/xx                    │
│                                                     │
│  ┌──────────────────┐      ┌──────────────────┐     │
│  │   Wazuh Server   │      │   Wazuh Agent    │     │
│  │  Ubuntu 24.04    │◄────►│  Ubuntu 24.04    │     │
│  │  192.168.56.xxx  │      │  192.168.56.xxx  │     │
│  │                  │      │                  │     │
│  │ • Wazuh Manager  │      │ • Wazuh Agent    │     │
│  │ • Wazuh Indexer  │      │ • auditd         │     │
│  │ • Wazuh Dashboard│      │                  │     │
│  │  RAM: 4 GB       │      │  RAM: 1 GB       │     │
│  │  Disk: 50 GB     │      │  Disk: 20 GB     │     │
│  └──────────────────┘      └──────────────────┘     │
└─────────────────────────────────────────────────────┘
                         │
                ┌────────▼────────┐
                │   Linux Host    │
                │  VirtualBox 7.x │
                │   Web Dashboard │
                │ https://192.168 │
                │       .56.xxx   │
                └─────────────────┘
```

---

## 🛠️ Technologies Used

| Technology | Version | Role |
|---|---|---|
| **Wazuh** | 4.7.5 | SIEM/XDR Platform |
| **Ubuntu Server** | 24.04 LTS | VM Operating System |
| **VirtualBox** | 7.x | Virtualization |
| **OpenSSH** | - | Remote VM Access |
| **auditd** | 3.1.2 | Linux System Auditing |

---

## ⚙️ Installation

### Prerequisites
- VirtualBox 7.x installed
- Ubuntu Server 24.04 LTS ISO
- Minimum 8 GB RAM on the host machine

### Step 1 — Wazuh Server VM

Create a VirtualBox VM with:
- RAM: 4096 MB
- CPU: 2
- Disk: 50 GB
- Network: Adapter 1 (NAT) + Adapter 2 (Host-Only 192.168.56.0/24)

Install Ubuntu Server 24.04, then:

```bash
# Download the Wazuh installation script
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh

# Run the full installation (Manager + Indexer + Dashboard)
sudo bash wazuh-install.sh -a --ignore-check
```

> ⚠️ `--ignore-check` is required because Wazuh 4.7 does not officially list Ubuntu 24.04 yet, but the installation works correctly.

### Step 2 — Wazuh Agent VM

Create a VirtualBox VM with:
- RAM: 1024 MB
- CPU: 1
- Disk: 20 GB
- Network: Adapter 1 (NAT) + Adapter 2 (Host-Only 192.168.xx.x/xx)

Install Ubuntu Server 24.04, then:

```bash
# Download and install the agent package
curl -o wazuh-agent.deb https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.7.5-1_amd64.deb
sudo WAZUH_MANAGER="192.168.56.xxx" dpkg -i wazuh-agent.deb

# Start the agent
sudo systemctl enable wazuh-agent && sudo systemctl start wazuh-agent

# Install auditd for system monitoring
sudo apt-get install auditd -y
sudo systemctl enable auditd && sudo systemctl start auditd
```

### Step 3 — Access the Dashboard

From the host machine browser:

```
https://192.168.56.xxx
```

Credentials: `admin` / *(password generated during installation)*

---

## 🔍 Attack Simulations

### 1. SSH Brute Force (Authentication Failure)
```bash
# From the host machine — failed SSH login attempts
ssh fakeuser@192.168.56.xxx
# Repeat with wrong passwords
```
**Result:** 6 "Authentication failure" alerts detected

### 2. Sensitive File Modification
```bash
# On the Agent VM
sudo nano /etc/passwd
sudo touch /etc/cron.d/test-attack
sudo sh -c "echo '* * * * * root echo test' >> /etc/crontab"
```
**Result:** File integrity monitoring alerts triggered

### 3. System Reconnaissance
```bash
# User and process enumeration
cat /etc/passwd
whoami && id && who
ps aux
netstat -an
```
**Result:** Suspicious system activity recorded in logs

---

## 📊 Results

| Metric | Value |
|---|---|
| Total alerts generated | **92** |
| Authentication failures | **6** |
| Authentication successes | **17** |
| Connected agents | **1 active** |
| Available modules | Security Events, Integrity Monitoring, Policy Monitoring, System Auditing, Vulnerabilities, MITRE ATT&CK, PCI DSS, NIST 800-53 |

---

## 📸 Screenshots

### Main Dashboard — Agent Overview
![Dashboard Wazuh](screenshots/screenshot1.png)

### Security Events — Real-time Alerts
![Security Events](screenshots/screenshot2.png)

### Top 5 Agents & Security Alerts
![Agents and alerts](screenshots/screenshot3.png)

### Events — Detailed Event List
![Events](screenshots/screenshot4.png)

---

## 🧠 Skills Demonstrated

- ✅ Deployment of an open-source SIEM solution in a virtualized environment
- ✅ Host-Only network configuration for VM isolation
- ✅ Wazuh agent installation and registration
- ✅ Security incident simulation and detection
- ✅ Reading and interpreting security alerts
- ✅ Using `auditd` for Linux system monitoring
- ✅ PAM, SSH, and file system log analysis

---

## 📚 Resources

- [Wazuh Official Documentation](https://documentation.wazuh.com)
- [MITRE ATT&CK Framework](https://attack.mitre.org)
- [Wazuh GitHub](https://github.com/wazuh/wazuh)

---

## 👤 Author

Project carried out as part of a cybersecurity learning path — SIEM/XDR deployment in a home lab.

---

*Project completed on June 23, 2026*
