# Introduction to Docker Examples

This file provides a basic introduction to using Docker, with specific examples to using on the Nvidia XAVIER NX.

These examples assume the use of one or more shells open.

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

This example will run a simple hello world container. You'll start by runninh the command `docker run hello-world`.
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
| Command | Description |
| --- | --------- |
| docker ps | Lists all running containers |
| docker ps -a | Lists all containers |
| docker images | Lista all local images |
| docker run  <imageName> | Run an image as a container |
| docker run --name <name> <imageName> | Run an image as a container, using the specified name for the container. |
| docker run --rm <imageName> | Run an image as a conatiner and automatically delete it when it exits. |

## Part 4: Interacting with a container

You'll now see a slightly more interesting example using the Ubuntu base image.  You'll start by running the command `docker pull ubuntu:latest`.  This will explictly download the image for you, making sure you have the most up to date version.  After the download is complete, run the command `docker run --name ubuntu --rm -it ubuntu bash`.  The `-it` option enable interactive mode and allocates a pseudo-TTY.  This allows us to interact with the container process.  The `bash` command tells docker to run the bash shell as the container's process.  This command will give you a shell that looks similar to 
```
root@eb3d250d4de8:/# 
```
Run some commands.  apt-get update && apt-get install vim, etc.  This will work as expected.

As you know, Docker is processed based.  On Linux systems, we can see this from the host by using the ps command.  Note, non-Linux hosts such as macOS run Docker in a Virutal Machine and the host OS cannot see the process.

I've installed VI (apt-get install -y vim) and am editing a file named helloWorld.txt via the command `vi helloWorld.txt`.

```
ps -ef | grep vi
root      6648  5776  0 07:55 pts/0    00:00:00 vi helloWorld.txt
```

As expected, this is just a process!

Exit out of your ubuntu container.

Let's get 2 containers talking to each other!.  In one shell, run the following

```
docker run --name web --hostname web --rm -it ubuntu bash
```

And in a second

```
docker run --name db --hostname db --rm -it ubuntu bash
```

We are using the hostname option to force docker to set the hostname as something recognizable vs the container id.

In the `web` container, let's install ping: `apt-get update && apt-get install iputils-ping -y`

Once installed, let's try to ping `db` with the command `ping db`.  You'll get an error back that says: `ping: db: No address associated with hostname`.  We need to find the container's IP Address; in a third shell, run the command `docker inspect db | grep IPAddress`.  You'll get back a field `IPAddress`; take note of the value.  For example, I get `"IPAddress": "172.17.0.3"`.  Now in web, ping your db's IP Address and you should get a response similar to:
```
root@web:/# ping 172.17.0.3
PING 172.17.0.3 (172.17.0.3) 56(84) bytes of data.
64 bytes from 172.17.0.3: icmp_seq=1 ttl=64 time=0.224 ms
```
Exit out of your 2 containers.

By default, containers can talk by IP Address, but not by name.  While this works, it is brittle and not portable.  There use to be a link option on run, but this was replace by user defined networks.
These user defined networks provide:
- DNS resultion between containers
- Better isolation - only containes attached may communicate with each other.
- Containers can be attached and detached on the fly

You'll create a network with the command `docker network create demo` to create a network named `demo`.  You can verify the network with the command `docker network ls`.  We'll recreate our 2 containers, but this time using our new network.

```
In shell one: 
docker run --name web --hostname web --network demo --rm -it ubuntu bash

In shell two:
docker run --name db --hostname db --network demo --rm -it ubuntu bash

```
In web, install ping again and run the command `ping db`.  This time, you'll get a resopnse!

```
root@web:/# ping db
PING db (172.19.0.3) 56(84) bytes of data.
64 bytes from db.demo (172.19.0.3): icmp_seq=1 ttl=64 time=0.483 ms
64 bytes from db.demo (172.19.0.3): icmp_seq=2 ttl=64 time=0.201 ms
```

Close out your containers and then delete your network with the command `docker network rm demo`.


