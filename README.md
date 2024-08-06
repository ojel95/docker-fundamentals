# docker-kubernetes-practical-guide

This is course to learn the fundamentals of Docker and Kubernetes. Here you can find exercises and notes made during the course.

## Key notes

### Docker

#### Images

- Images are read-only

- images have a layer based architecture. Dockerfile instructions represents a layers and every time you build an image Docker
caches the instruction result. So next time the images is rebuild it uses cache from unchanged previous layers. All instructions after
changed step will be executed again.

- When a container is run based on an image, an new extra read-write layer is added on top of the image.

## Docker commands

Check this cheat sheet to get most common commands [Docker cheat sheet](https://docs.docker.com/get-started/docker_cheatsheet.pdf)

### docker build

It is not good practice for the name and tag to be empty or none. Therefore, it is necessary to create with name and tag:

```
docker build -t website:latest .
```

### docker rmi

Removes existing images

```
docker rmi -f <image ID>
```

### docker run

To create and run a container from the terminal, use the following command:

```
docker run -it --rm -d -p 8080:80 --name web website
```

- -it: Interactively to view logs

- --rm: Removes container when it exits

- -d: The container runs in the background

- -p: Maps the container's port to the application's port to expose it

-p is the only required part when it comes to listening on a port. Still, it is a best practice to also add EXPOSE in the Dockerfile to document this behavior.

- --name: Name of the container

- Finally, add the image name

### docker start

This can start an existent Exited container without creating a new one.

```
docker start <CONTAINER_NAME/ID>
```

-a for attach mode
-i for interactive and allow STDIN

### docker attach

This will attach yourself to a detached container again.

```
docker container attach <CONTAINER_NAME>
```

### docker logs

This will print the past logs of the container. You can add -f (follow mode) to show logs in real time like attached mode.

### docker images

- To filter the list of docker images use --filter arg. In this example we filter by tag version:

```
docker images --filter=reference='*:1.0'
```

- To update image tag

```
docker image tag <previous-tag:version> <new-tag:version>
```

- Delete or untag an image tag. This is going to remove the specified tag but if you have multiple tags based on the same image it's going to keep the image with the other tags

```
docker rmi <tag:version>
```

- If you delete the image using the Image ID it is going to untag all tags associated to that image and delete the image since the Image ID
is the same for all created tags based on the same image.

```
docker rmi -f <IMAGE_ID>
```

- If you want to inspect an image and get info about it.

```
docker image inspect <IMAGE_ID>
```

- If you want to remove all images including tagged images:

```
docker image prune -a
```

### docker ps

- To get running container with its size:

```
docker ps --size
```

### docker stats

- To get the stats of resources usage by the containers:

```
docker stats
```

### docker cp

This allows you to copy files to/from running containers. The path of the container has same structure of an scp.

```
docker cp <CONTAINER_NAME>:/PATH/ <HOST_PATH>
```
