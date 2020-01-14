# Persistent Data and Docker Compose

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