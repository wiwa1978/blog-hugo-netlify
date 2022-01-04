---
title: DigitalOcean - Create K8S cluster with DOCTL
date: 2021-02-17T14:39:50+01:00
draft: True
categories:
  - DevOps
  - Infrastructure As Code
  - Public Cloud
  - All
tags:
  - Doctl
  - DigitalOcean
  - Kubernetes
---
### Introduction

![k8s_terraform](/images/2021-02-17-1.png)
![k8s_terraform](/images/2021-02-17-2.png)
![k8s_terraform](/images/2021-02-17-3.png)


```bash
~/S/Programming/blog-hugo-netlify-code/DigitalOcean_Terraform/kubernetes master !3 ?5 ❯ doctl kubernetes cluster create k8swim --count 3 --region ams3 --size s-2vcpu-2gb --version 1.19.3-do.3        Notice: Cluster is provisioning, waiting for cluster to be running
.....................................................
Notice: Cluster created, fetching credentials
Notice: Adding cluster credentials to kubeconfig file found in "/Users/wauterw/SynologyDrive/Programming/blog-hugo-netlify-code/DigitalOcean_Terraform/kubernetes/config"
Notice: Setting current-context to do-ams3-k8swim
ID                                      Name      Region    Version        Auto Upgrade    Status     Node Pools
a0eae8bb-8bb0-452e-8417-7bfa5347d1cb    k8swim    ams3      1.19.3-do.3    false           running    k8swim-default-pool

```


```bash
~/S/Programming/blog-hugo-netlify-code/DigitalOcean_Terraform/kubernetes master !3 ?5 ❯ doctl k8s cluster kubeconfig save a0eae8bb-8bb0-452e-8417-7bfa5347d1cb                                 12:41:01
W0130 12:41:28.556296   64770 loader.go:223] Config not found: /Users/wauterw/SynologyDrive/Programming/blog-hugo-netlify-code/DigitalOcean_Terraform/kubernetes/config
Notice: Adding cluster credentials to kubeconfig file found in "/Users/wauterw/SynologyDrive/Programming/blog-hugo-netlify-code/DigitalOcean_Terraform/kubernetes/config"
Notice: Setting current-context to do-ams3-k8swim
W0130 12:41:28.556574   64770 loader.go:223] Config not found: /Users/wauterw/SynologyDrive/Programming/blog-hugo-netlify-code/DigitalOcean_Terraform/kubernetes/config
W0130 12:41:28.556633   64770 loader.go:223] Config not found: /Users/wauterw/SynologyDrive/Programming/blog-hugo-netlify-code/DigitalOcean_Terraform/kubernetes/config
```


```bash
~/S/P/blog-hugo-netlify-code/DigitalOcean_T/kubernetes master !3 ?5 ❯ kubectl cluster-info
Kubernetes master is running at https://a0eae8bb-8bb0-452e-8417-7bfa5347d1cb.k8s.ondigitalocean.com
CoreDNS is running at https://a0eae8bb-8bb0-452e-8417-7bfa5347d1cb.k8s.ondigitalocean.com/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```



```bash

~/S/P/blog-hugo-netlify-code/DigitalOcean_T/kubernetes master !3 ?5 ❯ kubectl get nodes                                                                                       ○ do-ams3-k8swim 12:41:49
NAME                        STATUS   ROLES    AGE     VERSION
k8swim-default-pool-39dx0   Ready    <none>   4m25s   v1.19.3
k8swim-default-pool-39dxd   Ready    <none>   4m20s   v1.19.3
k8swim-default-pool-39dxv   Ready    <none>   4m16s   v1.19.3
```


```bash
~/SynologyDrive/Programming/blog-hugo-netlify-code master !3 ?5 ❯ doctl kubernetes cluster get a0eae8bb-8bb0-452e-8417-7bfa5347d1cb                                                         7s 12:45:27
ID                                      Name      Region    Version        Auto Upgrade    Status     Endpoint                                                               IPv4              Cluster Subnet    Service Subnet    Tags                                            Created At                       Updated At                       Node Pools
a0eae8bb-8bb0-452e-8417-7bfa5347d1cb    k8swim    ams3      1.19.3-do.3    false           running    https://a0eae8bb-8bb0-452e-8417-7bfa5347d1cb.k8s.ondigitalocean.com    104.248.198.89    10.244.0.0/16     10.245.0.0/16     k8s,k8s:a0eae8bb-8bb0-452e-8417-7bfa5347d1cb    2021-01-30 11:33:46 +0000 UTC    2021-01-30 11:39:02 +0000 UTC    k8swim-default-pool
```


```bash

~/SynologyDrive/Programming/blog-hugo-netlify-code master !3 ?5 ❯ doctl kubernetes cluster delete a0eae8bb-8bb0-452e-8417-7bfa5347d1cb                                                      4s 12:45:41
Warning: Are you sure you want to delete this Kubernetes cluster? (y/N) ? y
Notice: Cluster deleted, removing credentials
Notice: Removing cluster credentials from kubeconfig file found in "/Users/wauterw/.kube/config"

```