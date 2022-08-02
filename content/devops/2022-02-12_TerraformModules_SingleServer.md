---
title: DigitalOcean - Terraform Modules by example (1)
date: 2022-02-12T14:39:50+01:00
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

In this post, we will create a droplet on DigitalOcean using Terraform onto which we automatically want to install some software packages. Nothing new in any case as we spend a [blog post](https://blog.wimwauters.com/devops/2019-09-02_digitalocean_terraform/) on this topic already. We'll approach it a bit different though: first we will be using a normal Terraform script which we will then improve slightly by using the concept of [Terraform modules](https://www.terraform.io/language/modules/syntax).

### Create a single droplet

Creating a single droplet onto DigitalOcean is very straightforward. Have a look at the [docs](https://registry.terraform.io/providers/digitalocean/digitalocean/latest/docs/resources/droplet) to get an understanding of what is exactly required.

Additionally we will also create a DigitalOcean [project](https://registry.terraform.io/providers/digitalocean/digitalocean/latest/docs/resources/project), a [tag](https://registry.terraform.io/providers/digitalocean/digitalocean/latest/docs/resources/tag) and assign also a [DNS record](https://registry.terraform.io/providers/digitalocean/digitalocean/latest/docs/resources/record) to that droplet.

The main code to achieve this is as follows:

```hcl
data "digitalocean_ssh_key" "terraform" {
  name = var.ssh_key
}

data "digitalocean_domain" "server" {
  name = var.domain_name
}

resource "digitalocean_project" "terraform_project" {
  name        = var.project_name
  environment = "Development"
  resources   = [digitalocean_droplet.server.urn]
}

resource "digitalocean_record" "www" {
  domain    = data.digitalocean_domain.server.id
  type      = "A"
  name      = digitalocean_droplet.server.name
  value     = digitalocean_droplet.server.ipv4_address
}

resource "digitalocean_tag" "webserver" {
    name = "web"
}

resource "digitalocean_droplet" "server" {
    name    = var.name
    image   = var.image
    size    = "s-2vcpu-2gb"
    region  = var.region
    ssh_keys = [
      data.digitalocean_ssh_key.terraform.id
    ]
    tags   = [digitalocean_tag.webserver.id, var.tag]
}
```

Nothing really very complex but two 'special' things that might require a bit of explanation:

- during droplet creation we need to specify an SSH key. I have already an SSH key setup onto DigitalOcean which is why I'm reading it in through a data variable
- we need to create the DNS record under a given domain name. I have already a domain name configured onto DigitalOcean which is why I'm reading it in through a data variable.

### Using provisioners

As mentioned before, I also want to install some software onto the newly created droplet. This is pretty easy with Terraform using remote [provisioners](https://www.terraform.io/language/resources/provisioners/syntax). Therefore I created a script called `software_install.sh` under a `scripts' directory which contains some very basic install commands for some software packages.

The script is simply performing some apt updates and installs net-tools as well as nginx.

```hcl
#!/bin/env bash

#export DEBIAN_FRONTEND=noninteractive
echo "Configuring users"
sudo adduser ubuntu --gecos "First Last,RoomNumber,WorkPhone,HomePhone" --disabled-password
echo "ubuntu:ubuntu" | sudo chpasswd

echo 'Performing apt updates ...'
sudo apt -y update
echo 'Installing net-tools ...'
sudo apt install -y net-tools
echo 'Installing nginx ...'
sudo apt install -y nginx
```

Next, within the droplet resource block copy/paste below snippet.

```hcl
  # Create folder
  provisioner "remote-exec" {
    inline = [
      "mkdir -p  /tmp/scripts/",
    ]

    connection {
      type        = "ssh"
      user        = "root"
      private_key = file("ssh_keys/${var.ssh_key}")
      host        = digitalocean_droplet.server.ipv4_address
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
      host        = digitalocean_droplet.server.ipv4_address
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
      host        = digitalocean_droplet.server.ipv4_address
    }
  }
```

First, we are creating the `/tmp/scripts` folder on the droplet, next we are copying the files from our local source to that newly created scripts folder on the droplet and finally we are executing this bash script. Note that each time, we use an SSH connection into the server using the SSH key from the `ssh_keys` folder.

### Check out the results

After the usual `terraform init`, `terraform plan` and `terraform apply` rouutine, go ahead and check the DigitalOcean console. A new droplet will have been created under the `Project Terraform`.

![Terraform_DO](/images/2022-02-12-1.png)

As we also installed NGINX on the droplet, we should be able to see the default nginx webpage. Let's check that out also!

![Terraform_DO](/images/2022-02-12-2.png)

In case the above has been a bit confusing, find the code for this part [here](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/main/Terraform/DigitalOcean-Terraform-SingleServer).

### Create a single droplet using Terraform modules

While the above works perfectly well, it's not really according to Terraform's best practises. A better way would be to create re-usable [modules](https://www.terraform.io/language/modules/syntax). Let's go ahead and do just that. A module in fact is nothing else than just a collection of Terraform files under a specific `modules` directory.

We create a folder `modules\server` and this contains the following files:

- main.tf: contains the main code for the module, essentially the same as what we have shown in the first two sections of this blog post
- variables.tf: contains a list of variables that need to be passed to the module
- versions.tf: contains the Terraform provider
- outputs.tf: while optional, it contains the outputs from our module

##### Calling our module

Just create another file under the root containing the main Terraform script. We will call this `main.tf` as well. As you can see in below snippet, instead of creating a resource, we simply call the `server` module and we pass along the variables that we are reading in from our variables file.

```hcl
module "ubuntu-server" {
    source = "./modules/server"

    name           =       var.name
    image          =       var.image
    environment    =       var.environment
    tag            =       var.tag
    domain_name    =       var.domain_name
    region         =       var.region
    ssh_key        =       var.ssh_key
    project_name   =       var.project_name
}
```

This will essentially call the module it finds in the folder `modules/server` and execute the Terraform files it finds there.

##### Using outputs with modules

Remember in our module folder we had an output.tf file containing the following code:

```hcl
output "droplet_ip_address" {
  value = digitalocean_droplet.server.ipv4_address
}
```

This was essentially outputting the IP address of our newly created droplet. Now, we want to use this from the main file under the root (where we call the module). We can do this using the following code

```hcl
output "droplet_ip_address" {
  value = module.ubuntu-server.droplet_ip_address
}
```

You notice here that we refer to the output value of the module itself.

The code for this part can be found [here](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/main/Terraform/DigitalOcean-Terraform-SingleServer-Modules) in case you want to follow along.

### Improving the Terraform module

In the last section, we have created a module for a server. When you look into this module, you'll certainly have noticed that the module is responsible for - each time it's being called - creating a project, a DNS record and a droplet. While this might be desired behaviour it might just not be what you want to achieve. Why would you create a new project each time you create a droplet? And do you really want to create a DNS record each time a droplet is created? The answer to such questions determine what resources you combine in a module.

As this is merely an example, we will create three modules now: server, project and record (each having its own folder under the modules directory). The code for each is as follows:

**For project (/modules/project)**:

```hcl
resource "digitalocean_project" "terraform_project" {
  name        = var.project_name
  resources   = var.resources
}
```

**For record (/modules/record)**:

```hcl
data "digitalocean_domain" "server" {
  name = var.domain_name
}

resource "digitalocean_record" "www" {
  domain    = data.digitalocean_domain.server.id
  type      = "A"
  name      = var.name
  value     = var.value
}
```

**For server (/modules/server)**:

```hcl
data "digitalocean_ssh_key" "terraform" {
  name = var.ssh_key
}

resource "digitalocean_droplet" "server" {
    name    = var.name
    image   = var.image
    size    = "s-2vcpu-2gb"
    region  = var.region
    ssh_keys = [
      data.digitalocean_ssh_key.terraform.id
    ]
    tags   = [digitalocean_tag.webserver.id, var.tag]
}

resource "digitalocean_tag" "webserver" {
    name = "web"
}
```

Within the main Terraform file under the root directory we combine the modules we want to. In our example, we want to create a single droplet, associate a DNS record to that server and assign the server to the project so we'll go ahead and just call these modules passing along the various variables. It looks as follows:

```hcl
module "ubuntu-server" {
    source = "./modules/server"

    name           =       var.name
    image          =       var.image
    environment    =       var.environment
    tag            =       var.tag
    domain_name    =       var.domain_name
    region         =       var.region
    ssh_key        =       var.ssh_key
}

module "terraform-project" {
    source = "./modules/project"

    project_name   =       var.project_name
    resources      =       [module.ubuntu-server.droplet_urn]
}

module "server-record" {
    source = "./modules/record"

    domain_name    =       var.domain_name
    name           =       module.ubuntu-server.droplet_name
    value          =       module.ubuntu-server.droplet_ip_address
}

```

The code for this part can be found [here](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/main/Terraform/DigitalOcean-Terraform-SingleServer-Modules-Improved).
