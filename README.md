# 🛡️ Snort 2.9.20 Network Intrusion Detection System

A practical **Network Intrusion Detection System (NIDS)** project using **Snort 2.9.20** to monitor network traffic and detect suspicious activities such as ICMP attacks, high-rate ping traffic, large ICMP packets, and TCP SYN-based port scanning.

This project is designed as a network-security component for an **Active Directory Intrusion Detection System**, with the possibility of integrating Snort alerts with **Wazuh** for centralized monitoring.

---

## 📌 Project Overview

Snort is an open-source network intrusion detection and prevention system that analyzes network packets and generates alerts when traffic matches configured detection rules.

In this project, Snort is configured to monitor the `ens33` network interface and inspect traffic within the protected network.

The project demonstrates:

* Snort installation and configuration
* Network interface monitoring
* Promiscuous mode
* `HOME_NET` configuration
* Custom Snort local rules
* ICMP monitoring
* ICMP flood detection
* Large ICMP packet detection
* TCP SYN port-scan detection
* Rule threshold configuration
* IDS alert generation
* Wazuh integration concept
* Active Directory security monitoring

---

## 🎯 Objectives

The main objectives of this project are:

1. Configure Snort as a Network IDS.
2. Monitor network traffic through the `ens33` interface.
3. Detect ICMP Echo Requests.
4. Detect ICMP Echo Replies.
5. Detect unusually large ICMP packets.
6. Detect high-frequency ICMP traffic.
7. Detect possible TCP SYN port scans.
8. Create custom Snort detection rules.
9. Generate security alerts.
10. Integrate network detection with Wazuh.
11. Improve visibility in an Active Directory environment.

---

## 🏗️ Architecture

```text
                    Network Traffic
                           |
                           v
                    +-------------+
                    |    ens33    |
                    |  Interface  |
                    +-------------+
                           |
                           v
                  Promiscuous Mode
                           |
                           v
                    +-------------+
                    |    Snort    |
                    |    2.9.20   |
                    +-------------+
                           |
             +-------------+-------------+
             |                           |
             v                           v
       ICMP Detection              TCP Detection
             |                           |
             v                           v
       Ping / Flood                SYN Port Scan
             |                           |
             +-------------+-------------+
                           |
                           v
                     Snort Alerts
                           |
                           v
                        Wazuh
                           |
                           v
                    Security Dashboard
```

---

## 🧰 Technologies Used

| Technology       | Purpose                       |
| ---------------- | ----------------------------- |
| Snort 2.9.20     | Network Intrusion Detection   |
| Linux            | Snort operating environment   |
| TCP/IP           | Network communication         |
| ICMP             | Ping monitoring               |
| TCP              | Port-scan detection           |
| Nmap             | Authorized security testing   |
| Wazuh            | SIEM / endpoint monitoring    |
| Active Directory | Target enterprise environment |

---

## ⚙️ Network Configuration

The protected network is configured using:

```text
HOME_NET = 192.168.244.1/24
```

External traffic is configured as:

```text
EXTERNAL_NET = any
```

DNS servers are configured as:

```text
DNS_SERVERS = $HOME_NET
```

Example Snort configuration:

```text
ipvar HOME_NET 192.168.244.1/24
ipvar EXTERNAL_NET any
ipvar DNS_SERVERS $HOME_NET
```

---

## 🔍 Step 1 — Verify Snort Installation

Check the installed Snort version:

```bash
snort --version
```

Expected output contains:

```text
Version 2.9.20 GRE
```

This confirms that Snort is installed and available from the command line.

---

## 📡 Step 2 — Identify Network Interface

Check available network interfaces:

```bash
ip link
```

In this project, the monitored interface is:

```text
ens33
```

---

## 📶 Step 3 — Enable Promiscuous Mode

Enable promiscuous mode:

```bash
sudo ip link set ens33 promisc on
```

Verify the configuration:

```bash
ip link show ens33
```

The interface should show:

```text
PROMISC
```

Promiscuous mode allows the interface to receive additional network frames for packet inspection.

---

## 📝 Step 4 — Configure HOME_NET

Edit the Snort configuration file:

```bash
sudo nano /etc/snort/snort.conf
```

Configure:

```text
ipvar HOME_NET 192.168.244.1/24
ipvar EXTERNAL_NET any
ipvar DNS_SERVERS $HOME_NET
```

