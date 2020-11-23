---
title: "Geospatial object detection and segmentation"
header:
  iframe:
    url: https://geoseg.multiscale.ml/
  teaser: assets/images/apps/geo_teaser.png
categories:
  - app
  - deep learning
tags:
  - app
  - geospatial
---

> Dataset: [NWPU-VHR-10](https://drive.google.com/file/d/1X2aE3uDRckIqjXlUUuDNmYzXJohlh3RJ/view?usp=sharing)

> Model: Mask R-CNN with torchvision

> mAP (IOU = 0.5): 0.973

> mAR (max detections per image = 100): 0.770

> This application is deployed with two replicated sets on a Kubernetes cluster

> Things need to do next: 

* Evaluate FPS (Frames per seconds)
* Try to use a light weight weight model (e.g., mobilenet_v2, suqeezeNet...) to replace ResNet to improve the FPS
* Compare current metric with the work done by others
* Use a confusion matrix to recognize patterns which classes are mixed with which or which classes cannot be detect
* May need more samples for some classes to improve the recall.

> URL: [https://geoseg.multiscale.ml/](https://geoseg.multiscale.ml/)
