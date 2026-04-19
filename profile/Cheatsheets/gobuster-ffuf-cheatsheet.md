# 🔍 Directory Fuzzing — Gobuster & FFUF Cheatsheet

A quick-reference guide for discovering hidden files, directories, subdomains, and virtual hosts.

> [!TIP]
> Always save your output. Fuzzing results are noisy — filtering by status code, response size, and word count will surface the real targets.

## 🗂️ Wordlists (SecLists)
Install SecLists first — it's the gold standard wordlist collection.
```bash
sudo apt install seclists
# Or: git clone https://github.com/danielmiessler/SecLists /opt/SecLists
```
| Wordlist | Path | Use Case |
|----------|------|----------|
| Common directories | `/usr/share/seclists/Discovery/Web-Content/common.txt` | Fast initial scan |
| Medium directories | `/usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt` | Thorough scan |
| API endpoints | `/usr/share/seclists/Discovery/Web-Content/api/api-endpoints.txt` | REST API fuzzing |
| Subdomains | `/usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt` | Subdomain enumeration |

---

## 🚀 Gobuster

### Directory/File Fuzzing
```bash
# Basic directory scan
gobuster dir -u http://target.com -w /usr/share/seclists/Discovery/Web-Content/common.txt

# Scan for specific extensions
gobuster dir -u http://target.com -w /opt/SecLists/Discovery/Web-Content/common.txt -x php,html,txt,bak,zip

# Ignore specific status codes and add auth header
gobuster dir -u http://target.com -w common.txt -b 404,403 -H "Authorization: Bearer <TOKEN>"

# Output to file
gobuster dir -u http://target.com -w common.txt -o gobuster_results.txt
```

### Subdomain Enumeration
```bash
gobuster dns -d target.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```

### Virtual Host Discovery
```bash
gobuster vhost -u http://target.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt --append-domain
```

| Flag | Description |
|------|-------------|
| `-u` | Target URL |
| `-w` | Wordlist path |
| `-x` | File extensions to append |
| `-t` | Number of threads (default: 10) |
| `-b` | Status codes to ignore (e.g., `404,403`) |
| `-H` | Custom header |
| `-o` | Output file |
| `-k` | Skip TLS certificate verification |
| `--timeout` | HTTP timeout (default: 10s) |

---

## ⚡ FFUF (Fuzz Faster U Fool)

FFUF is more powerful and flexible — it supports fuzzing any part of the request.

### Directory Fuzzing
```bash
# Basic directory scan (FUZZ keyword replaces the target position)
ffuf -u http://target.com/FUZZ -w /usr/share/seclists/Discovery/Web-Content/common.txt

# Filter out 404s and specific sizes
ffuf -u http://target.com/FUZZ -w common.txt -fc 404

# Scan with extensions
ffuf -u http://target.com/FUZZ -w common.txt -e .php,.html,.txt,.bak
```

### Subdomain Fuzzing
```bash
ffuf -u http://FUZZ.target.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -fc 404
```

### Parameter Fuzzing (GET)
```bash
# Find hidden GET parameters
ffuf -u "http://target.com/page?FUZZ=test" -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt -fc 404
```

### POST Body Fuzzing
```bash
# Fuzz a POST body field
ffuf -u http://target.com/login -X POST -d "username=FUZZ&password=test" -w /usr/share/seclists/Usernames/top-usernames-shortlist.txt
```

### Filter & Match Options
| Flag | Description |
|------|-------------|
| `-fc` | Filter by HTTP status code (e.g., `-fc 404,403`) |
| `-fs` | Filter by response size in bytes |
| `-fw` | Filter by word count in response |
| `-mc` | Match only these status codes (e.g., `-mc 200,301`) |
| `-ms` | Match by response size |

### Output & Performance
```bash
# Save results to JSON
ffuf -u http://target.com/FUZZ -w common.txt -o results.json -of json

# Control speed with rate limiting
ffuf -u http://target.com/FUZZ -w common.txt -rate 100   # Max 100 req/s

# Use a proxy (e.g., Burp Suite)
ffuf -u http://target.com/FUZZ -w common.txt -x http://127.0.0.1:8080
```

---

## 🔑 Finding Sensitive Files
Always include these in a wordlist or scan for them explicitly.
```bash
# Common sensitive paths to check manually
/.git/HEAD
/.env
/backup.zip
/phpinfo.php
/robots.txt
/sitemap.xml
/wp-config.php
/config.yml
/.DS_Store
```

---
*Return to [SimplySec Playbook](../README.md)*
