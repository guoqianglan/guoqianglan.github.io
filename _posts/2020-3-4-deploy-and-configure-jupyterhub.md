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

## Create pvc for user shared space
Prepare `share-pvc.yaml`,
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
 name: jhub-shared-vol
 namespace: jhub
 annotations:
   volume.beta.kubernetes.io/storage-class: heketi-gluster
spec:
 accessModes:
  - ReadWriteMany
 resources:
   requests:
     storage: 200Gi
```

```bash
kubectl apply -f shared-sc.yml
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

**Note**:
To specify memory, cpu, storage and image for singleuser, you can modify the config.yaml as below:
```yaml
singleuser:
  memory:
    guarantee: 2G
    limit: 2G
  cpu:
    limit: 2
    guarantee: 1
  storage:
    capacity: 20Gi
    dynamic:
      storageClass: heketi-gluster
    extraVolumes:
      - name: jhub-shared
        persistentVolumeClaim:
          claimName: jhub-shared-vol
    extraVolumeMounts:
      - name: jhub-shared
        mountPath: /home/shared
  image:
    name: lanheken/jupyter
    tag: cms
    pullPolicy: "Always"
  profileList:
    - display_name: "CMS environment"
      description: "Image for computational material science"
      default: true
    - display_name: "Minimal environment"
      description: "To avoid too much bells and whistles: Python."
      kubespawner_override:
        image:  jupyter/minimal-notebook:latest
```

# update config.yaml
When you need to update configuration, you can modify the config.yaml file and run commands as follows.
```bash
RELEASE=jhub

helm upgrade $RELEASE jupyterhub/jupyterhub --version=0.9.0 --values config.yaml
```

