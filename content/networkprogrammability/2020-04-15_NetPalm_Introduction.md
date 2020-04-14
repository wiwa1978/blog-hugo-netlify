---
title: NetPalm Introduction
date: 2020-04-15T12:32:50+01:00
draft: True
categories:
  - Network Programming
  - Programming
tags:
  - Python
  - Netpalm
---

```bash
WAUTERW-M-65P7:Netpalm_Introduction wauterw$ python3 -m venv venv 
WAUTERW-M-65P7:Netpalm_Introduction wauterw$ source venv/bin/activate
```
(venv) WAUTERW-M-65P7:Netpalm_Introduction wauterw$ git clone https://github.com/tbotnz/netpalm.git
(venv) WAUTERW-M-65P7:Netpalm_Introduction wauterw$ cd netpalm/
(venv) WAUTERW-M-65P7:netpalm wauterw$ sudo docker build -t netpalm .
Sending build context to Docker daemon  4.302MB
Step 1/10 : FROM python:3.8
3.8: Pulling from library/python
f15005b0235f: Pull complete 
41ebfd3d2fd0: Pull complete 
b998346ba308: Pull complete 
f01ec562c947: Pull complete 
2447a2c11907: Pull complete 
fdd2d569da3e: Pull complete 
ac3886b74a9f: Pull complete 
3c783a9b35dd: Pull complete 
ce16dda809f6: Pull complete 
Digest: sha256:e02bda1a92a0dd360a976ec3ce6ebd76f6de18b57b885c0556d5af4035e1767d
Status: Downloaded newer image for python:3.8
Successfully built a56e1fb19bfb
Successfully tagged netpalm:latest
```

```
(venv) WAUTERW-M-65P7:netpalm wauterw$ docker images
REPOSITORY                                  TAG                 IMAGE ID            CREATED             SIZE
netpalm                                     latest              a56e1fb19bfb        4 minutes ago       1.04GB
```



```
(venv) WAUTERW-M-65P7:netpalm wauterw$ docker-compose up -d
Creating network "netpalm_default" with the default driver
Pulling redis (redis:)...
latest: Pulling from library/redis
c499e6d256d6: Pull complete
bf1bc8a5a7e4: Pull complete
7564fb795604: Pull complete
ec6e86f783e4: Pull complete
1371d6223f46: Pull complete
021fd554320f: Pull complete
Digest: sha256:a732b1359e338a539c25346a50bf0a501120c41dc248d868e546b33e32bf4fe4
Status: Downloaded newer image for redis:latest
Building netpalm
Step 1/10 : FROM python:3.8
 ---> d47898c6f4b0
Step 2/10 : WORKDIR /usr/local/lib/python3.8/site-packages
 ---> Using cache
 ---> e8360358c54c
Step 3/10 : RUN git clone https://github.com/networktocode/ntc-templates.git
 ---> Using cache
 ---> a2c5837b5d2a
Step 4/10 : RUN mv ntc-templates ntc_templates
 ---> Using cache
 ---> 20739bf83e15
Step 5/10 : ADD . /code
 ---> bff5e87c770b
Step 6/10 : WORKDIR /code
 ---> Running in c9846a352127
Removing intermediate container c9846a352127
 ---> b05753baae09
Step 7/10 : RUN pip3 install --upgrade pip
 ---> Running in e1fad7f6302c
Requirement already up-to-date: pip in /usr/local/lib/python3.8/site-packages (20.0.2)
Removing intermediate container e1fad7f6302c
 ---> ad05489fbcc2
Step 8/10 : RUN pip3 install -r requirements.txt
 ---> Running in 64ef2197499d
Collecting Flask
  Downloading Flask-1.1.2-py2.py3-none-any.whl (94 kB)
Collecting netmiko
  Downloading netmiko-3.1.0-py2.py3-none-any.whl (144 kB)
Collecting napalm
  Downloading napalm-2.5.0-py2.py3-none-any.whl (201 kB)
Collecting ncclient
  Downloading ncclient-0.6.7.tar.gz (605 kB)
Collecting requests
  Downloading requests-2.23.0-py2.py3-none-any.whl (58 kB)
