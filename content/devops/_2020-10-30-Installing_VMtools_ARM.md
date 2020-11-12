

```
#!/bin/sh
apt update -y && apt upgrade -y
```
```
apt install -y git automake make gobjc++ libtool pkg-config libmspack-dev libglib2.0-dev libpam0g-dev libssl-dev libxml2-dev libxmlsec1-dev libx11-dev libxext-dev libxinerama-dev libxi-dev libxrender-dev libxrandr-dev libxtst-dev libgdk-pixbuf2.0-dev libgtk-3-dev libgtkmm-3.0-dev
```

```
git clone https://github.com/vmware/open-vm-tools.git
cd open-vm-tools/open-vm-tools/
```

```
autoreconf -i
./configure --disable-dependency-tracking
make
make install
ldconfig
```

```
cat > /etc/systemd/system/vmtoolsd.service << EOF
[Unit]
Description=Service for virtual machines hosted on VMware
Documentation=http://github.com/vmware/open-vm-tools
After=network-online.target

[Service]
ExecStart=/usr/local/bin/vmtoolsd
Restart=always
TimeoutStopSec=5

[Install]
WantedBy=multi-user.target
EOF
```

```
systemctl enable vmtoolsd.service
systemctl start vmtoolsd.service
systemctl status vmtoolsd.service
```


If all went well, you should be seeing something along the lines of below output
```
● vmtoolsd.service - Service for virtual machines hosted on VMware
     Loaded: loaded (/etc/systemd/system/vmtoolsd.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2020-11-11 20:56:01 UTC; 47ms ago
       Docs: http://github.com/vmware/open-vm-tools
   Main PID: 102012 (vmtoolsd)
      Tasks: 1 (limit: 2233)
     Memory: 680.0K
     CGroup: /system.slice/vmtoolsd.service
             └─102012 /usr/local/bin/vmtoolsd

Nov 11 20:56:01 ubuntu-microk8s-1 systemd[1]: Started Service for virtual machines hosted on VMware.
```