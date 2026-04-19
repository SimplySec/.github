# Security Policy

## Supported Repositories

This security policy applies to all repositories under the **SimplySec** organization.

---

## ⚠️ Scope of This Policy

SimplySec publishes **educational** security tools and documentation. Many of the tools in this organization are intentionally offensive in nature (stress testers, enumeration scripts, etc.) and are designed for **authorized use in lab environments only**.

This policy covers:
- Security vulnerabilities in our **tool code** (e.g., `ddos-attack.py`, `web-stress.py`)
- Inaccuracies in documentation that could lead users to take **unsafe or illegal actions**
- Dependency vulnerabilities in any released packages

---

## 🔍 Reporting a Vulnerability

**Please do not file a public GitHub Issue for security vulnerabilities.**

To report a security issue privately:

1. **Email:** Send a report to the maintainer via the contact listed on [@SimplyLouie's GitHub profile](https://github.com/SimplyLouie)
2. **GitHub Private Vulnerability Reporting:** Use the **"Report a vulnerability"** button on the affected repository's **Security** tab (Settings → Security → Private vulnerability reporting)

### What to Include in Your Report
- Repository and file affected
- Description of the vulnerability
- Steps to reproduce
- Potential impact
- Suggested fix (optional but appreciated)

---

## ⏱️ Response Timeline

| Stage | Timeline |
|-------|----------|
| Initial acknowledgement | Within **72 hours** |
| Triage and assessment | Within **7 days** |
| Fix or mitigation | Within **30 days** (critical issues prioritized) |
| Public disclosure | After fix is released or 90 days, whichever comes first |

---

## ✅ What Qualifies as a Security Issue

| Qualifies | Does Not Qualify |
|-----------|-----------------|
| RCE or code injection in a SimplySec tool | Expected offensive behavior of our tools (they are stress testers by design) |
| Credential exposure in source code or CI/CD | Findings from using our tools against unauthorized targets |
| Dependency with a known CVE | General feature requests |
| Documentation errors that encourage illegal use | Policy disagreements |

---

## 🌐 Responsible Disclosure

SimplySec believes in responsible disclosure. We will:
- Acknowledge your report promptly
- Work with you to understand and validate the issue
- Credit you in the fix commit or release notes (unless you prefer anonymity)
- Not take legal action against good-faith security researchers

We ask that you:
- Give us reasonable time to fix the issue before public disclosure
- Do not exploit the vulnerability beyond what is needed to demonstrate it
- Do not access or modify data that does not belong to you

---

*SimplySec — Demystify. Defend. Deploy.*
