# Contributing to SimplySec

Thank you for your interest in contributing! SimplySec is a living security lab built on the philosophy that **complexity is the enemy of security**. We welcome contributions that keep things simple, practical, and educational.

---

## 📋 What We Accept

| Type | Examples |
|------|---------|
| **Cheatsheets** | Tool references, command quick-starts |
| **Methodologies** | Structured assessment guides |
| **Tools / Scripts** | Security utilities, automation scripts |
| **Bug Fixes** | Corrections to existing content or code |
| **Improvements** | Expanding incomplete sections, adding examples |

---

## 🚫 What We Do Not Accept

- Tools or techniques designed to facilitate **unauthorized access** to systems
- Offensive exploits without defensive context or hardening guidance
- Content that encourages illegal activity
- Unrelated marketing or promotional material

---

## 🔁 Contribution Workflow

### 1. Fork & Branch
```bash
# Fork the repo on GitHub, then clone your fork
git clone https://github.com/<your-username>/<repo-name>
cd <repo-name>

# Create a feature branch
git checkout -b feat/add-hashcat-cheatsheet
```

### 2. Make Your Changes
Follow the style guide below before committing.

### 3. Commit with a Clear Message
We use a simple format:
```
type: short description

# Types:
feat     – New cheatsheet, methodology, or tool
fix      – Bug fix or correction
docs     – Documentation update
refactor – Code cleanup with no functional change
```

**Examples:**
```
feat: add hashcat password cracking cheatsheet
fix: correct sqlmap --level flag range in cheatsheet
docs: expand linux-privesc cron job section
```

### 4. Open a Pull Request
- Target the `main` branch
- Fill out the PR description using the template provided
- Link any related issues

---

## ✍️ Style Guide

### Markdown Documents (Cheatsheets & Methodologies)
- Use a single `# H1` title at the top
- Use emoji section headers (e.g., `## 🔍 Enumeration`) for consistency with existing files
- Include a `> [!WARNING]` or `> [!TIP]` callout where ethical use or context is important
- Add a backlink at the bottom: `*Return to [SimplySec Playbook](../README.md)*`
- Keep tables clean and aligned
- Always favor clarity over completeness — this is a cheatsheet, not a book

### Python Tools
- Use `argparse` for all CLI arguments (no positional-only tools)
- Include the SimplySec header docstring at the top of every script
- Use ANSI colors from the shared `Colors` class for consistency
- Generate a **hardening report** at the end of every test run
- Never hardcode targets, IPs, or credentials
- Test on Python 3.8+

---

## 🔒 Ethical Use Reminder

Every document and tool in SimplySec must include an ethical use disclaimer. If your contribution involves offensive techniques, pair it with defensive recommendations or a hardening section. This is non-negotiable.

---

## ❓ Questions

Open an issue with the `question` label or reach out to [@SimplyLouie](https://github.com/SimplyLouie) directly.

---

*SimplySec — Demystify. Defend. Deploy.*
