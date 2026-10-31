# Azure VM Provisioning and Configuration with Ansible

Creates a small Ubuntu 22.04 VM and its supporting Azure networking inside a pre-created dedicated resource group, then connects over SSH and applies a Linux baseline. This is a lab/portfolio project; review region, VM size, permissions, CIDR and cost before running it.

## What it creates

- Virtual network and subnet
- Network Security Group with SSH allowed only from the current GitHub Actions runner egress IP (`/32`)
- Static Standard public IP and network interface
- Ubuntu 22.04 Gen2 VM with a Standard SSD managed OS disk
- Baseline packages and enabled SSH/chrony services

The workflow opens only TCP/22, and only from the GitHub runner's detected public egress IP as a `/32`. Do not change this to `0.0.0.0/0`.

## Prerequisites

1. An Azure subscription with quota for the selected VM size and region.
2. A dedicated resource group created once in Azure, for example `rg-ansible-vm-lab`. Do not use a resource group containing unrelated resources.
3. A Microsoft Entra application/service principal (or user-assigned managed identity) configured with a GitHub Actions federated credential.
4. Assign the identity the `Contributor` role scoped only to that dedicated resource group for this lab. This is broad within that resource group; use a narrower custom role for production.
5. An SSH key pair. Put the public key in the GitHub variable and the matching private key in the GitHub secret.
6. The GitHub-hosted runner must be able to reach the VM over SSH. The workflow discovers the runner's public egress IP at runtime and adds only that `/32` to the NSG. For production, prefer a self-hosted runner with fixed egress or private networking/Run Command rather than relying on public SSH.

## GitHub Environment configuration

Create the GitHub Environment `azure-dev` under Settings → Environments. Create the dedicated Azure resource group first in the Azure Portal and assign the federated identity `Contributor` on that resource group only.

### Environment secrets

- `AZURE_CLIENT_ID`: Microsoft Entra application or managed identity client ID.
- `AZURE_TENANT_ID`: Microsoft Entra tenant/directory ID.
- `AZURE_SUBSCRIPTION_ID`: Azure subscription ID.
- `AZURE_VM_SSH_PRIVATE_KEY`: private SSH key corresponding to the public key below.

### Environment variables

- `AZURE_SSH_PUBLIC_KEY`: matching SSH public key, one line beginning with `ssh-ed25519` or `ssh-rsa`.
- `AZURE_LOCATION`: e.g. `centralindia` (optional; default in group_vars is `centralindia`).
- `AZURE_RESOURCE_GROUP`: e.g. `rg-ansible-vm-lab` (optional).
- `AZURE_VM_NAME`: e.g. `ansible-ubuntu-vm` (optional).
- `AZURE_VM_SIZE`: e.g. `Standard_B2s` (optional; verify regional availability/quota).
- `AZURE_ADMIN_USERNAME`: e.g. `ansibleadmin` (optional).

The workflow uses `azure/login@v2` with GitHub OIDC. No Azure client secret is stored in GitHub. Configure a federated credential in Microsoft Entra to trust your exact repository and the `azure-dev` environment subject. Grant only the Azure permissions needed by this project.

## Configure Microsoft Entra federation

1. Register an application (or use a user-assigned managed identity) in Microsoft Entra ID and create its service principal if using an app registration.
2. Add a federated credential with issuer `https://token.actions.githubusercontent.com`, audience `api://AzureADTokenExchange`, and subject `repo:<OWNER>/<REPO>:environment:azure-dev`.
3. Assign `Contributor` to the identity on the dedicated resource group only for this lab. For production, use a narrower custom role scoped to that resource group.
4. Copy client ID, tenant ID and subscription ID to the `azure-dev` environment secrets.

## Run

1. Ensure the dedicated resource group exists and the GitHub OIDC identity has Contributor access to it.
2. Open Actions → `Provision and Configure Azure VM` → Run workflow.
3. Select `provision`. The workflow creates resources and configures the VM.
4. Review the output and Ansible recap. Check the VM in Azure Portal.

## Destroy and cost control

The workflow has a separate `destroy` operation. It requires the exact confirmation text `DELETE_AZURE_VM_RESOURCES` and removes the VM plus the network resources created by this project. It leaves the resource group itself in place. Use a dedicated lab resource group only. VM compute, managed disks and public IP resources can incur charges; run cleanup when finished and verify the resources are gone in Azure Portal.

## Local execution

Install Ansible, the `azure.azcollection` collection and its Python requirements. Authenticate with `az login`, then run:

```bash
ansible-galaxy collection install -r requirements.yml
python -m pip install -r ~/.ansible/collections/ansible_collections/azure/azcollection/requirements.txt
ansible-playbook playbooks/provision-vm.yml
```

Then connect and configure the VM using the IP from Azure CLI and your SSH key:

```bash
VM_IP=$(az vm show -d -g rg-ansible-vm-lab -n ansible-ubuntu-vm --query publicIps -o tsv)
ansible-playbook -i "${VM_IP}," -u ansibleadmin --private-key ~/.ssh/id_ed25519 playbooks/configure-vm.yml
```

## Important limitations

- The project provisions one Ubuntu 22.04 VM inside a pre-created resource group; it is not a reusable multi-VM module yet.
- The SSH host key is collected on first connection by the workflow. For production, verify the fingerprint through a trusted channel instead of trusting an unverified first scan.
- `Standard_B2s` and `centralindia` are examples. Availability, quota and price vary by subscription and region.
- The deployment workflow dynamically limits SSH ingress to the current GitHub-hosted runner egress IP for that run. For production, prefer private networking or a self-hosted runner with fixed egress and stronger host-key verification.
- This project does not configure a domain name, TLS, backup, monitoring agent, or production-grade private networking.
