# Prometheus + Grafana deployment with Ansible

Deploys a small Docker Compose monitoring stack: Prometheus, Grafana and Node Exporter.

## Target and requirements
- Ubuntu 22.04/24.04 or RHEL-family Linux VM.
- SSH account with sudo.
- Docker packages available from the host OS repositories.
- Open inbound ports only from trusted admin networks: TCP 3000 (Grafana), 9090 (Prometheus), 9100 (Node Exporter). Prefer firewall allowlists or a private VPN; do not expose these dashboards publicly without TLS/authentication and network controls.

## GitHub secrets
- `SSH_PRIVATE_KEY`: deployment private key.
- `SSH_KNOWN_HOSTS`: pre-verified SSH host key entry.
- `GRAFANA_ADMIN_PASSWORD`: strong password of at least 12 characters.

Update `inventory/dev.yml` with the VM address and SSH user. Add the `dev` GitHub Environment and configure its secrets.

## Deploy
Actions > Deploy Monitoring Stack > Run workflow. The workflow validates first and then deploys on manual dispatch.

## Verify
- Prometheus: `http://HOST:9090/targets`
- Grafana: `http://HOST:3000`
- Node Exporter metrics: `http://HOST:9100/metrics`

Grafana's first login uses username `admin` and the password stored in GitHub Secrets. For production, pin image digests, configure TLS/reverse proxy, backups, retention, firewall rules and secret rotation. The sample uses version tags for reproducible demos, not a complete production security posture.
