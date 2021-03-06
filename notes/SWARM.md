# Docker Swarm

## Initialization
Docker Swarm is not turned on by default. You can see this in `docker info`. Turn it on:
```
docker swarm init
Swarm initialized: current node (wsod2fsq4m1ed74m1lf56dkk3) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-3j3q8i6sa3wkzs29s9fd8mhmk54qrjuqefj73m7840lc27q7rx-cbd3nvv8l8r5t2txypxyyju37 10.5.4.241:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```
Commands fall into one of three categories: `docker node` (mange a node), `docker swarm` (manage swarm) and `docker service` (create, run, scale services/containers). The last of which replaces `docker run` when using swarms.

## Create/Update a service
Similar to docker run: `docker service create alpine ping 8.8.8.8`. Returns *service* id.

Update the number of containers running: `docker service update [container_name] --replicas 3`.

`docker update` allows the user to change the configuration of a container. `docker service update` is similar but with a lot more options. May require container restart, but will update with consistent availability.

## Remove
`docker container rm -f ...` will not remove a swarm. The manager will re-create the nodes. In order to remove a swarm service, you must do `docker service rm festive_volhard`.

## Swarm Cluster
An example of a swarm cluster with 3 nodes.
```
# Init swarm, returns key line. Use key line on other nodes.
sudo docker swarm init
sudo docker swarm join --token SWMTKN-1-56m3hpab934q1kpjpkot0wys192wnxaldxk68iq670o7qanfip-3vaoueqlscpz9fk7ka8iwusmk 172.31.32.181:2377

# See nodes in swarm
sudo docker node ls
ID       HOSTNAME       STATUS   AVAILABILITY    MANAGER STATUS   ENGINE VERSION
hxb4gn   172-31-32-36    Ready     Active         Reachable       18.09.9-ce
87hke1*  172-31-32-181   Ready     Active         Leader          18.09.9-ce
plylul   172-31-35-141   Ready     Active         Reachable       18.09.9-ce

# Make node a manager (puts reachable status)
sudo docker node update --role manager ip-172-31-32-36

# Give task to swarm, will run 1 on each node if 3 nodes
sudo docker service create --replicas 3 alpine ping 8.8.8.8

# Similar to docker-compose drupal site. Hostname is psql.
sudo docker service create --name psql --network mydrupal -e POSTGRES_PASSWORD=pass postgres:11
sudo docker service create --name drupal --network mydrupal -p 80:80 drupal

# Update number of replicas!
sudo docker service update --replicas 5 epic_wiles

# See all services
sudo docker service ls

# Show service status.
sudo docker service ps epic_wiles
```

## Swarm Overlay Networking
The overlay network driver creates a distributed network among multiple Docker daemon hosts. This network sits on top of (overlays) the host-specific networks, allowing containers connected to it (including swarm service containers) to communicate securely. Docker transparently *handles routing of each packet* to and from the correct Docker daemon host and the correct destination container.

