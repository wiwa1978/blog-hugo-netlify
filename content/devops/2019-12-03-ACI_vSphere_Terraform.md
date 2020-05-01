---
title: Create VM on vSphere and network on Cisco ACI (using Terraform)
date: 2019-12-03T14:39:50+01:00
draft: false
categories:
  - DevOps
  - Infrastructure As Code
  - Private Cloud
  - All
tags:
  - ACI
  - Terraform
  - vSphere
---

### Introduction

In past couple of posts, we have been experimenting with Terraform in combination with ACI and vSphere seperately. In this post, we will combine both and we will first create network constructs on Cisco's ACI solution and once done, we will create two VM instances on vSphere that are actually using the underlying ACI network constructs.

### ACI configuration

A lot has already been discussed in this post here, so I won't re-iterate all of that here and instead focus on the new items in the Terraform file.

In below snippet, you can see we create a tenant, VRF and BD. The BD will use a subnet of '10.16.100.1/24'. The VMs we sill create in a later phase will hence be using an IP address from that subnet. 

Furthermore, we will create an Application Profile and two EPGs into that Application Profile. Each EPG will contain a VM. For demonstration purposes, we have also defined a contract between those EPGs. The contracts will allow HTTPS and ICMP traffic (refer to the aci_filter_entry resources in the below snippet). You will see in the 'aci_application_epg' resources that we link the contract (aci_contract.Contract_EPG1_EPG2.name) to the application EPGs (resource aci\_application_epg).

The full content of the main_aci.tf file is:

```terraform
provider "aci" {
  username = var.apic_username
  password = var.apic_password
  url      = var.apic_server
  insecure = true
}

resource "aci_tenant" "Tenant_TF" {
  name        = var.aci_tenant
  description = "Tenant created by TF"
}

resource "aci_vrf" "VRF_TF" {
  tenant_dn          = aci_tenant.Tenant_TF.id
  name               = var.aci_vrf
  description        = "VRF created by TF"
  bd_enforced_enable = false
}

resource "aci_bridge_domain" "BD_TF" {
  tenant_dn          = aci_tenant.Tenant_TF.id
  name               = var.aci_bd
  description        = "BD created by TF"
  relation_fv_rs_ctx = aci_vrf.VRF_TF.name
}

resource "aci_subnet" "Subnet_BD" {
  bridge_domain_dn = aci_bridge_domain.BD_TF.id

  #name             = "Subnet"
  ip = var.aci_bd_subnet
}

resource "aci_application_profile" "AppProfile_TF" {
  tenant_dn   = aci_tenant.Tenant_TF.id
  name        = var.aci_app_profile
  description = "App profile created by TF"
}

resource "aci_application_epg" "EPG_TF_1" {
  application_profile_dn = aci_application_profile.AppProfile_TF.id
  name                   = var.aci_epg_1
  description            = "EPG created by TF"
  relation_fv_rs_bd      = aci_bridge_domain.BD_TF.name
  relation_fv_rs_dom_att = [var.vmm_domain]
  relation_fv_rs_cons    = [aci_contract.Contract_EPG1_EPG2.name]
}

resource "aci_application_epg" "EPG_TF_2" {
  application_profile_dn = aci_application_profile.AppProfile_TF.id
  name                   = var.aci_epg_2
  description            = "EPG created by TF"
  relation_fv_rs_bd      = aci_bridge_domain.BD_TF.name
  relation_fv_rs_dom_att = [var.vmm_domain]
  relation_fv_rs_prov    = [aci_contract.Contract_EPG1_EPG2.name]
}

resource "null_resource" "delay" {
  provisioner "local-exec" {
    command = "sleep 5"
  }

  triggers = {
    "epg1" = aci_application_epg.EPG_TF_1.id
    "epg2" = aci_application_epg.EPG_TF_2.id
  }
}

resource "aci_contract" "Contract_EPG1_EPG2" {
  tenant_dn   = aci_tenant.Tenant_TF.id
  name        = "Contract_EPG1_EPG2"
  description = "Contract created by TF"
}

resource "aci_contract_subject" "Contract_Subject" {
  contract_dn                  = aci_contract.Contract_EPG1_EPG2.id
  name                         = var.aci_contract_subject
  relation_vz_rs_subj_filt_att = [aci_filter.Filter_Allow_HTTPS.name, aci_filter.Filter_Allow_ICMP.name]
}

resource "aci_filter" "Filter_Allow_HTTPS" {
  tenant_dn = aci_tenant.Tenant_TF.id
  name      = var.aci_filter_allow_https
}

resource "aci_filter" "Filter_Allow_ICMP" {
  tenant_dn = aci_tenant.Tenant_TF.id
  name      = var.aci_filter_allow_icmp
}

resource "aci_filter_entry" "Filter_Entry_HTTPS" {
  name        = var.aci_filter_entry_https
  filter_dn   = aci_filter.Filter_Allow_HTTPS.id
  ether_t     = "ip"
  prot        = "tcp"
  d_from_port = "https"
  d_to_port   = "https"
  stateful    = "yes"
}

resource "aci_filter_entry" "Filter_Entry_ICMP" {
  name      = var.aci_filter_entry_icmp
  filter_dn = aci_filter.Filter_Allow_ICMP.id
  ether_t   = "ip"
  prot      = "icmp"
  stateful  = "yes"
}
```

We also have to define the following variables in the variables_aci.tf file.

