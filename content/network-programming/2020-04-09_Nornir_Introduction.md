---
title: Nornir Introduction
date: 2020-04-04T12:32:50+01:00
draft: true
categories:
  - Programming
tags:
  - Python
  - Nornir
---

### Installation

```bash
WAUTERW-M-65P7:Nornir_Intro wauterw$ python3 -m venv venv
WAUTERW-M-65P7:Nornir_Intro wauterw$ source venv/bin/activate
(venv) WAUTERW-M-65P7:Nornir_Intro wauterw$ 
```

```bash
(venv) WAUTERW-M-65P7:Nornir_Intro wauterw$ pip3 install nornir
Collecting nornir
***Truncated***
Installing collected packages: typing-extensions, certifi, idna, chardet, urllib3, requests, MarkupSafe, jinja2, six, pycparser, cffi, cryptography, pynacl, bcrypt, paramiko, scp, future, textfsm, pyserial, netmiko, lxml, pyIOSXR, ncclient, netaddr, pyYAML, junos-eznc, nxapi-plumbing, pyeapi, passlib, colorama, dnspython, ciscoconfparse, napalm, mypy-extensions, ruamel.yaml.clib, ruamel.yaml, nornir
  Running setup.py install for future ... done
  Running setup.py install for pyIOSXR ... done
  Running setup.py install for ncclient ... done
  Running setup.py install for pyYAML ... done
  Running setup.py install for nxapi-plumbing ... done
  Running setup.py install for pyeapi ... done
  Running setup.py install for ruamel.yaml.clib ... done
Successfully installed MarkupSafe-1.1.1 bcrypt-3.1.7 certifi-2019.11.28 cffi-1.14.0 chardet-3.0.4 ciscoconfparse-1.5.1 colorama-0.4.3 cryptography-2.8 dnspython-1.16.0 future-0.18.2 idna-2.9 jinja2-2.11.1 junos-eznc-2.2.1 lxml-4.5.0 mypy-extensions-0.4.3 napalm-2.5.0 ncclient-0.6.7 netaddr-0.7.19 netmiko-3.1.0 nornir-2.4.0 nxapi-plumbing-0.5.2 paramiko-2.7.1 passlib-1.7.2 pyIOSXR-0.53 pyYAML-5.3.1 pycparser-2.20 pyeapi-0.8.3 pynacl-1.3.0 pyserial-3.4 requests-2.23.0 ruamel.yaml-0.16.10 ruamel.yaml.clib-0.2.0 scp-0.13.2 six-1.14.0 textfsm-1.1.0 typing-extensions-3.7.4.1 urllib3-1.25.8
WARNING: You are using pip version 19.2.3, however version 20.0.2 is available.
You should consider upgrading via the 'pip install --upgrade pip' command.

```