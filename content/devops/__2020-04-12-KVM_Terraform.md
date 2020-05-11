---
title: Create VMs on KVM with Terraform
date: 2020-04-12T14:39:50+01:00
draft: True
categories:
  - DevOps
  - Continuous Integration/Deployment
  - Private Cloud
  - All
tags:
  - KVM
  - Terraform
---

### Introduction

### Install Terraform
```
WAUTERW-M-65P7:~ wauterw$ brew install terraform -v
***Truncated***
WAUTERW-M-65P7:~ wauterw$ terraform -v
Terraform v0.12.24
```
### Install Golang

```
WAUTERW-M-65P7:~ wauterw$ brew install golang
WAUTERW-M-65P7:~ wauterw$ brew install pkg-config
WAUTERW-M-65P7:~ wauterw$ brew install libvirt
***Truncated
WAUTERW-M-65P7:~ wauterw$ go version
go version go1.14.1 darwin/amd64
```

### Install Terraform KVM provider

```
WAUTERW-M-65P7:~ wauterw$ go get github.com/dmacvicar/terraform-provider-libvirt
WAUTERW-M-65P7:~ wauterw$ go install github.com/dmacvicar/terraform-provider-libvirt
WAUTERW-M-65P7:~ wauterw$ cd ~/.terraform.d/plugins/
WAUTERW-M-65P7:plugins wauterw$ ls
terraform-provider-libvirt
WAUTERW-M-65P7:plugins wauterw$ brew install cdrtools
```
