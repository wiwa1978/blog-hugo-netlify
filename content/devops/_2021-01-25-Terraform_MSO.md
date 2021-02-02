---
title: Terraform with Cisco MSO
date: 2021-01-25T14:39:50+01:00
draft: True
categories:
  - DevOps
  - All
tags:
  - Private Cloud
  - Terraform
  - ACI
---

### Introduction


### Terraform code


```terraform
terraform {
  required_providers {
    mso = {
      source  = "ciscodevnet/mso"
      version = "0.1.5"
    }
  }
}

provider "mso" {
  username = var.username
  password = var.password
  url      = var.url
  insecure = true
}

```
Next, we will create a main.tf file which contains the information we want to configure on ACI.

```terraform
# Remember to run Terraform with a tag -parallelism=1
data "mso_site" "site1" {
  name  = var.site1
}

resource "mso_tenant" "tenant1" {
  name         = var.tenant1
  display_name = var.tenant1
  description  = "Created by Terraform!"
  site_associations {
    site_id = data.mso_site.site1.id
  }
}

resource "mso_schema" "schema1" {
  name          = var.schema1
  template_name = var.template1
  tenant_id     = mso_tenant.tenant1.id
}

resource "mso_schema_site" "site1" {
  schema_id  = mso_schema.schema1.id
  site_id  = data.mso_site.site1.id
  template_name  = var.template1
}

resource "mso_schema_template_anp" "ap1" {
  schema_id    = mso_schema.schema1.id
  template     = var.template1
  name         = var.ap1
  display_name = var.ap1
}

resource "mso_schema_template_anp" "ap2" {
  schema_id    = mso_schema.schema1.id
  template     = var.template1
  name         = var.ap2
  display_name = var.ap2
}

resource "mso_schema_template_vrf" "vrf1" {
  schema_id    = mso_schema.schema1.id
  template     = var.template1
  name         = var.vrf1
  display_name = var.vrf1
  layer3_multicast= true
  vzany           = true
}

resource "mso_schema_template_bd" "bd1" {
  schema_id     = mso_schema.schema1.id
  template_name = var.template1
  name          = var.bd1
  display_name  = var.bd1
  vrf_name      = mso_schema_template_vrf.vrf1.name
}

resource "mso_schema_template_anp_epg" "epg1" {
  schema_id     = mso_schema.schema1.id
  template_name = var.template1
  anp_name      = mso_schema_template_anp.ap1.name
  name          = var.epg1
  bd_name       = mso_schema_template_bd.bd1.name
  vrf_name      = mso_schema_template_vrf.vrf1.name
  display_name  = var.epg1
}

resource "mso_schema_template_anp_epg" "epg2" {
  schema_id     = mso_schema.schema1.id
  template_name = var.template1
  anp_name      = mso_schema_template_anp.ap2.name
  name          = var.epg2
  bd_name       = mso_schema_template_bd.bd1.name
  vrf_name      = mso_schema_template_vrf.vrf1.name
  display_name  = var.epg2
}


resource "mso_schema_template_contract" "contract1" {
  schema_id     = mso_schema.schema1.id
  template_name = var.template1
  contract_name = var.contract_allow_all
  display_name  = var.contract_allow_all
  filter_type   = "bothWay"
  scope         = "context"
  filter_relationships = {
    filter_schema_id     = mso_schema.schema1.id
    filter_template_name = var.template1
    filter_name          = mso_schema_template_filter_entry.any.name
  }
  directives = ["none", "log"]
}


resource "mso_schema_template_filter_entry" "any" {
  schema_id          = mso_schema.schema1.id
  template_name      = var.template1
  name               = var.filter_any
  display_name       = var.filter_any
  entry_name         = "any"
  entry_display_name = "any"
  ether_type         = "ipv4"
  ip_protocol        = "tcp"
  destination_from   = "unspecified"
  destination_to     = "unspecified"
  source_from        = "unspecified"
  source_to          = "unspecified"
  arp_flag           = "unspecified"
}

resource "mso_schema_template_anp_epg_contract" "epg1_any" {
  schema_id         = mso_schema.schema1.id
  template_name     = var.template1
  anp_name          = mso_schema_template_anp.ap1.name
  epg_name          = mso_schema_template_anp_epg.epg1.name
  contract_name     = mso_schema_template_contract.contract1.contract_name
  relationship_type = "provider"
}

resource "mso_schema_template_anp_epg_contract" "epg2_any" {
  schema_id         = mso_schema.schema1.id
  template_name     = var.template1
  anp_name          = mso_schema_template_anp.ap2.name
  epg_name          = mso_schema_template_anp_epg.epg2.name
  contract_name     = mso_schema_template_contract.contract1.contract_name
  relationship_type = "consumer"
}

output "mso_tenant_name" {
  value = mso_tenant.tenant1.name
}

output "mso_schema_name" {
  value = mso_schema.schema1.name
}

output "mso_schema_template" {
  value = mso_schema.schema1.template_name
}

output "mso_schema_template_vrf" {
  value = mso_schema_template_vrf.vrf1.name
}

output "mso_schema_template_bd" {
  value = mso_schema_template_bd.bd1.name
}

output "mso_schema_template_anp1" {
  value = mso_schema_template_anp.ap1.name
}

output "mso_schema_template_anp2" {
  value = mso_schema_template_anp.ap2.name
}

output "mso_schema_template_anp_epg1" {
  value = mso_schema_template_anp_epg.epg1.name
}

output "mso_schema_template_anp_epg2" {
  value = mso_schema_template_anp_epg.epg2.name
}

output "mso_schema_template_contract" {
  value = mso_schema_template_contract.contract1.id
}

output "mso_schema_template_filter_entry" {
  value = mso_schema_template_filter_entry.any.id
}

output "mso_schema_template_anp_epg_contract_1" {
  value = mso_schema_template_anp_epg_contract.epg1_any.id
}

output "mso_schema_template_anp_epg_contract_2" {
  value = mso_schema_template_anp_epg_contract.epg2_any.id
}
```




