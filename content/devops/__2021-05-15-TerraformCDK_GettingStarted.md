---
title: Terraform-CDK - Create AWS EC2 instance
date: 2021-05-10T14:39:50+01:00
draft: True
categories:
  - DevOps
  - Infrastructure As Code
  - Public Cloud
  - All
tags:
  - AWS
  - Terraform
  - Terraform-CDK
---

### Introduction


```
/Webserver/TerraformCDK❯ npm install --global cdktf-cli
```

```
alias cdktf='/usr/local/bin/cdktf'
```

```
brew install pipenv
```

```
export AWS_ACCESS_KEY_ID=A****4X
export AWS_SECRET_ACCESS_KEY=Ii***prkN
```

```
/Webserver/TerraformCDK❯ cdktf init --template="python" --local
Note: By supplying '--local' option you have chosen local storage mode for storing the state of your stack.
This means that your Terraform state file will be stored locally on disk in a file 'terraform.tfstate' in the root of your project.

We will now set up the project. Please enter the details for your project.
If you want to exit, press ^C.

Project Name: (default: 'TerraformCDK') TerraformCDK_EC2instance
Project Description: (default: 'A simple getting started project for cdktf.') Simple project to create EC2 instance
Creating a virtualenv for this project...
Pipfile: /Users/wauterw/SynologyDrive/Programming/blog-hugo-netlify-code/InfraAsCode/Webserver/TerraformCDK/Pipfile
Using /usr/local/bin/python3.9 (3.9.5) to create virtualenv...
⠦ Creating virtual environment...created virtual environment CPython3.9.5.final.0-64 in 1134ms
  creator CPython3Posix(dest=/Users/wauterw/.local/share/virtualenvs/TerraformCDK-IpdiLeis, clear=False, no_vcs_ignore=False, global=False)
  seeder FromAppData(download=False, pip=bundle, setuptools=bundle, wheel=bundle, via=copy, app_data_dir=/Users/wauterw/Library/Application Support/virtualenv)
    added seed packages: pip==21.1.2, setuptools==57.0.0, wheel==0.36.2
  activators BashActivator,CShellActivator,FishActivator,PowerShellActivator,PythonActivator,XonshActivator

✔ Successfully created virtual environment! 
Virtualenv location: /Users/wauterw/.local/share/virtualenvs/TerraformCDK-IpdiLeis
Pipfile.lock not found, creating...
Locking [dev-packages] dependencies...
Locking [packages] dependencies...
Updated Pipfile.lock (c0c5a6)!
Installing dependencies from Pipfile.lock (c0c5a6)...
  🐍   ▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉ 0/0 — 00:00:00
To activate this project's virtualenv, run pipenv shell.
Alternatively, run a command inside the virtualenv with pipenv run.
Installing cdktf~=0.4.1...
Adding cdktf to Pipfile's [packages]...
✔ Installation Succeeded 
Pipfile.lock (c0c5a6) out of date, updating to (552952)...
Locking [dev-packages] dependencies...
Locking [packages] dependencies...
Building requirements...
Resolving dependencies...
✔ Success! 
Updated Pipfile.lock (552952)!
Installing dependencies from Pipfile.lock (552952)...
  🐍   ▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉ 0/0 — 00:00:00
To activate this project's virtualenv, run pipenv shell.
Alternatively, run a command inside the virtualenv with pipenv run.
========================================================================================================

  Your cdktf Python project is ready!

  cat help                Prints this message

  Compile:
    pipenv run ./main.py  Compile and run the python code.

  Synthesize:
    cdktf synth [stack]   Synthesize Terraform resources to cdktf.out/

  Diff:
    cdktf diff [stack]    Perform a diff (terraform plan) for the given stack

  Deploy:
    cdktf deploy [stack]  Deploy the given stack

  Destroy:
    cdktf destroy [stack] Destroy the given stack

  Learn more about using modules and providers https://cdk.tf/modules-and-providers

========================================================================================================

```

Add the AWS provider in cdktf.json
```
{
  "language": "python",
  "app": "pipenv run python main.py",
  "terraformProviders": [
    "hashicorp/aws@~> 3.46"
  ],
  "terraformModules": [],
  "codeMakerOutput": "imports",
  "context": {
    "excludeStackIdFromLogicalIds": "true",
    "allowSepCharsInLogicalIds": "true"
  }
}
```


```
/Webserver/TerraformCDK❯ cdktf get
Generated python constructs in the output directory: imports
```

```python

```