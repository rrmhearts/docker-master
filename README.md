# Docker Mastery Cheat Sheet

## Versions and Install docker-machine
```
cat /etc/group | grep docker
docker --version
docker version
docker-compose --version

docker info
docker       # see help
```

### Install docker-machine separately
On Linux, docker-machine is not included in main install.
```
curl -L https://github.com/docker/machine/releases/download/v0.13.0/docker-machine-`uname -s`-`uname -m` >/tmp/docker-machine && chmod +x /tmp/docker-machine && sudo cp /tmp/docker-machine /usr/local/bin/docker-machine
docker-machine --version
```

## Running and seeing containers
```
docker container run --publish 80:80 nginx
docker container run --publish 80:80 --detach nginx
docker ps
docker container stop fe324b77c79c

docker container ls -a | grep nginx
docker container run --publish 80:80 --detach --name webhost nginx
docker container ls

docker container run --publish 80:80 --detach --name webhost nginx
docker container ls -a  | grep webhost
docker container kill c753834789d9
docker rm c7538
docker container ls -a  | grep webhost
docker container run --publish 80:80 --detach --name webhost nginx
docker container logs webhost
docker container top webhost
docker container --help
```

## Deleting containers

```
docker container ls -a
docker container rm 1714 45b5 9136 494d bb36 4b4b 0d8d
docker container ls -a
docker container rm -f f56 # force kill
```
## Running multiple containers
```
docker run --name mongo -d mongo
docker top mongo
ps aux | grep mongod
docker stop mongo
docker ps
docker top mongo
ps aux | grep mongod
docker container run --publish 80:80 --detach nginx
docker container run --publish 8080:80 --detach httpd
docker container run --publish 3306:3306 --env MYSQL_ROOT_PASS=yes --detach mysql
docker container ls
docker container run --publish 3306:3306 --env MYSQL_RANDOM_ROOT_PASSWORD=yes --detach mysql
docker container ls
docker container logs mysql
docker container logs cfa5
ps aux | grep mysql
ps aux | grep httpd
ps aux | grep nginx
docker container ls
docker container stop charming_hypatia tender_diffie infallible_bose 
docker container rm cfa cfe 3a
docker ps
docker container ls
docker container run --publish 8080:80 --detach httpd
docker ps
docker container stop modest_mclean 
docker container rm modest_mclean 
docker ps -a
docker container rm f7 a6
docker container run --publish 3306:3306 --env MYSQL_RANDOM_ROOT_PASSWORD=yes --detach mysql
docker container run --publish 80:80 --detach nginx
docker ps
```
## Inspecting containers
```
docker container top mysql
docker container top 
docker container top d08
docker container top nginx
docker container top pensive_fermi 
docker container top strange_bardeen 
docker container inspect pensive_fermi 
docker container stats
docker container ls
docker container run -it --name proxy nginx bash
docker container ls
docker container ls -a
docker container run -it --name ubuntu ubuntu
docke container ls -a
docker container ls -a
docker container start -ai --name ubuntu ubuntu
docker container start -ai ubuntu
docker container exec -it mysql bash
docker ps
```
```
docker container exec -it pensive_fermi bash
docker ps
docker pull alpine
docker image ls
docker container run -it alpine bash
docker container run -it alpine sh
docker image
docker image ls
docker container ls
docker container ls -a
docker container port strange_bardeen 
docker container port pensive_fermi 
docker container port goofy_murdock 
docker container run -p 80:80 --name webhost -d nginx
docker ps
docker container port strange_bardeen 
docker container inspect strange_bardeen 
```