`HOME_NET` represents the protected network monitored by Snort.

---

# 🚨 Custom Snort Rules

The custom rules are stored in the local rules configuration.

Typical location:

```text
/etc/snort/rules/local.rules
```

---

## 🔵 Rule 1 — ICMP Ping Detection

```text
alert icmp any any -> $HOME_NET any \
(msg:"ICMP ping detected"; itype:8; sid:1000001; rev:1;)
```

### Purpose

Detects ICMP Echo Request packets.

ICMP type:

```text
8 = Echo Request
```

This rule can identify normal ping activity entering the protected network.

---

## 🟢 Rule 2 — ICMP Ping Reply Detection

```text
alert icmp $HOME_NET any -> any any \
(msg:"ICMP ping reply sent"; itype:0; sid:1000002; rev:1;)
```

### Purpose

Detects ICMP Echo Reply packets.

ICMP type:

```text
0 = Echo Reply
```

This helps monitor responses generated by systems inside the protected network.

---

## 🟠 Rule 3 — Large ICMP Packet Detection

```text
alert icmp any any -> $HOME_NET any \
(msg:"Large ICMP packet - possible ping flood"; \
dsize:>800; sid:1000003; rev:1;)
```

### Purpose

Detects ICMP packets with payloads larger than 800 bytes.

The detection condition is:

```text
dsize:>800
```

This can help identify abnormal ICMP traffic.

---

## 🔴 Rule 4 — High-Rate ICMP Detection

```text
alert icmp any any -> $HOME_NET any \
(msg:"Possible ICMP flood - high ping rate"; \
itype:8; \
detection_filter:track by_src, count 10, seconds 5; \
sid:1000004; rev:1;)
```

### Purpose

Detects repeated ICMP Echo Requests.

The detection filter:

```text
track by_src
```

tracks traffic based on the source address.

The threshold:

```text
count 10, seconds 5
```

means the rule triggers when the configured threshold is reached within 5 seconds for a source.

---

## 🟣 Rule 5 — TCP SYN Port Scan Detection

```text
alert tcp any any -> $HOME_NET any \
(msg:"Possible SYN port scan"; \
flags:S; \
detection_filter:track by_src, count 20, seconds 5; \
sid:1000010; rev:1;)
```

### Purpose

Detects repeated TCP SYN packets that may indicate port scanning.

The TCP flag:

```text
flags:S
```

represents the SYN flag.

Detection threshold:

```text
20 SYN packets
within 5 seconds
per source
```

---

# 📊 Detection Rule Summary

| SID     | Protocol | Detection     | Threshold  |
| ------- | -------- | ------------- | ---------- |
| 1000001 | ICMP     | Ping Request  | Type 8     |
| 1000002 | ICMP     | Ping Reply    | Type 0     |
| 1000003 | ICMP     | Large ICMP    | >800 bytes |
| 1000004 | ICMP     | ICMP Flood    | 10 / 5 sec |
| 1000010 | TCP      | SYN Port Scan | 20 / 5 sec |

---

# 🧪 Testing

> Perform security testing only against systems you own or are explicitly authorized to test.

## ICMP Testing

Generate normal ping traffic:

```bash
ping <TARGET_IP>
```

Snort should be able to identify the ICMP Echo Request.

---

## High-Rate ICMP Testing

Generate repeated ICMP traffic in your authorized lab and monitor Snort alerts.

The relevant rule is:

```text
sid:1000004
```

---

## SYN Scan Testing

Use Nmap against an authorized test system:

```bash
nmap -sS <TARGET_IP>
```

The Snort rule:

```text
sid:1000010
```

can identify repeated SYN traffic according to its configured threshold.

---

# ✅ Configuration Validation

Before running Snort, test the configuration:

```bash
sudo snort -T -c /etc/snort/snort.conf
```

The `-T` option tells Snort to test the configuration.

A successful result confirms that the configuration and rules can be loaded.

---

# ▶️ Running Snort

Start Snort using the monitored interface:

```bash
sudo snort -i ens33 -c /etc/snort/snort.conf
```

Snort will then inspect traffic arriving through the selected interface.

---

# 📁 Project Structure

```text
snort-ids/
│
├── README.md
│
├── rules/
│   └── local.rules
│
├── config/
│   └── snort.conf
│
├── screenshots/
│   ├── snort-version.png
│   ├── promiscuous-mode.png
│   ├── network-config.png
│   └── detection-rules.png
│
└── documentation/
    └── Snort_IDS_Report.pdf
```

