# Kubernetes Linux Node Preparation

Prepares Ubuntu hosts for a subsequent kubeadm-based Kubernetes installation. It configures kernel modules, networking sysctls and containerd with systemd cgroups.

## Scope
This project prepares the operating system and container runtime only. It does **not** install kubeadm/kubelet/kubectl, initialize a control plane, configure a CNI, or join nodes to a cluster. Those steps depend on the Kubernetes version and cluster design and should be a separate controlled workflow.

## Requirements
- Ubuntu 22.04 or newer (the playbook asserts Ubuntu).
- SSH access and sudo privileges.
- A disposable VM or lab node is recommended.
- Reboot may be required in some environments after kernel-level changes.

## GitHub setup
Update `inventory/dev.yml`. Create GitHub Environment `dev` with secrets `SSH_PRIVATE_KEY` and `SSH_KNOWN_HOSTS`.

## Deploy
Actions > Prepare Kubernetes Nodes > Run workflow.

## Verify on target
```bash
lsmod | grep -E 'overlay|br_netfilter'
sysctl net.ipv4.ip_forward
grep SystemdCgroup /etc/containerd/config.toml
systemctl status containerd
```

Do not run this against a production cluster node without a maintenance plan and compatibility review.
