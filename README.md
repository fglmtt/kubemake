# Kubernetes-based Cluster Maker :hammer:

`kubemake` is a tool based on Ansible to (un)make Kubernetes-based clusters.

## Supported Components

`kubemake` currently supports the components listed below.

### System Architectures

| System Architecture |
| --- |
| amd64 |
| arm64 |

### Operating Systems

| Operating System | Version(s) |
| --- | --- |
| Ubuntu | 20.04, 22.04 |

### Kubernetes Distributions

| Kubernetes Distribution | Version(s) |
| --- | --- |
| Kubernetes | > 1.24 |

### Container Runtimes

| Container Runtime | Version(s) |
| --- | --- |
| CRI-O | > 1.24 |
  
### Network Plugins

| Network Plugin | Version(s) |
| --- | --- |
| Flannel | See [here](https://github.com/flannel-io/flannel/blob/master/Documentation/kube-flannel.yml) |

### Device Plugins

| Device Plugin | Version(s) |
| --- | --- |
| NVIDIA | 0.14.4 |

### Service Meshes

| Service Mesh | Version(s) |
| --- | --- |
| Istio | 1.20.2 |

### Telemetry Addons

| Telemetry Addon | Version(s) |
| --- | --- |
| Grafana | See [here](https://github.com/istio/istio/tree/master/manifests/addons) |
| Prometheus | See [here](https://github.com/istio/istio/tree/master/manifests/addons) |
| Jaeger | See [here](https://github.com/istio/istio/tree/master/manifests/addons) |
| Kiali | See [here](https://github.com/istio/istio/tree/master/manifests/addons) |
| Kubernetes Metrics Server | See [here](https://github.com/kubernetes-sigs/metrics-server) |
| kube-state-metrics | See [here](https://github.com/kubernetes/kube-state-metrics) |

### Chaos Engineering Tools

| Chaos Engineering Tool | Version(s) |
| --- | --- |
| Chaos Mesh | 2.6.1 |

### Serverless Frameworks

| Serverless Framework | Version(s) |
| --- | --- |
| Fission | 1.20.5 |
  
**Note 1**: the related versions are the ones that has been tested. Other versions might work as well.
  
## Requirements

`kubemake` does **not** provision virtual machines. This means you need your cluster nodes up and running beforehand. See [here](#operating-systems) for the operating systems currently supported. The control node, i.e., the node where `kubemake` runs, needs:

- Ansible (>= 2.13.1)
- [Kubernetes Collection for Ansible](https://galaxy.ansible.com/kubernetes/core?extIdCarryOver=true&sc_cid=701f2000001OH6uAAG)

**Note 2**: `kubemake`does **not** check if your cluster nodes meet hardware requirements. Please check them out based on the [components](#supported-components) you need.

**Note 3**: Ansible manages target nodes over SSH. This means the control node needs SSH access to the cluster nodes.

## Quick Start

## Ansible

Install Ansible in a Python virtual environment:

```
$ python3 -m venv .venv
$ source .venv/bin/activate
$ pip install -r requirements.txt
```

### Inventory

The first thing you need is to build your [inventory](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html). An example of inventory is the following:

```
[masters]
master ansible_user=ubuntu ansible_host=192.168.1.1 kubelet_node_ip=192.168.1.1 apiserver_advertise_address=192.168.1.1 apiserver_cert_extra_sans=172.16.100.100

[workers]
worker-1 ansible_user=ubuntu ansible_host=192.168.1.2 kubelet_node_ip=192.168.1.2 crio_os=xUbuntu_22.04 cni_arch=amd64
worker-2 ansible_user=ubuntu ansible_host=192.168.1.3 kubelet_node_ip=192.168.1.3 crio_os=xUbuntu_22.04 cni_arch=amd64
worker-3 ansible_user=ubuntu ansible_host=192.168.1.4 kubelet_node_ip=192.168.1.4 crio_os=xUbuntu_20.04 cni_arch=arm64

[gpus]
worker-3 ansible_user=ubuntu ansible_host=192.168.1.4

[controllers]
controller-1 ansible_user=ubuntu ansible_host=192.168.1.5

[all:vars]
ansible_ssh_private_key_file=/path/to/your/pem
crio_version="1.28"
cri_socket="unix:/run/crio/crio.sock"
cni_version="v1.4.0"
kubernetes_version_repo="v1.29"
kubernetes_os="deb"
kubernetes_version_pkg="1.29.9-1.1"

[masters:vars]
pod_network_cidr=10.244.0.0/16
flannel_iface_regex=[eth1|eth0]
nvidia_device_plugin_version="v0.14.4"
istio_version="1.20.2"
istio_profile=demo
istio_ingress_domain=yourdomain.edu
chaos_mesh_version="v2.6.1"
fission_version="v1.20.5"

[controllers:vars]
fission_version="v1.20.5"
```

**Note 4**: for the sake of brevity, this inventory also contains variables. Please look [here](https://docs.ansible.com/ansible/2.8/user_guide/playbooks_best_practices.html) for Ansible best practices.

**Note 5**: `kubemake` does **not** currently support high-availability clusters, i.e., clusters with multiple masters.

### (Un)Make

To make a Kubernetes-based cluster provided with all the [supported components](#supported-components), run:

```
$ ansible-playbook site.yml -i hosts
```

To unmake, run:

```
$ ansible-playbook rollback-site.yml -i hosts
```

If you are only interested in a subset of those components, you can use [Ansible tags](https://docs.ansible.com/ansible/latest/user_guide/playbooks_tags.html) to limit what `kubemake` does. Specifically

- `setup`: installs packages Kubernetes needs to run
- `init`: initializes the Kubernetes master
- `join`: makes worker nodes joining the master
- `helm`: deploys Helm
- `mesh`: deploys Istio
- `telemetry`: deploys telemetry addons
- `chaos`: deploys Chaos Mesh
- `serverless`: deploys Fission

For example, the command
```
$ ansible-playbook site.yml -i hosts -t setup,init,join,chaos
```
makes a Kubernetes cluster and deploys Chaos Mesh on top of it.

Accodingly, to unmake
```
$ ansible-playbook rollback-site.yml -i hosts -t setup,init,join,chaos
```

### Expected Result

Let's assume you used the inventory above and ran:
```
$ ansible-playbook site.yml -i hosts
```

If you SSH into the master, you should get the following:
```
$ kubectl get pod --all-namespaces
chaos-testing   chaos-controller-manager-6c6747db85-2cd5r   1/1     Running   (...)
chaos-testing   chaos-controller-manager-6c6747db85-5mckg   1/1     Running   (...)
chaos-testing   chaos-controller-manager-6c6747db85-nzbfb   1/1     Running   (...)
chaos-testing   chaos-daemon-9ffzd                          1/1     Running   (...)
chaos-testing   chaos-daemon-hb9mh                          1/1     Running   (...)
chaos-testing   chaos-daemon-zsxc9                          1/1     Running   (...)
chaos-testing   chaos-dashboard-f8578859d-27bxr             1/1     Running   (...)
istio-system    grafana-78588947bf-nng9r                    1/1     Running   (...)
istio-system    istio-egressgateway-59d4c446df-d84qd        1/1     Running   (...)
istio-system    istio-ingressgateway-67d6475549-rpx5c       1/1     Running   (...)
istio-system    istiod-544bd8d6bc-fkhld                     1/1     Running   (...)
istio-system    jaeger-b5874fcc6-8n86n                      1/1     Running   (...)
istio-system    kiali-575cc8cbf-vfkqn                       1/1     Running   (...)
istio-system    prometheus-6544454f65-v286r                 2/2     Running   (...)
kube-flannel    kube-flannel-ds-gdnn9                       1/1     Running   (...)
kube-flannel    kube-flannel-ds-n7bkd                       1/1     Running   (...)
kube-flannel    kube-flannel-ds-v2whg                       1/1     Running   (...)
kube-flannel    kube-flannel-ds-wvvxv                       1/1     Running   (...)
kube-system     coredns-6d4b75cb6d-c8vdg                    1/1     Running   (...)
kube-system     coredns-6d4b75cb6d-grnh6                    1/1     Running   (...)
kube-system     etcd-kube-master                            1/1     Running   (...)
kube-system     kube-apiserver-kube-master                  1/1     Running   (...)
kube-system     kube-controller-manager-kube-master         1/1     Running   (...)
kube-system     kube-proxy-578rd                            1/1     Running   (...)
kube-system     kube-proxy-5vwvn                            1/1     Running   (...)
kube-system     kube-proxy-gdjvv                            1/1     Running   (...)
kube-system     kube-proxy-vzvbv                            1/1     Running   (...)
kube-system     kube-scheduler-kube-master                  1/1     Running   (...)
```

By default, `kubemake` 

- enables [sidecar injection](https://istio.io/latest/docs/setup/additional-setup/sidecar-injection/) on the `default` namespace and restricts [the scope of Chaos Mesh](https://chaos-mesh.org/docs/configure-enabled-namespace/) to the same namespace;
- exposes Grafana, Jaeger, Kiali, Prometheus, and Chaos Mesh dashboards through the Istio ingress gateway. To get the HTTP port of the Istio ingress gateway:

```
$ kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}'
```

If you do not hold the domain you specified in the [inventory](#inventory), upload the `/etc/hosts` file as follows:

```
<ip>     grafana.yourdomain.edu
<ip>     kiali.yourdomain.edu
<ip>     jaeger.yourdomain.edu
<ip>     prometheus.yourdomain.edu
<ip>     chaos.yourdomain.edu
```

Since the Istio ingress gateway is exposed as a NodePort service, you can use any cluster node's IP. The dashboards are available at the following links (where INGRESS_HTTP is the number you obtained before):

* http://grafana.yourdomain.edu:INGRESS_HTTP
* http://kiali.yourdomain.edu:INGRESS_HTTP
* http://jaeger.yourdomain.edu:INGRESS_HTTP
* http://prometheus.yourdomain.edu:INGRESS_HTTP
* http://chaos.yourdomain.edu:INGRESS_HTTP

**Note 6**: `kubemake` exposes such dashboards using the [unsecure method](https://istio.io/v1.1/docs/tasks/telemetry/gateways/). Do not use this in production.
