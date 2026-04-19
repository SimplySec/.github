# 🕵️ Burp Suite Cheatsheet

A quick-reference guide for using Burp Suite Community/Pro in web application security testing.

> [!TIP]
> Always set up your browser to proxy through `127.0.0.1:8080` and import Burp's CA certificate to avoid TLS errors.

## ⚙️ Essential Setup
```
Proxy → Options → Proxy Listeners → 127.0.0.1:8080
Import CA: http://burpsuite/ → Download CA Certificate → Import to browser
```
| Shortcut | Action |
|----------|--------|
| `Ctrl+I` | Send to Intruder |
| `Ctrl+R` | Send to Repeater |
| `Ctrl+S` | Send to Scanner (Pro) |
| `Ctrl+Shift+F` | Forward all intercepted requests |

## 🔁 Repeater — Manual Testing
The core tool for manually manipulating individual requests.
1. Intercept a request in **Proxy**
2. `Ctrl+R` to send to **Repeater**
3. Modify headers, parameters, cookies, body
4. Click **Send** and analyze the response

**Key uses:**
* Test IDOR by changing ID parameters
* Replay authentication requests with modified tokens
* Test SQLi/XSS payloads manually

## 🎯 Intruder — Automated Fuzzing
Automate payload injection across a target parameter.

### Attack Types
| Mode | Use Case |
|------|----------|
| **Sniper** | One payload list, one position at a time (most common) |
| **Battering Ram** | Same payload in all positions simultaneously |
| **Pitchfork** | Multiple lists, paired by index (e.g., usernames + passwords) |
| **Cluster Bomb** | All combinations across multiple lists (brute-force) |

### Rate Limiting Bypass (Community)
Community edition throttles Intruder. Use **Turbo Intruder** extension for high-speed attacks.

## 🕷️ Target & Site Map
```
Target → Site map → Right-click scope → Add to scope
Target → Site map → Filter by scope
```
* **Engagement Tools (Pro):** Find comments, discover content, generate CSRF PoC.

## 🔬 Scanner (Pro Only)
```
Right-click any request → Scan
Dashboard → New Scan → Crawl and Audit
```
* Configure scan type: Crawl only, Audit only, or both
* Review **Issue Activity** for findings with severity/confidence ratings

## 📡 Decoder — Encode/Decode Payloads
Transform encoded data quickly.

| Function | Use Case |
|----------|----------|
| URL Decode | Decode `%3C%73%63%72%69%70%74%3E` → `<script>` |
| Base64 Decode | Inspect JWT payloads or cookie values |
| HTML Decode | Decode `&lt;script&gt;` |
| Hash | Generate MD5/SHA hashes for comparison |

## 🔄 Match & Replace — Passive Header Modification
Automatically modify requests/responses without intercepting each one.
```
Proxy → Options → Match and Replace → Add Rule
```
**Examples:**
* Remove `X-Frame-Options` header from responses
* Add `X-Forwarded-For: 127.0.0.1` to all requests
* Replace `User-Agent` with a custom string

## 💾 Useful Extensions (BApp Store)
| Extension | Purpose |
|-----------|---------|
| **Turbo Intruder** | High-speed fuzzing (bypasses Community rate limit) |
| **Logger++** | Log and search all traffic |
| **Authorize** | Test IDOR/BOLA by replaying requests as another user |
| **JWT Editor** | Manipulate and forge JWT tokens |
| **Param Miner** | Discover hidden parameters and headers |
| **Active Scan++** | Enhances the Pro scanner with additional checks |

## 🧪 Common Testing Workflows
### Test for IDOR
1. Log in as User A, capture a request for `/api/users/105/orders`
2. Send to **Repeater**, change `105` → `106`
3. If response returns User B's data → **IDOR confirmed**

### Test for SQLi
1. Capture a request with a parameter (e.g., `?id=1`)
2. Send to **Repeater**, try: `1'`, `1 OR 1=1--`, `1; DROP TABLE users--`
3. Look for database errors or behavioral changes

### Brute-force Login
1. Capture login POST request, send to **Intruder**
2. Mark password field as payload position (`§password§`)
3. Load a wordlist, run **Sniper** attack
4. Sort by response length or status code to find valid credentials

---
*Return to [SimplySec Playbook](../README.md)*
