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