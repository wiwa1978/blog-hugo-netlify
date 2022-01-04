Deze tutorial werd gedaan op een bestaande Gitlab en Drone in Cisco lab. Idee zou zijn om thuis Gitlab en Drone installatie te documenteren en dan vervolgens het docker rate-limiting probleem op te lossen. De screenshots en tekst zijn eigenlijk een leidraad.

### Create Gitlab repository

1.png

2.png

### Setup drone

3.png

4.png

5.png

6.png

```bash
cisco@jumphost-acc:~/Code$ git clone git@10.48.34.155:cisco/drone-example.git
Cloning into 'drone-example'...
time="2021-11-23T10:42:26Z" level=info msg="SSL_CERT_DIR is configured" ssl_cert_dir=/opt/gitlab/embedded/ssl/certs/
remote: Enumerating objects: 3, done.
remote: Counting objects: 100% (3/3), done.
remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
Receiving objects: 100% (3/3), done.
```

```yml
name: drone-example
kind: pipeline
type: docker

steps:
  - name: Test
    image: ubuntu
    commands:
      - echo "Working fine!"
```

```bash
cisco@jumphost-acc:~/Code/drone-example$ git add .
cisco@jumphost-acc:~/Code/drone-example$ git commit -m "Adding drone.yml"
[main 4db4c2f] Adding drone.yml
 1 file changed, 9 insertions(+)
 create mode 100644 .drone.yml
cisco@jumphost-acc:~/Code/drone-example$ git push origin main
time="2021-11-23T10:47:03Z" level=info msg="SSL_CERT_DIR is configured" ssl_cert_dir=/opt/gitlab/embedded/ssl/certs/
Enumerating objects: 4, done.
Counting objects: 100% (4/4), done.
Delta compression using up to 2 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 381 bytes | 381.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To 10.48.34.155:cisco/drone-example.git
   21191cc..4db4c2f  main -> main
```

7.png

8.png

9.png

```
cisco@aac-drone:~$ cat /home/cisco/.docker/config.json
{
	"auths": {
		"https://index.docker.io/v1/": {
			"auth": "d2l3...hdA=="
		}
	}
}
```

10.png

11.png

```yml
name: drone-example
kind: pipeline
type: docker

steps:
  - name: Test
    image: ubuntu
    commands:
      - echo "Working fine!"

image_pull_secrets:
  - dockercredentials
```

```bash
cisco@jumphost-acc:~/Code/drone-example$ git add .
cisco@jumphost-acc:~/Code/drone-example$ git commit -m "Adding credentials"
[main 753ef55] Adding credentials
 1 file changed, 3 insertions(+)
cisco@jumphost-acc:~/Code/drone-example$ git push origin main
time="2021-11-23T10:54:42Z" level=info msg="SSL_CERT_DIR is configured" ssl_cert_dir=/opt/gitlab/embedded/ssl/certs/
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 2 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 347 bytes | 347.00 KiB/s, done.
Total 3 (delta 1), reused 0 (delta 0)
To 10.48.34.155:cisco/drone-example.git
   4db4c2f..753ef55  main -> main
```

12.png

13.png
