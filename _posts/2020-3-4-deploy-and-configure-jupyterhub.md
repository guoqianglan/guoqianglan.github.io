---
title: Deploy and configure JupyterHub
header:
  image: assets/images/blog_kube_jhub/academic_computing.png
categories:
  - tutorial
  - cloud
tags:
  - tutorial
  - cloud
  - jupyterhub
  - kubernetes
  - openstack
toc: true
toc_sticky: true
---


In this post, I will show you how to deploy and configure JupyterHub.
Here I assumed that you already have a Kubernetes cluster and have set up the storageclass based on Heketi-glusterfs.
if not, please follow my [previous blog]({{ "/tutorial/cloud/deploy-heketi-glusterfs-for-dynamic-storage" | relative_url }}) to build one.

## Create storageclass for pvc assignment
Prepare yaml files as follows.

- 'jhub-sc.yml' for hub-db-dir and user home folder pvc assignement
```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1beta1
metadata:
  name: gluster-jhub-sc
provisioner: kubernetes.io/glusterfs
parameters:
  resturl: "http://10.233.59.198:8080"
  restuser: "admin"
  restuserkey: "adminkey"
  restauthenabled: "true"
  clusterid: "d132fc523fcf0c89d7a26c176f4c934d"
  volumetype: "replicate:2"
allowVolumeExpansion: true
```
- 'jhub-shared-sc.yml' for user shared folder pvc assignment
```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1beta1
metadata:
  name: gluster-jhub-shared-sc
provisioner: kubernetes.io/glusterfs
parameters:
  resturl: "http://10.233.59.198:8080"
  restuser: "admin"
  restuserkey: "adminkey"
  restauthenabled: "true"
  clusterid: "5108ea8a533e7366b26755470cf6d242"
  volumetype: "replicate:2"
allowVolumeExpansion: true
```
Run
```bash
kubectl apply -f jhub-sc.yml -f jhub-shared-sc.yml
```

## Deploy jupyterhub using helm

Please go to [zero-to-jupyterhub](https://zero-to-jupyterhub.readthedocs.io/en/stable/setup-helm.html) and see
how to install helm and deploy jupyterhub first.
 
```bash
helm repo add jupyterhub https://jupyterhub.github.io/helm-chart/
helm repo update

RELEASE=jhub
NAMESPACE=jhub

helm upgrade --install $RELEASE jupyterhub/jupyterhub --namespace $NAMESPACE --version=0.9.0 --values config.yaml
```

# update config.yaml
When you need to update configuration, you can modify the config.yaml file and run commands as follows.
```bash
RELEASE=jhub

helm upgrade $RELEASE jupyterhub/jupyterhub --version=0.9.0 --values config.yaml
```

