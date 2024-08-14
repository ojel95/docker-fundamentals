# docker-kubernetes-practical-guide

This is course to learn the fundamentals of Docker and Kubernetes. Here you can find exercises and notes made during the course.

## Key notes

### Images

- Images are read-only - once they're created, they can't change (you have to rebuild them to update them).

- images have a layer based architecture. Dockerfile instructions represents a layers and every time you build an image Docker
caches the instruction result. So next time the images is rebuild it uses cache from unchanged previous layers. All instructions after
changed step will be executed again.

- Containers on the other hand can read and write - they add a thin "read-write layer" on top of the image. That means that they can make changes to the files and folders in the image without actually changing the image.

### Data & Volumes

Even with read-write Containers, two big problems occur in many applications using Docker:

1. Data written in a Container doesn't persist: If the Container is stopped and removed, all data written in the Container is lost

2. The container Container can't interact with the host filesystem: If you change something in your host project folder, those changes are not reflected in the running container. You need to rebuild the image (which copies the folders) and start a new container

Problem 1 can be solved with a Docker feature called "Volumes". Problem 2 can be solved by using "Bind Mounts".

### Volumes

Volumes are folders (and files) managed on your host machine which are connected to folders/files inside of a container.

There are **two types of Volumes**:

- **Anonymous Volumes**: Created via `-v /some/path/in/container` and removed automatically when a container is removed because of `--rm` added on the `docker run` command
- **Named Volumes**: Created via `-v some-name:/some/path/in/container` and NOT removed automatically

With Volumes, **data can be passed into a container** (if the folder on the host machine is not empty) and it can be saved when written by a container (changes made by the container are reflected on your host machine).

**Volumes are created and managed by Docker** - as a developer, you don't necessarily know where exactly the folders are stored on your host machine. Because the data stored in there is **not meant to be viewed or edited by you** - use "Bind Mounts" if you need to do that!

Instead, especially **Named Volumes** can help you with **persisting data**.

