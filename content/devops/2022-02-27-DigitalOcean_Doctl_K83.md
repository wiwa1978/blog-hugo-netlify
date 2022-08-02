---
title: DigitalOcean - Create K8S cluster with DOCTL
date: 2022-02-27T14:39:50+01:00
draft: False
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

In [this](https://blog.wimwauters.com/devops/2022-02-25-digitalocean_terraform_k8s/) blogpost, we have setup a Kubernetes cluster on DigitalOcean using Terraform. There is another, slightly lesser known, option to create Kubernetes clusters onto DigitalOcean so I wanted to dedicate a very small post to that option as well.

Essentially, DigitalOcean has a command line interface available which is called [DOCTL](https://docs.digitalocean.com/reference/doctl/). It allows us to interact with DigitialOcean through the command line. Most of the functionality is supported. So let's have a quick look.

The command to create the K8S cluster is shown in the snippet below. We want to create a K8S cluster with 3 nodes in the AMS3 region:

```bash
~/Kubernetes-DOCTL ❯ doctl kubernetes cluster create k8swim --count 3 --region ams3 --size s-2vcpu-2gb --version 1.22.8-do.1

Notice: Cluster is provisioning, waiting for cluster to be running
.............................................................................
Notice: Cluster created, fetching credentials
Notice: Adding cluster credentials to kubeconfig file found in "/Users/wauterw/.kube/config"
Notice: Setting current-context to do-ams3-k8swim
ID                                      Name      Region    Version        Auto Upgrade    Status     Node Pools
1af3ac86-2431-49ee-a82d-7e9ca06be966    k8swim    ams3      1.22.8-do.1    false           running    k8swim-default-pool
```

It takes a couple of minutes but if all goes well, you will see a newly provisioned cluster in your console:

![k8s_terraform](/images/2022-02-27-1.png)

With indeed 3 running nodes and located in the AMS3 region.

![k8s_terraform](/images/2022-02-27-2.png)

We can go ahead and save our K8S configuration file as follows:

```bash
~/Kubernetes-DOCTL ❯ doctl k8s cluster kubeconfig save 1af3ac86-2431-49ee-a82d-7e9ca06be966
Notice: Adding cluster credentials to kubeconfig file found in "/Users/wauterw/.kube/config"
Notice: Setting current-context to do-ams3-k8swim
```

The above command will set our current context to the newly created cluster. We can now interact with our cluster using `kubectl`

```bash
~/Kubernetes-DOCTL ❯ kubectl cluster-info
Kubernetes control plane is running at https://1af3ac86-2431-49ee-a82d-7e9ca06be966.k8s.ondigitalocean.com
CoreDNS is running at https://1af3ac86-2431-49ee-a82d-7e9ca06be966.k8s.ondigitalocean.com/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

Note: DigitalOcean displays the nice Kubernetes Dashboard. Click on the link in the console:

![k8s_terraform](/images/2022-02-27-3.png)

And we can use kubectl to get an overview of the nodes of course.

```bash
~/Kubernetes-DOCTL ❯ kubectl get nodes
NAME                        STATUS   ROLES    AGE     VERSION
k8swim-default-pool-cg736   Ready    <none>   8m48s   v1.22.8
k8swim-default-pool-cg73l   Ready    <none>   8m53s   v1.22.8
k8swim-default-pool-cg73t   Ready    <none>   8m43s   v1.22.8
```

Now let's remove the cluster through DOCTL. The command to do so is shown in below snippet.

```bash
~/Kubernetes-DOCTL ❯ doctl kubernetes cluster delete 1af3ac86-2431-49ee-a82d-7e9ca06be966
Warning: Are you sure you want to delete this Kubernetes cluster? (y/N) ? y
Notice: Cluster deleted, removing credentials
Notice: Removing cluster credentials from kubeconfig file found in "/Users/wauterw/.kube/config"
Notice: The removed cluster was set as the current context in kubectl. Run `kubectl config get-contexts` to see a list of other contexts you can use, and `kubectl config set-context` to specify a new one.
```
