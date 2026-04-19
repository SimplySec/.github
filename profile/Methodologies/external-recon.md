# 🛡️ External Recon & Surface Mapping

A methodology for discovering and mapping an organization's publicly exposed assets.

> [!IMPORTANT]
> Reconnaissance is 90% of a successful assessment. Knowing the attack surface allows you to find the "forgotten" server that hasn't been patched in years.

## 1. Passive Discovery
Find assets without ever touching the target's infrastructure.
*   **WHOIS/ASN:** Identify the IP ranges owned by the target.
*   **Search Engine Dorking:** Use Google/Bing to find indexed subdomains, documents, and login pages (e.g., `site:target.com -www`).
*   **Certificate Transparency:** Use `crt.sh` or `censys.io` to see every SSL certificate issued for the domain. This often reveals dev/test subdomains.

## 2. Active Subdomain Discovery
Brute-force and query DNS for subdomains.
*   **DNS Enumeration:** Use `subfinder` or `amass` to aggregate results from dozens of sources.
*   **Brute-Force:** Use a wordlist (e.g., `secLists`) to find subdomains that aren't indexed (e.g., `vpn`, `internal`, `jenkins`).

## 3. Service Mapping & Port Scanning
Identify what is actually running on the discovered IPs.
*   **Massive Scanning:** Use `nmap` or `naabu` to find open ports.
*   **Service Identification:** Run `-sV` in `nmap` to identify the software and version (e.g., Apache 2.4.41).
*   **Screenshotting:** Use `httpx` or `gowitness` to take screenshots of every web service found. This allows you to visually identify interesting targets quickly.

## 4. Content & JS Analysis
Dig deeper into found web applications.
*   **JavaScript Analysis:** Use `subjs` or `linkfinder` to extract endpoints and API keys hidden in `.js` files.
*   **Directory Fuzzing:** Search for common hidden files like `.env`, `.git/`, `README.md`, or `phpinfo()`.

## 5. Cloud Asset Discovery
*   **S3 Buckets:** Search for open storage buckets (e.g., `target-data`, `site-backups`).
*   **Azure/GCP Assets:** Identify public storage blobs or functions.

---
*Return to [SimplySec Playbook](../README.md)*
