# Docker Cheat Sheet

## Images

**Build image**

```bash
docker build .
docker build -t {tag_name} .
docker build -t {tag_name} Dockerfile
```

**List all images**

```bash
docker images
```

**Remove image**

```bash
docker rmi {image_name:tag_name}
```

**Download image**

```bash
docker pull {image_name}
```

## Containers

**List containers**

```bash
docker ps # show running containers
docker ps -a # show all container
```

**Get logs**

```bash
docker logs [-f/--follow] {container}
```

**Delete container**

```bash

docker rm {container}
```

**Run container**

```bash

docker run {image}
docker run -ti ubuntu:latest # -ti = terminal keyboardInteractive
docker run -d -ti ubuntu bash # -d : (detach) run docker process in background
docker run -e "DB_HOST=db" -e "DB_PORT=3306" -e "DB_USER=wikijs" -e "DB_PASS=wikijsrocks" image_name
docker run --name {container_name} -p {host_port}:{container_port} -v {/host_path}:{/container_path} -it {image_name} /bin/bash
```

docker run command = docker create command + docker start command

```bash
docker create ubuntu:latest
docker start {container}
```

**Inspect container**

```bash
docker inspect {container}
```

**Stop container**

```bash
docker stop {container}
```

**Convert detach mode to attach mode**

```bash
docker attach {container}
```

**Remove stopped contains**

```bash
docker system prune # Remove all unused containers, networks, images (both dangling and unreferenced), and optionally, volumes.
```

**Executing Commands**

```bash
docker exec -it {container} sh
```

**Copy file from host to container**

```bash
docker cp foo.txt mycontainer:/foo.txt
```

**Copy file from container to host**

```bash
docker cp mycontainer:/foo.txt foo.txt
```

For more information, please refer [here](https://github.com/wsargent/docker-cheat-sheet)
