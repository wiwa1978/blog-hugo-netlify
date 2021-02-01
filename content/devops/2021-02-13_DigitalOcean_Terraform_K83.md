---
title: DigitalOcean - Create K8S cluster with Terraform
date: 2021-02-13T14:39:50+01:00
draft: True
categories:
  - DevOps
  - Infrastructure As Code
  - Public Cloud
  - All
tags:
  - Terraform
  - DigitalOcean
  - Kubernetes
---
### Introduction

In this post, we are going to create a Kubernetes cluster on DigitalOcean using Terraform. Something I wanted to do already for quite some time but never got to it really. In the past we have used already some different methods to install a Kubernetes cluster. He's an overview:

- [On Digitalocean droplets using Ansible](https://blog.wimwauters.com/devops/2019-10-17_digitalocean_kubernetes_ansible/)
- [On AWS using Ansible](https://blog.wimwauters.com/devops/2019-11-25-aws_kubernetes_ansible/)
- [On vSphere using Ansible](https://blog.wimwauters.com/devops/2019-12-02_vsphere_kubernetes_ansible/)
- [On Raspberry Pi 4 ESXi host](https://blog.wimwauters.com/devops/2020-11-13-ha_microk8s/)


![k8s_terraform](/images/2021-02-08-1.png)
![k8s_terraform](/images/2021-02-08-2.png)
![k8s_terraform](/images/2021-02-08-3.png)
![k8s_terraform](/images/2021-02-08-4.png)

### Terraform script

```hcl
provider "digitalocean"{
  token = var.do_token
}

resource "digitalocean_kubernetes_cluster" "kubernetes_cluster" {
  name    = var.k8s_clustername
  region  = var.region
  version = var.k8s_version

  tags = ["k8s"]

  # This default node pool is mandatory
  node_pool {
    name       = var.k8s_poolname
    size       = "s-2vcpu-2gb" # minimum size, list available options with `doctl compute size list`
    auto_scale = false
    node_count = var.k8s_count
    tags       = ["node-pool-tag"]
  }

}

output "cluster-id" {
  value = digitalocean_kubernetes_cluster.kubernetes_cluster.id
}
```

The variables file:

```tcl
variable "do_token" {
  default = "2c***5"
}

variable "region" {
  default = "ams3"
}

variable "k8s_clustername" {
  default = "clusterwim"
}

variable "k8s_version" {
  default = "1.19.3-do.3"
}

variable "k8s_poolname" {
  default = "worker-pool"
}

variable "k8s_count" {
  default = "3"
}

```

How do we know what slug to fill in as a valid version? We can use DOCTL for this

```
~/S/Programming/blog-hugo-netlify-code/DigitalOcean_Terraform/kubernetes master !3 ?5 ❯ doctl compute size list                                                                                12:14:52
Slug                  Memory    VCPUs    Disk    Price Monthly    Price Hourly
s-1vcpu-1gb           1024      1        25      5.00             0.007440
512mb                 512       1        20      5.00             0.007440
s-1vcpu-2gb           2048      1        50      10.00            0.014880
1gb                   1024      1        30      10.00            0.014880
s-3vcpu-1gb           1024      3        60      15.00            0.022320
s-2vcpu-2gb           2048      2        60      15.00            0.022320
```


```
~/S/P/blog-hugo-netlify-code/DigitalOcean_Terraform/kubernetes master !3 ?5 ❯ doctl compute region list                                                                                     4s 12:15:02
Slug    Name               Available
nyc1    New York 1         true
sfo1    San Francisco 1    false
nyc2    New York 2         false
ams2    Amsterdam 2        false
sgp1    Singapore 1        true
lon1    London 1           true
nyc3    New York 3         true
ams3    Amsterdam 3        true
fra1    Frankfurt 1        true
tor1    Toronto 1          true
sfo2    San Francisco 2    true
blr1    Bangalore 1        true
sfo3    San Francisco 3    true
```



```bash
~/S/Programming/blog-hugo-netlify-code/DigitalOcean_Terraform/kubernetes master !3 ?5 ❯ terraform init

Initializing the backend...

Initializing provider plugins...
- Finding latest version of digitalocean/digitalocean...
- Installing digitalocean/digitalocean v2.4.0...
- Installed digitalocean/digitalocean v2.4.0 (signed by a HashiCorp partner, key ID F82037E524B9C0E8)

Partner and community providers are signed by their developers.
If you'd like to know more about provider signing, you can read about it here:
https://www.terraform.io/docs/plugins/signing.html

The following providers do not have any version constraints in configuration,
so the latest version was installed.

To prevent automatic upgrades to new major versions that may contain breaking
changes, we recommend adding version constraints in a required_providers block
in your configuration, with the constraint strings suggested below.

* digitalocean/digitalocean: version = "~> 2.4.0"

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```


```bash
~/S/P/blog-hugo-netlify-code/DigitalOcean_Terraform/kubernetes master !3 ?5 ❯ terraform plan
Refreshing Terraform state in-memory prior to plan...
The refreshed state will be used to calculate this plan, but will not be
persisted to local or remote state storage.


------------------------------------------------------------------------

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # digitalocean_kubernetes_cluster.kubernetes_cluster will be created
  + resource "digitalocean_kubernetes_cluster" "kubernetes_cluster" {
      + cluster_subnet = (known after apply)
      + created_at     = (known after apply)
      + endpoint       = (known after apply)
      + id             = (known after apply)
      + ipv4_address   = (known after apply)
      + kube_config    = (sensitive value)
      + name           = "k8scluster_wim"
      + region         = "ams3"
      + service_subnet = (known after apply)
      + status         = (known after apply)
      + tags           = [
          + "k8s",
        ]
      + updated_at     = (known after apply)
      + version        = "1.19.3-do.3"
      + vpc_uuid       = (known after apply)

      + node_pool {
          + actual_node_count = (known after apply)
          + auto_scale        = false
          + id                = (known after apply)
          + name              = "worker-pool"
          + node_count        = 3
          + nodes             = (known after apply)
          + size              = "s-2vcpu-2gb"
          + tags              = [
              + "node-pool-tag",
            ]
        }
    }

Plan: 1 to add, 0 to change, 0 to destroy.

------------------------------------------------------------------------

Note: You didn't specify an "-out" parameter to save this plan, so Terraform
can't guarantee that exactly these actions will be performed if
"terraform apply" is subsequently run.
```



```bash
~/S/P/blog-hugo-netlify-code/DigitalOcean_Terraform/kubernetes master !3 ?5 ❯ terraform apply --auto-approve                                                                                4s 12:16:40
digitalocean_kubernetes_cluster.kubernetes_cluster: Creating...
digitalocean_kubernetes_cluster.kubernetes_cluster: Still creating... [10s elapsed]
digitalocean_kubernetes_cluster.kubernetes_cluster: Still creating... [20s elapsed]
digitalocean_kubernetes_cluster.kubernetes_cluster: Still creating... [30s elapsed]
digitalocean_kubernetes_cluster.kubernetes_cluster: Still creating... [40s elapsed]
digitalocean_kubernetes_cluster.kubernetes_cluster: Still creating... [50s elapsed]
digitalocean_kubernetes_cluster.kubernetes_cluster: Still creating... [1m0s elapsed]
<TRUNCATED>
digitalocean_kubernetes_cluster.kubernetes_cluster: Still creating... [5m10s elapsed]
digitalocean_kubernetes_cluster.kubernetes_cluster: Still creating... [5m20s elapsed]
digitalocean_kubernetes_cluster.kubernetes_cluster: Still creating... [5m30s elapsed]
digitalocean_kubernetes_cluster.kubernetes_cluster: Creation complete after 5m35s [id=02b7c276-b262-4a32-8178-26d812a1623b]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.

Outputs:

cluster-id = 02b7c276-b262-4a32-8178-26d812a1623b


```

Set environment variable
```bash
~/S/Programming/blog-hugo-netlify-code/DigitalOcean_Terraform/kubernetes master !3 ?5 ❯ export CLUSTER_ID=02b7c276-b262-4a32-8178-26d812a1623b  
```

Download kubectl config file

```bash
~/S/Programming/blog-hugo-netlify-code/DigitalOcean_Terraform/kubernetes master !3 ?5 ❯ curl -X GET \                                                                                          12:27:15
-H "Content-Type: application/json" \
-H "Authorization: Bearer 2c57a3370eb46f3a929b82d264e729fc675b9a242f19b7e95e8f6b9b75714dd5" \
"https://api.digitalocean.com/v2/kubernetes/clusters/$CLUSTER_ID/kubeconfig" \
> config
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  2020    0  2020    0     0    952      0 --:--:--  0:00:02 --:--:--   952
```
This will download a config file in your current repository:

```bash
~/S/P/blog-hugo-netlify-code/DigitalOcean_T/kubernetes master !3 ?5 ❯ kubectl cluster-info                                                                                ○ do-ams3-clusterwim 12:27:18
Kubernetes master is running at https://02b7c276-b262-4a32-8178-26d812a1623b.k8s.ondigitalocean.com
CoreDNS is running at https://02b7c276-b262-4a32-8178-26d812a1623b.k8s.ondigitalocean.com/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

```bash
~/S/P/blog-hugo-netlify-code/DigitalOcean_T/kubernetes master !3 ?5 ❯ kubectl get nodes                                                                                6s ○ do-ams3-clusterwim 12:27:57
NAME                STATUS   ROLES    AGE     VERSION
worker-pool-39dfa   Ready    <none>   6m53s   v1.19.3
worker-pool-39dfe   Ready    <none>   6m50s   v1.19.3
worker-pool-39dfg   Ready    <none>   7m3s    v1.19.3
```