### Docker command recap
| Command | Description |
| --- | --------- |
| docker ps | Lists all running containers |
| docker ps -a | Lists all containers |
| docker images | Lista all local images |
| docker run  <imageName> | Run an image as a container |
| docker run --name <name> <imageName> | Run an image as a container, using the specified name for the container. |
| docker run --rm <imageName> | Run an image as a conatiner and automatically delete it when it exits. |
| docker network create <name> | Create a user defined network |
| docker network ls | List networks |
| docker network rm <name> | Delete a network |
| docker run -it ... | Enable the abiliy to interact with a container via stdin and stdout |
| docker run --hostname ... | Set the hostname of the container |
| docker run --network ... | Set the container's network |
| docker inspect <name> | Return low-level information on Docker objects |

## Part 5: More netowrking and Persistent files##
Note, this is just an example.  For production applications, the files used here should be baked into the image.

In this example, we'll be working with Nginx, a robost HTTP server. You'll launch your instance with the following command `docker run -d --name web --hostname web --rm nginx`.  The `-d` option runs the container in detached mode.  This means the container is running in the background!

```
rdejana@nx:~$ docker run -d --name web --hostname web --rm nginx 
Unable to find image 'nginx:latest' locally
latest: Pulling from library/nginx
c9648d7fcbb6: Pull complete 
af2653e2da79: Pull complete 
1af64ee707c7: Pull complete 
3bdc08a2d3ea: Pull complete 
fed23bd0d00d: Pull complete 
Digest: sha256:4cf620a5c81390ee209398ecc18e5fb9dd0f5155cd82adcbae532fec94006fb9
Status: Downloaded newer image for nginx:latest
2d5a5aa602e5e3c6198ad3ae9733d33255c486fef1ef7a29e56bc42d6cadd9e1
rdejana@nx:~$
```

You'll use the `exec` command to access the container.  Run `docker exec -ti web bash`.  This will attach us to the container and start a bash process for you to interact with.
Once in the command, verify that Nginx is running by curling the server: `curl http://localhost` and you'll see:
```
root@web:/# curl http://localhost
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

Now how to access this container from OUTSIDE the container? Exit out of the shell and stop the container with `docker stop web`.  Recreate the container with the command: 
`docker run -d --name web --hostname web -p 8080:80 --rm nginx`.  The `-p #:#` maps a port on the host, 8080 in this case, to a port in the container, in this case 80.  From a web browser, access you NX, by name or IP on port 8080, eg. http://nx:8080 and you should get back your web page.

You can run the command `docker logs web` to see container's output.
```
docker logs web
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
192.168.1.199 - - [05/Jan/2021:15:51:26 +0000] "GET / HTTP/1.1" 200 612 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_6) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/14.0.2 Safari/605.1.15" "-"
```

You'll now customize the HTML page. By default, the HTML files are located in the directory `/usr/share/nginx/html`. You'll want to install VI (or another editor) (apt-get update && apt-get install -y vim).  Change to the HTML dir, and edit the fine index.html replacing the text with:
```
<!DOCTYPE html>
<html>
<head>
<title>Hello World</title>
</head>
<body>
<h1>Hello World from Docker and Nginx!</h1>
</body>
</html>
```
Save the file and access the web page again.  Now stop your container and start it again.  Notice that your changes are now gone.  This is because, when the contaienr is deleted, all of your changes are deleted.  We'll address this by creating a persistent file system.  On your host, create a directory were you'll want to store HTML.  Change to that directory and create the index.html file with the content you used before.  Now launch your container with the following:
```
docker run -d --name web --hostname web -p 8080:80 --rm  -v <your HTML directory>:/usr/share/nginx/html  nginx
```
replacing `<your HTML directory>` with your actual directory.  Access your page again and you'll see your changes!  You've created a container using a bind mount.  This is used to share content from your host to your container.  Another option is to use a volume.  A volume is a more power and portable solution, but one in which the "where" abastraced out. If you are intersted in volumes, see the Docker documentation.

Stop your container.

## Part 6. Building and sharing your own custom image.
Using exiting containers is great, but the Docker also allows you to create and share your own images.  