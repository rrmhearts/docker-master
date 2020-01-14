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
