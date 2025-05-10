# Kubernetes Cluster Deployment with Ansible and kubeadm

This repository contains Ansible playbooks to automate the deployment of a Kubernetes cluster using kubeadm.

## Prerequisites

- Ubuntu 22.04 on all nodes
- SSH access to all nodes
- Ansible 2.15.13 installed on the control node
- Minimum requirements for each node:
  - 2 CPU cores
  - 2 GB RAM
  - 20 GB disk space

### Installing Ansible 2.15.13

To install Ansible 2.15.13 on the control node, run the following commands:

```bash
# Add the Ansible PPA repository
sudo apt update
sudo apt install -y software-properties-common
sudo add-apt-repository --yes --update ppa:ansible/ansible

# Install Ansible
sudo apt install -y ansible

# Verify the installation
ansible --version
```

This will install Ansible via apt. Make sure the installed version is 2.15.13 or compatible.

## Cluster Architecture

- Master Node: Runs the Kubernetes control plane components
- Worker Nodes: Run the application workloads

## Configuration

### Inventory

Edit the `inventory/hosts.yaml` file to specify your master and worker nodes:

```yaml
---
# Kubernetes Cluster Inventory

kube_masters:
  hosts:
    kangdeploy-master1:
      ansible_host: 10.184.0.2
      ansible_user: ubuntu

kube_workers:
  hosts:
    kangdeploy-worker1:
      ansible_host: 10.184.0.5
      ansible_user: ubuntu
    kangdeploy-worker2:
      ansible_host: 10.184.0.6
      ansible_user: ubuntu

kubernetes:
  children:
    kube_masters:
    kube_workers:
```

### Variables

Edit the `group_vars/all.yml` file to customize your Kubernetes deployment:

- Kubernetes version
- Container runtime settings
- Network configuration
- CNI plugin selection

## Deployment

### 1. Verify SSH connectivity to all nodes

```bash
ansible all -m ping
```

Note: Since we've set the default inventory in `ansible.cfg`, you don't need to specify the inventory file with `-i`.

When connecting to hosts for the first time, you'll be prompted to verify the SSH host keys. This repository includes an `ansible.cfg` file that disables host key checking by default, so you won't be prompted to verify host keys. If you still see prompts, you can handle them in several ways:

1. Manually accept the host keys by answering "yes" to the prompts
2. Disable host key checking temporarily:
   ```bash
   ANSIBLE_HOST_KEY_CHECKING=False ansible -i inventory/hosts.yaml all -m ping
   ```
3. Ensure you're running the command from the repository root directory where the `ansible.cfg` file is located

### 2. Deploy the Kubernetes cluster

```bash
ansible-playbook site.yml
```

Note: Since we've set the default inventory in `ansible.cfg`, you don't need to specify the inventory file with `-i`.

### 3. Verify the deployment

After the playbook completes, you can verify the cluster by running:

```bash
# On any node in the cluster (master or worker)
kubectl get nodes
kubectl cluster-info
```

The kubeconfig file is automatically distributed to all nodes, allowing you to use kubectl commands from any node in the cluster.

### 4. Deploy a test application

To verify that the cluster is working correctly, deploy the Nginx test application:

```bash
kubectl apply -f verify/deploy-nginx.yaml
```

Check that the pods are running:

```bash
kubectl get pods
kubectl get services
```

## Roles

- **common**: Prepares all nodes with the necessary prerequisites
- **kubernetes-master**: Initializes the Kubernetes control plane and distributes kubeconfig
- **kubernetes-worker**: Joins worker nodes to the cluster and sets up kubeconfig for kubectl access

## Troubleshooting

If you encounter issues during deployment:

1. Check the Ansible logs for errors
2. Verify that all prerequisites are met
3. Check the Kubernetes logs on the affected nodes:
   ```bash
   journalctl -xeu kubelet
   ```

### Repository Information

This playbook uses the official Kubernetes repository for version 1.30 from `https://pkgs.k8s.io/core:/stable:/v1.30/deb/`. This is the recommended repository for installing Kubernetes 1.30.x on Ubuntu systems.

## License

MIT
