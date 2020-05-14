---
title: Python Virtual Environments
date: 2020-01-08T10:19:50+01:00
draft: false
categories:
  - Network Programming
  - Programming
  - All
tags:
  - Python
---
### What is a virtual environment
The main purpose of virtual environments is to create isolated environments for Python projects. As such, each project can have its own dependencies, regardless of the dependencies in other projects or on your 'main' PC.

### Setup a virtual environment
Installing a virtual environment is pretty easy. First, use pip3 (for Python3) to install the package.
```bash
WAUTERW-M-65P7:Parse_YAML_Python wauterw$ pip3 install virtualenv
```
Next, create a virtual environment:
```bash
WAUTERW-M-65P7:Parse_YAML_Python wauterw$ python3 -m venv
usage: venv [-h] [--system-site-packages] [--symlinks | --copies] [--clear] [--upgrade] [--without-pip]
            [--prompt PROMPT]
            ENV_DIR [ENV_DIR ...]
```
The above learns us that the main command is `python3 -m venv`, but we also need to give the virtual environment a name. Most people use `venv` as the name, but this is entirely up to you. In any case, it explains a bit why you see two times the word `venv` in below command.
```bash
WAUTERW-M-65P7:Parse_YAML_Python wauterw$ python3 -m venv venv
```
Next, we need to activate the virtual environment:
```bash
WAUTERW-M-65P7:Parse_YAML_Python wauterw$ source venv/bin/activate
(venv) WAUTERW-M-65P7:Parse_YAML_Python wauterw$ 
```
Observe that the command prompt now has this `(venv)` statement in front of it. This means you are into the virtual environment. Any install request under that virtual environment is entirely isolated from other environments.