Collecting redis
  Downloading redis-3.4.1-py2.py3-none-any.whl (71 kB)
Collecting rq
  Downloading rq-1.3.0-py2.py3-none-any.whl (59 kB)
Collecting xmltodict
  Downloading xmltodict-0.12.0-py2.py3-none-any.whl (9.2 kB)
Collecting jinja2
  Downloading Jinja2-2.11.1-py2.py3-none-any.whl (126 kB)
Collecting jinja2schema
  Downloading jinja2schema-0.1.4.tar.gz (19 kB)
Collecting jsonschema
  Downloading jsonschema-3.2.0-py2.py3-none-any.whl (56 kB)
Collecting itsdangerous>=0.24
  Downloading itsdangerous-1.1.0-py2.py3-none-any.whl (16 kB)
Collecting Werkzeug>=0.15
  Downloading Werkzeug-1.0.1-py2.py3-none-any.whl (298 kB)
Collecting click>=5.1
  Downloading click-7.1.1-py2.py3-none-any.whl (82 kB)
Requirement already satisfied: setuptools>=38.4.0 in /usr/local/lib/python3.8/site-packages (from netmiko->-r requirements.txt (line 2)) (46.1.3)
Collecting pyserial
  Downloading pyserial-3.4-py2.py3-none-any.whl (193 kB)
Collecting textfsm
  Downloading textfsm-1.1.0-py2.py3-none-any.whl (37 kB)
Collecting paramiko>=2.4.3
  Downloading paramiko-2.7.1-py2.py3-none-any.whl (206 kB)
Collecting scp>=0.13.2
  Downloading scp-0.13.2-py2.py3-none-any.whl (9.5 kB)
Collecting cffi>=1.11.3
  Downloading cffi-1.14.0-cp38-cp38-manylinux1_x86_64.whl (409 kB)
Collecting ciscoconfparse
  Downloading ciscoconfparse-1.5.1-py3-none-any.whl (88 kB)
Collecting netaddr
  Downloading netaddr-0.7.19-py2.py3-none-any.whl (1.6 MB)
Collecting pyeapi>=0.8.2
  Downloading pyeapi-0.8.3.tar.gz (137 kB)
Collecting pyIOSXR>=0.53
  Downloading pyIOSXR-0.53.tar.gz (15 kB)
Collecting nxapi-plumbing>=0.5.2
  Downloading nxapi_plumbing-0.5.2.tar.gz (11 kB)
Collecting future
  Downloading future-0.18.2.tar.gz (829 kB)
Collecting junos-eznc==2.2.1
  Downloading junos_eznc-2.2.1-py2.py3-none-any.whl (159 kB)
Collecting pyYAML
  Downloading PyYAML-5.3.1.tar.gz (269 kB)
Collecting lxml>=3.3.0
  Downloading lxml-4.5.0-cp38-cp38-manylinux1_x86_64.whl (5.6 MB)
Collecting six
  Downloading six-1.14.0-py2.py3-none-any.whl (10 kB)
Collecting urllib3!=1.25.0,!=1.25.1,<1.26,>=1.21.1
  Downloading urllib3-1.25.8-py2.py3-none-any.whl (125 kB)
Collecting chardet<4,>=3.0.2
  Downloading chardet-3.0.4-py2.py3-none-any.whl (133 kB)
Collecting certifi>=2017.4.17
  Downloading certifi-2020.4.5.1-py2.py3-none-any.whl (157 kB)
Collecting idna<3,>=2.5
  Downloading idna-2.9-py2.py3-none-any.whl (58 kB)
Collecting MarkupSafe>=0.23
  Downloading MarkupSafe-1.1.1-cp38-cp38-manylinux1_x86_64.whl (32 kB)
Collecting pyrsistent>=0.14.0
  Downloading pyrsistent-0.16.0.tar.gz (108 kB)
Collecting attrs>=17.4.0
  Downloading attrs-19.3.0-py2.py3-none-any.whl (39 kB)
Collecting bcrypt>=3.1.3
  Downloading bcrypt-3.1.7-cp34-abi3-manylinux1_x86_64.whl (56 kB)
