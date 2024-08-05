# docker-kubernetes-practical-guide

This is course to learn the fundamentals of Docker and Kubernetes. Here you can find exercises and notes made during the course.

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

To run a container from the terminal, use the following command:

```
docker run -it --rm -d -p 8080:80 --name web website
```

- -it: Interactively to view logs

- --rm: Removes previous versions of the container

- -d: The container runs in the background

- -p: Maps the container's port to the application's port to expose it

- --name: Name of the container

- Finally, add the image name

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
