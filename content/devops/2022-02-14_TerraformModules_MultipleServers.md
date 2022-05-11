---
title: DigitalOcean - Terraform Modules by example (2)
date: 2022-02-14T14:39:50+01:00
draft: False
categories:
  - DevOps
  - Infrastructure As Code
  - Public Cloud
  - All
tags:
  - Terraform
  - DigitalOcean
---

### Introduction

In [this](https://blog.wimwauters.com/devops/2022-02-12_terraformmodules_singleserver/) post, we described how to create a droplet, project, tag and DNS record on DigitalOcean using Terraform. We've shown a regular Terraform script (cfr. Create a single droplet) and we have also show how to do this using modules (cfr. Create a single droplet using Terraform modules). However in that post, we limited ourselves to creating a single droplet, assigned it to a single project and created a single DNS record for that droplet. In the upcoming post, we'll see how to make it work if we want to create multiple droplets.

### Create multiple droplets using regular Terraform files

Imagine we have the following variable 'servers':

```hcl
servers = {
  server1 = {
    size   = "s-2vcpu-2gb"
    image  = "ubuntu-21-10-x64"
    region = "ams3",
    tags   = ["web", "development"]
  },
  server2 = {
    size   = "s-2vcpu-2gb"
    image  = "ubuntu-20-04-x64"
    region = "lon1",
    tags   = ["web", "staging"]
  }
}
```

We want to create a droplet for each of these entries. We could simply achieve this with a `for_each` loop as follows:

```hcl
resource "digitalocean_droplet" "server" {
  for_each = var.servers

  name   = each.key
  image  = each.value.image
  size   = each.value.size
  region = each.value.region
  ssh_keys = [
    data.digitalocean_ssh_key.terraform.id
  ]
  tags = each.value.tags
}
```

### Using provisioners

As explained in this post, when we want to install some software onto each of these servers. We could do so by using provisioners.

```hcl
  # Create directories for deployment scripts
  provisioner "remote-exec" {
    inline = [
      "mkdir -p  /tmp/scripts/",
    ]

    connection {
      type        = "ssh"
      user        = "root"
      private_key = file("ssh_keys/${var.ssh_key}")
      host        = digitalocean_droplet.server[each.key].ipv4_address
    }
  }

  # Copy  Scripts
  provisioner "file" {
    source      = "${local.script_directory}/"
    destination = "/tmp/scripts/"

    connection {
      type        = "ssh"
      user        = "root"
      private_key = file("ssh_keys/${var.ssh_key}")
      host        = digitalocean_droplet.server[each.key].ipv4_address
    }
  }

  provisioner "remote-exec" {
    inline = [
      "bash /tmp/scripts/software_install.sh"
    ]

    connection {
      type        = "ssh"
      user        = "root"
      private_key = file("ssh_keys/${var.ssh_key}")
      host        = digitalocean_droplet.server[each.key].ipv4_address
    }
  }
```

The important piece here is the statement `host = digitalocean_droplet.server[each.key].ipv4_address`. We are addressing the current server through the `[each_key]`. When iterating this will become digitalocean_droplet.server["server1"] and digitalocean_droplet.server["server2"]. As such, we assign the IP address to the host attribute of the connection.

However, when we execute this code (e.g. terraform plan), you will notice we receive the following error:

> Error: Cycle: digitalocean_droplet.server["server2"], digitalocean_droplet.server["server1"]

The problem here is that we are creating a cyclic dependency because we are referring a resource by its name within its own block. This issue (and solution as a matter of fact) is explained [here](https://www.terraform.io/language/resources/provisioners/connection#the-self-object). So the solution to our little problem here is to use:

```hcl
host        = self.ipv4_address
```

instead of

```hcl
host        = digitalocean_droplet.server[each.key].ipv4_address
```

### Create multiple droplets using modules

Let's now see how we can achieve the same using Terraform modules. We have covered this already in part 1 of course but in what follows we will focus on creating multiple droplets (rather than a single droplet as we did in [part 1](https://blog.wimwauters.com/devops/2022-02-12_terraformmodules_singleserver/)).

In essence, we just have to call the module multiple times. We can do this using a `for_each` loop (see example below) or using a `count` variable. In below example, we are simply looping over the servers variable while assigning the values to the attributes using the `each.value` construct.

Then for the DNS records, we are looping over the created droplets using a count variable. Each element is addressed with the `count.index`.

```hcl
module "ubuntu-server" {
    source = "./modules/server"

    for_each = var.servers

    name           =       each.value.name
    image          =       each.value.image
    environment    =       each.value.environment
    tag            =       each.key
    domain_name    =       var.domain_name
    region         =       each.value.region
    ssh_key        =       var.ssh_key
}

module "terraform-project" {
    source = "./modules/project"

    project_name   =       var.project_name
    resources      =       values(module.ubuntu-server)[*].droplet_urn
}

module "server-record" {
    source = "./modules/record"

    count = length(module.ubuntu-server)

    domain_name    =       var.domain_name
    name           =       values(module.ubuntu-server)[count.index].droplet_name
    value          =       values(module.ubuntu-server)[count.index].droplet_ip_address
}
```
