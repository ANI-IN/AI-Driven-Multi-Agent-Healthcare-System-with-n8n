# Security Policy

## Supported Versions

| Version | Supported |
| ------- | --------- |
| Latest  | Yes       |

## Reporting a Vulnerability

If you discover a security vulnerability in this project, please report it responsibly.

**Do not open a public GitHub issue for security vulnerabilities.**

Instead, please use one of the following methods:

1. **GitHub Private Security Advisory**: Navigate to the
   [Security tab](https://github.com/ANI-IN/AI-Driven-Multi-Agent-Healthcare-System-with-n8n/security/advisories)
   of this repository and click "Report a vulnerability."
2. **Email**: Contact the maintainer directly at the email listed on the
   [GitHub profile](https://github.com/ANI-IN).

Please include:

- A description of the vulnerability
- Steps to reproduce the issue
- The potential impact
- Any suggested fixes (if you have them)

You can expect an initial response within 72 hours. We will work with you to
understand the issue and coordinate a fix before any public disclosure.

## Known Risk Areas

This project handles healthcare-related data and integrates with external
services. The following areas are particularly security-sensitive:

1. **Chat Trigger Authentication**: The Coordinator Agent's chat trigger is
   configured as public. In a production deployment, enable authentication
   (Header Auth or Basic Auth) on the webhook to prevent unauthorized access.

2. **Credential Management**: The workflow JSON files reference credential IDs
   for OpenAI, Microsoft Graph, Outlook, and Airtable. Never commit actual
   credential values. Use n8n's built-in credential store and environment
   variables for all secrets.

3. **Email Sending**: The Diagnostics and Doctor Booking agents send emails
   via Microsoft Outlook. The recipient address comes from user input without
   server-side validation. In a production setting, add input validation and
   rate limiting to prevent abuse.

4. **External Data Access**: The workflows read data from SharePoint and
   Airtable. Ensure that the OAuth2 tokens and API keys used have the minimum
   required permissions (principle of least privilege).

## Best Practices for Deployment

- Run n8n behind a reverse proxy with TLS termination.
- Use environment variables for all secrets; never hard-code them.
- Enable n8n's built-in authentication for the web UI.
- Restrict network access to the n8n instance (firewall rules or VPN).
- Regularly rotate API keys and OAuth2 tokens.
- Review n8n execution logs for unexpected activity.
- Keep n8n and all community nodes updated to the latest stable versions.