```terraform
variable "apic_server" {
  default = "https://10.16.2.1"
}

variable "apic_username" {
  default = "admin"
}

*** Note: truncated ***
```
The entire ACI variables file can be found [here](https://github.com/wiwa1978/blog-hugo-netlify-code/blob/master/ACI_vSphere_Terraform/variables_aci.tf)

### vSphere configuration

Next, we will focus on the vSphere portion and more in particular, how we can link the VM instances to the ACI network. 

You will notice in below snippet that we are using quite some Terraform data sources. Data sources allow data to be fetched from your system to be used elsewhere in the Terraform code. For vSphere, we will read information from our system such as the datacenter, the datastore, the cluster, the template to be used. All this because we will use it elsewhere to tell Terraform where it needs to create these resources.

One thing to point out, and potentially a little more complicated, is the data source for the network. ACI uses the concept of a VMM domain (Virtual Machine Manager). It is a constuct to bind the ACI network constructs to the VMware network constructs. Upon creation, ACI will create a port group on VMWare's DVS and it will call this portgroup according to the following name format: tenant|applicationprofile|EPGname. In the datasource 'data vsphere_network', you will see that we are retrieving the network from VMWare using that name format.

For the rest, it's business as usual, we will just create two instances on vSphere and tell it to use that portgroup ( network\_id = data.vsphere\_network.vm1_net.id)

```hcl
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
  depends_on = [null_resource.delay]
  name = format(
    "%v|%v|%v",
    aci_tenant.Tenant_TF.name,
    aci_application_profile.AppProfile_TF.name,
    aci_application_epg.EPG_TF_1.name,
  )
  datacenter_id = data.vsphere_datacenter.dc.id
}

data "vsphere_network" "vm2_net" {
  depends_on = [null_resource.delay]
  name = format(
    "%v|%v|%v",
    aci_tenant.Tenant_TF.name,
    aci_application_profile.AppProfile_TF.name,
    aci_application_epg.EPG_TF_2.name,
  )
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

In order for everything to work, you will of course need to specify some variables. These variables will be used in the vSphere Terrafrom file.

```hcl
variable "vsphere_server" {
  default = "10.16.2.99"
}

*** Note: truncated ***

```
The entire vSphere variable file can be found [here](https://github.com/wiwa1978/blog-hugo-netlify-code/blob/master/ACI_vSphere_Terraform/variables_vsphere.tf)


### Deploying ACI and vSphere

Let's run the 'terraform init' command. You will see that we are downloading the required providers.

```bash
cisco@wauterw-ubuntu-desktop:~/software/Terraform/ACI-vSphere$ terraform init

Initializing the backend...

Initializing provider plugins...

The following providers do not have any version constraints in configuration,
so the latest version was installed.

To prevent automatic upgrades to new major versions that may contain breaking
changes, it is recommended to add version = "..." constraints to the
corresponding provider blocks in configuration, with the constraint strings
suggested below.

* provider.aci: version = "~> 0.1"
* provider.null: version = "~> 2.1"
* provider.vsphere: version = "~> 1.13"

*** Note: truncated output ***
```

Next, let's take a look at the plan. You will see that 16 resources will be created (both for VMware as for ACI).

```bash
cisco@wauterw-ubuntu-desktop:~/software/Terraform/ACI-vSphere$ terraform plan
Refreshing Terraform state in-memory prior to plan...
The refreshed state will be used to calculate this plan, but will not be
persisted to local or remote state storage.

data.vsphere_datacenter.dc: Refreshing state...
data.vsphere_virtual_machine.template: Refreshing state...
data.vsphere_compute_cluster.cl: Refreshing state...
data.vsphere_datastore.ds: Refreshing state...

------------------------------------------------------------------------

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
 
 *** Note: truncated output ***
         
Plan: 16 to add, 0 to change, 0 to destroy.

------------------------------------------------------------------------

Note: You didn't specify an "-out" parameter to save this plan, so Terraform
can't guarantee that exactly these actions will be performed if
"terraform apply" is subsequently run.
```

And finally, let's apply the configurations

```bash
cisco@wauterw-ubuntu-desktop:~/software/Terraform/ACI-vSphere$ terraform apply
data.vsphere_datacenter.dc: Refreshing state...
data.vsphere_datastore.ds: Refreshing state...
data.vsphere_compute_cluster.cl: Refreshing state...
data.vsphere_virtual_machine.template: Refreshing state...

*** Note: truncated output ***

aci_tenant.Tenant_TF: Creating...
aci_tenant.Tenant_TF: Creation complete after 1s [id=uni/tn-Tenant_Terraform_ACI_Vsphere_demo]
aci_contract.Contract_EPG1_EPG2: Creating...

*** Note: truncated output ***

vsphere_virtual_machine.aci_vm2[0]: Creation complete after 6m25s [id=423c297e-9f95-6f53-6c49-2f4fc41e3e16]

Apply complete! Resources: 16 added, 0 changed, 0 destroyed.
```

You will notice now that some objects are created on ACI and on vSphere. Below, I have added some screenshots.

First, you will see that the ACI tenant got created and that it has a VRF, a BD and 2 EPG's.

![Tenat](/images/2019-12-03-1.png)

And you will also notice that it has a VRF, a BD and 2 EPG's.

![VRF and BD](/images/2019-12-03-2.png)

Drilling into the Tenant, you will see the two EPGS:
  
![EPGs](/images/2019-12-03-3.png)

And here you will see the two EPGs with the contracts in between.

![EPGs](/images/2019-12-03-4.png)

On vSphere, you will see our two VM instances:

![EPGs](/images/2019-12-03-5.png)

And on vSphere you will see the following network configuration.

![vSphere network](/images/2019-12-03-6.png)

And because we have an ICMP contract between both EPGs, we can also verify if the ping works.

![vSphere network](/images/2019-12-03-7.png)

The source code for this example can be found at [Github](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/master/ACI_vSphere_Terraform)

