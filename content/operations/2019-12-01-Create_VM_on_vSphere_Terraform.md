---
title: Create VM on vSphere with Terraform
date: 2019-12-01T14:39:50+01:00
draft: false
categories:
  - InfraAsCode
tags:
  - Terraform
  - vSphere
---
## Introduction
In this post and this post, we created respectively some EC2 instances on AWS and some network contructs on Cisco’s ACI solution.

We will apply the same principle but instead of creating some servers on AWS, we will create some servers on vSphere.

## Terraform code
We will create a main.tf file which contains the entire configuration. It’s quite a lengthy file but it’s not really complicated to be honest.

First, we retrieve all values from our existing vSphere setup. Reason for doing this is that we will use them somewhat later in the file. For the server creation, we are essentially cloning a template, in my case a Ubuntu 16.04 server template which was created already before. We will also assign a static IP address (and DNS, domain…) to the servers.

```terraform
provider "vsphere" {
  user                 = var.vsphere_user
  password             = var.vsphere_password
  vsphere_server       = var.vsphere_server
  allow_unverified_ssl = true
}

data "vsphere_datacenter" "dc" {
  name = var.vsphere_datacenter
}

data "vsphere_network" "vm1_net" {
  name          = "VM Network"
  datacenter_id = data.vsphere_datacenter.dc.id
}

data "vsphere_network" "vm2_net" {
  name          = "VM Network"
  datacenter_id = data.vsphere_datacenter.dc.id
}

data "vsphere_datastore" "ds" {
  name          = var.vsphere_datastore
  datacenter_id = data.vsphere_datacenter.dc.id
}

data "vsphere_compute_cluster" "cl" {
  name          = var.vsphere_compute_cluster
  datacenter_id = data.vsphere_datacenter.dc.id
}

data "vsphere_virtual_machine" "template" {
  name          = var.vsphere_template
  datacenter_id = data.vsphere_datacenter.dc.id
}

resource "vsphere_virtual_machine" "aci_vm1" {
  count            = 1
  name             = var.aci_vm1_name
  resource_pool_id = data.vsphere_compute_cluster.cl.resource_pool_id
  datastore_id     = data.vsphere_datastore.ds.id

  num_cpus  = 2
  memory    = 4096
  guest_id  = data.vsphere_virtual_machine.template.guest_id
  scsi_type = data.vsphere_virtual_machine.template.scsi_type

  disk {
    label            = "disk0"
    size             = data.vsphere_virtual_machine.template.disks[0].size
    thin_provisioned = data.vsphere_virtual_machine.template.disks[0].thin_provisioned
  }

  folder = var.folder

  network_interface {
    network_id   = data.vsphere_network.vm1_net.id
    adapter_type = data.vsphere_virtual_machine.template.network_interface_types[0]
  }

  clone {
    linked_clone  = "false"
    template_uuid = data.vsphere_virtual_machine.template.id

    customize {
      linux_options {
        host_name = var.aci_vm1_name
        domain    = var.domain_name
      }

      network_interface {
        ipv4_address = var.aci_vm1_address
        ipv4_netmask = "24"
      }

      ipv4_gateway    = var.gateway
      dns_server_list = [var.dns_list]
      dns_suffix_list = [var.dns_search]
    }
  }
}

resource "vsphere_virtual_machine" "aci_vm2" {
  count            = 1
  name             = var.aci_vm2_name
  resource_pool_id = data.vsphere_compute_cluster.cl.resource_pool_id
  datastore_id     = data.vsphere_datastore.ds.id

  num_cpus = 2
  memory   = 4096
  guest_id = data.vsphere_virtual_machine.template.guest_id

  scsi_type = data.vsphere_virtual_machine.template.scsi_type

  disk {
    label            = "disk0"
    size             = data.vsphere_virtual_machine.template.disks[0].size
    thin_provisioned = data.vsphere_virtual_machine.template.disks[0].thin_provisioned
  }

  folder = var.folder

  network_interface {
    network_id   = data.vsphere_network.vm2_net.id
    adapter_type = data.vsphere_virtual_machine.template.network_interface_types[0]
  }

  clone {
    linked_clone  = "false"
    template_uuid = data.vsphere_virtual_machine.template.id

    customize {
      linux_options {
        host_name = var.aci_vm2_name
        domain    = var.domain_name
      }

      network_interface {
        ipv4_address = var.aci_vm2_address
        ipv4_netmask = "24"
      }

      ipv4_gateway    = var.gateway
      dns_server_list = [var.dns_list]
      dns_suffix_list = [var.dns_search]
    }
  }
}
```
The corresponding variables file is:
```terraform
variable "vsphere_server" {
  default = "10.x.y.z"
}

variable "vsphere_user" {
  default = "administrator@vsphere.local"
}

variable "vsphere_password" {
  default = "****!"
}

variable "vsphere_datacenter" {
  default = "SaS-DC"
}

variable "vsphere_datastore" {
  default = "datastore-UCS-POD1-2"
}

variable "vsphere_compute_cluster" {
  default = "SaS-Cluster"
}

variable "vsphere_template" {
  default = "ubuntu-1604-server-template"
}

variable "folder" {
  default = "wauterw"
}

variable "aci_vm1_name" {
  default = "vSphere1"
}

variable "aci_vm2_name" {
  default = "vSphere2"
}

variable "aci_vm1_address" {
  default = "10.16.2.233"
}

variable "aci_vm2_address" {
  default = "10.16.2.234"
}

variable "gateway" {
  default = "10.16.2.254"
}

variable "dns_list" {
  default = "10.9.15.1"
}

variable "dns_search" {
  default = "cisco.com"
}

variable "domain_name" {
  default = "cisco.com"
}
```

