---
title: DigitalOcean - Create K8S cluster with Pulumi
date: 2022-05-12T14:39:50+01:00
draft: True
categories:
  - DevOps
  - Infrastructure As Code
  - Public Cloud
  - All
tags:
  - DigitalOcean
  - Pulumi
---

### Introduction

In this post, I will continue my experiments with Pulumi a bit. We'll focus on creating a Kubernetes cluster. Very much similar to what we did in [this](https://blog.wimwauters.com/devops/2022-02-25-digitalocean_terraform_k8s/) and [this](https://blog.wimwauters.com/devops/2022-02-27-digitalocean_doctl_k83/) post but focused on Pulumi instead.

To summarize, Pulumi is an Infra As Code tool, similar to the likes of Terraform. Unlike Terraform, which has its own language (Hashicorp Configuration Language) and syntax for defining infrastructure as code, Pulumi uses real programming languages. This means that you can write your configuration languages in languages like Python, Javascript, Typescript, Go or .NET languages like C# and F#. In this blog post we'll focus on Python as our language of choice.

### Getting started

We first need to create our Pulumi stack and indicate we want to use Python as our preferred language. Go ahead and use the `pulumi new python` command. That will guide us through some questions to set up our stack.

```bash
~/Pulumi_DigitalOcean_K8S ❯ pulumi new python
This command will walk you through creating a new Pulumi project.

Enter a value or leave blank to accept the (default), and press <ENTER>.
Press ^C at any time to quit.

project name: (Pulumi_DigitalOcean_K8S)
project description: (A minimal Python Pulumi program) Pulumi K8S tryout
Created project 'Pulumi_DigitalOcean_K8S'

Please enter your desired stack name.
To create a stack in an organization, use the format <org-name>/<stack-name> (e.g. `acmecorp/dev`).
stack name: (dev)
Created stack 'dev'

Failed to resolve python version command: fork/exec /Users/wauterw/SynologyDrive/Programming/Pulumi/Pulumi_DigitalOcean_K8S/venv/bin/python: no such file or directory

Installing dependencies...

Creating virtual environment...
Finished creating virtual environment
Updating pip, setuptools, and wheel in virtual environment...
Requirement already satisfied: pip in ./venv/lib/python3.9/site-packages (21.3.1)
...
...
Finished installing dependencies

Your new project is ready to go! ✨

To perform an initial deployment, run 'pulumi up'
```

Go to `https://app.pulumi.com` and see that our stack got created:

![pulumi_k8s](/images/2022-05-13-1.png)

Next, we will activate the virtual environment and install the `pulumi-digitalocean` Python package.

```bash
~/Pulumi_DigitalOcean_K8S ❯ source venv/bin/activate
~/Pulumi_DigitalOcean_K8S ❯ pip3 install pulumi-digitalocean
~/Pulumi_DigitalOcean_K8S ❯ pip3 freeze > requirements.txt
```

Also, don't forget to set the `DIGITALOCEAN_TOKEN` environment variable. As mentioned in previous posts, you should create an API token on the DigitalOcean console in order to retrieve the token.

```bash
~/Pulumi_DigitalOcean_K8S ❯ export DIGITALOCEAN_TOKEN=dop_v1_e***
```

### Creating Kubernetes cluster

Now that the basic stuff is being taken care off, we can focus on the Python code to create a Kubernetes cluster. You will see that Pulumi already scaffolded quite some code. The important file for us is the `__main__.py` file.

Have a look at the [documentation](https://www.pulumi.com/registry/packages/digitalocean/api-docs/kubernetescluster/) in order to understand below snippet better. We simply create a `KubernetesCluster` and add some nodes to it using `KubernetesClusterNodePoolArgs`.

```python
"""A Python Pulumi program"""

import pulumi
import pulumi_digitalocean as digitalocean

# ------------------------------------------------------------------
# VARIABLES
# ------------------------------------------------------------------
name = "clusterwim"
region = "ams3"
version = "1.22.8-do.1"
size = "s-2vcpu-2gb"
node_count = 3

cluster = digitalocean.KubernetesCluster(
    name,
    auto_upgrade=True,
    region=region,
    version=version,
    node_pool=digitalocean.KubernetesClusterNodePoolArgs(
        name="front-end-pool",
        size="s-2vcpu-2gb",
        node_count=3,
        auto_scale=False,
        tags=["node-pool-tag"]
    ),
)

pulumi.export(f"{name}", cluster.ipv4_address)
```

Next, just run `pulumi up`. It will take a couple of minutes to provision our Kubernetes cluster.

```bash
~/Pulumi_DigitalOcean_K8S ❯ pulumi up
Previewing update (dev)

View Live: https://app.pulumi.com/wymedia/Pulumi_DigitalOcean_K8S/dev/previews/8b7baf54-4caf-42bc-898e-28c072ba55b5

     Type                                     Name                         Plan
     pulumi:pulumi:Stack                      Pulumi_DigitalOcean_K8S-dev

- └─ digitalocean:index:KubernetesCluster clusterwim create

Outputs:

- clusterwim: output<string>

Resources: + 1 to create
1 unchanged

Do you want to perform this update? yes
Updating (dev)

View Live: https://app.pulumi.com/wymedia/Pulumi_DigitalOcean_K8S/dev/updates/4

     Type                                     Name                         Status
     pulumi:pulumi:Stack                      Pulumi_DigitalOcean_K8S-dev

- └─ digitalocean:index:KubernetesCluster clusterwim created

Outputs:

- clusterwim: "161.35.89.15"

Resources: + 1 created
1 unchanged

Duration: 7m56s

```

Go to the DigitalOcean console and you should see your Kubernetes cluster being provisioned.

![pulumi_k8s](/images/2022-05-13-2.png)

Some detailed information:

![pulumi_k8s](/images/2022-05-13-3.png)

In Pulumi (https://app.pulumi.com) you will see that our stack got upgraded.

![pulumi_k8s](/images/2022-05-13-4.png)

You can also see the objects that got created by Pulumi:

![pulumi_k8s](/images/2022-05-13-5.png)

### Deleting Kubernetes cluster

It's very easy to now also destroy our Kubernetes cluster. Just use the `pulumi destroy` command to do so:

```bash
~/Pulumi_DigitalOcean_K8S ❯ pulumi destroy
Previewing destroy (dev)

View Live: https://app.pulumi.com/wymedia/Pulumi_DigitalOcean_K8S/dev/previews/77365210-42bc-44a3-b946-cb50f193cc5b

     Type                                     Name                         Plan

- pulumi:pulumi:Stack Pulumi_DigitalOcean_K8S-dev delete
- └─ digitalocean:index:KubernetesCluster clusterwim delete

Outputs:

- clusterwim: "161.35.89.15"

Resources: - 2 to delete

Do you want to perform this destroy? yes
Destroying (dev)

View Live: https://app.pulumi.com/wymedia/Pulumi_DigitalOcean_K8S/dev/updates/5

     Type                                     Name                         Status

- pulumi:pulumi:Stack Pulumi_DigitalOcean_K8S-dev deleted
- └─ digitalocean:index:KubernetesCluster clusterwim deleted

Outputs:

- clusterwim: "161.35.89.15"

Resources: - 2 deleted

Duration: 3s

The resources in the stack have been deleted, but the history and configuration associated with the stack are still maintained.
If you want to remove the stack completely, run 'pulumi stack rm dev'.

```
