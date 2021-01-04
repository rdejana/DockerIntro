# Introduction to Docker Examples

This file provides a basic introduction to using Docker, with specific examples to using on the Nvidia XAVIER NX.

## Part 1: Installation
Docker is installed by default as part of the Nvidia Jetpack Install.  For other platforms, see the Docker installation instructions.  See https://docs.docker.com/engine/install/ for installation instructions on non-NX platforms, e.g. macOS x86_64, Windows, Linux (CentOS, Ubuntu, etc.)  Unless noted, these examples will run on any docker example.

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