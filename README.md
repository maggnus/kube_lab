# Kubernetes Lab
Single run Kubernetes deployment with Vagrant. I use this script to setup test Kubernetes cluster for my CKA exam prepartion:
- upgrade cluster version with `kubeadm` (default version is `1.22.0-00`
- backup `etcd`
- join cluster nodes (`kubeadm token create --print-join-command`)
- create Kubernetes resources
- etc.

## Prerequisites
Tools required to apply this deployemnt:
- Ansible
- Vagrant
- Vritualbox

## Installation

1. Clone repository and change working directory
    ```sh
    git clone git@github.com:maggnus/kube_lab.git
    cd kube_lab
    ```
2. Bootstrap Kubernetes with Vagrant scripts
   ```sh
   vagrant up
   ```
3. Login to master nodes and check the cluster
    ```sh
    vagrant ssh kube-master01
    sudo kubectl get nodes,pods,svc -A
    ```

## VM Settings
`etc/config.yml`
```yaml
---
box_name: "ubuntu/hirsute64"

providers:
  # VirtualBox
  virtualbox:
    enable: true

servers:
  - name: "kube-master01"
    cpu: 2
    ram: 4096
    ip: 192.168.56.11
    role: "master"

  - name: "kube-node01"
    cpu: 2
    ram: 2048
    ip: 192.168.56.21
    role: "node"
```

## Cluster settings
`ansible/roles/kubernetes/defaults/main.yml`
```yaml

# Kubernetes version
kube_version: 1.22.0-00

kube_master_host: "{{ groups['master'][0] }}"
kube_master_ip: "{{ hostvars[kube_master_host].instance_ip }}"

kube_apiserver_ip: "{{ kube_master_ip }}"
kube_apiserver_dns: "{{ kube_master_host }}"
kube_apiserver_port: 6443
kube_kubelet_port: 10250

# Docker config
docker_config:
  exec-opts:
    - native.cgroupdriver=systemd
```

## Ansible deployment
Ansible script could be run independently without Vagrant provisioning
```sh
cd ansible
./ansible-local.sh playbook.yaml
```
