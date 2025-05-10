# Kubernetes Cluster Provisioning with Ansible (kubeadm)

## 🖥️ Prerequisites
- OS: Ubuntu 22.04 LTS (master & workers)
- Ansible >= 2.13
- SSH access to all nodes with passwordless sudo
- Python 3 and pip installed
- Virtualization: Tested on VM Instance GCP

## 🔧 How to Use

1. Edit `inventory/hosts.ini` with IPs of your nodes.
2. Edit `group_vars/all.yml` with the desired kubeadm token, pod CIDR, etc.
3. Run the playbook:

```bash
ansible-playbook -i inventory/hosts.ini site.yml
