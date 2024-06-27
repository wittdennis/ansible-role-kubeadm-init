# kubeadm_init

Initializes a Kubernetes cluster with kubeadm

## Requirements

Requires kubeadm to be installed on the system

## Role Variables

```yaml
kubeadm_init_apiserver_bind_port: 443 # Port where the apiserver is listening on
kubeadm_init_pod_network_cidr: "10.128.0.0/16" # Pod netword cidr range

# The above values only work while not overriding the following variables
kubeadm_init_init_configuration: see: https://kubernetes.io/docs/reference/config-api/kubeadm-config.v1beta3/

kubeadm_init_cluster_configuration: see: https://kubernetes.io/docs/reference/config-api/kubeadm-config.v1beta3/

kubeadm_init_kubelet_configuration: see: https://kubernetes.io/docs/reference/config-api/kubeadm-config.v1beta3/

kubeadm_init_kube_proxy_configuration: see: https://kubernetes.io/docs/reference/config-api/kubeadm-config.v1beta3/
```

Default values:

```yaml
---
kubeadm_init_apiserver_bind_port: 443
kubeadm_init_pod_network_cidr: "10.128.0.0/16"

kubeadm_init_init_configuration:
  localAPIEndpoint:
    advertiseAddress: "{{ ansible_all_ipv4_addresses | ansible.utils.ipaddr('private') | first }}"
    bindPort: "{{ kubeadm_init_apiserver_bind_port }}"

kubeadm_init_cluster_configuration:
  networking:
    podSubnet: "{{ kubeadm_init_pod_network_cidr }}"
  kubernetesVersion: "{{ kubeadm_init_version }}"
  apiServer:
    certSANs:
      - "{{ ansible_all_ipv4_addresses | ansible.utils.ipaddr('private') | first }}"
  controllerManager:
    extraArgs:
      bind-address: "0.0.0.0"
  scheduler:
    extraArgs:
      bind-address: "0.0.0.0"

kubeadm_init_kubelet_configuration:
  cgroupDriver: "systemd"

kubeadm_init_kube_proxy_configuration: {}
```

## Example Playbook

    - hosts: control_plane
      roles:
         - { role: wittdennis.kubeadm_init }

## License

MIT
