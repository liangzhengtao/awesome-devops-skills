# Security Policy

## Supported Versions

| Version | Supported          |
| ------- | ------------------ |
| latest  | :white_check_mark: |

## Reporting a Vulnerability

If you discover a security vulnerability in this project, please report it responsibly.

### How to Report

1. **Do NOT** open a public GitHub issue
2. Email: security@example.com (or use GitHub's private vulnerability reporting)
3. Include:
   - Description of the vulnerability
   - Steps to reproduce
   - Potential impact
   - Suggested fix (if any)

### Response Timeline

- **Acknowledgment**: Within 48 hours
- **Initial assessment**: Within 5 business days
- **Fix or mitigation**: Within 30 days for critical issues

### Scope

This project contains documentation and code templates. Security concerns may include:

- Hardcoded credentials or secrets in example code
- Insecure configurations in templates
- Vulnerable dependency versions referenced in examples

### What to Expect

- We will acknowledge your report promptly
- We will work with you to understand the issue
- We will credit you in the fix (unless you prefer anonymity)
- We will not take legal action against researchers who report responsibly

## Security Proven Patterns for Users

When using the skills in this repository:

- Never commit real credentials or API keys
- Use secrets management (Vault, AWS Secrets Manager, etc.)
- Review all code templates before deploying to production
- Keep dependencies updated
- Follow the principle of least privilege
