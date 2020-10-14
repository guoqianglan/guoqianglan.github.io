---
title: Infrastructure for a SaaS platform for material informatics 
header:
  image: assets/images/blog_infrastructure/header.png
categories:
  - idea
  - cloud
tags:
  - idea
  - cloud
  - Infrastructure
  - kubernetes
  - openstack
  - HPC
---

In this post, I would like to share the infrastructure of **McGill MatNavi**, a material informatics platform I just designed for McGill University.


# Technical
- **Virtual Hardware**: The McGill MatNavi cloud runs on the [OpenStack at Compute Canada cloud](https://docs.computecanada.ca/wiki/OpenStack), which provides virtual machines and virtual disk volumn.
- **Server Management**: The McGill MatNavi servers is managed via [ansible](https://www.ansible.com/)
- **Data storage**: Data are stored in persistent volume created by Heketi-gluster. In the future, the data will be stored in Cinder volumes when they are available in Compute Canada Cloud.
- **Backend**: all services, applications and databases are deployed via container managed by Kubernetes.
- **Frontend**: For simplicity, the frontend will be created using [Jekll](https://jekyllrb.com/) template with various applications embedded as iframe, just like my personal blog.


![RC file download]({{ site.url }}{{ site.baseurl }}/assets/images/blog_infrastructure/infrastructure.png)


