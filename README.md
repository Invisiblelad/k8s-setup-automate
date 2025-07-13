
# 🚀 Kubernetes Cluster Setup with Ansible

This project automates the setup of a Kubernetes cluster using Ansible. It provisions one master node and one or more worker nodes. The setup includes Docker installation, system configuration, Kubernetes component installation, and joining workers to the cluster.


## Prerequisites

### 1️⃣ Generate and Use SSH Key

Create a key if you haven't already:

```bash
ssh-keygen -t rsa -f ~/.ssh/vm -C "your_email@example.com"
```

Add the **public key** (`~/.ssh/vm.pub`) to each remote VM under:

```bash
~/.ssh/authorized_keys
```

### 2️⃣ Inventory File (edit `inventory/hosts.ini`)

```ini
[master]
master ansible_host=<IP Address> ansible_user=<user> node_type=master ansible_ssh_private_key_file=<Path to ssh file>

[worker]
worker ansible_host=<IP Address> ansible_user=<user> node_type=worker ansible_ssh_private_key_file=<Path to ssh file>
```
---

##  What Each Role Does

### `common/` Role

- Installs Docker
- Loads `br_netfilter` module and applies sysctl settings
- Installs `kubelet`, `kubeadm`, `kubectl`
- Optionally resets Kubernetes if already initialized

### `k8s-master/` Role

- Initializes control plane (`kubeadm init`)
- Sets up `kubectl` for root
- Extracts `kubeadm join` token and CA cert hash
- Shares these values for workers to use

### `k8s-worker/` Role

- Installs Docker and Kubernetes packages
- Retrieves and uses `kubeadm join` values from the master
- Joins the cluster

---

##  Run the Playbook

1. Ensure you can SSH into your nodes:

```bash
ansible all -i inventory/hosts.ini -m ping
```

2. Execute the cluster setup:

```bash
ansible-playbook -i inventory/hosts.ini playbook.yml
```

---

## Validation

After the playbook completes:

1. SSH into your master node:

```bash
ssh -i ~/.ssh/vm ubuntu@<MASTER_PUBLIC_IP>
```

2. Check if all nodes joined the cluster:

```bash
kubectl get nodes
```

You should see both the master and worker nodes in a `Ready` state.

---

## Troubleshooting

| Problem | Solution |
|--------|----------|
| `Permission denied (publickey)` | Ensure the SSH private key path is correct and the public key is added to `authorized_keys` |
| `br_netfilter` not found | Ensure the module is loaded using `modprobe br_netfilter` and sysctl config is applied |
| `api_server_endpoint is undefined` | Ensure `master-components.yaml` sets it via `set_fact` and uses `add_host` |
| Worker fails to join | Check token and cert hash are registered properly and shared using `hostvars` |

---

## Resetting the Cluster

You can manually reset your Kubernetes setup with:

```bash
ansible -i inventory/hosts.ini all -a "kubeadm reset -f" --become
```

---

## Maintainer

Sharath Veerapaneni  
GitHub: [Invisiblelad](https://github.com/Invisiblelad)

---
Happy Kubernetes-ing! 🎉
