# Named volumes
```
docker container run -d --name postgres_9.6.1 -e POSTGRES_PASSWORD=password -v postgres-db:/var/lib/postgresql/data postgres:9.6.1
docker container -f postgres_9.6.1 ## -f lets you watch logs
docker inspect postgres_9.6.1
docker volume ls
docker container rm -f postgres_9.6 
docker container run -d --name postgres_9.6.2 -e POSTGRES_PASSWORD=password -v postgres-db:/var/lib/postgresql/data postgres:9.6.2
docker container -f postgres_9.6.2
docker inspect postgres_9.6.2
docker volume ls              ## uses same volume as the first image.
docker rm -f postgres_9.6.2 
docker volume rm postgres-db
```