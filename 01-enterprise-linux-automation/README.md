# Enterprise Linux Automation with Ansible

Role-based automation for common Linux server configuration and operational health checks.

## What it does
- Installs baseline packages.
- Creates named administrator accounts.
- Ensures SSH service is enabled.
- Applies a small security baseline.
- Collects filesystem, memory and OS health information.
- Keeps OS patching in a separate, explicitly gated playbook.

## Requirements
- Ubuntu 22.04/24.04 or a supported RHEL-family Linux host.
- SSH access and a remote account with sudo privileges.
- Python 3 on managed hosts.
- Ansible installed by the GitHub Actions workflow.

## Repository layout
- `inventory/`: development and production host definitions.
- `group_vars/`: shared configuration.
- `roles/`: reusable configuration units.
- `playbooks/`: entry-point playbooks.
- `.github/workflows/`: validation and manual deployment.

## Configure GitHub
Create GitHub Actions secrets:
- `SSH_PRIVATE_KEY`: private key matching the public key installed on target hosts.
- `SSH_KNOWN_HOSTS`: verified host-key lines from a trusted source (do not disable host-key checking).
- `ANSIBLE_USER`: SSH account name.
- `PROD_APP01_HOST`: production host/IP used by `inventory/prod.yml`.

Create GitHub Environments named `dev` and `prod`. Add production approval protection in repository Settings > Environments. Replace the example IP in `inventory/dev.yml` with your lab host.

## Run
From Actions > Ansible Linux Automation > Run workflow:
1. Select `dev` or `prod`.
2. Select `configure` or `health-check`.
3. Run the workflow.

Local validation:
```bash
python -m pip install ansible ansible-lint
ansible-galaxy collection install -r requirements.yml
ansible-playbook -i inventory/dev.yml playbooks/site.yml --syntax-check
ansible-lint playbooks roles
```

Local deployment:
```bash
ansible-playbook -i inventory/dev.yml playbooks/site.yml --ask-become-pass
ansible-playbook -i inventory/dev.yml playbooks/health-check.yml
```

## Patching safety
Patching can restart services or change kernel packages. Review the playbook, approve a maintenance window, set `enable_os_patching: true`, and run `playbooks/patch.yml` deliberately. Do not enable unattended production patching through a push trigger.

## Notes
The sample inventories are examples. Use private addresses with a self-hosted runner for private infrastructure. Never commit private keys, passwords, tokens or vault passwords.