```terraform
variable "username" {
  default = "***"
}
variable "password" {
  default = "***"
}
variable "url" {
  default = "https://10.48.109.10"
}
variable "site1" {
  default = "SaS_APIC"
}
variable "tenant1" {
  default = "tn-mso-wim"
}
variable "schema1" {
  default = "sch-mso-wim"
}
variable "template1" {
  default = "tmpl-mso-wim"
}
variable "ap1" {
  default = "app-mso-app1"
}
variable "ap2" {
  default = "app-mso-app2"
}
variable "epg1" {
  default = "epg-mso-1"
}
variable "epg2" {
  default = "epg-mso-2"
}
variable "vrf1" {
  default = "vrf-mso"
}
variable "bd1" {
  default = "bd-mso"
}
variable "contract_allow_all" {
  default = "ctr-mso-allow-all"
}
variable "filter_any" {
  default = "flt-any"
}
```


### Deploy infrastructure


```bash
~/Terraform/MSO main ❯ terraform init

Initializing the backend...

Initializing provider plugins...
- Finding ciscodevnet/mso versions matching "0.1.5"...
- Installing ciscodevnet/mso v0.1.5...
- Installed ciscodevnet/mso v0.1.5 (signed by a HashiCorp partner, key ID 433649E2C56309DE)

Partner and community providers are signed by their developers.
If you'd like to know more about provider signing, you can read about it here:
https://www.terraform.io/docs/plugins/signing.html

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```


