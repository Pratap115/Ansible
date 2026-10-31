# Ansible Automation Portfolio

A single GitHub repository containing six practical Ansible automation projects, each with an independent GitHub Actions workflow.

## Repository structure

```text
Ansible/
├── .github/workflows/
│   ├── linux-automation.yml
│   ├── monitoring-deployment.yml
│   ├── nginx-deployment.yml
│   ├── kubernetes-preparation.yml
│   ├── aws-ec2-automation.yml
│   └── azure-vm-automation.yml
├── 01-enterprise-linux-automation/
├── 02-prometheus-grafana-deployment/
├── 03-nginx-webserver-automation/
├── 04-kubernetes-node-automation/
├── 05-aws-ec2-automation/
├── 06-azure-vm-provisioning-automation/
└── README.md
```

## Projects

| Folder | Purpose | Workflow |
|---|---|---|
| `01-enterprise-linux-automation` | Linux baseline, users, security, health checks and gated patching | Ansible Linux Automation |
| `02-prometheus-grafana-deployment` | Prometheus, Grafana and Node Exporter using Docker Compose | Deploy Monitoring Stack |
| `03-nginx-webserver-automation` | Nginx web server, landing page and health endpoint | Deploy Nginx |
| `04-kubernetes-node-automation` | Ubuntu kernel, networking and containerd prerequisites | Prepare Kubernetes Nodes |
| `05-aws-ec2-automation` | AWS EC2 dynamic inventory and configuration of tagged instances | Configure AWS EC2 with Ansible |
| `06-azure-vm-provisioning-automation` | Provision Azure networking and an Ubuntu VM, then configure it over SSH | Provision and Configure Azure VM |

## How workflows work

All six workflows are stored under the repository-level `.github/workflows/` directory because that is where GitHub Actions discovers workflows. Each workflow sets a project-specific `working-directory`, so playbook and inventory paths remain local to that project.

Most workflows validate on relevant pushes and deploy only when manually started from the Actions tab. The AWS and Kubernetes workflows are manual-only. The Azure workflow validates on pushes and provisions or destroys infrastructure only when manually started.

## Setup

1. Create one GitHub repository named `Ansible`.
2. Upload the contents of this folder to the repository root. Ensure the hidden `.github` folder is included.
3. Update example host addresses and usernames in each project's `inventory/` files.
4. Configure the GitHub Actions secrets and variables listed in each project's README.
5. Create the required GitHub Environments (`dev`, `prod`, `aws-dev`, `azure-dev`) where applicable.
6. For Azure, configure Microsoft Entra GitHub OIDC federation and the `azure-dev` secrets/variables described in `06-azure-vm-provisioning-automation/README.md`.
7. Start with disposable lab infrastructure and run one workflow at a time.

## Security and deployment notes

- Do not commit private keys, passwords, tokens, certificates or real secrets.
- Keep SSH host-key checking enabled and provide verified known-host entries.
- Use a self-hosted runner or private connectivity for hosts that are not publicly reachable. Do not expose production SSH to the internet just to use a GitHub-hosted runner.
- Configure production Environment approvals and restrict deployment branches.
- The Kubernetes project prepares a host; it does not install or initialize a complete Kubernetes cluster.
- The AWS project discovers and configures existing tagged EC2 instances; it does not create or terminate EC2 resources.
- The Azure project creates network resources and one Ubuntu VM inside a pre-created dedicated resource group. Its cleanup operation removes the VM and project network resources but leaves the resource group in place.
- Review each README for prerequisites, secrets, network ports and verification steps.

## Suggested portfolio evidence

Add sanitized screenshots of successful Actions runs, Ansible play recaps, Nginx health checks, Prometheus targets, Grafana dashboards, Kubernetes prerequisite checks and AWS dynamic inventory and Azure VM provisioning/configuration. Remove public IPs, account IDs, internal hostnames and secrets before publishing.