## Deploy infrastructure

As we did with AWS resources, we will follow exactly the same pattern.

First we will perform ‘terraform init’. This will essentially download the vSphere provider from Terraform repo.

```bash
WAUTERW-M-65P7:Create_VM_on_vSphere_Terraform wauterw$ terraform init

Initializing the backend...
* provider.vsphere: version = "~> 1.17"

Terraform has been successfully initialized!

***Truncated***
```
Then, we will perform a ‘terraform plan’:
```bash
WAUTERW-M-65P7:Create_VM_on_vSphere_Terraform wauterw$ terraform plan
Refreshing Terraform state in-memory prior to plan...
The refreshed state will be used to calculate this plan, but will not be
persisted to local or remote state storage.

data.vsphere_datacenter.dc: Refreshing state...
data.vsphere_datastore.ds: Refreshing state...
data.vsphere_compute_cluster.cl: Refreshing state...
data.vsphere_virtual_machine.template: Refreshing state...
data.vsphere_network.vm2_net: Refreshing state...
data.vsphere_network.vm1_net: Refreshing state...

***Truncated***

Plan: 2 to add, 0 to change, 0 to destroy.

------------------------------------------------------------------------

Note: You didn't specify an "-out" parameter to save this plan, so Terraform
can't guarantee that exactly these actions will be performed if
"terraform apply" is subsequently run.
```
And finally, let's apply the configuration:
```bash
WAUTERW-M-65P7:Create_VM_on_vSphere_Terraform wauterw$ terraform apply
data.vsphere_datacenter.dc: Refreshing state...
data.vsphere_network.vm2_net: Refreshing state...

***Truncated***

vsphere_virtual_machine.aci_vm1[0]: Creation complete after 5m59s [id=423c55ad-850a-d79a-bdc8-f796567f5f2d]
vsphere_virtual_machine.aci_vm2[0]: Still creating... [6m0s elapsed]
vsphere_virtual_machine.aci_vm2[0]: Still creating... [6m10s elapsed]
vsphere_virtual_machine.aci_vm2[0]: Creation complete after 6m11s [id=423c2218-c832-0a79-2158-53f7dee57546]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
```
Let's verify on vSphere:

![vSphereTerraform](/images/2019-12-01-1.png)

## Destroy configuration

Next, we will destroy the two servers as we don't immediately require them. Note that destroying goes much faster (6s) compared to creating the servers (6min). Reason is obviously the cloning step of the template during the creation.

```bash
WAUTERW-M-65P7:Create_VM_on_vSphere_Terraform wauterw$ terraform destroy
data.vsphere_datacenter.dc: Refreshing state...
data.vsphere_network.vm2_net: Refreshing state...
data.vsphere_network.vm1_net: Refreshing state...
data.vsphere_datastore.ds: Refreshing state...
data.vsphere_compute_cluster.cl: Refreshing state...

*** Truncated ***

vsphere_virtual_machine.aci_vm1[0]: Destruction complete after 16s
vsphere_virtual_machine.aci_vm2[0]: Destruction complete after 16s

Destroy complete! Resources: 2 destroyed.
```
Check on vSphere and the servers will be gone.

The files used in this blog can be found on [Github](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/master/Create_VM_on_vSphere_Terraform)

Pretty simple but good to make a blog post about it for future reference. Makes it easier to remember for next time.
Ciao!
