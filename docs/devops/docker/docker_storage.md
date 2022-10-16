# Docker Storage

On a linux system, docker stores data pertaining to images, containers, volumes, etc under `/var/lib/docker`.

When we run the `docker build` command, docker builds one layer for each instruction in the Dockerfile. These image layers are **read-only** layers. When we run the `docker run` command, docker builds container layer(s), which are **read-write** layers.

![](https://user-images.githubusercontent.com/17776979/196044306-179252ea-a38f-40f3-abf4-472a4c0a59fb.png)

You can create new files on the container, for instance, **temp.txt**. You can also modify a file that belongs to the image layers on the container, for instance, **app.py**. When you do this, a local copy of that file is created on the container layer and the changes **only** live on the container — this is called the **Copy-on-Write** mechanism. This is important as several containers and child images use the same image layers. The life of the files on the container is as long as the container is alive. When the container is destroyed, the files/modifications on it are also destroyed.

**Copy on Write**

Copy-on-write or CoW is a technique to efficiently copy data resources in a computer system. If a resource is duplicated but not modified, it is not necessary to create a new resource; the resource can be **shared between the copy and the original**. Modifications must still create a copy, hence the technique: **the copy operation is deferred to the first write**. By sharing resources in this way, it is possible to significantly reduce the resource consumption of unmodified copies, while adding a small overhead to resource-modifying operations.

![](https://user-images.githubusercontent.com/17776979/196044678-a9998796-ee85-4a3e-ba4c-9dadb46e146d.png)

You can create a docker volume by using the `docker volume create` command. This command will create a volume in the `/var/lib/docker/volumes` directory.

```bash
docker volume create data_volume
```

Now when you run the docker run command, you can specify which volume to use using the `-v` flag. This is called Volume Mounting.

```bash
docker run -v data_volume:/var/lib/postgres postgres
```

If the volume does not exist, docker creates one for you. Now, **even if the container is destroyed the data will persist in the volume**.

If you want to have your data on a specific location on the docker host or already have existing data on the disk, you can mount this location on the container as well. This is called Bind Mounting.

```bash
docker run -v /data/postgres:/var/lib/postgres postgres
```

![](https://user-images.githubusercontent.com/17776979/196044916-c1738fd4-52e5-4e15-b33e-67a6d2047b77.png)
