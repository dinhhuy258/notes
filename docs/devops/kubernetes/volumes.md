# Volumes

A Volume in Kubernetes represents a directory with data that is accessible across multiple containers in a Pod.
The container data in a Pod is deleted or lost when a container crashes or restarts, but when you use a volume, the new container can pick up the data at the state before the container crashes.
The volume outlives the containers in a Pod and can be consumed by any number of containers within that Pod.

## Types of Volumes

K8s supportes several types of volumes. We can categorize the Kubernetes Volumes based on their lifecycle.

Considering the lifecycle of the volumes, we can have:

- **Ephemeral Volumes**, which are tightly coupled with the lifetime of the Node (for example emptyDir, or hostPath) and they are deleted if the Node goes down.
- **Persistent Volumes**, which are meant for long-term storage and are independent of the Pods or Nodes lifecycle. These can be cloud volumes (like gcePersistentDisk, awsElasticBlockStore, azureFile or azureDisk), NFS (Network File Systems) or Persistent Volume Claims (a series of abstraction to connect to the underlying cloud provided storage volumes).

## Ephemeral Volumes

### emptyDir

![](https://user-images.githubusercontent.com/17776979/240946303-8b4b3c3e-fb41-4ac6-8055-b9ae1a9747e1.png)

The emptyDir volume is primarily designed for sharing files between containers within the same pod. When multiple containers are running within a pod and they need to exchange data or share files, an emptyDir volume can be used as a common location for storing and accessing that data.

Here are some key characteristics of the emptyDir volume:

- Lifetime: An emptyDir volume is tied to the lifecycle of a pod. It is created when the pod is created and deleted when the pod is terminated.
- Accessibility: The emptyDir volume is accessible by **all containers** within the same pod. This means that multiple containers running in the same pod can read from and write to the same emptyDir volume.
- Storage Medium: The storage medium used for the emptyDir volume depends on the underlying infrastructure of the cluster. It could be a directory on the host's filesystem or a memory-based filesystem, depending on the configuration.
- Persistence: An emptyDir volume is not designed for persistent storage. If the pod is restarted or rescheduled to a different node, the contents of the emptyDir volume will be lost.

Some use cases for an emptyDir are:

- Scratch space, for a sort algorithm for example
- Checkpointing a long computation for recovery from crashes
- As a cache
- Holding files that a content-manager container fetches while a webserver Container serves the data

Example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-nginx
spec:
  containers:
    - image: nginx
      name: test-nginx
      volumeMounts:
        - mountPath: /cache
          name: cache-volume
  volumes:
    - name: cache-volume
      emptyDir: {}
```

Let’s try applying the YAML file and get into the Pod.

```sh
kubectl apply -f emptydir.yml
pod/test-nginx created

kubectl get pods
NAME         READY   STATUS    RESTARTS   AGE
test-nginx   1/1     Running   0          7s

kubectl exec -it test-nginx -- /bin/bash
root@test-nginx:/# mount | grep -i cache
/dev/vda1 on /cache type ext4 (rw,relatime)
```

If we see the storage medium used for the `emptyDir` mounted on the container we just created, it shows up as `/dev/vda1` a paravirtualization disk driver.

If you set the `emptyDir.medium` field to `Memory`. Kubernetes mounts a tmpfs (RAM-backed filesystem) for you instead.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-nginx
spec:
  containers:
    - image: nginx
      name: test-nginx
      volumeMounts:
        - mountPath: /cache
          name: cache-volume
  volumes:
    - name: cache-volume
      emptyDir:
        medium: Memory
```

### hostPath

![](https://user-images.githubusercontent.com/17776979/240946334-0ebb9765-d7d8-4e1b-80f8-86bcb68569bf.png)

A hostPath volume mounts a file or directory from the host node's filesystem into your Pod.

Here are some key characteristics of the hostPath volume:

- Accessibility: The hostPath volume allows containers within the pod to access files on the host node's filesystem. It enables sharing of files between the host and the containers.
- Storage Medium: The hostPath volume mounts a directory or file from the host's filesystem. The actual storage medium depends on the host node's configuration and can be a local disk, network-attached storage, or any other filesystem accessible to the node.
- Persistence: The hostPath volume is not tied to the lifecycle of the pod. If the pod is restarted or rescheduled to a different node, the hostPath volume will still be available as long as the file or directory exists on the host node.
- Security Considerations: The hostPath volume grants access to the host's filesystem, so it should be used with caution. Granting containers access to sensitive files or system directories on the host can pose security risks.

Some use cases for an hostPath are:

- Running a container that needs access to Docker internals; use a hostPath of /var/lib/docker
- Persist database data, application data

Example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
    - image: k8s.gcr.io/test-webserver
      name: test-container
      volumeMounts:
        - mountPath: /test-pd
          name: test-volume
  volumes:
    - name: test-volume
      hostPath:
        # directory location on host
        path: /data
        # this field is optional
        type: Directory
```

The supported values for field `type` are:

- `DirectoryOrCreate`: If nothing exists at the given path, an empty directory will be created there as needed with permission set to `0755`, having the same group and ownership with Kubelet.
- `Directory`: A directory must exist at the given path
- `FileOrCreate`: If nothing exists at the given path, an empty file will be created there as needed with permission set to `0644`, having the same group and ownership with Kubelet.
- `File`: A file must exist at the given path
- `Socket`: A UNIX socket must exist at the given path
- `CharDevice`: A character device must exist at the given path
- `BlockDevice`: A block device must exist at the given path