Overlay acts like everything is on the same subnet. It permits incoming traffic on node to be re-directed to the node where the service is running. For more information see [Docker Docs](https://docs.docker.com/network/overlay/).
**Note:** *Postgres hostname is still psql.*
```
docker network create --driver overlay mydrupal
docker service create --name psql --network mydrupal -e POSTGRES_PASSWORD=pass postgres
docker service ls       # show number of replicas
docker service ps psql  # show node running on - node1

docker service create --name drupal --network mydrupal -p 80:80 drupal

# This will run a command over and over again.
watch docker service ls 

docker service ps drupal # will run on node2
```
### Aside on Swarm Networks
**Overlay networks** manage communications among the Docker daemons participating in the swarm. You can create overlay networks, in the same way as user-defined networks for standalone containers. You can attach a service to one or more existing overlay networks as well, to enable service-to-service communication. Overlay networks are Docker networks that use the overlay network driver.

The **ingress network** is a special *overlay network* that facilitates load balancing among a service's nodes. When any swarm node receives a request on a published port, it hands that request off to a module called IPVS. IPVS keeps track of all the IP addresses participating in that service, selects one of them, and routes the request to it, over the ingress network.

The ingress network is created automatically when you initialize or join a swarm. Most users do not need to customize its configuration, but Docker 17.05 and higher allows you to do so.

The **docker_gwbridge** is a *bridge network* that connects the overlay networks (including the ingress network) to an individual Docker daemon's physical network. By default, each container a service is running is connected to its local Docker daemon host's docker_gwbridge network.

The docker_gwbridge network is created automatically when you initialize or join a swarm. Most users do not need to customize its configuration, but Docker allows you to do so.

## Swarm Routing Mesh
Rouing mesh routes ingress (incoming) packets for a service to proper task. It distributes packets to the right services. It uses IPVS from Linux Kernel. It load balances swarm services across their tasks. Containers talk to other containers using an *overlay network* using VIP. External traffic incoming to published ports can come into any node in the swarm. It will then re-route to the proper container based on load balancing. You don't care what node a service is on, it can vary depending on failures and load balancing.

A **swarm load balancer** exists on each node and may re-direct traffic to anther node.

### Example of Use
```
docker service create --name search --replicas 3 -p 9200:9200 elasticsearch:2
```
See load balancing in action by `curl localhost:9200`. Stateless load balancing. It is a TCP/IP load balancer, not DNS.
This can be overcome with **Nginx** or **HAProxy LB proxy** or Docker Enterprise.

## Swarm Stacks
Stacks accept compose files as their declarative definition for services, networks, and volumes. Now it's possible to make use of `docker-compose.yml` files to bring up stacks of Docker containers, without having to install Docker Compose.

The command is called `docker stack`, and it looks exactly the same to `docker-compose`. Here’s a comparison:
```
$ docker-compose -f docker-compose.yml up

$ docker stack deploy -c docker-compose.yml somestackname
```
However, in stacks, you cannot `build` images like in docker-compose. But you can now `deploy` in compose file to determine # replicas and network things. `docker-compose` will ignore `deploy` commands and `docker stack` will ignore `build` commands.

In **stack** you have multiple services, volumes and networks under one stack. Bring up all your services in one file. A stack only runs on one swarm. An example yml service can be seen below, a full picture can be found at [example-voting](./resources/swarm/swarm-stack-1/example-voting-app-stack.yml).
```
services:
  redis:
    image: redis:alpine
    ports:
      - "6379"
    networks:
      - frontend
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure
```

## Secrets Storage
Secure solution for storing secret data. Usernames, passwords, keys, certificates, etc. You need it but don't need it published! Supports strings and binaries. Secrets are stored in swarm manager nodes and then assigned to services. Only containers in assigned services can see them.
```
cat secret.txt
docker secrete create psql_user secret_username.txt

echo "mypass" | docker secret create psql_pass -

docker secret ls
docker service create --name psql --secret psql_user --secret psql_pass -e POSTGRES_PASSWORD_FILE=/run/secrets/psql_pass -e POSTGRES_USER_FILE=/run/secrets/psql_user postgres
```
This is mapped into tmpfs. And you must shell into the container in order to see the secrets.
```
docker exec -it psql.1.fjxa37tl79hzhwx2xesxvdkfd bash
cat /run/secrets/psql_user
```
Services are immutable, if you remove a secret from an existing container, it will destroy and recreate container. `docker service update --secret-rm`.

### Secrets on Stacks
Version has to be **3.1** or greater to use stacks with secrets. See [secrets-sample](resources/sample/secrets-sample-2) for example code.
```
version: "3.1"

services:
  psql:
    image: postgres
# assign secrets to this container.
    secrets:
      - psql_user
      - psql_password
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/psql_password
      POSTGRES_USER_FILE: /run/secrets/psql_user

secrets:
  psql_user:
    file: ./psql_user.txt
  psql_password:
    file: ./psql_password.txt
```
From that folder, the stack can be run accordingly:
```
docker stack deploy -c docker-compose.yml mydb
docker secret ls

# This will remove secrets as well!
docker stack rm mydb
```

#### Secrets on Docker-Compose
Secrets are **NOT** secret in Docker-compose, however they do work. They are essentially bind mounted into the image. This is only for development and testing off of a swarm. See [secrets-sample-2](./resources/sample/secrets-sample-2) and run `docker-compose up -d` for testing. You can see the "secret" by `docker-compose exec psql cat /run/secrets/psql_user`.

## Single Compose, Full App Lifestyle
We will be using [swarm-stack-3](./resources/swarm/swarm-stack-3) to see the full app lifestyle of development, build, test, and deploy. Docker compose will read in a file called `docker-compose.override.yml` and override any settings default in the `docker-compose.yml`.

### Development Environment
```docker-compose up -d```
This will use the `docker-compose.yml` as nomal. It will also read in the `docker-compose.override.yml` and override what it may. You know this was called in when you inspect `docker inspect swarm-stack-3_drupal_1`. This is for a **local** *development* environment. (bring down when down).

### Test Environment
```docker-compose -f docker-compose.yml -f docker-compose.test.yml up -d```
The base file always comes first, and the test file comes second. The second *yml* file will override the first. This is for a *test* or CI environment and may be remote.

### Production Environment
```docker-compose -f docker-compose.yml -f docker-compose.prod.yml config > output.yml```
For production, you can use config to combine the input files and create an output file to be used in production. This kind of command could be used in a remote *production* environment. Another option for a remote production environment is `docker stack deploy` using the listed `output.yml` file.

Look up **compose extends**. May not yet work in Stacks.

### Service Updates
Provides rolling replacement of tasks/containers in a service. Limits downtime. Will replace containers for most changes. Has many options to control the update *+70*. Many are create options and will usually change adding *-add* or *-rm* to them. Includes rollback and healthcheck options. Also has scale and rollback subcommands for quicker access: `docker service scale web=4` and `docker service rollback web`. A stack deplay when pre-existing will issue service updates.
* Update the image used to a newer version `docker service update --image myapp:1.2.1 <servicename>`
* Adding an environment variable or remove a port 
```docker service --env-add NODE_ENV=production --publish-rm 8080```
* Change number of replicas of two services `docker service scale web=8 api=6`
* Same command to update stack. Edit yml, then `docker stack deploy -c file.yml <stackname>`

**Live example**
```
docker service create -p 8088:80 --name web nginx:1.13.7
docker service ls

# scale up
docker service scale web=5 

# change image in service.
docker service update --image nginx:1.13.6 web 

# remove old port, publish new port.
docker service update --publish-rm 8088 --publish-add 9090:80

# re-balancing nodes. Will replace tasks and put on unused nodes.
docker service update --force web
```

### Healthchecks
Added in 1.12 and supported by everything. Docker engine will exec the command in the container. It expects exit 0 (OK) or exit 1 (Error). Three container states: *starting, healthy, unhealthy*. Much better then *"is binary still running?"* But not a external monitoring replacement. ***basic health***

Healthcheck status shows up in `docker container ls` and last 5 in `docker container inspect`. Docker does nothing with healthchecks but services will replace tasks. Will pause / change service update commands.
**Example**
```
docker container run \
  --health-cmd="curl -f localhost:9300/_cluser/health || false" \
  --health-interval=5s \
  --health-retries=3 \
  --health-timeout=2s \
  --health-start-period=15s \
  elasticsearch:2
```
If you put `localhost:9200` as the port, the health check will fail. Use `docker container ls` to see health under *STATUS*. In a Dockerfile, you use `HEALTHCHECK curl -f http://localhost/ || false`. OR
```
FROM nginx:1.13

HEALTHCHECK --interval=30s --timeout=3s \
   CMD curl -f http:http://localhost/ || exit 1
```

**Live Example**
```
docker container run --name p1 -d postgres
docker container ls

# Look at difference
docker container run --name p2 -d --health-cmd="pg_isready -U postgres || exit 1" postgres
docker container ls

docker container inspect p2
docker service create --name p1 postgres
docker service create --name p2 --health-cmd="pg_isready -U postgres || exit 1" postgres
```
