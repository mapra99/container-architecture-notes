# Docker

## What are Containers?

A container is a group of processes that run in isolation, running on a shared kernel

The isolation is provided by a Linux feature called Namespaces:

- PID
- USER
- UTS
- NS
- NET
- IPC

- Control Groups

### VM vs Containers

#### Virtual Machines

- Run over an Hypervisor
- Includes a full blown operating system and processes
- Includes Dependencies
- Includes one or more Apps

#### Containers

- Run over a Kernel
- Doesn't include the whole OS, just some specific files
- Includes Dependencies
- Includes one Application

Containers are more lightweight and fast than VMs. However, this doesn't mean that containers replace VMs. Rather, containers can be run directly on top of VMs, complementing the VMs infrastructure.

### Docker

Tooling for using containers is glacking. Docker comes in there: At its core, Docker is a tooling to manage containers.

Enable developers to use containers for the applications: "Build once, run anywhere".

#### Advantages

- No more 'Works on my machine'. Given that we are packaging the app and its dependencies, we are creating an isolated environment that can be shared among all developers machines and production.
- Lightweight and fast. Easy to ship into the CI/CD pipeline
- Ecosystem and tooling provided by community.

## LAB 1: First Docker Container

#### Running a container

- Run the container: The command `docker container run -t ubuntu top` will run a docker container that uses an Ubuntu image and inside runs a command called `top`.

  Internally, it `docker container run` first checks if the `ubuntu` image exists in the machine; if not, ir will run `docker pull` to download the image.

  The flag `-t` creates a TTY. It's just to let the command `top` work fine.

  The `top` command is an utility of Ubuntu that lists all the processes that are running and sorts them by the resource usage. It can be seen that the result of running this command is a list that contains only 1 process running: `top` itself. This proves the isolation that the container is creating.

- List all running containers: The command `docker container ls` lists all the containers that are running, and their ID. With this ID we can reference each container for further commands

- Execute another task in the container: The command `docker container exec -it CONTAINER_ID bash` runs a bash terminal inside the container.

#### Running multiple containers

##### The Docker Hub

It's a public central repository to host Docker images.

#### More flags for `docker container`

`docker container run --detach --publish 8080:80 --name nginx`

`--detach` runs the container in the background

`--name nginx` gives the container a custom name (`nginx`)

`nginx` is the name of the image.

#### Inspecting the containers

`docker container ls` lists all the active containers and their details

`docker container inspect CONTAINER_ID` gives more details about the specified container.

### Removing the containers

`docker container stop CONTAINER_ID`  stops the containers.

`docker system prune` removes any stopped container, unused volumne and network.

## Creating custom Docker Images

A Docker image is a Tar file containing a container's filesystem and metadata. It can be used for sharing and redistribution.

Images are downloaded and cached in the host's memory.

### Docker Registry

It's the system that pushes and pulls images. The default registry is Docker Hub.

- Public and free for public images
- Many pre-packaged images available

### Private Registries

- Self-host or cloud provider options
- A Docker Registry is itself an image

### Creating a Docker Image

1. Create a Dockerfile: It's a list of instructions for how to construct the container
2. `docker build -f Dockerfile` builds the Docker image

A typical Dockerfile example:

```dockerfile
FROM ubuntu
ADD myapp /
EXPOSE 80
ENTRYPOINT /myapp
```

`FROM` must be the first instruction at the begining of the file. It specifies on top of which image this new build is going to be.

`ADD` adds a new file

`RUN` runs some commands needed for the container setup

`CMD` runs the command that starts the container. There must be only one.

`COPY` takes one file and copies it from the host to the container

### Image layers architecture

A Docker image is based on many images one on top of the other. Each line of the Dockerfile is an expression that states a change between the previous image and a new one. This is important to understand, as it has some KPIs:

- Docker will cache all these images and reuse them when it is possible. It reuses the images in further rebuilds of the container, and also to build other containers. This allows a faster CI/CD integration.
- When any of the lines in the Docker file produces a change in the Container build, all the subsequent images will need to be rebuilt. This means that, in order to have a faster CI/CD pipeline, the lines that implement more changes should be as much as possible at the end of the Dockerfile.