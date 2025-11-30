# Docker Commands Guide

```bash
# docker run creates and starts a container.
# -d runs the container in the background.

docker run -d -p 6379:6379 --name redis-older redis:7.0

docker run -d \
  -p <HOST_PORT>:<CONTAINER_PORT> \
  --name <CONTAINER_NAME> \
  <IMAGE_NAME>:<VERSION_TAG>

# download image from docker hub
docker pull <IMAGE_NAME>

# list all containers including stopped ones
docker ps -a
```

View container log

```bash
# container logs
docker logs <container_name>

# container logs in real time
docker logs -f <container_name>
```

Enter inside a running container

```bash
# If container uses bash
docker exec -it <container_name> bash

# If container uses sh
docker exec -it <container_name> sh
```

Docker Network
```bash
# Shows all existing Docker networks
docker network ls

# Create new docker network
docker network create mongo-network

# -e sets environment varible
# Uses mongo-network to communicate with other containers
docker run -p 27017:27017 -d -e MONGO_INITDB_ROOT_USERNAME=admin -e MONGO_INITDB_ROOT_PASSWORD=1234 --name mongodb --net mongo-network mongo

# Running mongo express, a web interface for MongoDB
docker run -d -p 8081:8081 -e ME_CONFIG_MONGODB_ADMINUSERNAME=admin -e ME_CONFIG_MONGODB_ADMINPASSWORD=1234 -e ME_CONFIG_MONGODB_SERVER=mongodb --net mongo-network --name mongo-express mongo-express
```

Enter Inside Running Container
```bash
# if bash
docker exec -it <container_name> bash

# if using sh
docker exec -it <container_name> sh

```

Build an image from dockerfile

```bash
docker build -t <image_name>:<tag> .

# tag an image name
docker tag <source_image> <target_image>

# Save docker image to tar file
docker save -o <OUTPUT_FILE>.tar <IMAGE_NAME>:<TAG>

# Load docker image from tar file
docker load -i <OUTPUT_FILE>.tar
```
Inspect docker

```bash
# Inspect a container
docker inspect <container_name>

# Inspect a network
docker network inspect <network_name>
```

Docker cleanup

```bash
docker container prune

docker image prune
docker system prune

```
Docker stats
```bash
docker stats
```


