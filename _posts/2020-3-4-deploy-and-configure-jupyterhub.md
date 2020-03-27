---
layout: post
title: Deploy and configure JupyterHub
---

![_config.yml]({{ site.baseurl }}/images/academic_computing.png)

In this post, I will show you how to deploy and configure JupyterHub.
Here I assumed that you already have a Kubernetes cluster and have set up the storageclass based on Heketi-glusterfs.
if not, please follow my [previous blog]({{ site.baseurl }}/deploy-heketi-glusterfs-for-dynamic-storage) to build one.

## Cluster Status

Master nodes:
- k8s-cluster-k8s-master-1 192.168.147.6

Worker nodes:
- k8s-cluster-k8s-node-nf-1 192.168.147.13 /dev/vdb
- k8s-cluster-k8s-node-nf-2 192.168.147.17 /dev/vdb
- k8s-cluster-k8s-node-nf-3 192.168.147.25 /dev/vdb
- k8s-cluster-k8s-node-nf-4 192.168.147.16 /dev/vdb

Storageclass:
```bash
[centos@k8s-cluster-k8s-master-1 build_jhub]$ kubectl get storageclass
NAME             PROVISIONER               AGE
gluster-heketi   kubernetes.io/glusterfs   45m
```

## Deploy jupyterhub using helm

First, you need to [install and initialize helm](https://zero-to-jupyterhub.readthedocs.io/en/stable/setup-helm.html).

Then please visit [zero-to-jupyterhub](https://zero-to-jupyterhub.readthedocs.io/en/stable/setup-jupyterhub.html) and go through the introduction first.

```bash
RELEASE=jhub
NAMESPACE=jhub

helm upgrade --install $RELEASE jupyterhub/jupyterhub --namespace $NAMESPACE --version=0.9.0-beta.4 --values config.yaml
```

# update config.yaml
```
RELEASE=jhub

helm upgrade $RELEASE jupyterhub/jupyterhub --version=0.9.0-beta.4 --values config.yaml
```


https://portworx.com/deploy-ha-jupyterhub-google-kubernetes-engine/