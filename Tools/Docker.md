# Docker

## Introduction

- An open-source project that facilitates the deployment of software
  applications inside sandboxed _containers_ to run on the host OS
- An application is packaged with all of its dependencies into 
  a standardized unit, called _container_, that can be run anywhere

## Containers

- In traditional virtualization, a hypervisor is used to virtualize
  physical hardware
- Each VM contains a guest OS, a virtual copy of the required hardware
  and a program with its dependencies
- Instead of virtualizing the underlying hardware, containers
  virtualize the operating system they are run on
- In case of Linux kernel, they leverage namespaces and cgroups primitives 
  for process isolation and resource control
- In summary, they offer a way to abstract applications from their
  runtime environment allowing for easy and consistent deployment on variety
  of targets (desktops, clouds, data centers etc.)

## Installation

- On Alpine, simply type 

```bash
apk add docker
```

- To start a Docker Daemon, type `service docker start` as root
- To run Docker as a non-priviledged user (it needs access to a Unix socket),
  either use [rootless mode](https://docs.docker.com/engine/security/rootless) 
  or add an appropriate line in the `sudoers` file

## Basics

- Each command interacts with a _Docker Daemon_ - a background service 
  responsible for building, running and distributing Docker containers

### Images

- A blueprint of an application, which forms the basis of containers
  - Base images are images that have no parent image
  - Child images are images built on base images
  - Official images are maintained by Docker developers; usually 1 word long
  - User images are maintained by Docker community - same naming like at Github
- `docker pull` fetches an image from the [Docker registry](https://hub.docker.com)
- `docker rmi` removes an image
- `docker image` provides a couple of operations on images
  - `ls` prints available images (equivalent to `docker images`)
- `docker search` queries Docker registry for a given string

### Containers

- `docker run` finds a given image, creates a container based on that image and
  runs it
  - `-i` keeps `stdin` open
  - `-t` allocates a pseudo tty
  - `--rm` automatically removes container once it's done
  - `-d` detaches the terminal
  - `-P` publishes all exposed ports to random ports
  - `-p` specifies custom port
  - `--name` assigns a name to a container
- `docker ps` shows all containers that are currently up
  - `-a` queries all containers that were run
  - `-q` shows only their IDs
  - `-f` filters 
- `docker rm` deleted a container based on its ID
- `docker container` provides a handful of operations on containers
  - `prune` deletes all stopped containers
  - `ls` prints running containers
- `docker port` lists port mappings for a container
- `docker stop` stops a given container
- `docker build` builds an image basing on the `Dockerfile` (kinda like `make`)
  - `-t` specifies name and a tag list

## Custom Docker containers

- A Dockerfile is a text file that contains a list of commands called by Docker
  client when creating an image
- A convenient tool for automation

### Keywords

- `FROM` initializes a new build stage and sets base image for the next instruction
  - `AS` names the build stage
- `WORKDIR` sets the working directory - if it doesn't exist, creates it
- `COPY` copies files from the current directory and adds them to 
  the file system created by `WORKDIR`
- `RUN` executes any commands in a new layer on top of the current image
  and commits the result to the image
- `EXPOSE` specifies a port number to expose
- `CMD [...]` provides defaults for an executing container - must be unique

# Credits

- Prakhar Srivastav - [Docker for beginners](https://docker-curriculum.com/)
- IBM Cloud Education - [Containers](https://www.ibm.com/cloud/learn/containers)
- Docker docs - [Dockerfile reference](https://docs.docker.com/engine/reference/builder/)

# Author

- [Harutekku](https://github.com/harutekku)