# 🗺️ Nmap Scanning & Enumeration Cheatsheet

A quick-reference guide for the Network Mapper (Nmap) tool.

## 🚀 Basic Scans
| Command | Description |
|---------|-------------|
| `nmap <target>` | Basic default scan (Top 1,000 ports) |
| `nmap -sn <target/CIDR>` | Ping sweep (Host discovery, no port scan) |
| `nmap -p- <target>` | Scan all 65,535 ports |
| `nmap -p 22,80,443 <target>` | Scan specific ports |

## 🔍 Service & OS Detection
| Command | Description |
|---------|-------------|
| `nmap -sV <target>` | Attempt to determine service/version info |
| `nmap -O <target>` | Attempt to determine the Operating System |
| `nmap -A <target>` | Aggressive scan (OS, Services, Traceroute, Scripts) |

## ⚡ Scan Types & Performance
| Command | Description |
|---------|-------------|
| `nmap -sS <target>` | TCP SYN "Stealth" Scan (Requires sudo) |
| `nmap -sU <target>` | UDP Scan |
| `nmap -T4 <target>` | Set timing template (0=paranoid to 5=insane). T4 is standard for fast, reliable networks. |

## 📜 Nmap Scripting Engine (NSE)
| Command | Description |
|---------|-------------|
| `nmap -sC <target>` | Run default safe scripts |
| `nmap --script vuln <target>` | Run scripts in the "vuln" category |
| `nmap --script smb-enum* <target>` | Run all scripts starting with "smb-enum" |

## 💾 Output Formats
It is highly recommended to *always* save your output.
| Command | Description |
|---------|-------------|
| `nmap -oN nmap.txt <target>` | Save as normal text |
| `nmap -oX nmap.xml <target>` | Save as XML (Good for importing into other tools) |
| `nmap -oA nmap_scan <target>` | Save in all three formats (Normal, XML, Grepable) |

---
*Return to [SimplySec Playbook](../README.md)*
