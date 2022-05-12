---
title: Terraform use cases complex variables
date: 2022-03-01T14:39:50+01:00
draft: False
categories:
  - DevOps
  - All
tags:
  - Terraform
---

### Introduction

As much as I like Terraform, it can be sometimes a nightmare to work with complex variables and loops etc. In this extensive blog post, I have defined a number of use cases to show how some of these more difficult concepts work. They are all based on DigitalOcean but can be easily applied to other cloud providers as well. Most of the inspiration for the use cases can be found in [this](https://blog.wimwauters.com/devops/2022-02-12_terraformmodules_singleserver/) and [this](https://blog.wimwauters.com/devops/2022-02-14_terraformmodules_multipleservers/) post.

### Use Case 1: list of objects with count

**Description**: Create instances on DigitalOcean based on a `list of objects` and using the `count` method

**Code**: See [here](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/main/Terraform/UseCases_Terraform/UseCase1)

**Variables definition**: We will use a list of objects. Each object represents a server and has multiple attributes (e.g. name, size...)

```hcl
variable "servers" {
    type = list(object({
        name  = string,
        size = string,
        image = string,
        region = string
        tags = list(string)
    }))
}
```

**Variables example**: An example of the previous definition would be:

```hcl
servers = [
    {
        name  = "server1",
        size = "s-2vcpu-2gb"
        image = "ubuntu-21-10-x64"
        region = "ams3",
        tags = ["web", "development"]
    },
    {
        name  = "server2",
        size = "s-2vcpu-2gb"
        image = "ubuntu-20-04-x64"
        region = "lon1",
        tags = ["web", "staging"]
    }
]
```

**Main file**: We will use the `count` method to create the servers on DigitalOcean

```hcl
resource "digitalocean_droplet" "server" {
    count = length(var.servers)

    name    =  var.servers[count.index].name
    image   =  var.servers[count.index].image
    size    =  var.servers[count.index].size
    region  =  var.servers[count.index].region
    ssh_keys = [
      data.digitalocean_ssh_key.terraform.id
    ]
    tags   = var.servers[count.index].tags
}

output "droplet_ip_addresses" {
  value = digitalocean_droplet.server[*].ipv4_address
}
```

**Explanation**: In our example, our list counts two servers so we can set the count variable to the length of the list. Next, we assign the attributes using `var.servers[count.index]` (e.g. var.servers[0]). As you already could guess, in the first execution the index will be 0, the next execution the index will be 1. When the count loop has finished (in our example after two iterations), two servers have been created on DigitalOcean. Inside of Terraform, these are known as:

- digitalocean_droplet.server[0]
- digitalocean_droplet.server[1]

In our case, we want to output all of the IP-addresses of the created servers, so we could write it as `digitalocean_droplet.server[*].ipv4_address`. The `[*]` refers to all servers.

### Use Case 2: list of objects with for_each

**Description**: Create instances on DigitalOcean based on a `list of objects` and using the `for_each` method

**Code**: See [here](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/main/Terraform/UseCases_Terraform/UseCase2)

**Variables file**: We will use the same variables file as in use case 1

```hcl
variable "servers" {
    type = list(object({
        name  = string,
        size = string,
        image = string,
        region = string
        tags = list(string)
    }))
}
```

**Variables example**: We will use the same variables example as in use case 1 (again a list of objects1)

```hcl
servers = [
    {
        name  = "server1",
        size = "s-2vcpu-2gb"
        image = "ubuntu-21-10-x64"
        region = "ams3",
        tags = ["web", "development"]
    },
    {
        name  = "server2",
        size = "s-2vcpu-2gb"
        image = "ubuntu-20-04-x64"
        region = "lon1",
        tags = ["web", "staging"]
    }
]
```

**Main file**: We will use the `for_each` method to create the servers on DigitalOcean

```hcl
resource "digitalocean_droplet" "server" {
    for_each = { for server in var.servers : server.name => server }

    name    =  each.key
    image   =  each.value.image
    size    =  each.value.size
    region  =  each.value.region
    ssh_keys = [
      data.digitalocean_ssh_key.terraform.id
    ]
    tags   = each.value.tags
}

output "droplet_ip_addresses" {
  value = values(digitalocean_droplet.server)[*].ipv4_address
}
```

**Explanation**: The `for_each` works with a map. But what we provide is not a map, it's a list. So we need to do some magic to convert that list to a map. For that reason, a `for-loop` is used. I'll try to explain it to the best of my abilities.

As mentioned, `for_each` constructs works with a map. So we will ensure that what comes after the `for_each = ` statement is in fact a map. Hence, we surround the `for-loop` with curly braces. Next we have a `for-loop` which loops over the servers in the servers variable. Each server needs to be uniquely identified by a key (e. in our case we chose the server name 'server.name', assuming we have unique names for the servers in the servers variable) and we assign the object 'servers' (with all the attributes) as a value to the key identified by the name of the server.

With this for-loop, we end up having something like:

```hcl
{ # the curly braces indicating a map structure
  server1 = {  # the key
      size = "s-2vcpu-2gb"  # the attributes of the server object
      image = "ubuntu-21-10-x64"
      region = "ams3",
      tags = ["web", "development"]
  },
  server2 = {
      size = "s-2vcpu-2gb"
      image = "ubuntu-20-04-x64"
      region = "lon1",
      tags = ["web", "staging"]
  }
}
```

and this is a map that is usable with a `for_each` loop.

How do we address the various elements in this map? The answer is that we can use `each.value` to refer to the current server object. Hence, we can use the attribute (e.g name, image) immediately with the `each.value`. `each.key` refers to current key (e.g. server1, server2)

What concerns the output, we use the `values` function (see [docs](https://www.terraform.io/language/functions/values)). The `values` function takes a map and returns a list containing the values of the elements in that map. What does this mean? It means that for `values(digitalocean_droplet.server)` we will get something like the below:

```hcl
[
   {
    size = "s-2vcpu-2gb"
    image = "ubuntu-21-10-x64"
    region = "ams3",
    tags = ["web", "development"]
  },
  {
    size = "s-2vcpu-2gb"
    image = "ubuntu-20-04-x64"
    region = "lon1",
    tags = ["web", "staging"]
  }
]
```

And these elements are in a list so can be referred to with [*] in case we want all servers. This explains why we have written is as `values(digitalocean_droplet.server)[*]`

### Use Case 3: map of objects with count

**Description**: Previous use case (use case 2) was based on a list of objects in combination with the `for_each` method but as you could witness that resulted in some complexities here and there. In this use case, we will create some instances on DigitalOcean based on a map of objects using the `count` method.

**Code**: See [here](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/main/Terraform/UseCases_Terraform/UseCase33)

**Variables definition**: We will use a map of objects. Each object represents a server and has multiple attributes (e.g. name, size...)

```hcl
variable "servers" {
    type = map(object({
        name = string
        size = string,
        image = string,
        region = string
        tags = list(string)
    }))
}
```

**Variables example**: Below is an example variable adhering to the previous definition (note the key/value pairs). We have a variable called 'servers' and it contains a map of servers, each server identified by a key (e.g server1) and an object as defined between the two curly braces.

```hcl
servers = {
  server1 = {
      name = "server1"
      size = "s-2vcpu-2gb"
      image = "ubuntu-21-10-x64"
      region = "ams3",
      tags = ["web", "development"]
  },
  server2 = {
      name = "server2"
      size = "s-2vcpu-2gb"
      image = "ubuntu-20-04-x64"
      region = "lon1",
      tags = ["web", "staging"]
  }
}
```

**Main file**:

```hcl
resource "digitalocean_droplet" "server" {

    count = length(var.servers)

    name    =  values(var.servers)[count.index]["name"]
    image   =  values(var.servers)[count.index]["image"]
    size    =  values(var.servers)[count.index]["size"]
    region  =  values(var.servers)[count.index]["region"]
    ssh_keys = [
      data.digitalocean_ssh_key.terraform.id
    ]
    tags   = var.servers[format("server%s", count.index+1)]["tags"]
}

output "droplet_ip_addresses" {
  value = digitalocean_droplet.server[*].ipv4_address
}
```

**Explanation**: Use case 1 gave an example of how to use the `count` method. The difference though with use case 1 is that now we have a `map` and not a list. Our map contains two servers and we set the count variable to the length of the map. Next, you see that we are using Terraform's `values` function (see [docs](https://www.terraform.io/language/functions/values)). The `values` function takes a map and returns a list containing the values of the elements in that map. What does this mean?

It means that for `values((var.servers)` we will get something like the below:

```hcl
[
   {
    size = "s-2vcpu-2gb"
    image = "ubuntu-21-10-x64"
    region = "ams3",
    tags = ["web", "development"]
  },
  {
    size = "s-2vcpu-2gb"
    image = "ubuntu-20-04-x64"
    region = "lon1",
    tags = ["web", "staging"]
  }
]
```

Since this is a list we can address the individual servers with `values((var.servers)[0]`, `values((var.servers)[1]`. In a loop with count, we use `count.index` to iterate over the servers.

### Use Case 4: map of objects with for_each

**Description**: Create instances on DigitalOcean based on a `map of objects` (not a list) while still using the `for_each` method.

**Code**: See [here](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/main/Terraform/UseCases_Terraform/UseCase4)

**Variables definition**: Note the curly braces in the below definition. Here we expect the variables to be a map of objects, each object having several attributes (e.g. size, image ...)

```hcl
variable "servers" {
    type = map(object({
        size = string,
        image = string,
        region = string
        tags = list(string)
    }))
}
```

**Variables example**: Below is an example variable adhering to the previous definition (note the key/value pairs). We have a variable called 'servers' and it contains a map of servers, each server identified by a key (e.g server1) and an object as defined between the two curly braces.

```hcl
servers = {
  server1 = {
      size = "s-2vcpu-2gb"
      image = "ubuntu-21-10-x64"
      region = "ams3",
      tags = ["web", "development"]
  },
  server2 = {
      size = "s-2vcpu-2gb"
      image = "ubuntu-20-04-x64"
      region = "lon1",
      tags = ["web", "staging"]
  }
}
```

**Main file**: We will use the `for_each` method to create the servers on DigitalOcean

```hcl
resource "digitalocean_droplet" "server" {
    for_each = var.servers

    name    =  each.key
    image   =  each.value.image
    size    =  each.value.size
    region  =  each.value.region
    ssh_keys = [
      data.digitalocean_ssh_key.terraform.id
    ]
    tags   = each.value.tags
}

output "droplet_ip_addresses" {
  value = values(digitalocean_droplet.server)[*].ipv4_address
}
```

**Explanation**: Looping over a map is very straigthforward with `for_each` as we can just loop over the incoming variable. We can then use `each.value` to refer to the current server object. Hence, we can use the attribute (e.g name, image) immediately with the `each.value`. `each.key` refers to current key (e.g. server1, server2)

What concerns the output, we use the `values` function again (see [docs](https://www.terraform.io/language/functions/values)). Let's see why we need to use that function.

Once the servers are created, Terraform provides them as follows:

```hcl
droplets = {
  "server1" = {
    "id" = "297638825"
    "image" = "ubuntu-21-10-x64"
    "ipv4_address" = "164.92.219.154"
    "urn" = "do:droplet:297638825"
    ...
    "vpc_uuid" = "d31d4da8-8b9b-4145-bb87-0b07c285234c"
  }
  "server2" = {
    "id" = "297638826"
    "image" = "ubuntu-20-04-x64"
    "ipv4_address" = "138.68.154.243"
    "urn" = "do:droplet:297638826"
    ...
    "vpc_uuid" = "7fb12eee-afbf-4c8c-b499-fa5c107d9991"
  }
}
```

As explained above already, the `values` function takes a map (which we have indeed as per the above snippet) and returns a list containing the values of the elements in that map. So once the `values` function does its magic, we have a list of server objects so they can be referred to with [*].

### Use Case 5: list of strings with count

**Description**: Create projects on DigitalOcean based on a `list of strings` and using the `count` method

**Code**: See [here](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/main/Terraform/UseCases_Terraform/UseCase5)

**Variables definition**: We will use a list of strings as defined in the below variables definition:

```hcl
variable "projects" {
  type = list(string)
}

```

**Variables example**: Below is an example of a list of strings:

```hcl
projects = ["Project Development", "Project Staging"]
```

**Main file**:

```hcl
resource "digitalocean_project" "project" {
  count = length(var.projects)

  name = var.projects[count.index]
}

output "project_names" {
  value = digitalocean_project.project[*].name
}
```

**Explanation**: This is a very easy use case. We take a list of strings as an input and determine the length of the list. Next, as we did in the previous use cases using the `count` method, we simply loop over the projects and assign the attribute. For the output method, we can simply address the `project` resource as it's returned by Terraform as a list, hence we can display the name with a simple `digitalocean_project.project[*].name` statement.

### Use Case 6: list of strings with for_each

**Description**: Create projects on DigitalOcean based on a `list of strings` and using the `for_each` method

**Code**: See [here](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/main/Terraform/UseCases_Terraform/UseCase6)

**Variables definition**: We will use a list of strings as defined in the below variables definition:

```hcl
variable "projects" {
  type = list(string)
}

```

**Variables example**: Below is an example of a list of strings:

```hcl
projects = ["Project Development", "Project Staging"]
```

**Main file**:

```hcl
resource "digitalocean_project" "project" {
  for_each = toset(var.projects)

  name = each.key
}

output "project_names" {
  value = values(digitalocean_project.project)[*].name
}
```

**Explanation**: Again this is a very easy use case with a little complexity. As we are making use of the `for_each` construct. we can simply loop over the projects. Bit if we would simply use `for_each = toset(var.projects)`, you will get back the following error message:

> The given "for_each" argument value is unsuitable: the "for_each" argument must be a map, or set of strings, and you have provided a value of type list of string.

The issue is that `var.projects` is a list and a for_each only works with maps or a set of strings (not a list of strings which is what we have). Hence we need to convert the `list of strings` to a `set of strings` using the `toset` function (see [docs](https://www.terraform.io/language/functions/toset)) so we are able to loop over the projects.

Note 1: you might wonder why in use case 2 we are not using the `toset` function. The reason is that -in use case 2- we are dealing with a list of objects (not a list of strings) and hence we cannot use the `toset` function in use case 2.

Note 2: this use case is very easy as we are only using a list of strings. Normally a list of objects is what you would use more often so we could also use the same methodology as we did in use case 1 or 2.

- For an example of how we used a `list of objects` with the `count` (method explained in use case 1 but applied to projects), refer to the 'project' resource in the example [here](https://github.com/wiwa1978/blog-hugo-netlify-code/blob/main/Terraform/UseCases_Terraform/UseCase6-list_of_objects/main.tf).

### Use Case 7: map of strings with count

**Description**: Create projects on DigitalOcean based on a `map of strings` and using the `count` method

**Code**: See [here](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/main/Terraform/UseCases_Terraform/UseCase7)

**Variables definition**: We will use a map of strings as defined in the below variables definition:

```hcl
variable "projects" {
  type = map(string)
}
```

**Variables example**: below is an example of a map of strings

```hcl
projects = {
  project1 = "Project Development"
  project2 = "Project Staging"
}
```

Quick update: a question I have received already quite a few times is why the following is not considered a map of strings:

```hcl
projects = {
  "Project Development",
  "Project Staging"
}
```

The answer is that essentially a map consists of key/value pairs seperated by a '=' sign.

**Main file**:

```hcl
resource "digitalocean_project" "project" {
  count = length(var.projects)

  name = values(var.projects)[count.index]
}

output "project_names" {
  value = digitalocean_project.project[*].name
}

```

**Explanation**: this use case is very similar to Use Case 3 (map of objects with count). The explanation from Use Case 3 is very applicable to this use case

### Use Case 8: map of strings with for_each

**Description**: Create projects on DigitalOcean based on a `map of strings` and using the `for_each` method

**Code**: See [here](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/main/Terraform/UseCases_Terraform/UseCase8)

**Variables definition**: We will use a map of strings as defined in the below variables definition:

```hcl
variable "projects" {
  type = map(string)
}
```

**Variables example**: below is an example of a map of strings

```hcl
projects = {
  project1 = "Project Development",
  project2 = "Project Staging"
}
```

**Main file**:

```hcl
resource "digitalocean_project" "project" {
  for_each = var.projects

  name = each.value
}

output "project_names" {
  value = values(digitalocean_project.project)[*].name
}
```

**Explanation**: this use case is very similar to Use Case 4 (map of objects with for_each). The explanation from Use Case 4 is very applicable to this use case. Here though we don't have multiple attributes, we only have one. Hence, we only use the `each.value` statement to assign the value (string) to the name attribute.

### Use Case 9: assign created resources to attributes

**Description**: In this use case we will create projects on DigitalOcean as we have done in the previous use cases, but we will assign previously created resources (in this case servers) to the project. This means we need to find a way to address these resources.

**Code**: See [here](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/main/Terraform/UseCases_Terraform/UseCase9)

**Variables definition**: For simplicity reasons, we will use a `map of objects` with a `for_each` loop (similar to use case 4). We could of course also have chosen to add the projects and servers using a `map of objects` with the `count` method of course (use case 3). We leave that as an exercise to the reader.

```hcl
variable "servers" {
  type = map(object({
    size        = string,
    image       = string,
    region      = string
    tags        = list(string)
  }))
}

variable "projects" {
  type = map(object({
    name          = string,
    description   = string,
    purpose       = string,
    environment   = string
  }))
}
```

**Variables example**: An example of the above variable definition can be found below. Again, pretty similar to what we have done in use case 4.

```hcl
servers = {
  server1 = {
    size        = "s-2vcpu-2gb"
    image       = "ubuntu-21-10-x64"
    region      = "ams3",
    tags        = ["web", "development"]
  },
  server2 = {
    size        = "s-2vcpu-2gb"
    image       = "ubuntu-20-04-x64"
    region      = "lon1",
    tags        = ["web", "staging"]
  }
}

projects = {
  project1 = {
    name = "Project Development"
    description   = "Description for Project Development"
    purpose       = "Web Application"
    environment   = "Development"
  }
}
```

**Main file**:

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

resource "digitalocean_project" "project" {
  for_each = var.projects

  name = each.value.name
  description = each.value.description
  purpose = each.value.purpose
  environment = each.value.environment

  resources = values(digitalocean_droplet.server)[*].urn
}
```

**Explanation**: This is very similar to what we have explained as part of use case 4, but we focus in this explanation on the `resources` attribute (documentation can be found [here](https://registry.terraform.io/providers/digitalocean/digitalocean/latest/docs/resources/project). The idea is to add the servers (created with the resource digitalocean_droplet) to the projects. The way to achieve this is by referencing the created resource. We can do this using `digitalocean_droplet.server`. But why using the `values` function again? As mentioned in use case 4, Terraform would provide back the created resources as follows:

```hcl
droplets = {
  "server1" = {
    "id" = "297638825"
    "image" = "ubuntu-21-10-x64"
    "ipv4_address" = "164.92.219.154"
    "urn" = "do:droplet:297638825"
    ...
    "vpc_uuid" = "d31d4da8-8b9b-4145-bb87-0b07c285234c"
  }
  "server2" = {
    "id" = "297638826"
    "image" = "ubuntu-20-04-x64"
    "ipv4_address" = "138.68.154.243"
    "urn" = "do:droplet:297638826"
    ...
    "vpc_uuid" = "7fb12eee-afbf-4c8c-b499-fa5c107d9991"
  }
}
```

This is a map as you can see from the curly braces (and key/value pairs). The `values` function would produce a list containing the values of the elements in the map. Essentially, `values(digitalocean_droplet.server)` gives us back a list of server objects. Given this is a list, we can reference all the servers using `values(digitalocean_droplet.server)[*]`. The urn attribute is just required as per the DigitalOcean Terraform provider (see [docs](https://registry.terraform.io/providers/digitalocean/digitalocean/latest/docs/resources/project#example-usage))

### Use Case 10: conditionally assign created resources to attributes

**Description**: Use case 9 works well but given we only used one project, all the server resources are assigned to that single project. Even in case we would define multiple projects, that would mean all server resources would still be assigned to one project. You could try this out easily based on use case 9 but defining two or more projects in the variables file. You will notice that all servers get assigned to the project that was created last!

Let's get that fixed!

We will use a filter/condition to ensure that all servers with a project attribute "Staging" would be added to the "Staging" project and all servers with the attribute "Development" would be added to the "Development" project.

**Code**: See [here](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/main/Terraform/UseCases_Terraform/UseCase10)

**Variables definition**: For simplicity reasons, we will use a `map of objects` with a `for_each loop` (similar to use case 4). We could of course also have chosen to add the projects and servers using a `map of objects` with the `count` method of course (use case 3). We leave that as an exercise to the reader.

```hcl
variable "servers" {
  type = map(object({
    name    = string,
    size    = string,
    image   = string,
    region  = string
    tags    = list(string)
    project = string
  }))
}

variable "projects" {
  type = map(object({
    name        = string,
    description = string,
    purpose     = string,
    environment = string,
  }))
}
```

**Variables example**: An example of the above variable definition can be found below. Take note of the projects attribute in the servers variable.

```hcl
servers = {
  server1 = {
    name    = "Server1"
    size    = "s-2vcpu-2gb"
    image   = "ubuntu-21-10-x64"
    region  = "ams3",
    tags    = ["web", "development"]
    project = "Development"
  },
  server2 = {
    name    = "Server2"
    size    = "s-2vcpu-2gb"
    image   = "ubuntu-20-04-x64"
    region  = "lon1",
    tags    = ["web", "staging"]
    project = "Staging"
  },
  server3 = {
    name    = "Server3"
    size    = "s-2vcpu-2gb"
    image   = "ubuntu-20-04-x64"
    region  = "lon1",
    tags    = ["web", "staging"]
    project = "Staging"
  }
}

projects = {
  project1 = {
    name        = "Project Development"
    description = "Description for Project Development"
    purpose     = "Web Application"
    environment = "Development"
  },
  project2 = {
    name        = "Project Staging"
    description = "Description for Project Staging"
    purpose     = "Service or API"
    environment = "Staging"
  }
}
```

**Main file**:

```hcl
resource "digitalocean_project" "project_development" {
  for_each = {
    for key, value in var.projects : key => value
    if value.environment == "Development"
  }

  name        = each.value.name
  description = each.value.description
  purpose     = each.value.purpose
  environment = each.value.environment

  resources = [for key, value in var.servers : digitalocean_droplet.server[key].urn
      if value.project == "Development"
  ]
}

resource "digitalocean_project" "project_staging" {
  for_each = {
    for key, value in var.projects : key => value
    if value.environment == "Staging"
  }

  name        = each.value.name
  description = each.value.description
  purpose     = each.value.purpose
  environment = each.value.environment

  resources = [for key, value in var.servers : digitalocean_droplet.server[key].urn
    if value.project == "Staging"
  ]
}

output "project_names_development" {
  value = values(digitalocean_project.project_development)[*].name
}

output "project_names_staging" {
  value = values(digitalocean_project.project_staging)[*].name
}

```

**Explanation**:
The important part here is the condition in the resources attribute. We are looping through the servers variable but only the ones where the project attribute is set to either "Development" or "Staging" respectively. For this filtered list of servers, we are assigning them to the project using `digitalocean_droplet.server[key]`. The `key` here equals 'server1`, `server2`, ...

### Use Case 11: using Flatten function

**Description**: Although using conditional statements is a perfectly valid method to assign resources (see use case 10), it still forces us to define multiple project resources. In the example for use case 10, we have written a 'Staging' project resource and a 'Development' project resource. This feels a bit inefficient. In this use case example we will make use of the Flatten function in Terraform (see [docs](https://www.terraform.io/language/functions/flatten).)

**Code**: See [here](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/main/Terraform/UseCases_Terraform/UseCase11)

**Variables definition**: In the variables definition, we embed the servers object as a map.

```hcl
variable "projects" {
  type = map(object({
    description = string,
    purpose     = string,
    environment = string,
    servers = map(object({
      name  = string
      size   = string,
      image  = string,
      region = string
      tags   = list(string)
    }))
  }))
}
```

**Variables example**: In the below example, you see we don't have a specific 'servers' variable, rather only the 'projects' variable. But as you can see, we are inserting the 'servers' under each project respectively so we automatically know which server should be created under which project.

```hcl
projects = {
  development = {
        description = "Description for Project Development"
        purpose     = "Web Application"
        environment = "Development"
        servers = {
          server1 = {
            name   = "Server1"
            size   = "s-2vcpu-2gb"
            image  = "ubuntu-21-10-x64"
            region = "ams3",
            tags   = ["web", "development"]
          }
      }
    },
    staging = {
        description = "Description for Project Staging"
        purpose     = "Service or API"
        environment = "Staging"
        servers = {
          server2 = {
            name   = "Server2"
            size   = "s-2vcpu-2gb"
            image  = "ubuntu-20-04-x64"
            region = "lon1",
            tags   = ["web", "staging"]
          },
          server3 = {
            name   = "Server3"
            size   = "s-2vcpu-2gb"
            image  = "ubuntu-20-04-x64"
            region = "lon1",
            tags   = ["web", "staging"]
        }
      }
    }
}
```

**Main file**:

```hcl
locals {
  all_servers = flatten([
    for project_key, project_value in var.projects : [
      for server_key, server_value in project_value.servers  : {
        project_key = project_key
        server_key = server_key
        name     = server_value["name"]
        image    = server_value["image"]
        size     = server_value["size"]
        region   = server_value["region"]
        tags     = server_value["tags"]
      }
    ]
  ])
}


resource "digitalocean_droplet" "server" {
 for_each = {
    for server in local.all_servers : server.server_key => server
  }

  name   = each.value.name
  image  = each.value.image
  size   = each.value.size
  region = each.value.region
  ssh_keys = [
    data.digitalocean_ssh_key.terraform.id
  ]
  tags = each.value.tags
}

resource "digitalocean_project" "project" {
  for_each = var.projects

  name        = each.key
  description = each.value.description
  purpose     = each.value.purpose
  environment = each.value.environment

  resources = [ for key, value in each.value.servers : digitalocean_droplet.server[key].urn]

}
```

**Explanation**:
Let's start with the explanation of flatten function: we eventually want to have a 'servers' variable which contains all the servers embedded in the 'projects' variable. Therefore we need to loop two times:

- first loop over the 'projects' variable
- second loop over the 'servers' attribute specific to the current project we are looping over (e.g. therefore we use `project_value.servers`).

Next, we will just assign the correct attributes (e.g name, image ...) as we have easy access to them from the second loop. Note, we also assign the project_key and server_key as they will help us later.

Anyway, the flatten function will result in the following local variable called `all_servers`. As you can see, it's now a flattened object containing all servers.

```hcl
all_servers = [
      {
        image       = "ubuntu-21-10-x64"
        name        = "Server1"
        project_key = "development"
        region      = "ams3"
        server_key  = "server1"
        size        = "s-2vcpu-2gb"
        tags        = [
          "web",
          "development",
        ]
      },
      {
        image       = "ubuntu-20-04-x64"
        name        = "Server2"
        project_key = "staging"
        region      = "lon1"
        server_key  = "server2"
        size        = "s-2vcpu-2gb"
        tags        = [
          "web",
          "staging",
        ]
      },
      {
        image       = "ubuntu-20-04-x64"
        name        = "Server3"
        project_key = "staging"
        region      = "lon1"
        server_key  = "server3"
        size        = "s-2vcpu-2gb"
        tags        = [
          "web",
          "staging",
        ]
      },
    ]
```

In the droplet resource, we can now just loop over the local 'all_servers' variable in order to have them created. Also the creation of the projects are very straigthforward now.

### Use Case 12: create one record automatically when another is created (using count)

**Description**: Create one resource on DigitalOcean when another is created first. In this use case, we will create a DNS record each time a droplet is created.

**Code**: See [here](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/main/Terraform/UseCases_Terraform/UseCase12)

**Variables definition**: We have two definitions for servers and projects.

```hcl
variable "servers" {
  type = map(object({
    size        = string,
    image       = string,
    region      = string
    tags        = list(string)
  }))
}

variable "projects" {
  type = map(object({
    name          = string,
    description   = string,
    purpose       = string,
    environment   = string
  }))
}
```

**Variables example**: We define two projects and two servers

```
servers = {hcl
  server1 = {
    size        = "s-2vcpu-2gb"
    image       = "ubuntu-21-10-x64"
    region      = "ams3",
    tags        = ["web", "development"]
  },
  server2 = {
    size        = "s-2vcpu-2gb"
    image       = "ubuntu-20-04-x64"
    region      = "lon1",
    tags        = ["web", "staging"]
  }
}
projects = {
  project1 = {
    name = "Project Development"
    description   = "Description for Project Development"
    purpose       = "Web Application"
    environment   = "Development"
  },
  project2 = {
    name = "Project Staging"
    description   = "Description for Project Staging"
    purpose       = "Service or API"
    environment   = "Staging"
    }
}
```

**Main file**: The relevant section is shown below:

```hcl
resource "digitalocean_record" "www" {
  count = length(digitalocean_droplet.server)

  domain    = data.digitalocean_domain.server.id
  type      = "A"
  name      = values(digitalocean_droplet.server)[count.index].name
  value     = values(digitalocean_droplet.server)[count.index].ipv4_address
}

```

**Explanation**: We are using the count method to loop over the list of created servers. We can use the `count.index` to each time assign the name and IP address to the respective attributes. Note again that we should use the values function. This means `values(digitalocean_droplet.server)` returns a list containing the values of the elements in the map (e.g the created servers).

### Use Case 13: create one record when another is created (using for_each)

**Description**: Create one resource on DigitalOcean when another is created first. In this use case, we will create a DNS record each time a droplet is created.

**Code**: See [here](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/main/Terraform/UseCases_Terraform/UseCase13)

**Variables definition**: We have two definitions for servers and projects.

```hcl
variable "servers" {
  type = map(object({
    name        = string,
    size        = string,
    image       = string,
    region      = string
    tags        = list(string)
  }))
}

variable "projects" {
  type = map(object({
    name          = string,
    description   = string,
    purpose       = string,
    environment   = string
  }))
}
```

**Variables example**:

```hcl
servers = {
  server1 = {
    name        = "Server1"
    size        = "s-2vcpu-2gb"
    image       = "ubuntu-21-10-x64"
    region      = "ams3",
    tags        = ["web", "development"]
  },
  server2 = {
    name        = "Server2"
    size        = "s-2vcpu-2gb"
    image       = "ubuntu-20-04-x64"
    region      = "lon1",
    tags        = ["web", "staging"]
  }
}
projects = {
  project1 = {
    name = "Project Development"
    description   = "Description for Project Development"
    purpose       = "Web Application"
    environment   = "Development"
  },
  project2 = {
    name = "Project Staging"
    description   = "Description for Project Staging"
    purpose       = "Service or API"
    environment   = "Staging"
    }
}
```

**Main file**: The relevant section is shown below:

```hcl
resource "digitalocean_record" "www" {
  for_each = {
    for server in digitalocean_droplet.server : server.name => server
  }

  domain    = data.digitalocean_domain.server.id
  type      = "A"
  name      = each.value.name
  value     = each.value.ipv4_address
}

```

**Explanation**: The trick here is the `for server in digitalocean_droplet.server : server.name => server` statement. We used something similar in use case 2 as well. We need to ensure that what comes after the for_each is a map. We construct that map with a for-loop that loops over the created resources. As the key we use the name of the server (assuming it's unique). Next, we can identify each element with the `each.value` clause.
