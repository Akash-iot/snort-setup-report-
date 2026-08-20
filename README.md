# 🛡️ Snort Network Intrusion Detection System (IDS)

A practical **Network Intrusion Detection System (NIDS)** built using **Snort 2.9.20** to monitor network traffic and detect suspicious activities through custom detection rules.

## 🎯 Objective

The main objective of this project is to implement a lightweight network-based IDS capable of monitoring traffic and generating alerts when predefined suspicious patterns are detected.

## 🔍 IDS Features

* Network traffic monitoring
* ICMP traffic detection
* ICMP Echo Request detection
* ICMP Echo Reply detection
* Large ICMP packet detection
* High-rate ICMP traffic detection
* TCP SYN port-scan detection
* Source-based traffic threshold detection
* Custom Snort IDS rules
* Real-time security alerts

## ⚙️ Technologies

* **Snort 2.9.20**
* **Linux**
* **TCP/IP**
* **ICMP**
* **TCP**
* **Nmap**

## 🚨 Detection Rules

| Detection              |       SID |
| ---------------------- | --------: |
| ICMP Ping Detection    | `1000001` |
| ICMP Reply Detection   | `1000002` |
| Large ICMP Packet      | `1000003` |
| Possible ICMP Flood    | `1000004` |
| Possible SYN Port Scan | `1000010` |

## 🌐 Network Monitoring

The IDS monitors network traffic through the configured network interface:

```bash
ens33
```

Promiscuous mode is enabled to allow Snort to inspect network traffic:

```bash
sudo ip link set ens33 promisc on
```

## 🏠 Protected Network

The monitored network is configured using:

```text
ipvar HOME_NET 192.168.244.1/24
```

External traffic is configured as:

```text
ipvar EXTERNAL_NET any
```

## 📡 ICMP Detection

The IDS detects ICMP Echo Requests:

```text
itype:8
```

and ICMP Echo Replies:

```text
itype:0
```

This allows the system to identify and monitor ping-based network activity.

## 🚨 ICMP Flood Detection

The IDS uses a detection threshold to identify repeated ICMP requests:

```text
detection_filter:track by_src, count 10, seconds 5
```

This helps identify unusually high-frequency ICMP traffic from a single source.

## 🔎 SYN Port Scan Detection

TCP SYN packets are monitored using:

```text
flags:S
```

The IDS uses a threshold of:

```text
20 SYN packets / 5 seconds / source
```

to identify possible SYN-based port scanning.

## 🧪 IDS Testing

ICMP detection can be tested using:

```bash
ping <TARGET_IP>
```

SYN detection can be tested in an authorized lab using:

```bash
nmap -sS <TARGET_IP>
```

## ▶️ Run Snort IDS

Validate the Snort configuration:

```bash
sudo snort -T -c /etc/snort/snort.conf
```

Run Snort on the monitored interface:

```bash
sudo snort -i ens33 -c /etc/snort/snort.conf
```

## 📊 IDS Detection Flow

```text
Network Traffic
       ↓
Network Interface
       ↓
Snort IDS
       ↓
Packet Inspection
       ↓
Detection Rules
       ↓
Suspicious Traffic
       ↓
Security Alert
```

## 🔐 Purpose

This project demonstrates how **Snort can be configured as a Network Intrusion Detection System** to identify suspicious network behavior and generate security alerts.

## 📌 Status

**Network IDS — Laboratory Implementation**

## ⚠️ Disclaimer

This project is intended for **educational purposes and authorized security testing only**.
