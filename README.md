# The Kubernetes Cluster Builder

`kubemake` is a tool based on Ansible to (un)build Kubernetes clusters.

## Supported Components

- Core
  - Kubernetes 1.21

- Container Runtime
  - Flannel
  
- Network Plugin
  - CRI-O 1.20

- Service Mesh
  - Istio 1.10
  
## Requirements

### Cluster Nodes

- Ubuntu 20.04 LTS
- SSH access enabled
- [K8s-specific requirements](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#before-you-begin)

### Control Node
- Ansible 2.10
- [Kubernetes collection](https://galaxy.ansible.com/community/kubernetes?extIdCarryOver=true&sc_cid=701f2000001OH6uAAG) for Ansible
