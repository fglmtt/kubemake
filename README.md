# Kubernetes-based Cluster Maker :hammer:

`kubemake` is a tool based on Ansible to (un)make Kubernetes-based clusters.

## Supported Components

`kubemake` currently supports the components listed below. 

### Operating Systems

| Component | Version(s) |
| --- | --- |
| Ubuntu | 20.04 |

### Kubernetes Distributions

| Component | Version(s) |
| --- | --- |
| Kubernetes | 1.24.1 |

### Container Runtimes

| Component | Version(s) |
| --- | --- |
| CRI-O | 1.24.1 |
  
### Network Plugins

| Component | Version(s) |
| --- | --- |
| Flannel | 0.19.0 |

### Service Meshes

| Component | Version(s) |
| --- | --- |
| Istio | 1.14.1 |

### Telemetry Addons

| Component | Version(s) |
| --- | --- |
| Grafana | 8.3.1 |
| Prometheus | 2.31.1 |
| Jaeger | 1.29 |
| Kiali | 1.50 |

### Chaos Engineering Tools

| Component | Version(s) |
| --- | --- |
| Chaos Mesh | 2.2.2 |
  
**Note 1**: the related versions are the ones that has been tested. Other versions might work as well.
  
## Requirements

`kubemake` does **not** provision virtual machines. This means you need your cluster nodes up and running beforehand. See [here](#operating-systems) for the operating systems currently supported. The control node, i.e., the node where `kubemake` runs, needs:

- Ansible (>= 2.13.1)
- [Kubernetes Collection for Ansible](https://galaxy.ansible.com/kubernetes/core?extIdCarryOver=true&sc_cid=701f2000001OH6uAAG)

**Note 2**: `kubemake`does **not** check if your cluster nodes meet hardware requirements. Please check them out based on the [components](#supported-components) you need.

**Note 3**: Ansible manages target nodes over SSH. This means the control node needs SSH access to the cluster nodes.

## Quick Start

### Inventory

The first thing you need is to build your [inventory](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html). An example of inventory is the following:

```
[masters]
master ansible_user=ubuntu ansible_host=192.168.1.1 kubelet_node_ip=192.168.1.1 apiserver_advertise_address=192.168.1.1

[workers]
worker-1 ansible_user=ubuntu ansible_host=192.168.1.2 kubelet_node_ip=192.168.1.2
worker-2 ansible_user=ubuntu ansible_host=192.168.1.3 kubelet_node_ip=192.168.1.3
worker-3 ansible_user=ubuntu ansible_host=192.168.1.4 kubelet_node_ip=192.168.1.4

[all:vars]
ansible_password=your_sudo_password
crio_version="1.24"
crio_os=xUbuntu_20.04
kubernetes_version="1.24.1-00"

[masters:vars]
pod_network_cidr=10.244.0.0/16
flannel_iface_regex=[eth1|eth0]
istio_version="1.14.1"
istio_profile=demo
istio_ingress_domain=yourdomain.edu
chaos_mesh_version="v2.2.2"
```

**Note 4**: for the sake of brevity, this inventory also contains variables. Please look [here](https://docs.ansible.com/ansible/2.8/user_guide/playbooks_best_practices.html) for Ansible best practices.

**Note 5**: `kubemake` does **not** currently support high-availability clusters, i.e., clusters with multiple masters.

### (Un)Make

To make a Kubernetes-based cluster provided with all the (supported components)[#supported-components], run:

```
anisble-playbook site.yml -i hosts
```

To unmake, run:

```
anisble-playbook rollback-site.yml -i hosts
```

If you are only interested in a subset of those components, you can use [Ansible tags](https://docs.ansible.com/ansible/latest/user_guide/playbooks_tags.html) to limit what `kubemake` does. Specifically

- `setup`: installs packages Kubernetes needs to run
- `init`: initializes the Kubernetes master
- `join`: makes worker nodes joining the master
- `mesh`: deploys Istio
- `telemetry`: deploys telemetry addons
- `chaos`: deploys Chaos Mesh

For example, the command
```
anisble-playbook site.yml -i hosts -t setup,init,join,chaos
```
makes a Kubernetes cluster and deploys Chaos Mesh on top of it.

Accodingly, to unmake
```
anisble-playbook rollback-site.yml -i hosts -t setup,init,join,chaos
```
