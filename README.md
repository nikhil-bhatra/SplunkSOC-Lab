<div align="center">

# 🛡️ SOC Home Lab — Threat Detection & Incident Analysis

**A fully virtualized Security Operations Center (SOC) built to simulate real-world cyberattacks, collect and analyze system logs, and monitor security events using industry-standard tools.**

![Ubuntu](https://img.shields.io/badge/Ubuntu-E95420?style=flat&logo=ubuntu&logoColor=white)
![Kali Linux](https://img.shields.io/badge/Kali_Linux-557C94?style=flat&logo=kalilinux&logoColor=white)
![Splunk](https://img.shields.io/badge/Splunk-000000?style=flat&logo=splunk&logoColor=white)
![Metasploit](https://img.shields.io/badge/Metasploit-2596CD?style=flat&logo=metasploit&logoColor=white)

</div>

---

## 📌 Table of Contents
- [Overview](#-overview)
- [Lab Architecture](#-lab-architecture)
- [Tools & Technologies](#-tools--technologies)
- [Setup & Installation](#-setup--installation)
- [Attack Simulations](#-attack-simulations)
- [Log Collection & Analysis](#-log-collection--analysis)
- [Splunk Detection Queries](#-splunk-detection-queries-spl)
- [Alerts & Dashboards](#-alerts--dashboards)
- [Key Findings](#-key-findings)
- [Skills Demonstrated](#-skills-demonstrated)
- [Future Improvements](#-future-improvements)

---

## 📖 Overview

This project simulates a real-world SOC environment built entirely on a home lab setup. The lab uses **Kali Linux** as the attacker machine, **Ubuntu** as the victim/target machine, and **Splunk** as the SIEM platform to detect, monitor, and analyze threats in real time.

**Goals of this project:**
- Simulate common cyberattacks in a safe, isolated environment
- Collect and forward system/network logs to a centralized SIEM
- Write detection rules and build dashboards to identify threats
- Perform hands-on incident analysis and develop response skills

---

## 🏗️ Lab Architecture

```
┌─────────────────────┐          ┌─────────────────────┐
│    Kali Linux       │  ──────▶ │   Ubuntu Server     │
│   (Attacker VM)     │  Attack  │   (Victim/Target)   │
│                     │          │  Splunk UF Agent    │
└─────────────────────┘          └──────────┬──────────┘
                                            │
                                   Logs forwarded
                                   via port 9997
                                            │
                                            ▼
                                 ┌──────────────────────┐
                                 │   Splunk Enterprise   │
                                 │   (on Ubuntu Host)    │
                                 │   Search, Alert,      │
                                 │   Dashboard & Monitor │
                                 └──────────────────────┘
```

> All machines run as Virtual Machines on the same host.
> Network is configured in **Host-Only / NAT mode** to fully isolate all traffic.

---

## 🛠️ Tools & Technologies

| Tool | Role | Category |
|---|---|---|
| **Ubuntu** | Victim machine & Splunk host | Target |
| **Kali Linux** | Attack simulation platform | Attacker |
| **Splunk Enterprise** | SIEM — log ingestion, search, alerting, dashboards | SIEM |
| **Splunk Universal Forwarder** | Ships logs from Ubuntu to Splunk indexer | Agent |
| **Nmap** | Network scanning and service enumeration | Recon |
| **Metasploit** | Exploitation framework for attack simulation | Exploit |
| **Hydra** | Brute-force attack simulation over SSH | Brute Force |
| **Syslog / Auth.log** | Linux system log sources | Log Source |

---

## ⚙️ Setup & Installation

### Prerequisites
- Virtualization software: **VirtualBox** or **VMware Workstation**
- Minimum: **8GB RAM**, **50GB free disk space**
- ISO images for **Ubuntu** and **Kali Linux**
- **Splunk Enterprise** (free trial at [splunk.com](https://www.splunk.com))

### Step 1 — Set Up Ubuntu (Target + Splunk Host)
```bash
sudo apt update && sudo apt upgrade -y

# Install Splunk Enterprise
wget -O splunk.deb 'https://download.splunk.com/...'
sudo dpkg -i splunk.deb
sudo /opt/splunk/bin/splunk start --accept-license
sudo /opt/splunk/bin/splunk enable boot-start
```

### Step 2 — Set Up Kali Linux (Attacker)
```bash
# Verify key tools are installed
nmap --version
msfconsole --version
hydra --version
```

### Step 3 — Install & Configure Splunk Universal Forwarder
```bash
# Install the forwarder
wget -O splunkforwarder.deb 'https://download.splunk.com/...'
sudo dpkg -i splunkforwarder.deb

# Point it to your Splunk indexer
sudo /opt/splunkforwarder/bin/splunk add forward-server <SPLUNK_IP>:9997

# Add log sources to monitor
sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/syslog
sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/auth.log
sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/kern.log

# Start the forwarder
sudo /opt/splunkforwarder/bin/splunk start
```

### Step 4 — Configure Splunk to Receive Logs
In the **Splunk Web UI**:
```
Settings → Forwarding and Receiving → Configure Receiving → New Receiving Port → 9997
```

---

## ⚔️ Attack Simulations

### 01 — Network Scanning with Nmap
```bash
# Full service and version detection scan
nmap -sV -sC -A <UBUNTU_IP>

# Stealth SYN scan
nmap -sS -T4 <UBUNTU_IP>
```
| Detail | Value |
|---|---|
| **Goal** | Enumerate running services and map the attack surface |
| **Log Generated** | `/var/log/syslog` |
| **Indicator of Compromise** | Rapid sequential port connection attempts |

---

### 02 — SSH Brute Force with Hydra
```bash
# Brute-force SSH login using rockyou.txt wordlist
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://<UBUNTU_IP>

# Target multiple users
hydra -L users.txt -P /usr/share/wordlists/rockyou.txt ssh://<UBUNTU_IP>
```
| Detail | Value |
|---|---|
| **Goal** | Simulate a credential stuffing / brute-force attack over SSH |
| **Log Generated** | `/var/log/auth.log` |
| **Indicator of Compromise** | Hundreds of "Failed password" entries from a single IP |

---

### 03 — System Exploitation with Metasploit
```bash
# Launch Metasploit
msfconsole

# Search for applicable exploits
msf> search type:exploit platform:linux
msf> use exploit/<module>
msf> set RHOSTS <UBUNTU_IP>
msf> set LHOST <KALI_IP>
msf> run
```
| Detail | Value |
|---|---|
| **Goal** | Simulate a full system compromise attempt |
| **Log Generated** | `/var/log/syslog`, `/var/log/kern.log` |
| **Indicator of Compromise** | Unexpected processes, unusual network connections |

---

## 📊 Log Collection & Analysis

### Log Sources Monitored

| Log File | Path | What It Captures |
|---|---|---|
| **Auth Log** | `/var/log/auth.log` | SSH logins, sudo usage, failed authentications |
| **Syslog** | `/var/log/syslog` | General system events and service activity |
| **Kernel Log** | `/var/log/kern.log` | Kernel-level events, device activity |
| **dpkg Log** | `/var/log/dpkg.log` | Software installation and removal events |

---

## 🔎 Splunk Detection Queries (SPL)

### Detect SSH Brute Force
```spl
index=main source="/var/log/auth.log" "Failed password"
| stats count by src_ip, user
| where count > 10
| sort - count
```

### Detect Successful Login After Multiple Failures
```spl
index=main source="/var/log/auth.log" ("Failed password" OR "Accepted password")
| stats
    count(eval(match(_raw,"Failed")))   AS failures,
    count(eval(match(_raw,"Accepted"))) AS successes
    by src_ip
| where failures > 5 AND successes > 0
```

### Detect Nmap Scan Signatures
```spl
index=main source="/var/log/syslog" ("port scan" OR "nmap" OR "SYN")
| table _time, host, src_ip, _raw
| sort - _time
```

### Monitor New User Account Creation
```spl
index=main source="/var/log/auth.log" ("useradd" OR "adduser" OR "new user")
| table _time, host, _raw
```

### Detect Sudo Command Usage
```spl
index=main source="/var/log/auth.log" "sudo"
| rex field=_raw "sudo:\s+(?<sudo_user>\S+)"
| table _time, sudo_user, host, _raw
```

---

## 📈 Alerts & Dashboards

### Splunk Alerts Configured

| Alert Name | Trigger Condition | Severity |
|---|---|---|
| SSH Brute Force Detected | 10+ failed SSH logins from one IP within 5 minutes | 🔴 High |
| Login After Failures | Successful login preceded by 5+ failures from same IP | 🔴 High |
| New User Account Created | A new system account is created | 🟡 Medium |
| Sudo Command Executed | Any sudo usage is logged and flagged for review | 🟡 Medium |

### Dashboards Built in Splunk
- **Security Overview** — Real-time counts of failed logins, active alerts, and network events
- **Authentication Monitor** — Timeline of login attempts broken down by IP address and username
- **Top Threat Sources** — Ranked list of IPs generating the most suspicious activity

---

## 🔍 Key Findings

- **Brute-force attacks generate massive log volume** — Hydra produced hundreds of failed authentication entries within minutes, demonstrating why account lockout policies and rate limiting are essential
- **Nmap scans leave clear fingerprints** — Port sweep patterns are easy to detect with targeted Splunk queries once you know what to look for
- **Real-time alerting is highly effective** — Splunk's alert engine flagged suspicious activity within seconds of each simulated attack
- **Logs tell a complete story** — Even without an agent on the attacker machine, the victim logs alone provided a full timeline of every attack stage, usable for post-incident forensic analysis
- **Correlating failed + successful logins is critical** — A single successful login after many failures is a much stronger indicator of compromise than failures alone

---

## 🎓 Skills Demonstrated

- ✅ Setting up and managing a virtualized multi-VM lab environment
- ✅ Installing and configuring Splunk Enterprise as a SIEM
- ✅ Deploying and managing the Splunk Universal Forwarder
- ✅ Simulating attacks with Nmap, Hydra, and Metasploit
- ✅ Writing SPL (Splunk Processing Language) queries for threat detection
- ✅ Building dashboards and real-time alerts in Splunk
- ✅ Analyzing logs to identify Indicators of Compromise (IoCs)
- ✅ Understanding attacker methodology through hands-on offensive simulation
- ✅ Linux system administration and log management
- ✅ Fundamentals of incident response and security monitoring

---

## 🚀 Future Improvements

- [ ] Add a **Windows 10 VM** as an additional target for Windows Event Log analysis
- [ ] Integrate **Snort or Suricata** as a network-based intrusion detection system (NIDS)
- [ ] Deploy **Wazuh** as a host-based intrusion detection system (HIDS)
- [ ] Simulate advanced attack scenarios — privilege escalation, lateral movement, persistence
- [ ] Build automated **incident response playbooks** triggered by Splunk alerts
- [ ] Integrate live **threat intelligence feeds** into Splunk for real-time IoC matching
- [ ] Set up **pfSense** as a network firewall to capture and analyze traffic at the perimeter

---

## 👤 Author

Nikhil Sharma
- GitHub: [@nikhil-bhatra](https://github.com/nikhil-bhatra)
- LinkedIn: [linkedin.com/in/nikhil-bhatra](https://www.linkedin.com/in/nikhil-sharma-75599721b/)

---

## 📄 License

This project is for **educational purposes only**.
All attack simulations were conducted in a fully isolated lab environment.
> ⚠️ Do not use these techniques on systems you do not own or have explicit written permission to test.

---

<div align="center">

⭐ If you found this project useful, consider giving it a star!

</div>
