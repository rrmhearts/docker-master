# Docker Mastery Cheat Sheet

## Tables of contents

1. [Containers and Images](./notes/CONTAINERS.md)
2. [Data Volumes + Docker Compose](./notes/VOLUMES.md)
3. [Docker Swarm](./notes/SWARM.md)
4. [Docker Registries](./notes/REGISTRY.md)
5. [Kubernetes](./notes/KUBERNETES.md)
6. [How Does Docker Work](#How-Docker-Works)
7. [Further Notes](#Further-Notes)

*Below is repeated from the files except* **Docker Swarm** through **Kubernetes**, (3)-(5). *that is only found in [SWARM.md](./notes/SWARM.md) etc.*

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
docker container [stop|rm] fe324b77c79c

docker container run --publish 80:80 --detach --name webhost nginx
docker container ls -a  | grep webhost
docker container logs webhost            # see logs piped to stdout
docker container top webhost             # see running process associated w/ container.
docker container [kill|rm] c753834789d9
docker container ls -a  | grep webhost
```

## Deleting containers
Containers must be stopped before they can be removed.
```
docker container [stop|rm] fe324b77c79c
docker container rm 1714 45b5 (...)      # rm containers
docker container rm -f f56               # force rm even if not stopped
```
## Running multiple containers
```
docker run --name mongo -d mongo
docker top mongo
ps aux | grep mongod
docker stop mongo
#####################################################################
docker container run --publish 80:80 --detach nginx
docker container run --publish 8080:80 --detach httpd
docker container run --publish 3306:3306 --env MYSQL_RANDOM_ROOT_PASSWORD=yes --detach mysql
docker container ls
docker container logs mysql    # look for randomized root password in logs. Setup by env variable.
ps aux | grep mysql
ps aux | grep httpd
ps aux | grep nginx
docker container ls
docker container [stop|rm] charming_hypatia tender_diffie infallible_bose 
```
## Inspecting containers
```
docker container top mysql
docker container top d08
docker container top nginx
docker container top pensive_fermi 
docker container inspect pensive_fermi # inspect config file
docker container stats                 # see stats about container.
```
## Starting containers that already exist.
```
docker container run -it --name proxy nginx bash
docker container run -it --name ubuntu ubuntu
docke container ls -a
docker container start -ai --name ubuntu ubuntu
docker container start -ai ubuntu
docker container exec -it mysql bash
docker ps
```
## Pull and run commands in an image.
```
docker pull alpine
docker image ls
docker container run -it alpine bash
docker container run -it alpine sh
docker image
docker image ls
docker container ls

# See port and ipaddress information
docker container run -p 80:80 --name webhost -d nginx
docker ps
docker container port strange_bardeen 
docker container inspect strange_bardeen 
```

## Network / Bridge usage
A network bridge allows multiple docker containers to communicate with the outside world and each other. Creating a new bridge by `docker network create my_app_net` will include a DNS server that allows containers to communicate with each other through their container name as a hostname.
```
docker container inspect --format '{{ .NetworkSettings.IPAddress }}' strange_bardeen 
ifconfig 
docker container run -p 80:80 --name nginx_alpine -d nginx:alpine
docker network inspect 
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
docker container run -d --name new_nginx --network my_app_net nginx:alpine 
docker container ls
docker network inspect my_app_net 
docker network connect my_app_net nginx_alpine      # connect container to bridge!
docker container inspect nginx_alpine               # now two bridges w/ two IPs
docker network disconnect my_app_net nginx_alpine 
docker container inspect nginx_alpine 
docker stop pensive_fermi 

docker network inspect my_app_net 
docker container run -d --name my_nginx --network my_app_net nginx:alpine
docker container exec -it  my_nginx ping new_nginx    # ping between two containers using my_app_net hostnames
docker container exec -it  new_nginx ping my_nginx 
docker container stats my_nginx 
docker container inspect my_nginx 

docker container run -it centos:7 bash
docker container run -it ubuntu:14.04 bash
docker container prune                             # will delete containers that are not running!!
docker container run --rm -it ubuntu:14.04 bash    # will rm container when quit running.
docker container start -ia ubuntu:14.04 bash       # start existing container, rm when closed.
docker container start -ia ubuntu:14.04
docker container ls
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
docker run -it --network my_app_net alpine nslookup  search   # lookup search, resolves to 1/2 elasticsearch*
docker run -it --network my_app_net alpine nslookup search 
docker run -it alpine nslookup search
docker run -it --network my_app_net alpine nslookup search

docker run -it --network my_app_net centos curl -s search:9200 --net
docker run -it --network my_app_net centos curl -s search:9200
docker container ls
docker run -it --network my_app_net alpine nslookup search
docker run -it --network my_app_net centos curl -s search:9200
docker run -it --rm --network my_app_net centos curl -s search:9200
docker run -it --rm --network my_app_net alpine nslookup search
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

## Dockerfile
It's a recipe for creating an image. See `dockerfile-sample-1/` in `resources/`. 
Five stanzas to a generic Dockerfile:
1. FROM (req)
2. ENV
3. RUN 
4. EXPOSE
5. CMD (req)

Caches steps in build, won't re-run. Short build times. But things after new changes will have to re-run because of dependencies. If part of file is going to change often, write it LATE in the Dockerfile.
```
docker image build -t custom_nginx .
docker image ls

# Make a change in Dockerfile, watch build use cached layers.
docker image build -t custom_nginx . 
```

See `dockerfile-sample-2/` in `resources/`.
```
# Default
$ docker container run -p 80:80 --rm nginx

# W/ New Dockerfile
$ docker image build -t nginx-with-html .
Sending build context to Docker daemon  3.584kB
Step 1/3 : FROM nginx:latest
 ---> f7bb5701a33c
Step 2/3 : WORKDIR /usr/share/nginx/html
 ---> Running in a5be4f14ba3f
Removing intermediate container a5be4f14ba3f
 ---> e7d0440a9409
Step 3/3 : COPY index.html index.html
 ---> e23a2320335e
Successfully built e23a2320335e
Successfully tagged nginx-with-html:latest

# Run new image!
docker container run -p 80:80 --rm nginx-with-html
```

## Volumes
Persistent data that outlives the container is possible. We want containers to be immutable, but we may also be storing a database. The database should live as a volume apart from the container.
```
# Run mysql with NAMED volume at default location. Normal id is a hash, use a name.
docker container run -d --name mysql -e MYSQL_ALLOW_EMPTY_PASSWORD=true -v mysql-db:/var/lib/mysql mysql
docker volume ls
# See the volume attached is named appropriately.
docker container inspect mysql
```
### Bind mounts
Mapping host file or dir to container file or dir. It's a link. Allows for persistent data as host files.
Mounting is similar to choosing a volume path. Can maintain container code from host.
```
# use local index.html linked to nginx/html in container.
cd resources/dockerfile/dockerfile-sample-2
docker container run -d --name nginx -p 80:80 -v $(pwd):/usr/share/nginx/html nginx
```

## Docker compose
### yml
What is in a `docker-compose.yml` file?
```
version: '3.1'  # if no version is specificed then v1 is assumed. Recommend v2 minimum
services:  # containers. same as docker run
  servicename: # a friendly name. this is also DNS name inside network
    image: # Optional if you use build:
    command: # Optional, replace the default CMD specified by the image
    environment: # Optional, same as -e in docker run
    volumes: # Optional, same as -v in docker run
  servicename2:
volumes: # Optional, same as docker volume create
networks: # Optional, same as docker network create
```
### docker-compose CLI
```
git clone github.com/some/software
docker-compose [-d] up
docker-compose down [-v] [--rmi local] ## remove volumes (v) and/or images (rmi)

docker-compose ps
docker-compose top
```
## Cleanup
### Prune to keep system clean
You can use "prune" commands to clean up images, volumes, build cache, and containers. Examples include:

- `docker image prune` to clean up just "dangling" images

- `docker system prune` will clean up everything

- The big one is usually `docker image prune -a` which will remove all images you're not using. Use docker system df to see space usage.

Remember each one of those commands has options you can learn with `--help`.

Here's a YouTube video I made about it: https://youtu.be/_4QzP7uwtvI

#### Remove dangling images
Images with `<none>` as the repo and tag are "dangling" dependencies if you've deleted some other images.
```
docker rmi $(docker images -f "dangling=true" -q)
```

# How Docker Works
Every Docker container is a Linux namespace. Docker executes processes under a specific namespace in Linux. Docker is written in Go and takes advantages of the Linux kernel to function. It uses `namespaces` to provide isolated workspaces called the *container*. When you run a container, Docker creates a set of namespaces for that container, such as: 
* `pid` (process isolation)
* `net` (network interfaces)
* `ipc` (managing access to IPC resources)
* `mnt` (managing filesystem mount points)
* `uts` (isolating kernel and version identifiers)
Docker also uses `control groups` to limit an application to a specific set of resources, including files, hardware and constraints on memory available, etc.

A container may share the same user ID as the  host user, also, process running in a container are running on the *host*, but have different PIDs between the two. Run `watch ps ax` inside a container and look for it on the host... 

Use `pstree -p` and you can find the parent of the `watch..` process running in the container. It is a child of `systemd`-->`containerd`-->`containerd-shim`-->`bash`-->`watch`. *Containerd* is a container runtime that manages the container lifecycle of the host system (storage, execution, supervision, network attachements, etc). Containerd requires runc. *Runc* is a CLI tool for spawning and running containers.

Docker is a client CLI tool that communicates with the Docker daemon (`dockerd`) over REST or unix sockets. By default, the Docker daemon automatically starts containerd. Containerd uses runc to spawn and run containers. Let's use ```sudo strace -f -p `pid containerd` -o strace_log``` and start a docker container with `sudo docker run -d --rm -i ubuntu...`. Look at the syscall trace: search for `exec` syscalls. (look for runc, unshare, containerd, etc). `unshare` is called by runc. Unshare allows a process to disassociate parts of its execution context that are currently being shared with other processes. It will disassociate, among other things, the PID: a new PID namespace will be created for its children. It's first child process will have the PID of 1. `clone` is used to create this child. In the namespace, each new child of runc will have PIDs of 1,2,3,4... but outside the namespace where strace is running, they will have their normal, high-valued PID values.

The PID namespace lives in the host's universe, but also has it's own *microuniverse* where it has a new INIT and reset of PIDs etc. 
So, `containerd`-->`runc` ----> `runc(1)` --> `runc(2)`...

You can check the namespace of a file in the proc filesystem. Compare the following: 
* `pidof watch` (30232) and `sudo ls -lah /proc/30232/ns` (container process namespace)
* `sudo ls -lah /proc/$$/ns` (current shell namespace)
But they share the same user namespace. Namespaces are a kernel feature. Docker is a kernel feature.

Use the PID of watch on the host system, you can log into the docker container using: `sudo nsenter -t 27585 -m -u -i -n -p /bin/bash`. How does this differ from `docker exec -it container_name /bin/bash`??? Look at `id` and `mount`, it will match docker. One differnce is that, nsenter will make you *truly* root inside the container. Docker exec will drop down priveleges to normal user level. Nsenter can be used to install new tools that require root, instead of building sudo into the container. You can also use `docker exec --user root`.

## Kernel namespaces
**Namespaces** are a feature of the Linux kernel that partitions kernel resources such that one set of processes sees one set of resources while another set of processes sees a different set of resources. The feature works by having the same namespace for a set of resources and processes, but those namespaces refer to distinct resources. Resources may exist in multiple spaces. Examples of such resources are process IDs, hostnames, user IDs, file names, and some names associated with network access, and interprocess communication.

## Implementation
The kernel assigns each process a symbolic link per namespace kind in /proc/<pid>/ns/. The inode number pointed to by this symlink is the same for each process in this namespace. This uniquely identifies each namespace by the inode number pointed to by one of its symlinks.

Reading the symlink via readlink returns a string containing the namespace kind name and the inode number of the namespace.

### Syscalls
Three syscalls can directly manipulate namespaces:

* *clone*, flags to specify which new namespace the new process should be migrated to.
* *unshare*, allows a process (or thread) to disassociate parts of its execution context that are currently being shared with other processes (or threads)
* *setns*, enters the namespace specified by a file descriptor.

## Docker usage
Docker is entirely based around *kernel namespaces*. A namespace categories and isolates PIDs from some perspective. A process will have multiple PIDs based on what namespace you are looking at it from.
* `getpid` will look up PID inside a namespace. Uses the current process to get namespace, uses namespace to get the right PID.

Root access and/or kernel modules that go around namespace isolation will destroy any Docker isolation/security. Docker containers can run in **privileged** mode which would allow you to install kernel modules and have complete access to `/dev`, etc. Watch out! (someone could mount the root filesystem)

# Further Notes
## Docker in Production
These tools may be used for production. Choose one from each category.

| Concern            | Solution                    |
|--------------------|-----------------------------|
| Swarm GUI          | Portainer.io                |
| Central Monitoring | Librato, Sysdig             |
| Central Loggin     | Docker for AWS/Azure        |
| Layer 7 Proxy      | Flow-Proxy, Traefik         |
| Registry           | Docker Hub, Quay            |
| CI/CD              | Codeship, TravisCI, Jenkins |
| Storage            | Docker for AWS/Azure        |
| Networking         | Docker Swarm                |
| Orchestration      | Docker Swarm, Kubernetes    |
| Runtime            | Docker                      |
| HW/OS              | Docker for AWS/Azure        |

## Security notes
* See [Brett's list](https://github.com/BretFisher/ama/issues/17)
* Create and change user from root if possible in container. See [https://github.com/BretFisher/dockercon19/blob/master/1.Dockerfile](https://github.com/BretFisher/dockercon19/blob/master/1.Dockerfile). DO not run as root inside container...
* User namespaces need to be enabled. So "root user in a container isn't really root on the host."
* Use snyk or github to scan for vulnerability issues.
* Use an image/package scanner for CVE vulnerabilities. Try **Trivy**, Clair, Quay, or aqua MicroScanner
* Falco identifies bad behavior in containers. Made by Sysdig. If you did something you shouldn't have, Falco will audit and log it.
* Content Trust, sign code, only signed code can be run. Requires code and image signing.
* Lock down your containers as best as possible. Use best security practices for each container such as modifying security policies for a PostgreSQL database.
* Docker root-less. Run *dockerd* daemon as a normal user on the host. **NEW** feature, may want to wait on it. Will prevent virtual networking which requires root.

## DevOps Notes
* Do not run a DB in a container unless you want multiple separate DBs on one machine. Cloud hosted solutions or bare-metal are better options. 
* Always use Swarm for single node in production instead of Docker Compose. You can prevent downtime and replace/update containers without customers losing service.
* No hard coded environment that would change between different environments. Pull environment variables out of app. **Strict separation of config from code.** See the *Twelve Factor App* principles. Store config in environment variables that are passed into the container. You can pass ENV into a docker container through an entrypoint script.