
# Introduction

JupyterHub is a powerful environment to host a [Jupyter][] notebook server. This lets one or more users access computational resources and work in a Jupyter on the host (server) computer, but requires only a web browser to access from the user's (client) computer. Docker is a platform for packaging and executing *containers*, which behave like lightweight deployable virtual machines. The purpose of this tutorial is to cover the basics necessary to get a JupyterHub container up and running on a Docker host.

The combination of these two technologies makes it possible to put together a working environment that can be quickly started up on any (modern) Linux system. By leveraging Docker's features to build on top of a standard JupyterHub container, it's also straightforward to automatically customize the environment with all of the features that you or your users need.

# Installing and enabling Docker

Docker uses Linux containers, so it needs a fairly up-to-date Linux system to run. You can think of a Docker container as conceptually similar to a virtual machine, but they're actually much more efficient. This is especially true if you have lots of similar containers running. Each filesystem change is dependent on previous layers (kind of like Git's directed graph). This is one of the strengths of Docker; we can build a custom JupyterHub container on top of the Jupyter project's official release (and benefit from the work already done).

You can run a Linux environment for Docker inside a virtual machine like VirtualBox, VMWare or Parallels if you don't have access to a recent Linux host. Docker works well on recent Ubuntu releases, though I mostly use it either on Debian 7 or 8, or CentOS 7. If you have access to a cloud virtual machine, that's also a great way to run it, but doing much in JupyterHub is probably going to need more memory than a basic \$5/mo virtual machine provides.

On Debian/Ubuntu, most of the setup should be as simple as:

    sudo apt-get update
    sudo apt-get install docker-engine

> In Debian-based systems, the package "docker" is a graphical system dock like the Mac OS dock, but for X11. The package for the Docker container tool has been variously known as " docker.io", "lxc-docker" and "docker-engine", though the latter seems to be the new standard. On CentOS or RHEL, you would type:

    sudo yum install docker

The Docker daemon (system service) runs automatically at boot on Debian-derived systems, including Ubuntu. On CentOS you may need to do something like:

    systemctl enable docker
    systemctl start docker

The specifics of this will depend on the system; what I've put here will work on CentOS 7 or Debian 8, both of which use `systemd` to manage services.

# Basic Docker usage

All of the Docker commands are triggered from a master control script (appropriately named `docker`), which you may need to run with administrator privileges. There are ways around this, but being able to run Docker commands more or less confers "root" access on the system (because of the potential for *privilege escalation*; i.e., you can something equivalent to root privileges by setting up a Docker container in the ~~right~~ wrong way). So, most systems require you to run the Docker commands with `sudo` or equivalent. Below is a (somewhat abbreviated) output of some of the relevant commands. You can always type `docker --help` to get more info.

    $ sudo docker
    Usage: docker [OPTIONS] COMMAND [arg...]
        docker [--help | -v | --version ]

    A self-sufficient runtime for containers.

    Commands:
        build       Build an image from a Dockerfile
        exec        Run a command in a running container
        images      List images
        logs        Fetch the logs of a container
        ps          List containers
        pull        Pull an image or a repository from a registry
        restart     Restart a container
        rm          Remove one or more containers
        rmi         Remove one or more images
        run         Run a command in a new container
        start       Start one or more stopped containers
        stop        Stop a running container

The only command you really need to get things running is `docker run`, but the (abridged) list of commands above is an overview of the ones you're most likely to use on a regular basis. I won't aim to give a complete overview of these commands here; rather, I'll move on to showing you how to actually set up the container for hosting JupyterHub.

# Installing JupyterHub

To get a copy of the basic [JupyterHub container][] from the main Docker repository [Docker Hub][], you would type:

    docker pull jupyter/jupyterhub

This fetches all of the filesystem layers that build up a container called `jupyterhub` from the user `jupyter`. This is the official release from the Jupyter developers, and it is constructed automatically by the Docker Hub system as an "Automated Build". In order to verify how it is built you should look at the [project page][JupyterHub Container], and note that it links to a [source repository](https://github.com/jupyter/jupyterhub) on GitHub. This repository contains all of the instructions for the automated build, and the [Dockerfile](https://github.com/jupyter/jupyterhub/blob/master/Dockerfile) controls the build process. The fact that the JupyterHub container is based on an automated build may inspire some confidence that it is what it claims to be, since it adds transparency (and convenience) in the assembly process.

> Note that in general, pulling a set of filesystem images from Docker places a great deal of trust in the person or persons who constructed the image. Automated builds are better than nothing (if you trust Docker Hub and validate the Dockerfile). There are still potential security implications, even if you don't run the container. You should make yourself aware of them before using any of these sorts of tools in critical infrastructure.

By default, it will get the latest version, though there are ways to tag different containers to allow for versioned releases. I have a version that is based on this, but adds support for LDAP users, which you would get with:

docker pull bsmithyman/jupyterhub-ldap

Of course, in general you should probably validate how these things are built. To check how my container is created, you can go to the following link:

You'll see that this is an "Automated Build", meaning that it is built automatically by the Docker Hub environment, and that it comes from a configuration stored in Git. It turns out that the source repository is here:

The changes from the base jupyter/jupyterhub version are pretty minor, but they make it possible for us to use this with our research group's domain logins. There are also ways to get this working with Git OAuth logins, for example; to be honest, I haven't played with that very extensively.

# Running Containers

[Docker Hub]: https://hub.docker.com/
[Jupyter]: http://jupyter.org/
[JupyterHub Container]: https://hub.docker.com/r/jupyter/jupyterhub/