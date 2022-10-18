# Docker Network

Docker networking enables a user to link a Docker container to as many networks as he/she requires. Docker Networks are used to provide complete isolation for Docker containers.

At the highest level, Docker networking comprises three major components:

- The Container Network Model (CNM)
- libnetwork
- Drivers

## Container Network Model

![](https://user-images.githubusercontent.com/17776979/196164607-a43ee36b-d1c8-40f6-aad6-6301f955d732.png)

CNM has 3 main components: Sandbox, Endpoint, and Network.

**Network Sandbox** is an isolated network stack. It includes; Ethernet interfaces, ports, routing tables, and DNS config.

**Endpoint** is virtual network interfaces (E.g. veth). Like normal network interfaces, they’re responsible for making connections. In the case of the CNM, it’s the job of the endpoint to connect a sandbox to a network.

**Network** is a software implementation of a switch (802.1d bridge). As such, they group together and isolate a collection of endpoints that need to communicate.

## Libnetwork

The CNM is the design doc, and libnetwork is the canonical implementation. It’s [open-source](https://github.com/moby/libnetwork), written in Go, cross-platform (Linux and Windows), and used by Docker.

## Network drivers

### None Driver

The `none` driver simply disables networking for a container, making it isolated from other containers.

![](https://user-images.githubusercontent.com/17776979/196165054-4bfc8d89-53fa-4770-8d87-598a3f617bc0.png)

**How to use none network**

```bash
docker run -it --name app --network none alpine
```

**Use case**:

- To run network-isolation application that only perform file operations
- To run a one-off command which requires network-isolation

### Host Driver

When using the host driver, the container shares the network stack of the Docker host - appearing as if the container is the host itself, from a networking perspective.

![](https://user-images.githubusercontent.com/17776979/196167085-1bcce825-97a5-43b8-9597-276a4ea5d666.png)

**How to use host network**

```bash
docker run -d --name app --network host nginx:alpine
```

**Use case**:

- When the highest network performance is required
- When a single container needs to handle a large number of pods
- When network-isolation is not required

**Limitation**

- Lack of network isolation
- Port conflict
- Only works on Linux machines

### Bridge driver

The bridge driver creates an internal network within a single Docker host. Containers placed within this network can communicate with each other but are isolated from containers, not on the internal network

![](https://user-images.githubusercontent.com/17776979/196402872-db810282-4de1-4cef-8476-ba1da4c80f48.png)

**Default bridge network**

**How to use default bridge network**

```bash
docker run -d --name app nginx:alpine
```

Note: The default network of the container is the `default bridge network`

**Retrieve the ip address of the container**

```bash
docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' container_name_or_id
```

**User-defined bridge network**

```bash
docker network create my-net # You can specify the subnet, the IP address range, the gateway, and other options
docker network rm my-net
docker create --name my-nginx --network my-net --publish 8080:80 nginx:latest
docker network connect my-net my-nginx # connect a running container to an existing user-defined bridge
docker network disconnect my-net my-nginx # disconnect a running container from a user-defined bridge
```

**User-defined bridges vs default bridges**

- User-defined bridges provides automatic DNS resolution between containers: Containers on the default bridge network can only access each other by IP addresses. On a user-defined bridge network, containers can resolve by name or alias.
- User-defined bridges provide better isolation: All containers without a specified -network is attached to the default bridge network. This can be a risk, as unrelated stacks/services/containers can then communicate. Using a user-defined network provides a scoped network in which only containers attached to that network can communicate.
- Containers can be attached or detached from User-defined network on the fly: You can connect or disconnect them from user-defined networks on the fly during a container's lifetime. To remove a container from the default bridge network, you need to stop it and recreate it with different network options.
- Each User-defined network creates a configurable bridge: If your containers use the default bridge network, you can configure it, but all the containers use the same settings, such as MTU and iptables rules. In addition, configuring the default bridge network happens outside of Docker itself and requires a restart of Docker. User-defined bridge networks are created and configured using `docker network create`. If different groups of applications have different network requirements, you can configure each user-defined bridge separately as you create it.
- Containers connected to the same user-defined bridge network effectively expose all ports to each other: For a port to be accessible to containers or non-Docker hosts on different networks, that port must be published using the `-p` or `--publish` flag.

**Limitation**

- Limited to a single host only
- Bridge driver is slower than Host driver

### Overlay driver

The overlay driver creates a distributed network that can span multiple Docker hosts, and therefore is the preferred driver for managing container communication within a multi-host cluster. overlay is the default driver for Docker swarm services.

![](https://user-images.githubusercontent.com/17776979/196169068-ea591b7e-20b3-4ccd-a463-90ea3c853be5.png)