```bash
~Terraform/MSO main ❯ terraform plan -parallelism=1
Refreshing Terraform state in-memory prior to plan...
The refreshed state will be used to calculate this plan, but will not be
persisted to local or remote state storage.

data.mso_site.site1: Refreshing state...

------------------------------------------------------------------------

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # mso_schema.schema1 will be created
  + resource "mso_schema" "schema1" {
      + id            = (known after apply)
      + name          = "sch-mso-wim"
      + template_name = "tmpl-mso-wim"
      + tenant_id     = (known after apply)
    }

  # mso_schema_site.site1 will be created
  + resource "mso_schema_site" "site1" {
      + id            = (known after apply)
      + schema_id     = (known after apply)
      + site_id       = "5f992fdd1301000d011168f6"
      + template_name = "tmpl-mso-wim"
    }

  # mso_schema_template_anp.ap1 will be created
  + resource "mso_schema_template_anp" "ap1" {
      + display_name = "app-mso-app1"
      + id           = (known after apply)
      + name         = "app-mso-app1"
      + schema_id    = (known after apply)
      + template     = "tmpl-mso-wim"
    }

  # mso_schema_template_anp.ap2 will be created
  + resource "mso_schema_template_anp" "ap2" {
      + display_name = "app-mso-app2"
      + id           = (known after apply)
      + name         = "app-mso-app2"
      + schema_id    = (known after apply)
      + template     = "tmpl-mso-wim"
    }

  # mso_schema_template_anp_epg.epg1 will be created
  + resource "mso_schema_template_anp_epg" "epg1" {
      + anp_name                   = "app-mso-app1"
      + bd_name                    = "bd-mso"
      + bd_schema_id               = (known after apply)
      + bd_template_name           = (known after apply)
      + display_name               = "epg-mso-1"
      + id                         = (known after apply)
      + intersite_multicast_source = false
      + intra_epg                  = (known after apply)
      + name                       = "epg-mso-1"
      + preferred_group            = (known after apply)
      + proxy_arp                  = false
      + schema_id                  = (known after apply)
      + template_name              = "tmpl-mso-wim"
      + useg_epg                   = (known after apply)
      + vrf_name                   = "vrf-mso"
      + vrf_schema_id              = (known after apply)
      + vrf_template_name          = (known after apply)
    }

  # mso_schema_template_anp_epg.epg2 will be created
  + resource "mso_schema_template_anp_epg" "epg2" {
      + anp_name                   = "app-mso-app2"
      + bd_name                    = "bd-mso"
      + bd_schema_id               = (known after apply)
      + bd_template_name           = (known after apply)
      + display_name               = "epg-mso-2"
      + id                         = (known after apply)
      + intersite_multicast_source = false
      + intra_epg                  = (known after apply)
      + name                       = "epg-mso-2"
      + preferred_group            = (known after apply)
      + proxy_arp                  = false
      + schema_id                  = (known after apply)
      + template_name              = "tmpl-mso-wim"
      + useg_epg                   = (known after apply)
      + vrf_name                   = "vrf-mso"
      + vrf_schema_id              = (known after apply)
      + vrf_template_name          = (known after apply)
    }

  # mso_schema_template_anp_epg_contract.epg1_any will be created
  + resource "mso_schema_template_anp_epg_contract" "epg1_any" {
      + anp_name               = "app-mso-app1"
      + contract_name          = "ctr-mso-allow-all"
      + contract_schema_id     = (known after apply)
      + contract_template_name = (known after apply)
      + epg_name               = "epg-mso-1"
      + id                     = (known after apply)
      + relationship_type      = "provider"
      + schema_id              = (known after apply)
      + template_name          = "tmpl-mso-wim"
    }

  # mso_schema_template_anp_epg_contract.epg2_any will be created
  + resource "mso_schema_template_anp_epg_contract" "epg2_any" {
      + anp_name               = "app-mso-app2"
      + contract_name          = "ctr-mso-allow-all"
      + contract_schema_id     = (known after apply)
      + contract_template_name = (known after apply)
      + epg_name               = "epg-mso-2"
      + id                     = (known after apply)
      + relationship_type      = "consumer"
      + schema_id              = (known after apply)
      + template_name          = "tmpl-mso-wim"
    }

  # mso_schema_template_bd.bd1 will be created
  + resource "mso_schema_template_bd" "bd1" {
      + dhcp_policy            = (known after apply)
      + display_name           = "bd-mso"
      + id                     = (known after apply)
      + intersite_bum_traffic  = (known after apply)
      + layer2_stretch         = (known after apply)
      + layer2_unknown_unicast = (known after apply)
      + layer3_multicast       = (known after apply)
      + name                   = "bd-mso"
      + optimize_wan_bandwidth = (known after apply)
      + schema_id              = (known after apply)
      + template_name          = "tmpl-mso-wim"
      + vrf_name               = "vrf-mso"
      + vrf_schema_id          = (known after apply)
      + vrf_template_name      = (known after apply)
    }

  # mso_schema_template_contract.contract1 will be created
  + resource "mso_schema_template_contract" "contract1" {
      + contract_name        = "ctr-mso-allow-all"
      + directives           = [
          + "none",
          + "log",
        ]
      + display_name         = "ctr-mso-allow-all"
      + filter_relationships = (known after apply)
      + filter_type          = "bothWay"
      + id                   = (known after apply)
      + schema_id            = (known after apply)
      + scope                = "context"
      + template_name        = "tmpl-mso-wim"
    }

  # mso_schema_template_filter_entry.any will be created
  + resource "mso_schema_template_filter_entry" "any" {
      + arp_flag             = "unspecified"
      + destination_from     = "unspecified"
      + destination_to       = "unspecified"
      + display_name         = "flt-any"
      + entry_description    = (known after apply)
      + entry_display_name   = "any"
      + entry_name           = "any"
      + ether_type           = "ipv4"
      + id                   = (known after apply)
      + ip_protocol          = "tcp"
      + match_only_fragments = (known after apply)
      + name                 = "flt-any"
      + schema_id            = (known after apply)
      + source_from          = "unspecified"
      + source_to            = "unspecified"
      + stateful             = (known after apply)
      + tcp_session_rules    = (known after apply)
      + template_name        = "tmpl-mso-wim"
    }

  # mso_schema_template_vrf.vrf1 will be created
  + resource "mso_schema_template_vrf" "vrf1" {
      + display_name     = "vrf-mso"
      + id               = (known after apply)
      + layer3_multicast = true
      + name             = "vrf-mso"
      + schema_id        = (known after apply)
      + template         = "tmpl-mso-wim"
      + vzany            = true
    }

  # mso_tenant.tenant1 will be created
  + resource "mso_tenant" "tenant1" {
      + description  = "Created by Terraform!"
      + display_name = "tn-mso-wim"
      + id           = (known after apply)
      + name         = "tn-mso-wim"

      + site_associations {
          + aws_access_key_id         = (known after apply)
          + aws_account_id            = (known after apply)
          + aws_secret_key            = (known after apply)
          + azure_access_type         = (known after apply)
          + azure_active_directory_id = (known after apply)
          + azure_application_id      = (known after apply)
          + azure_client_secret       = (known after apply)
          + azure_shared_account_id   = (known after apply)
          + azure_subscription_id     = (known after apply)
          + is_aws_account_trusted    = (known after apply)
          + site_id                   = "5f992fdd1301000d011168f6"
          + vendor                    = (known after apply)
        }

      + user_associations {
          + user_id = (known after apply)
        }
    }

Plan: 13 to add, 0 to change, 0 to destroy.

------------------------------------------------------------------------

Note: You didn't specify an "-out" parameter to save this plan, so Terraform
can't guarantee that exactly these actions will be performed if
"terraform apply" is subsequently run.
```
Next, let’s apply the configuration. You should know the drill by now as we did it many times in previous blog posts already.

