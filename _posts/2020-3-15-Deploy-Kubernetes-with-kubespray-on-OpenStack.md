---
layout: post
title: Deploy Kubernetes with Kubespray on OpenStack
---

In this post, I will show you how to deploy Kubernetes OpenStack.
More specifically, [Kubespary](https://github.com/kubernetes-sigs/kubespray) with 
[Terraform suport](https://github.com/kubernetes-sigs/kubespray/tree/master/contrib/terraform) will be applied to simplify the deployment.

## Status

- *Local:* Ubuntu WSL system (Any Linux system should work as well)
- *Cloud:* Persitent zone belong to [Compute Canada Cloud, Arbhutus](https://arbutus.cloud.computecanada.ca/) (Other clouds based on OpenStack should work as well)

## Requirements

- *Local:*

  1. Create a new python virtual environment, e.g., 'openstack_env', activate it, and install openstackclient.
  ```bash
  virtualenv --python=python3 env/openstack_env
  source env/openstack_env/bin/activate
  pip install python-openstackclient
  ```
  2. Install the dependencies of Kubespary.
  ```bash
  git clone https://github.com/kubernetes-sigs/kubespray
  cd kubespray
  pip install -r requirements.txt
  ```
  3. [Install Terraform 0.12](https://www.terraform.io/intro/getting-started/install.html) or later.
  4. Download the OpenStack RC File from your cloud provider and load it.
  ![_config.yml]({{ site.baseurl }}/images/blog_kube_jhub/download_rc_file.png)
  ```bash
  source <your_project_name>-openrc.sh
  ```
  
## Setup cluster

In kubespary directory, exectucate following commands.
  ```bash
  CLUSTER=my-kube
  cp -LRp contrib/terraform/openstack/sample-inventory \
  inventory/$CLUSTER
  cd inventory/$CLUSTER
  ln -s ../../contrib/terraform/openstack/hosts
  ln -s ../../contrib
  ```
Edit the cluster variable file, `inventory/$CLUSTER/cluster.tfvars`.

For example:
```yaml
# your Kubernetes cluster name here
cluster_name = "k8s-cluster"

# list of availability zones available in your OpenStack cluster
az_list = ["Persistent_01", "Persistent_02"]
az_list_node = ["Persistent_01", "Persistent_02"]

dns_nameservers=["100.125.4.25", "8.8.8.8"]

# SSH key to use for access to nodes
public_key_path = "~/.ssh/id_rsa.pub"

# image to use for bastion, masters, standalone etcd instances, and nodes
image = "CentOS-7-x64-2019-07"

# user on the node (ex. core on Container Linux, ubuntu on Ubuntu, etc.)
ssh_user = "centos"

# 0|1 bastion nodes
number_of_bastions = 0
flavor_bastion = "448319d3-2417-4eb1-9da2-63a2fdbc23f6"

# standalone etcds
number_of_etcd = 0
flavor_etcd = "448319d3-2417-4eb1-9da2-63a2fdbc23f6"

# masters
number_of_k8s_masters = 1

number_of_k8s_masters_no_etcd = 0

number_of_k8s_masters_no_floating_ip = 0

number_of_k8s_masters_no_floating_ip_no_etcd = 0

flavor_k8s_master = "448319d3-2417-4eb1-9da2-63a2fdbc23f6"

# nodes
number_of_k8s_nodes = 0

number_of_k8s_nodes_no_floating_ip = 4

flavor_k8s_node = "448319d3-2417-4eb1-9da2-63a2fdbc23f6"

# GlusterFS
# either 0 or more than one
# number_of_gfs_nodes_no_floating_ip = 1
# gfs_volume_size_in_gb = 500
# Container Linux does not support GlusterFS
image_gfs = "CentOS-7-x64-2019-07"
# May be different from other nodes
ssh_user_gfs = "centos"
flavor_gfs_node = "448319d3-2417-4eb1-9da2-63a2fdbc23f6"

# networking
network_name = "k8s-network"

external_net = "6621bf61-6094-4b24-a9a0-f5794c3a881e"

subnet_cidr = "192.168.147.0/24"

floatingip_pool = "Public-Network"

bastion_allowed_remote_ips = ["0.0.0.0/0"]

```
To start the Terraform deployment, you need to install some plugins using command as follows.
```
terraform init contrib/terraform/openstack
```
Start to build the cluster.
```
terraform apply -var-file=cluster.tfvars ../../contrib/terraform/openstack
```
If it is finished successfully, you will get output as follows.
```
Apply complete! Resources: 5 added, 0 changed, 0 destroyed.

Outputs:

bastion_fips = []
floating_network_id = 6621bf61-****************
k8s_master_fips = [
  "206.**.**.***",
]
k8s_node_fips = []
private_subnet_id = dfa59b71-**************
router_id = cf695cfb-******************
```
Then go back to the kubespary root directory.

Try if ansible can successfully reach our clusters using

```
ansible -i inventory/my-kube/hosts -m ping all
```
Note:
If the cluster is unreachable, please add additional security group, SSH (TCP, port 22), to the k8s-cluster-k8s security group, and then try agian.


modify `inventory/$CLUSTER/group_vars/all/all.yml` `cloud_provider: openstack`

modify `inventory/$CLUSTER/group_vars/k8s-cluster/k8s-cluster.yml`,  `kube_network_plugin: flannel` resolvconf_mode we will use “docker_dns”, `use_access_ip: 0`

modify `inventory/$CLUSTER/group_vars/k8s-cluster/addons.yml` `helm_enabled: true`

Note:
If you failed to pass the etcd cluster healthy check, you may need to open port, TCP 2379. If it doesn't help, you would need login the master node 
and run commamd, `sudo chmod 755 -R /etc/ssl/etcd`. After that, you can check the healthy status by running
`etcdctl --endpoints https://<master_ip>:2379 --ca-file=/etc/ssl/etcd/ssl/ca.pem --cert-file=/etc/ssl/etcd/ssl/member-k8s-cluster-k8s-master-1.pem --key-file=/etc/ssl/etcd/ssl/member-k8s-cluster-k8s-master-1-key.pem cluster-health`


Make sure you have good internet connection, or it is very easy to get timeout exception when you run the ansible playbook.
```


```
![_config.yml]({{ site.baseurl }}/images/academic_computing.png)