## Network / Bridge usage
A network bridge allows multiple docker containers to communicate with the outside world and each other. Creating a new bridge by `docker network create my_app_net` will include a DNS server that allows containers to communicate with each other through their container name as a hostname.
```
docker container inspect --format '{{ .NetworkSettings.IPAddress }}' strange_bardeen 
ifconfig en0
ifconfig eno0
ifconfig 
docker container run -p 80:80 --name webhost -d nginx:alpine
docker container run -p 80:80 --name nginx_alpine -d nginx:alpine
docker container ls
docker container stop strange_bardeen 
docker container run -p 80:80 --name nginx_alpine -d nginx:alpine
docker container rm strange_bardeen 
docker network ls
docker network inspect 
docker container run -p 80:80 --name nginx_alpine -d nginx:alpine
docker container ls
docker container ls -a | grep nginx
docker container rm nginx_alpine webhost 
docker container run -p 80:80 --name nginx_alpine -d nginx:alpine
docker network inspect nginx_alpine
docker network inspect bridge 
docker network ls
docker network create my_app_net
docker network ls
docker network inspect my_app_net 
```
### Connect container to a new bridge.
```
#docker container run -p 80:80 --network my_app_net --name nginx_my_app -d nginx:alpine

docker container run -d --name new_nginx --network my_app_net nginx:alpine 
docker container ls
docker network inspect my_app_net 
docker network connect my_app_net nginx_alpine      # connect container to bridge!
docker container inspect nginx_alpine               # now two bridges w/ two IPs
docker network disconnect my_app_net nginx_alpine 
docker container inspect nginx_alpine 
docker container ls
docker stop pensive_fermi 
docker network inspect my_app_net 
docker container run -d --name my_nginx --network my_app_net nginx:alpine
docker container exec my_nginx -it ping
docker container exec -it  my_nginx ping new_nginx 
docker container exec -it  new_nginx ping my_nginx 
docker container --help
docker container create --help
docker container stats my_nginx 
docker container inspect my_nginx 
docker container run -it centos:7 bash
docker container run -it ubuntu:14.04 bash
docker container prune                          # will delete containers that are not running!!
docker container ls
docker container run --rm -it ubuntu:14.04 bash # will rm container when quit running.
docker container ls
docker container start --rm -it ubuntu:14.04 bash
docker container start -it ubuntu:14.04 bash
docker container start -ia ubuntu:14.04 bash
docker container start -ia ubuntu:14.04
docker container ls
docker container stop --rm nginx_alpine 
docker container stop nginx_alpine 
docker run -d --name elasticsearch --network my_app_net  --network-alias search elasticsearch:2
docker run -d --name elasticsearch_2 --network my_app_net  --network-alias search elasticsearch:2
docker container ls
docker run -it ubuntu bash
docker start -ai ubuntu bash
docker run -it ubuntu nslookup search --net
docker run -it ubuntu
docker ls
docker ps
docker run -it alpine nslookup search --net
docker run -it alpine nslookup --net search
docker run -it --network my_app_net alpine nslookup --net search
docker run -it --network my_app_net alpine nslookup  search
docker run -it --network my_app_net alpine nslookup search
docker run -it alpine nslookup search
docker run -it --network my_app_net alpine nslookup search
docker run -it --network my_app_net centos curl -s search:9200 --net
docker run -it --network my_app_net centos curl -s search:9200
docker container ls
docker run -it --network my_app_net alpine nslookup search
docker run -it --network my_app_net centos curl -s search:9200
docker container ls
docker container ls -a
docker container prune 
docker container ls -a
docker run -it --rm --network my_app_net centos curl -s search:9200
docker run -it --rm --network my_app_net alpine nslookup search
docker container ls -a
docker container rm elas*
docker container rm f1c 3b2
docker container rm f1c 3b2 -f
docker container ls -a
```

## Images
On docker hub, there are several tags for the same image. If you pull multiple tags you will get a new reference to the same image.
```
docker pull nginx:1.11.9-alpine
docker pull nginx:1.11.9
docker pull nginx
docker pull nginx:1.17
docker image ls | grep nginx
docker image ls
```
### Image layers
A container is an abstraction on top of an image. A history of image layers can be seen below. Layers are added as changes occur. A lot of the history are from Docker developers.
```
$ docker image history nginx:latest
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
f7bb5701a33c        12 days ago         /bin/sh -c #(nop)  CMD ["nginx" "-g" "daemon…   0B                  
<missing>           12 days ago         /bin/sh -c #(nop)  STOPSIGNAL SIGTERM           0B                  
<missing>           12 days ago         /bin/sh -c #(nop)  EXPOSE 80                    0B                  
<missing>           12 days ago         /bin/sh -c ln -sf /dev/stdout /var/log/nginx…   22B                 
<missing>           12 days ago         /bin/sh -c set -x     && addgroup --system -…   57.1MB              
<missing>           12 days ago         /bin/sh -c #(nop)  ENV PKG_RELEASE=1~buster     0B                  
<missing>           12 days ago         /bin/sh -c #(nop)  ENV NJS_VERSION=0.3.7        0B                  
<missing>           12 days ago         /bin/sh -c #(nop)  ENV NGINX_VERSION=1.17.6     0B                  
<missing>           12 days ago         /bin/sh -c #(nop)  LABEL maintainer=NGINX Do…   0B                  
<missing>           12 days ago         /bin/sh -c #(nop)  CMD ["bash"]                 0B                  
<missing>           12 days ago         /bin/sh -c #(nop) ADD file:04caaf303199
```
Each layer gets it's own SHA to show equality or not.
```
[[ Ubuntu --> Apt --> ENV var change]]
^^ One Image
[[                --> Copy ]]
^^ Second Image with shared layers.

[[ Custom --> Apache --> PORT 80 --> CopyA | CopyB ]]
^^ Two images, Five layers
```
Each time you run a container, a new image is created based off the base image. Changes in container produce **copy on write** new image by looking at diffs.

Inspect gives you the metadata about an image.
```
$ docker image inspect nginx:latest
[
    {
        "Id": "sha256:f7bb5701a33c0e572ed06ca554edca1bee96cbbc1f76f3b01c985de7e19d0657",
        "RepoTags": [
            "nginx:1.17",
            "nginx:latest"
        ],
        ...
          "Cmd": [
                "/bin/sh",
                "-c",
                "#(nop) ",
                "CMD [\"nginx\" \"-g\" \"daemon off;\"]"
            ],
        ...
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "NGINX_VERSION=1.17.6",
                "NJS_VERSION=0.3.7",
                "PKG_RELEASE=1~buster"
            ],
            "Cmd": [
    ...
```

### Image Tags and Upload
Tagged image as `rrmhearts/nginx` then pushed to my personal account.
```
docker image tag nginx rrmhearts/nginx
docker image ls | grep ngin
docker image push rrmhearts/nginx
docker image tag nginx rrmhearts/nginx:testing
docker image push rrmhearts/nginx:testing # will keep previous layers, just add testing label. IT KNOWS.

```