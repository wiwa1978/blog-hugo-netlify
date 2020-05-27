
### Preparing the servers

cisco@wauterw-k8s-master:~$ sudo apt-get remove docker docker-engine docker.io containerd runc

cisco@wauterw-k8s-master:~$ sudo apt-get update

cisco@wauterw-k8s-master:~$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common

cisco@wauterw-k8s-master:~$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

cisco@wauterw-k8s-master:~$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

cisco@wauterw-k8s-master:~$ sudo apt-get update
cisco@wauterw-k8s-master:~$ sudo apt-get install docker-ce docker-ce-cli containerd.io

cisco@wauterw-k8s-master:~$ sudo nano /etc/docker/daemon.json

 {
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
cisco@wauterw-k8s-master:~$ sudo su
root@wauterw-k8s:/home/cisco# mkdir -p /etc/systemd/system/docker.service.d
root@wauterw-k8s:/home/cisco# systemctl daemon-reload
root@wauterw-k8s:/home/cisco# systemctl restart docker




wim@k8s-master:/etc$ kubectl get nodes
NAME          STATUS   ROLES    AGE   VERSION
k8s-master    Ready    master   13m   v1.18.2
k8s-worker1   Ready    <none>   10m   v1.18.2
k8s-worker2   Ready    <none>   10m   v1.18.2