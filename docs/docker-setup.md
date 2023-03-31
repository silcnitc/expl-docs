---
title: Docker based setup
---

## Install and setup Docker on host machine

Follow the instructions available [here](https://docs.docker.com/get-docker/){ target="_blank" } to install docker on your machine.

You could also go through the [:material-docker: Docker quick start quide](https://docs.docker.com/get-started/){ target="_blank" }  to know more about Docker .

!!! warning
    The following has **not** been tested on _Windows_.
    
    If you encounter any issues/ has suggestions raise an issue [here](https://github.com/silcnitc/expl-docs/issues/new)

## Setting up

We'll assume the following directory structure

``` plaintext
.
├── Dockerfile
└── workdir/ # <- user files will be stored here and mapped to container
```

We'll store the user written code in `workdir` and map the same into the container.

We can create the structure using the below commands

=== "Windows (Powershell)"

    ``` powershell
    cd <your directory>
    New-Item Dockerfile
    New-Item -path workdir -ItemType directory
    ```

=== "Unix / Linux"

    ``` sh
    cd <your directory>
    touch Dockerfile
    mkdir workdir
    ```

The contents of `Dockerfile` are given below

``` Dockerfile
FROM ubuntu:20.04

RUN apt-get update \
    && apt-get install -y bison flex libreadline-dev libc6-dev libfl-dev wget vim make gcc curl git build-essential

RUN useradd -m expl
USER expl

RUN cd /home \
    && git clone https://github.com/silcnitc/xsm_expl.git  \
    && cd ./xsm_expl \
    && make

WORKDIR /home/xsm_expl
```

The given `Dockerfile` will setup an expl environment as specified in [Installation Page](./install.md)

### Building the container image

We'll now build the container image using the `Dockerfile`

```sh
docker build -t expl:ubuntu20.04 .
```

### Start the container instance

We'll start an instance of Container and map the local folder `workdir` into `/home/expl/workdir` directory of container.

=== "Windows (PowerShell)"
    ``` powershell
    docker run -v ${PWD}/workdir:/home/expl/xsm_expl/workdir -d --name expl -i expl:ubuntu20.04
    ```

=== "Unix / Linux"
    ``` sh
    docker run -v $PWD/workdir:/home/expl/xsm_expl/workdir -d --name expl -i expl:ubuntu20.04 
    ```

We now have a container instance running in background with the name `expl` and required volume mounts

### Connecting to the container

We can connect to the container instance using the following commands

```sh
docker start expl # if the container instance is not already running

docker exec -it expl /bin/bash # to get a bash shell inside the container
```

After connecting to the container you can use `xfs-interface`, and `xsm` binaries as mentioned in [Installation Page](./install.md)