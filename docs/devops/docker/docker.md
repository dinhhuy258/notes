# Docker Overview

Docker provides the ability to package and run an application in a loosely isolated environment called a container. Containers are lightweight because they don’t need the extra load of a hypervisor, but run directly within the host machine’s kernel.

## Docker(Containerization) vs VMware(Virtualization)

![](https://user-images.githubusercontent.com/17776979/195768196-c8e5d5b9-de40-4df0-a629-c6d7c9e35703.png)

### Virtual machine

Virtual machines are heavy software packages that provide complete emulation of low level hardware devices like CPU, Disk and Networking devices.

**Pros**

- **Full isolation security**: Virtual machines run in isolation as a fully standalone system. This means that virtual machines are immune to any exploits or interference from other virtual machines on a shared host.
- **Interactive development**: Virtual machines are dynamic and can be interactively developed. Once the basic hardware definition is specified for a virtual machine the virtual machine can then be treated as a bare bones computer. Software can manually be installed to the virtual machine and the virtual machine can be snapshotted to capture the current configuration state.

**Cons**

- **Iteration speed**: Virtual machines are time consuming to build and regenerate because they encompass a full stack system.
- **Storage size cost**: Virtual machines can take up a lot of storage space.

**Containerization** is the packaging together of software code with all its necessary components like libraries, frameworks, and other dependencies so that they are isolated in their own "container"

### Container

Containers are lightweight software packages that contain all the dependencies required to execute the contained software application.

**Pros**

- **Iteration speed**: Because containers are lightweight and only include high level software, they are very fast to modify and iterate on.
- **Robust ecosystem**: We can download image from image public repository to run the container application, saving time for development teams

**Cons**

**Shared host exploits**: Containers all share the same underlying hardware system below the operating system layer, it is possible that an exploit in one container could break out of the container and affect the shared hardware.

## Docker architecture

![](https://user-images.githubusercontent.com/17776979/195769644-997a5d9d-4a6d-4ac8-96e5-5ef518b6b5d2.png)

Docker follows Client-Server architecture, which includes the three main components that are **Docker Client**, **Docker Host**, and **Docker Registry**.

### Docker Registry

Docker Registry manages and stores the Docker images.

There are two types of registries in the Docker:

- Pubic Registry - Public Registry is also called as Docker hub.
- Private Registry - It is used to share images within the enterprise.

### Docker client

The Docker client (docker) is the primary way that many Docker users interact with Docker. When you use commands such as `docker run`, the client sends these commands to `dockerd`, which carries them out. The docker command uses the Docker API. The Docker client can communicate with more than one daemon.

### Docker Host

Docker Host is used to provide an environment to execute and run applications. It contains the docker daemon, images, containers, networks, and storage.

### Docker daemon

The Docker daemon (`dockerd`) listens for Docker API requests and manages Docker objects such as images, containers, networks, and volumes. A daemon can also communicate with other daemons to manage Docker services.

### Docker objects

**Docker Images**

An image is a read-only template with instructions for creating a Docker container.

**Docker Containers**

A container is a runnable instance of an image.

**Docker Networking**

Docker Networking allows isolated packages communicate to each another.

**Docker Storage**

Docker Storage is used to store data on the container
