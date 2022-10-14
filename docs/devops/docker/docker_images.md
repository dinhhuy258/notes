# Docker Images

## Dockerfile

A Dockerfile is a text configuration file written using a special syntax. It describes step-by-step instructions of all the commands you need to run to assemble a Docker Image.

It contains 3 main parts:

1. Specify a base image
2. Run some commands to install additional programs
3. Specify a command to run on container startup

Example

```bash
FROM ubuntu:18.04
COPY . /app
RUN make /app
CMD python /app/app.py
```

### Dockerfile commands

**FROM**

```bash
FROM <image> [AS <name>]
```

FROM is used to define the base image to start the build process. Every Dockerfile must start with the FROM instruction.

**LABEL**

```bash
LABEL <key>="<value>"
```

Labels are used in Dockerfile to help organize your Docker Images. A label is a key-value pair, stored as a string. You can specify multiple labels for an object, but each key must be unique within an object.

```bash
LABEL version="1.0"
```

Using command `docker inspect [OPTIONS] NAME|ID [NAME|ID...]` to inspect image's metadata.

**ENV**

```bash
ENV <key>="<value>"
```

This command used to set the environment variables that is required to run the project.

```bash
ENV HTTP_PORT="9000"
```

**WORKDIR**

```bash
WORKDIR /path/to/workdir
```

WORKDIR tells Docker that the rest of the commands will be run in the context of the `/path/to/workdir` folder inside the image.

**RUN**

RUN has 2 forms:

```bash
RUN <command>
RUN ["executable", "param1", "param2"]
```

The RUN instruction will execute any commands in a new layer on top of the current image and commit the results. The resulting committed image will be used for the next step in the Dockerfile.

Example

```bash
RUN /bin/bash -c 'source $HOME/.bashrc; echo $HOME'
```

**COPY**

COPY has two forms:

```bash
COPY <src>... <dest>
COPY ["<src>",... "<dest>"] (this form is required for paths containing whitespace)
```

The COPY command is used to copy one or many local files or folders from source and adds them to the filesystem of the containers at the destination path.

It builds up the image in layers, starting with the parent image, defined using FROM.The Docker instruction WORKDIR defines a working directory for the COPY instructions that follow it.

The <dest> is an absolute path, or a path relative to `WORKDIR`, into which the source will be copied inside the destination container.

```bash
COPY test relativeDir/   # adds "test" to `WORKDIR`/relativeDir/
COPY test /absoluteDir/  # adds "test" to /absoluteDir/
```

**ADD**

ADD has two forms:

```bash
ADD <src>... <dest>
ADD ["<src>",... "<dest>"]
```

The ADD command is used to add one or many local files or folders from the source and adds them to the filesystem of the containers at the destination path.

It is Similar to COPY command but it has some additional features:

- If the source is a local tar archive in a recognized compression format, then it is automatically unpacked as a directory into the Docker image.
- If the source is a URL, then it will download and copy the file into the destination within the Docker image. However, Docker discourages using ADD for this purpose.

```bash
ADD rootfs.tar.xz /
ADD http://example.com/big.tar.xz /usr/src/things/
```

**EXPOSE**

```bash
EXPOSE <port> [<port>/<protocol>...]
```

The EXPOSE command informs the Docker that the container listens on the specified network ports at runtime. You can specify whether the port listens on TCP or UDP, and the default is TCP if the protocol is not specified.

**EXPOSE will not** allow communication via the defined ports to containers outside of the same network or to the host machine.

**VOLUME**

```bash
VOLUME ["/data"]
```

The VOLUME instruction creates a mount point with the specified name and marks it as holding externally mounted volumes from native host or other containers.

**USER**

```bash
USER user
```

By default, a Docker Container runs as a Root user. You can change or switch to a different user inside a Docker Container using the USER Instruction. For this, you first need to create a user and a group inside the Container:

```bash
RUN groupadd -r postgres && useradd --no-log-init -r -g postgres postgres
```

**ENTRYPOINT**

ENTRYPOINT has two forms:

```bash
ENTRYPOINT ["executable", "param1", "param2"] (exec form, preferred)
ENTRYPOINT command param1 param2 (shell form)
```

An ENTRYPOINT allows you to configure a container that will run as an executable. ENTRYPOINT sets the command and parameters that will be executed first when a container is run. Any command-line arguments passed to `docker run <image>` will be appended to the ENTRYPOINT command.

You can override ENTRYPOINT instructions using the `docker run --entrypoint`

```bash
ENTRYPOINT [ "sh", "-c", "echo $HOME" ]
```

**CMD**

The CMD instruction has three forms:

```bash
CMD ["executable","param1","param2"] (exec form, this is the preferred form)
CMD ["param1","param2"] (as default parameters to ENTRYPOINT)
CMD command param1 param2 (shell form)
```

The main purpose of a CMD is to provide defaults when executing a container. These will be executed after the entry point.

Any command-line arguments passed to `docker run <image>` will override all elements specified using CMD.

Example

```bash
CMD ["executable","param1","param2"]
```

If they omit the executable, you must specify an ENTRYPOINT instruction as well.

```bash
CMD ["param1","param2"] (as default parameters to ENTRYPOINT)
```

**NOTE**: There can only be one CMD instruction in a Dockerfile. If you want to list more than one CMD, then only the last CMD will take effect.

**Understand how CMD and ENTRYPOINT interact**

1. Dockerfile should specify at least one of CMD or ENTRYPOINT commands.
2. ENTRYPOINT should be defined when using the container as an executable.
3. CMD should be used as a way of defining default arguments for an ENTRYPOINT command or for executing an ad-hoc command in a container.
4. CMD will be overridden when running the container with alternative arguments.

**Build image**

```bash
docker build .
docker build -t {tag_name} .
docker build -t {tag_name} Dockerfile
```

**List image**

```bash
docker images
```

**Remove image**

```bash
docker rmi {image_name:tag_name}
```

## Docker Layer

A Docker image consists of several layers. Each layer corresponds to certain instructions in your Dockerfile. The following instructions create a layer: `RUN`, `COPY`, `ADD`. The other instructions will create intermediate layers and do not influence the size of your image.

![](https://user-images.githubusercontent.com/17776979/195864994-3117fe56-683c-4438-b43c-0891a17af1fc.png)