Since data is not just written in the container but also on your host machine, the **data survives even if a container is removed** (because the Named Volume isn't removed in that case). Hence you can use Named Volumes to persist container data (e.g. log files, uploaded files, database files etc).

Anonymous Volumes can be useful for ensuring that some Container-internal folder is **not overwritten** by a "Bind Mount" for example.

By default, **Anonymous Volumes are removed** if the Container was started with the `--rm` option and was stopped thereafter. They are **not removed** if a Container was started (and then removed) without that option.

**Named Volumes are never removed**, you need to do that manually (via `docker volume rm VOL_NAME`, see reference below).

### Bind Mounts

**Bind Mounts are very similar to Volumes** - the key difference is that you, the developer, set the path on your host machine that should be connected to some path inside of a Container.

You do that via `-v /absolute/path/on/your/host/machine:/some/path/inside/of/container`.

The path in front of the `:` (i.e. the path on your host machine, to the folder that should be shared with the container) has to be an absolute path when using `-v` on the `docker run` command.

Bind Mounts are very useful for sharing data with a Container which might change whilst the container is running - e.g. your source code that you want to share with the Container running your development environment.

Don't use Bind Mounts if you just want to persist data - Named Volumes should be used for that (exception: You want to be able to inspect the data written during development).

In general, Bind Mounts are a great tool during development - they're not meant to be used in production (since your container should run isolated from its host machine).

### ENVironment variables

- This can be defined in the Dockerfile using `ENV <KEY> <VALUE>`. Ex. In node apps you can use this env variable using `process.env.ENV_VAR`

- It can be also defined directly in the docker run command adding the `--env KEY=VALUE` parameter.

- You can use a env file, defining the file with a key-value pair per line and then call it in the docker run command with `--env-file FILE_PATH`. The path can be relative. The file content should be like this `PORT=80`.

- Depending on which kind of data you're storing in your environment variables, you might not want to include the secure data directly in your Dockerfile. Instead, go for a separate environment variables file which is then only used at runtime (i.e. when you run your container with docker run). Otherwise, the values are "baked into the image" and everyone can read these values via `docker history <image>`.

### ARGument variables

- It allows to build multiple images with specific variable values without changing the Dockerfile each time.
- This is declared in the Dockerfile using `ARG ARG_VARIABLE_NAME=DEFAULT_VALUE`. The default value is not mandatory.
- It can be used inside the Dockerfile to set for example ENV variables with different values in different images.
- It can be defined in the **docker build** command adding `--build-arg ARG_VAR_NAME=VALUE`.

### Networking

#### Communicating with the World Wide Web (WWW)

Consider this JavaScript example that tries to send a GET request to some-api.com/my-data:

``` javascript
fetch('https://some-api.com/my-data').then(...)
```

This will work out of the box, no extra configuration is required! The application, running in a
Container, will have no problems sending this request.

#### Communicating with the Host Machine

Again, consider this JS example that tries to send a GET request to some web server running on the local host
machine (i.e. outside of the Container but not the WWW).:

```
fetch('localhost:3000/demo').then(...)
```

On your local machine, this would work - inside of a Container, **it will fail**. Because localhost inside of the Container refers to the container environment, not to your local host machine which is running the Container / Docker!

You just need to change this snippet like this:

```
fetch('host.docker.internal:3000/demo').then(...)
```

`host.docker.internal` is a special address / identifier which is translated to the IP address of the machine hosting the Container by Docker.

#### Communicating with Other Containers

You have two main options:

1. Manually find out the IP of the other Container (it may change though): `docker container inspect CONTAINER_NAME`
2. Use Docker Networks and put the communicating Containers into the same Network

Option 1 is not great since you need to search for the IP on your own and it might change over time.

Option 2 is perfect though. With Docker, you can create Networks via `docker network create SOME_NAME` and you can then attach multiple Containers to one and the same Network.

Like this:

```
docker run --network my-network --name cont1 my-image
docker run --network my-network --name cont2 my-other-image
```

Both *cont1* and *cont2* will be in the same Network.

Now, you can simply use the Container names to let them communicate with each other - again, Docker will resolve the IP for you (see above).

```
fetch('cont1/my-data').then(...)
```

### Docker Compose

- You can put your Container configuration into a docker-compose.yaml file and then use just one
command to bring up the entire environment: `docker-compose up`.

- You can find the full list of configurations here: https://docs.docker.com/compose/compose-file/

- **Important**: When using Docker Compose, you automatically get a Network
for all your Containers - so you don't need to add your own Network unless you need multiple
Networks!

- **Note**: Like in docker commands, named volumes are not automatically deleted after exit. You need to remove them
manually.

#### Docker compose files

``` yaml
version: "3.8" # version of the Docker Compose spec which is being used
services: # "Services" are in the end the Containers that your app needs
  web:
    build: ./web # Define the path to your Dockerfile for the image of this container
    volumes: # Define any required volumes / bind mounts
      - logs:/app/logs
  db:
    build: # In case the Dockerfile has another name or other location, the image build can be more specific.
      context: ./db
      dockerfile: Dockerfile-web
    volumes:
      - data:/data/db
```

### Utility containers

This refers to those containers created from a based image in order to start a project and install all the dependencies required to
use the image in the future.

In this case we need to do the following:

  1. Create a simple Dockerfile specifying the image to use and the workdir.
  2. You can just build and run the container giving the command you want to run at the end of the run command. e.g:
  `docker run -it -v "/home/orlando/source/courses/docker-kubernetes-practical-guide/excercises/module-7-utility-containers:/app" node-util npm init`. In this case, it will start up a Node project creating the package.json file in the bind path.
  3. You can protect or make more specific the image created with Dockerfile adding a **ENTRYPOINT**. This is similar to COMMAND
  but only specify the starting part of the command. E.g `docker run -it -v "/home/orlando/source/courses/docker-kubernetes-practical-guide/excercises/module-7-utility-containers:/app" node-util:npm install`. In this example, in the image node-util:npm the ENTRYPOINT was npm, so all
  arguments at the end of the run command will complement the npm entrypoint (`npm install` for this example).
  4. You can define a docker-compose file in order to avoid writing the long docker commands. You define the service to run and you can start it
  with `docker-compose up`. However, if you want to execute the extra command as explained in the previous point you have to use `docker-compose run SERVICE_NAME COMMAND_ARG`. E.g `docker-compose run --build --rm npm init` note that in the example the "npm" is not part of the entrypoint
  command, it is the name of the service defined in the docker-compose.yaml file.

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

### Volumes

- This give you some options to list, inspect or remove created volumes. The example is --help so you can check the options.

```
docker volume --help
```

- `docker run -v /path/in/container IMAGE`: Create an Anonymous Volume inside a Container
- `docker run -v some-name:/path/in/container IMAGE`: Create a Named Volume (named `some-name`) inside a Container
- `docker run -v /path/on/your/host/machine:path/in/container IMAGE`: Create a Bind Mount and connect a local path on your host machine to some path in the Container. You can make read-only adding an extra colon with "ro" `docker run -v /path/on/your/host/machine:path/in/container:ro IMAGE`. This behavior is overwritten when a more specific directory volume is defined in the same run command.
- `docker volume ls`: List all currently active/stored Volumes (by all Containers)
- `docker volume create VOL_NAME`: Create a new (Named) Volume named `VOL_NAME`. You typically don't need to do that, since Docker creates them automatically for you if they don't exist when running a container
- `docker volume rm VOL_NAME`: Remove a Volume by its name (or ID)
- `docker volume prune`: Remove all unused Volumes (i.e. not connected to a currently running or stopped container)

## Docker compose commands

Start all containers / services mentioned in the Docker Compose file

```
docker-compose up
```
- -d : Start in detached mode

- --build : Force Docker Compose to re-evaluate / rebuild all images (otherwise, it only
does that if an image is missing)

Stop and remove all containers / services

```
docker-compose down
```

- -v : Remove all Volumes used for the Containers - otherwise they stay around, even if
  the Containers are removed

Run a specific service/container defined in the compose file.

```
docker-compose run SERVICE_NAME ENTRYPOINT_COMMAND_ARG
```

- --rm to the delete container when stopped. (like docker run command).

- --build to force the image build again.
