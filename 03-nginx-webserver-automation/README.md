# Nginx Web Server Automation

Ansible role that installs Nginx, deploys a simple landing page, configures a health endpoint and validates configuration before reload.

## Requirements
- Ubuntu 22.04/24.04 or RHEL-family VM.
- SSH + sudo access.
- TCP/80 reachable from your test client.

## Configure
Update `inventory/dev.yml` with the target address and SSH user. Add GitHub Environment `dev` with:
- `SSH_PRIVATE_KEY`
- `SSH_KNOWN_HOSTS`

The host key must be collected and verified independently, then stored as a known_hosts line. Do not use `StrictHostKeyChecking=no`.

## Deploy
Actions > Deploy Nginx > Run workflow.

## Verify
Open `http://HOST/` and `http://HOST/health`, or run:
```bash
curl -i http://HOST/health
sudo nginx -t
sudo systemctl status nginx
```

## TLS extension
The example intentionally serves HTTP only. For real TLS, add a certificate lifecycle (for example, ACME/Certbot), securely provision certificates, redirect HTTP to HTTPS and restrict access to private keys. Do not commit certificate private keys to Git.