```bash
~/Terraform/MSO main ❯ terraform apply --auto-approve -parallelism=1
data.mso_site.site1: Refreshing state...
mso_tenant.tenant1: Creating...
mso_tenant.tenant1: Creation complete after 0s [id=6019519d6e00001a273512d7]
mso_schema.schema1: Creating...
mso_schema.schema1: Creation complete after 0s [id=6019519d6e000020273512d8]
mso_schema_template_anp.ap1: Creating...
mso_schema_template_anp.ap1: Creation complete after 1s [id=app-mso-app1]
mso_schema_template_vrf.vrf1: Creating...
mso_schema_template_vrf.vrf1: Creation complete after 0s [id=vrf-mso]
mso_schema_template_filter_entry.any: Creating...
mso_schema_template_filter_entry.any: Creation complete after 0s [id=any]
mso_schema_template_anp.ap2: Creating...
mso_schema_template_anp.ap2: Creation complete after 0s [id=app-mso-app2]
mso_schema_site.site1: Creating...
mso_schema_site.site1: Creation complete after 0s [id=5f992fdd1301000d011168f6]
mso_schema_template_bd.bd1: Creating...
mso_schema_template_bd.bd1: Creation complete after 0s [id=bd-mso]
mso_schema_template_contract.contract1: Creating...
mso_schema_template_contract.contract1: Creation complete after 1s [id=ctr-mso-allow-all]
mso_schema_template_anp_epg.epg1: Creating...
mso_schema_template_anp_epg.epg1: Creation complete after 0s [id=epg-mso-1]
mso_schema_template_anp_epg.epg2: Creating...
mso_schema_template_anp_epg.epg2: Creation complete after 0s [id=epg-mso-2]
mso_schema_template_anp_epg_contract.epg1_any: Creating...
mso_schema_template_anp_epg_contract.epg1_any: Creation complete after 0s [id=ctr-mso-allow-all]
mso_schema_template_anp_epg_contract.epg2_any: Creating...
mso_schema_template_anp_epg_contract.epg2_any: Creation complete after 0s [id=ctr-mso-allow-all]

Apply complete! Resources: 13 added, 0 changed, 0 destroyed.

Outputs:

mso_schema_name = sch-mso-wim
mso_schema_template = tmpl-mso-wim
mso_schema_template_anp1 = app-mso-app1
mso_schema_template_anp2 = app-mso-app2
mso_schema_template_anp_epg1 = epg-mso-1
mso_schema_template_anp_epg2 = epg-mso-2
mso_schema_template_anp_epg_contract_1 = ctr-mso-allow-all
mso_schema_template_anp_epg_contract_2 = ctr-mso-allow-all
mso_schema_template_bd = bd-mso
mso_schema_template_contract = ctr-mso-allow-all
mso_schema_template_filter_entry = any
mso_schema_template_vrf = vrf-mso
mso_tenant_name = tn-mso-wim
```

