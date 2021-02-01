---
title: Deploy Flask Application with Waypoint to Docker
date: 2021-02-10T14:39:50+01:00
draft: True
categories:
  - DevOps
  - Private Cloud
  - All
tags:
  - KVM
  - Ubuntu
  - Waypoint
---

### Introduction


https://medium.com/@michael_41345/getting-started-with-hashicorp-waypoint-how-to-create-a-python-app-using-flask-docker-80b279c88b2a


### What is Hashicorp Waypoint

### Installing Hashicorp Waypoint CLI

```
~ ❯ brew install hashicorp/tap/waypoint 
==> Installing waypoint from hashicorp/tap
==> Downloading https://releases.hashicorp.com/waypoint/0.2.0/waypoint_0.2.0_darwin_amd64.zip
######################################################################## 100.0%
🍺  /usr/local/Cellar/waypoint/0.2.0: 3 files, 135.3MB, built in 4 seconds
```
Check whether waypoint is working correctly.

```
~Hashicorp_Waypoint master ❯ waypoint version
Waypoint v0.2.0 (360b2b3a)
```

### Installing Hashicorp Waypoint Server

```
~Flask/Flask-Basic-Waypoint main !4 ?6 ❯ waypoint install --platform=docker -accept-tos 
✓ Pulling image: hashicorp/waypoint:latest
✓ Installing Waypoint server to docker
✓ Server container started!
✓ Configuring server...
Waypoint server successfully installed and configured!

The CLI has been configured to connect to the server automatically. This
connection information is saved in the CLI context named "install-1612115989".
Use the "waypoint context" CLI to manage CLI contexts.

The server has been configured to advertise the following address for
entrypoint communications. This must be a reachable address for all your
deployments. If this is incorrect, manually set it using the CLI command
"waypoint server config-set".

Advertise Address: waypoint-server:9701
Web UI Address: https://localhost:9702
```

![flask-basic](/images/2021_02_10-1.png)


![flask-basic](/images/2021_02_10-2.png)

```bash
~/Flask/Flask-Basic-Waypoint main !4 ?6 ❯ waypoint token new 
bM152PWkXxfoy4vA51JFhR7LsYkVs9ssTRzjjJTjKh51sahtxoswohZ9YQPLDgE8MdxPFyjhjz71ZGfmnjh6igdqs23ijB9ciYQWU
```
![flask-basic](/images/2021_02_10-3.png) 

### Deploying Flask application

```bash
~Flask/Flask-Basic-Waypoint main !4 ?6 ❯ waypoint init 
```

![flask-basic](/images/2021_02_10-4.png) 

This will create a `waypoint.hcl` file in your current directory. Note: I removed the comments for display purposes

```tcl
project = "my-project"

app "web" {
    build {
        use "docker" {}
    }

    deploy {
        use "docker" {}
    }
}
```

Change the file to:

```tcl
project = "Flask Todo application"

app "flask_todo_app" {
    labels = {
    "service" = "flask-todo",
    "env"     = "dev"
  }
    build {
        use "docker" {
        }

    }

    deploy {
        use "docker" {
            service_port = 8081
        }
    }
}
```


```bash
~/Flask/Flask-Basic-Waypoint main !4 ?6 ❯ waypoint up  

» Building...
✓ Initializing Docker client...
⠏ Building image...
 │ Step 1/10 : FROM ubuntu:20.04
 <TRUNCATED>
» Deploying...
✓ Setting up waypoint network
✓ Starting container
✓ App deployed as container: flask_todo_app-01EXCSXAMW7QYZ1T3CC99YDPC2

» Releasing...

The deploy was successful! A Waypoint deployment URL is shown below. This
can be used internally to check your deployment and is not meant for external
traffic. You can manage this hostname using "waypoint hostname."

           URL: https://correctly-decent-mammoth.waypoint.run
Deployment URL: https://correctly-decent-mammoth--v1.waypoint.run

```
![flask-basic](/images/2021_02_10-5.png) 
![flask-basic](/images/2021_02_10-6.png) 