Collecting pynacl>=1.0.1
  Downloading PyNaCl-1.3.0-cp34-abi3-manylinux1_x86_64.whl (759 kB)
Collecting cryptography>=2.5
  Downloading cryptography-2.9-cp35-abi3-manylinux2010_x86_64.whl (2.7 MB)
Collecting pycparser
  Downloading pycparser-2.20-py2.py3-none-any.whl (112 kB)
Collecting passlib
  Downloading passlib-1.7.2-py2.py3-none-any.whl (507 kB)
Collecting colorama
  Downloading colorama-0.4.3-py2.py3-none-any.whl (15 kB)
Collecting dnspython
  Downloading dnspython-1.16.0-py2.py3-none-any.whl (188 kB)
Building wheels for collected packages: ncclient, jinja2schema, pyeapi, pyIOSXR, nxapi-plumbing, future, pyYAML, pyrsistent
  Building wheel for ncclient (setup.py): started
  Building wheel for ncclient (setup.py): finished with status 'done'
  Created wheel for ncclient: filename=ncclient-0.6.7-py2.py3-none-any.whl size=99677 sha256=a33e0f83bb7f61f34d603121c2bd0a3318ad946d0c7155770bb07abc58dc6f0e
  Stored in directory: /root/.cache/pip/wheels/1f/c8/fa/007d8803ed8c576c7779e2fe37fa4f667bd56a24053f3c115c
  Building wheel for jinja2schema (setup.py): started
  Building wheel for jinja2schema (setup.py): finished with status 'done'
  Created wheel for jinja2schema: filename=jinja2schema-0.1.4-py3-none-any.whl size=23697 sha256=1f79ee42900847e39b9f7bb2972e7898ea3d424a1ebda5f5721e6751e3cd8473
  Stored in directory: /root/.cache/pip/wheels/a0/c1/ed/311122ee9f5cb1327f552f42e728d989a2bedc784913c72ae1
  Building wheel for pyeapi (setup.py): started
  Building wheel for pyeapi (setup.py): finished with status 'done'
  Created wheel for pyeapi: filename=pyeapi-0.8.3-py3-none-any.whl size=90262 sha256=c8801d99f0912365ff792c80f52d1fc90debe3a032c1f1340d894eb3e8df6b52
  Stored in directory: /root/.cache/pip/wheels/d1/56/be/60a2a048f7510cc33f862dafa06471e928494ebdf0a0558ec0
  Building wheel for pyIOSXR (setup.py): started
  Building wheel for pyIOSXR (setup.py): finished with status 'done'
  Created wheel for pyIOSXR: filename=pyIOSXR-0.53-py3-none-any.whl size=10297 sha256=1ef910edb7a1a3c3bddefd8cccd4d90351c5d68e35ed199163ea2a4c97356625
  Stored in directory: /root/.cache/pip/wheels/b3/6b/f1/8d56a9a5c225cdaff9d0761f3f77349f68d220d10b7fd68c11
  Building wheel for nxapi-plumbing (setup.py): started
  Building wheel for nxapi-plumbing (setup.py): finished with status 'done'
  Created wheel for nxapi-plumbing: filename=nxapi_plumbing-0.5.2-py3-none-any.whl size=10797 sha256=43aa84d250df2a0eef23ef8b5565b0511c5d0f8e4274b3bc00082cb0cd383529
  Stored in directory: /root/.cache/pip/wheels/16/0d/46/9acd1b15c05930d886cc2ece3d1377d9ece380240cdc2b5786
  Building wheel for future (setup.py): started
  Building wheel for future (setup.py): finished with status 'done'
  Created wheel for future: filename=future-0.18.2-py3-none-any.whl size=491058 sha256=2040a146fb62a209d39b86c7289c554f903121423c9d53ea67a7906d2e44623d
  Stored in directory: /root/.cache/pip/wheels/8e/70/28/3d6ccd6e315f65f245da085482a2e1c7d14b90b30f239e2cf4
  Building wheel for pyYAML (setup.py): started
  Building wheel for pyYAML (setup.py): finished with status 'done'
  Created wheel for pyYAML: filename=PyYAML-5.3.1-cp38-cp38-linux_x86_64.whl size=572462 sha256=8046fc98d28b62a5a23f55f398264f314ed85e0b558ea85d10f6f1ba571b15d1
  Stored in directory: /root/.cache/pip/wheels/13/90/db/290ab3a34f2ef0b5a0f89235dc2d40fea83e77de84ed2dc05c
  Building wheel for pyrsistent (setup.py): started
  Building wheel for pyrsistent (setup.py): finished with status 'done'
  Created wheel for pyrsistent: filename=pyrsistent-0.16.0-cp38-cp38-linux_x86_64.whl size=126692 sha256=55f6fd3078cc0232d4cee578caece07bfca93aa28f0412a207db914c47bb0a24
  Stored in directory: /root/.cache/pip/wheels/17/be/0f/727fb20889ada6aaaaba861f5f0eb21663533915429ad43f28
