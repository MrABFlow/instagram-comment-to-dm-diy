# Security notes

This repository contains a workflow template and placeholder values only. It is not connected to an Instagram account and contains no Meta App Secret or access token.

## Never publish credentials

Keep all of these out of commits, screenshots, issues, and README edits:

- Instagram User access tokens.
- Meta App Secret.
- Webhook verification token value.
- Passwords, recovery codes, or private guide URLs.

Use your host’s protected environment for META_VERIFY_TOKEN and META_APP_SECRET, and n8n Credentials for the Instagram bearer token. If you accidentally commit a secret, revoke or rotate it in Meta immediately; deleting the commit or editing the file does not remove it from Git history or forks.

## Environment access warning

The workflow uses process.env in Code nodes and Node’s built-in crypto module to validate Meta’s X-Hub-Signature-256. Self-hosted n8n may require NODE_FUNCTION_ALLOW_BUILTIN=crypto. In n8n 2.x, Code-node environment access is blocked by default; this workflow requires N8N_BLOCK_ENV_ACCESS_IN_NODE=false if used unchanged.

That switch exposes environment variables to all Code nodes and expressions on the instance. Do not enable it on a shared or multi-user instance or where untrusted users can edit or run workflows. Use a dedicated, locked-down, single-user n8n host or move signature verification into a separate trusted gateway. If Task Runners are enabled, follow n8n’s runner-specific configuration instructions too.

## Hardening checklist

- Serve the n8n webhook and editor only over HTTPS; keep the editor behind strong authentication and MFA.
- Restrict who can edit workflows, credentials, environment variables, or access logs.
- Use Meta’s official OAuth/token flow. Never ask a user for their Instagram password.
- Grant only the current permissions required; check Meta’s docs before launch.
- Keep the workflow inactive until webhook verification and a controlled test pass.
- Review proxy logs, n8n execution settings, backups, and data retention. Infrastructure may retain request data.
- Avoid repeated tests against other people’s accounts.

## Reporting a problem

Do not post tokens, signatures, personal request bodies, or exploitable details in a public issue. Revoke exposed secrets first. Use GitHub’s private vulnerability reporting if enabled; otherwise contact the repository owner privately through GitHub.