Log into Cisco MSO and see that the tenant is created there:

![ACI](/images/2021-01-25-1.png)

Also the Schema was created:

![ACI](/images/2021-01-25-2.png)

Also the VRF and the BD got created as you can see in below screenshot (see the left colum):

![ACI](/images/2021-01-25-3.png)


### Destroying the infrastructure

```bash
~/Terraform/MSO main ❯ terraform destroy -parallelism=1 -auto-approve
data.mso_site.site1: Refreshing state... [id=5f992fdd1301000d011168f6]
mso_tenant.tenant1: Refreshing state... [id=6019519d6e00001a273512d7]
mso_schema.schema1: Refreshing state... [id=6019519d6e000020273512d8]
mso_schema_site.site1: Refreshing state... [id=5f992fdd1301000d011168f6]
mso_schema_template_filter_entry.any: Refreshing state... [id=any]
mso_schema_template_vrf.vrf1: Refreshing state... [id=vrf-mso]
mso_schema_template_anp.ap2: Refreshing state... [id=app-mso-app2]
mso_schema_template_anp.ap1: Refreshing state... [id=app-mso-app1]
mso_schema_template_contract.contract1: Refreshing state... [id=ctr-mso-allow-all]
mso_schema_template_bd.bd1: Refreshing state... [id=bd-mso]
mso_schema_template_anp_epg.epg1: Refreshing state... [id=epg-mso-1]
mso_schema_template_anp_epg.epg2: Refreshing state... [id=epg-mso-2]
mso_schema_template_anp_epg_contract.epg1_any: Refreshing state... [id=ctr-mso-allow-all]
mso_schema_template_anp_epg_contract.epg2_any: Refreshing state... [id=ctr-mso-allow-all]
mso_schema_site.site1: Destroying... [id=5f992fdd1301000d011168f6]
mso_schema_site.site1: Destruction complete after 0s
mso_schema_template_anp_epg_contract.epg2_any: Destroying... [id=ctr-mso-allow-all]
mso_schema_template_anp_epg_contract.epg2_any: Destruction complete after 0s
mso_schema_template_anp_epg_contract.epg1_any: Destroying... [id=ctr-mso-allow-all]
mso_schema_template_anp_epg_contract.epg1_any: Destruction complete after 0s
mso_schema_template_anp_epg.epg2: Destroying... [id=epg-mso-2]
mso_schema_template_anp_epg.epg2: Destruction complete after 0s
mso_schema_template_anp_epg.epg1: Destroying... [id=epg-mso-1]
mso_schema_template_anp_epg.epg1: Destruction complete after 0s
mso_schema_template_contract.contract1: Destroying... [id=ctr-mso-allow-all]
mso_schema_template_contract.contract1: Destruction complete after 0s
mso_schema_template_anp.ap2: Destroying... [id=app-mso-app2]
mso_schema_template_anp.ap2: Destruction complete after 0s
mso_schema_template_bd.bd1: Destroying... [id=bd-mso]
mso_schema_template_bd.bd1: Destruction complete after 0s
mso_schema_template_anp.ap1: Destroying... [id=app-mso-app1]
mso_schema_template_anp.ap1: Destruction complete after 0s
mso_schema_template_filter_entry.any: Destroying... [id=any]
mso_schema_template_filter_entry.any: Destruction complete after 0s
mso_schema_template_vrf.vrf1: Destroying... [id=vrf-mso]
mso_schema_template_vrf.vrf1: Destruction complete after 0s
mso_schema.schema1: Destroying... [id=6019519d6e000020273512d8]
mso_schema.schema1: Destruction complete after 0s
mso_tenant.tenant1: Destroying... [id=6019519d6e00001a273512d7]
mso_tenant.tenant1: Destruction complete after 1s

Destroy complete! Resources: 13 destroyed.
```

Code can be found [here](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/main/Terraform/MSO)