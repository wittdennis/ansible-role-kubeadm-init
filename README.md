# kubeadm_init

Initializes a Kubernetes cluster with kubeadm

## Requirements

Requires kubeadm to be installed on the system

## Role Variables

```yaml
kubeadm_init_apiserver_bind_port: 443 # Port where the apiserver is listening on
kubeadm_init_pod_network_cidr: "10.128.0.0/16" # Pod netword cidr range
kubeadm_init_config_path: /etc/kubernetes/kubeadm-config.yaml # Where the rendered kubeadm config is stored
kubeadm_init_reconcile: true # Reconcile config drift on already-initialized clusters

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
kubeadm_init_config_path: /etc/kubernetes/kubeadm-config.yaml
kubeadm_init_reconcile: true

kubeadm_init_init_configuration:
  localAPIEndpoint:
    advertiseAddress: "{{ ansible_facts['all_ipv4_addresses'] | ansible.utils.ipaddr('private') | first }}"
    bindPort: "{{ kubeadm_init_apiserver_bind_port }}"

kubeadm_init_cluster_configuration:
  networking:
    podSubnet: "{{ kubeadm_init_pod_network_cidr }}"
  kubernetesVersion: "{{ kubeadm_init_version }}"
  apiServer:
    certSANs:
      - "{{ ansible_facts['all_ipv4_addresses'] | ansible.utils.ipaddr('private') | first }}"
  controllerManager:
    extraArgs:
      - name: bind-address
        value: "0.0.0.0"
  scheduler:
    extraArgs:
      - name: bind-address
        value: "0.0.0.0"

kubeadm_init_kubelet_configuration:
  cgroupDriver: "systemd"

kubeadm_init_kube_proxy_configuration: {}
```

## Reconciliation

On first run the role bootstraps the cluster with `kubeadm init`. On subsequent
runs, if the rendered kubeadm config changes (e.g. new apiserver `extraArgs`),
the role re-uploads the config to the cluster and regenerates the control plane
static pod manifests via `kubeadm init phase`. The kubelet then restarts the
affected static pods automatically.

Each control plane node owns its own static pod manifests, so run the role
against all control plane hosts (not `run_once`) when reconciling config drift.
Set `kubeadm_init_reconcile: false` to opt out.

## Example Playbook

    - hosts: control_plane
      roles:
         - { role: wittdennis.kubeadm_init }

## License

MIT
