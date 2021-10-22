# Ansible Kubeadm Setup

Build a Kubernetes cluster using Ansible with kubeadm. The goal is easily install a Kubernetes cluster on machines running:

  - Ubuntu 20.04
  - Ubuntu 16.04
  - CentOS 7
  - RedHat 8

System requirements:

  - Deployment environment must have Ansible
  - Node with atleast 2GB RAM and 2vCPU

# Usage

Add the system information gathered above into a file called `hosts`. You must have connectivity between all the nodes. 
For example:
```
[master]
#ip_of_master_node
192.156.68.101

[slave]
#ip_of_slave_nodes
192.168.56.10[2:5]
```

Edit `remote_user` and `private_key_file` in `ansible.cfg` before proceed to next steps. 

Before continuing, edit `group_vars/all.yml` to your specified configuration.

For example, to run `docker` instead of crio, change:

```yaml
# container runtimes ('docker', 'crio')
container_runtime: docker
```

# Requirements

Some modules in roles is also dependent on some other ansible galaxy collection. So run the `requirements.yml` first...

```sh
$ ansible-galaxy collection install -r requirements.yml
...
Process install dependency map
Starting collection install process
Installing 'community.general:2.5.1' to '/root/.ansible/collections/ansible_collections/community/general'
```

# Playbook

```yaml
- name: Install Necessary Software
  hosts: all
  vars_files:
    - group_vars/all.yml
  roles:
  -  role: runtime/{{ container_runtime }}-setup
  -  role: common
- name: Setup Master node
  hosts: master
  roles:
  -  role: master
  -  role: cni
- name: Setup Slave  node
  hosts: slave
  roles:
  -  role: node
```

Run this playbook for provision cluster

```sh
$ ansible-playbook role-kube.yml
```

Now, Verify cluster running using Kubectl command in master node (su as `root` user):

```sh
$ kubectl get nodes
```