---

# 🖼️ Screenshots

Add the screenshots from the laboratory setup to:

```text
screenshots/
```

Recommended names:

```text
snort-version.png
promiscuous-mode.png
network-config.png
detection-rules.png
```

Then display them in the README using:

```markdown
![Snort Version](screenshots/snort-version.png)
```

```markdown
![Promiscuous Mode](screenshots/promiscuous-mode.png)
```

```markdown
![Network Configuration](screenshots/network-config.png)
```

```markdown
![Detection Rules](screenshots/detection-rules.png)
```

---

# 🔗 Wazuh Integration

Snort can be used as the network detection component while Wazuh provides centralized security monitoring.

A possible architecture is:

```text
             Attacker / Test Machine
                       |
                       v
                Network Traffic
                       |
                       v
                     Snort
                       |
                       v
                Network Alerts
                       |
                       v
                    Wazuh
                       |
             +---------+---------+
             |                   |
             v                   v
        Wazuh Manager       Wazuh Dashboard
             |
             v
       Security Analysis
```

Wazuh can also monitor Windows and Active Directory events, providing endpoint visibility alongside Snort's network visibility.

---

# 🏢 Active Directory Integration

For an Active Directory security monitoring project, Snort can help identify network reconnaissance and suspicious traffic.

Example monitored activities include:

```text
Port Scanning
     |
     v
TCP SYN Traffic
     |
     v
Snort Detection
     |
     v
Alert
     |
     v
Wazuh
```

At the same time, Wazuh can monitor Windows security events, authentication activity, and endpoint behavior.

This creates a layered monitoring approach.

---

# 🔐 Security Benefits

This project provides several security benefits:

* Network traffic visibility
* ICMP monitoring
* Basic network reconnaissance detection
* Port-scan detection
* Custom rule development
* Threshold-based detection
* Security alert generation
* Integration capability with SIEM
* Active Directory network monitoring

---

# ⚠️ Limitations

The rules in this project are intended for a laboratory environment.

They should not be considered complete protection against all attacks.

For example:

* Large ICMP packets are not automatically malicious.
* High-rate ICMP traffic may have legitimate causes.
* SYN traffic can be generated by normal applications.
* Thresholds may need tuning for production networks.
* Additional rules are required for broader attack coverage.

False positives should therefore be investigated before treating an alert as a confirmed attack.

---

# 🚀 Future Improvements

Possible improvements include:

* Integrate Snort alerts directly with Wazuh.
* Add DNS attack detection.
* Add SMB traffic detection.
* Add Kerberos-related detection.
* Add LDAP monitoring.
* Detect brute-force authentication attempts.
* Create custom Active Directory detection rules.
* Forward alerts to a centralized SIEM.
* Build automated incident-response workflows.
* Add MITRE ATT&CK mapping.
* Create custom Wazuh dashboard visualizations.
* Add alert correlation between Snort and Windows events.

---

# 📚 Learning Outcomes

Through this project, the following concepts can be learned:

* Network IDS fundamentals
* Snort rule syntax
* Packet inspection
* ICMP analysis
* TCP flag analysis
* Promiscuous mode
* Network monitoring
* Detection filters
* Security alert generation
* Port-scan detection
* SIEM integration
* Active Directory security monitoring

---

# 👨‍💻 Project Purpose

This project is developed as a cybersecurity laboratory project demonstrating how network-based intrusion detection can be implemented using Snort.

It is particularly useful as a foundation for an **Intrusion Detection in Active Directory** project where Snort and Wazuh are combined to monitor both network and endpoint activity.

---

# 📌 Conclusion

Snort 2.9.20 provides a flexible platform for building a network-based intrusion detection system.

The custom rules implemented in this project demonstrate the detection of:

```text
ICMP Ping
ICMP Reply
Large ICMP Packets
High-Rate ICMP Traffic
TCP SYN Port Scanning
```

By integrating Snort with Wazuh and Active Directory monitoring, the project can be expanded into a more comprehensive security monitoring and intrusion detection platform.

---

## ⭐ Project Status

```text
Status: Completed - Laboratory Implementation
```

## 📜 License

This project is intended for educational and authorized cybersecurity testing purposes.
