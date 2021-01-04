# Introduction to Docker Examples

This file provides a basic introduction to using Docker, with specific examples to using on the Nvidia XAVIER NX.

## Part 1: Installation
Docker is installed by default as part of the Nvidia Jetpack Install.  For other platforms, see the Docker installation instructions.  See https://docs.docker.com/engine/install/ for installation instructions on non-NX platforms, e.g. macOS x86_64, Windows, Linux (CentOS, Ubuntu, etc.)  Unless noted, these examples will run on any docker example.

Note, at the time of writing, Docker Desktop fo Apple M1 is still a tech preview.  

### Optional for the NX and Linux installations.
By default, Docker is owned by and runs as the user root.  This requires commands to be executed with sudo.  If you don't want to use sudo, a group may be used instead.  This group  group grants privileges equivalent to the root user. For details on how this impacts security in your system, see Docker Daemon Attack Surface (https://docs.docker.com/engine/security/#docker-daemon-attack-surface)
.  The examples will assume this has been done.  If you do not do this, you'll need to prefix the docker commands with `sudo`.

1. Create the group docker.  Note, this group may already exist.
```
sudo groupadd docker
```
2. Add your user to the docker group.
```
sudo usermod -aG docker $USER
```
3. Log out and log back in so that your group membership is re-evaluated.
If testing on a virtual machine, it may be necessary to restart the virtual machine for changes to take effect.

## Part 2: Registries
An image registry provides a means of sharing images.  Registries can be hosted, e.g. DockerHub or NVIDIA's NCG, or may be self hosted.  You'll want to sign up for a free account with DockerHub (https://hub.docker.com).  This will allow you to easily share images. 

See https://docs.docker.com/docker-hub/ for the steps.  Once your account is created, login via the command
```
docker login
```
and follow the prompts.

## Part 3: Hello World

Note, The following will run any docker instance.
1. Run the command `docker run hello-world`.
If things are correctly installed, you'll see an output similar to:
```
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
256ab8fe8778: Pull complete 
Digest: sha256:1a523af650137b8accdaed439c17d684df61ee4d74feac151b5b337bd29e7eec
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (arm64v8)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

This command "runs" a container based on the specified image, in this case, an image named hello-world.  
When the command was executed, Docker first checked if the image was availalbe locally, and as it was not, it downloaded it for us.  As we didn't specify the tag, it automatically download the "latest" one.  Next, it started the container, which printed out the message, then exited.  

We can run the command `docker images` to see that we have the hello-world image locally.  The output will look similar to:
```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-world         latest              a29f45ccde2a        12 months ago       9.14kB
```

The command `docker ps` is used to list containers.  The default command lists only running containers while the -a option is neede to list all containers.

`docker ps`:
```
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```
vs
`docker ps -a`:
```
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                      PORTS               NAMES
697c6c524995        hello-world         "/hello"            10 minutes ago      Exited (0) 10 minutes ago                       tender_galois
```

A stopped container may be deleted with the command `docker rm <container id or container name>`.  Delete your container and verify it has been removed.

When running a container, we can also give it a name with the flag `--name <name>`.  Note, docker provents containers from having duplicate names.  Running hello-world again with this flag:
```
docker run --name helloWorld hello-world
```
Note, the image is not downloaded again, rather the local "cached" image is used.

Run `docker ps -a` again and you should now see something similar to:
```
CONTAINER ID        IMAGE               COMMAND             CREATED              STATUS                          PORTS               NAMES
1ee8c4789e21        hello-world         "/hello"            About a minute ago   Exited (0) About a minute ago                       helloWorld
```

This container can now be deleted with the command `docker rm helloWorld`.

If you don't care to save the container when it exits, docker can automatically delete the container with the option --rm.
Run the command `docker run --name helloWorld --rm hello-world` and then when it exists, rerun `docker ps -a`.  You'll find that the container has been removed.  

When done, you can remove the image with the command `docker rmi hello-world:latest`.  An image may only be deleted when there are no containers using that image.

### Docker command recap
Command | Description
docker ps | Lists all running containers
docker ps -a | Lists all containers
docker images | Lista all local images
docker run  <imageName> | Run an image as a container
docker run --name <name> <imageName> | Run an image as a container, using the specified name for the container.
docker run --rm <imageName> | Run an image as a conatiner and automatically delete it when it exits.