Successfully built ncclient jinja2schema pyeapi pyIOSXR nxapi-plumbing future pyYAML pyrsistent
ERROR: napalm 2.5.0 has requirement netmiko==2.4.2, but you'll have netmiko 3.1.0 which is incompatible.
Installing collected packages: itsdangerous, Werkzeug, MarkupSafe, jinja2, click, Flask, pyserial, future, six, textfsm, pycparser, cffi, bcrypt, pynacl, cryptography, paramiko, scp, netmiko, passlib, colorama, dnspython, ciscoconfparse, netaddr, pyeapi, lxml, pyIOSXR, urllib3, chardet, certifi, idna, requests, nxapi-plumbing, ncclient, pyYAML, junos-eznc, napalm, redis, rq, xmltodict, jinja2schema, pyrsistent, attrs, jsonschema
Successfully installed Flask-1.1.2 MarkupSafe-1.1.1 Werkzeug-1.0.1 attrs-19.3.0 bcrypt-3.1.7 certifi-2020.4.5.1 cffi-1.14.0 chardet-3.0.4 ciscoconfparse-1.5.1 click-7.1.1 colorama-0.4.3 cryptography-2.9 dnspython-1.16.0 future-0.18.2 idna-2.9 itsdangerous-1.1.0 jinja2-2.11.1 jinja2schema-0.1.4 jsonschema-3.2.0 junos-eznc-2.2.1 lxml-4.5.0 napalm-2.5.0 ncclient-0.6.7 netaddr-0.7.19 netmiko-3.1.0 nxapi-plumbing-0.5.2 paramiko-2.7.1 passlib-1.7.2 pyIOSXR-0.53 pyYAML-5.3.1 pycparser-2.20 pyeapi-0.8.3 pynacl-1.3.0 pyrsistent-0.16.0 pyserial-3.4 redis-3.4.1 requests-2.23.0 rq-1.3.0 scp-0.13.2 six-1.14.0 textfsm-1.1.0 urllib3-1.25.8 xmltodict-0.12.0
Removing intermediate container 64ef2197499d
 ---> 3e523c048abc
Step 9/10 : RUN ln -sf /usr/local/lib/python3.8/site-packages/ntc_templates/templates/ backend/plugins/ntc-templates
 ---> Running in bd98e4a24c31
Removing intermediate container bd98e4a24c31
 ---> 117c14c0f388
Step 10/10 : CMD python3 netpalm.py
 ---> Running in fd258706db81
Removing intermediate container fd258706db81
 ---> 3bef4e5dcc00
Successfully built 3bef4e5dcc00
Successfully tagged netpalm_netpalm:latest
WARNING: Image for service netpalm was built because it did not already exist. To rebuild this image you must use `docker-compose build` or `docker-compose up --build`.
Creating netpalm_redis_1 ... done
Creating netpalm_netpalm_1 ... done
```

```
(venv) WAUTERW-M-65P7:netpalm wauterw$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
603fe3088632        netpalm_netpalm     "/bin/sh -c 'python3…"   33 minutes ago      Up 33 minutes       0.0.0.0:9000->9000/tcp   netpalm_netpalm_1
654a0b2623ec        redis               "docker-entrypoint.s…"   33 minutes ago      Up 33 minutes       6379/tcp                 netpalm_redis_1
```