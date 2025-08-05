---
title:  "Part 01: Docker for Network Automation"
date:   2022-07-14 11:02:15 +0500
categories: docker
tags:
  - network automation
  - docker
---

## What is Docker?
Docker is software that runs on Linux and Windows. It creates, manages, and can even orchestrate containers. The software is currently built from various tools from the open-source project

### The Docker technology

When most people talk about Docker, they’re referring to the technology that runs containers. However, there are at least three things to be aware of when referring to Docker as a technology:

1. _The runtime_
2. _The daemon (a.k.a. engine)_
3. _The orchestration_

### Windows containers vs Linux containers

It’s vital to understand that a running container shares the kernel of the host machine it is running on. Its means that a containerized Windows app will not run on a Linux-based Doer host, and vice-versa. Windows containers require a Windows host, and Linux containers require a Linux host.

It is possible to run Linux containers on Windows machines. For example, Docker Desktop running on Windows has two modes — “Windows containers” and “Linux containers”. Depending on your version of Docker Desktop, the Linux container runs either inside a lightweight Hyper-V VM or using the Windows Subsystem for Linux (WSL). The WSL option is newer and the strategic option for the future as it doesn’t require a Hyper-V VM and oﬀers better performance and compatibility.

## Docker Installation

You can install Docker Engine in different ways, depending on your needs. [Download docker from the official website for your platform](https://docs.docker.com/engine/install/).

### Installing Docker on Linux

In this tutorial, we’ll look at one of the ways to install Ubuntu Linux 20.04 LTS.

Before you install Docker Engine for the first time on a new host machine, you need to set up the Docker repository. Afterwards, you can install and update Docker from the repository.

#### Set up the repository

* Update the apt package index and install packages to allow apt to use a repository over HTTPS:

```console
$ sudo apt-get update

$ sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

* Add Docker’s official GPG key:

```console
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

* Use the following command to set up the stable repository.

```console
$ echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

#### Install Docker Engine

* Update the apt package index, and install the latest version of Docker Engine and container, or go to the next step to install a specific version:

```console
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

* Verify that Docker Engine is installed correctly by running the hello-world image.

```console
sudo docker run hello-world
```

See other ways to [Install Docker Engine](https://docs.docker.com/engine/install/).

#### Docker Commands

Docker is now installed and you can test running some commands to check more information about docker.

```console
sudo docker --version
sudo docker version
sudo docker info
```

## Docker Image

A Docker container image is a lightweight, standalone, executable package of software that includes everything needed to run an application: code, runtime, system tools, system libraries and settings.

It’s helpful to think of a Docker image as an object that contains an OS ﬁle system, an application, and all application dependencies as a virtual machine template is a stopped virtual machine. In the Docker world, an image is effectively a stopped container.

* Run the _docker image ls_ command

```console
~ » docker image ls
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
```

* Getting images onto your Docker host is called “pulling”.

```console
ubuntu@ubuntu:~$ docker pull ubuntu:latest
Using default tag: latest
latest: Pulling from library/ubuntu
Digest: sha256:626ffe58f6e7566e00254b638eb7e0f3b11d4da9675088f4781a50ae288f3322
Status: Image is up to date for ubuntu:latest
docker.io/library/ubuntu:latest
```

* To list install docker images:

```console
$ docker image ls
REPOSITORY                    TAG       IMAGE ID       CREATED        SIZE
ubuntu                        latest    ba6acccedd29   2 weeks ago    72.8MB
```

Or

```console
$ docker images
REPOSITORY                    TAG       IMAGE ID       CREATED        SIZE
ubuntu                        latest    ba6acccedd29   2 weeks ago    72.8MB
```

## Docker Containers

A container is a standard unit of software that packages up code and all its dependencies so the application runs quickly and reliably from one computing environment to another.

Now that we have an image pulled locally, we can use the _docker container run_ command.

```console
$ docker container run -it ubuntu:latest /bin/bash
root@20e3b1f9bab4:/#
```

docker container run command tells the Docker daemon to start a new container. the -it ﬂags tells Docker to make the container interactive and to attach the current shell to the container’s terminal. Next, the command tells Docker that we want the container to be based on the ubuntu:latest image. Finally, we tell Docker which process we want to run inside of the container.

```console
-d — run in a detached mode as a background service.
-p — forward the ports between container and the host operating system.
-i — Keep STDIN open even if not attached.
-t — tty session with the container.
-V — mount the local partition with the container partition.
--name flag is used to name your container with a custom name.
```

Press _Ctrl+PQ_ to exit the container without terminating it. This will land your shell bash at the terminal of your Docker host.

You can see all _running_ containers on your system using the _docker container ls_ command.

```console
$ docker container ls
20e3b1f9bab4   ubuntu:latest   "/bin/bash"   11 minutes ago   Up 11 minutes             frosty_williams
```

Or

```console
~ » docker ps
CONTAINER ID   IMAGE           COMMAND       CREATED          STATUS          PORTS     NAMES
20e3b1f9bab4   ubuntu:latest   "/bin/bash"   11 minutes ago   Up 11 minutes             frosty_williams
```

To list all containers, we can use `docker ps -a`

### Attaching to running containers

You can attach your shell to the terminal of a running container with the _docker container exec_ command. As the container from the previous steps is still running.

```console
~ » docker container exec -it frosty_williams bash
root@20e3b1f9bab4:/# ls
bin  boot  dev  etc  home  lib  lib32  lib64  libx32  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@20e3b1f9bab4:/#
```

Or

```console
~ » docker attach frosty_williams
root@20e3b1f9bab4:/# ls
bin  boot  dev  etc  home  lib  lib32  lib64  libx32  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@20e3b1f9bab4:/#
```

We can also use _CONTAINER ID_ instead of _NAMES_ in the above command.

Exit the container again by pressing _Ctrl-PQ_. Stop the container and remove it using the docker _container stop_ and _docker container rm_ commands.

```console
docker container stop frosty_williams
frosty_williams
```

```console
docker container rm frosty_williams
frosty_williams
```

## Mount Host machine and container volumes

I am currently inside the _doc_ folder which I have created and it’s empty. With the below command using the -v flag. we will mount the current working directory(doc folder) with the/tmp folder inside the container.

```console
╭─$ ~/doc
╰─➤  docker run -it --name ubuntu -v $PWD:/tmp ubuntu:focal
root@7e349527f674:/# cd tmp
root@7e349527f674:/tmp# ls
root@7e349527f674:/tmp# touch file.txt
root@7e349527f674:/tmp# ls
file.txt
root@7e349527f674:/tmp# cd
root@7e349527f674:~# exit
exit
```

Now, we can exchange files to and from the container and to the host operating system.

```console
╭─$ ~/doc
╰─➤  docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
╭─$ ~/doc
╰─➤  ls
 file.txt
```

> We can not mount a volume to an already created/running container.
{: .prompt-tip }
