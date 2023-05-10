# Pods

## What is a pod?

Pods are the smallest deployable units of computing that you can create and manage in Kubernetes.

A Pod is a group of one or more containers, with shared storage and network resources, and a specification for how to run the containers.

The shared context of a Pod is a set of Linux namespaces, cgroups, and potentially other facets of isolation - the same things that isolate a Docker container.

In terms of Docker concepts, a Pod is similar to a group of Docker containers with shared namespaces and shared filesystem volumes.

![](https://user-images.githubusercontent.com/17776979/207418910-001b3462-a687-4f53-a7f7-75c67c1eaf50.png)

## Pod networking

Each Pod is assigned a unique IP address for each address family. Every container in a Pod shares the network namespace, including the IP address and network ports. Inside a Pod (and only then), the containers that belong to the Pod can communicate with one another using `localhost`. The containers in a Pod can also communicate with each other using standard inter-process communications

Containers in different Pods have distinct IP addresses and can not communicate by IPC without special configuration. Containers that want to interact with a container running in a different Pod can use IP networking to communicate.

## Pod lifetime

Pods are created, assigned a unique ID (UID), and scheduled to nodes where they remain until termination (according to restart policy) or deletion. If a Node dies, the Pods scheduled to that node are scheduled for deletion after a timeout period.

Pods do not, by themselves, self-heal. If a Pod is scheduled to a node that then fails, the Pod is deleted; likewise, a Pod won't survive an eviction due to a lack of resources or Node maintenance. Kubernetes uses a higher-level abstraction, called a controller, that handles the work of managing the relatively disposable Pod instances.

## How can we create Pods in Kubernetes?

Pod definition template

```yaml
apiVersion: v1
kind: Pod
metadata: ...
spec: ...
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-2
  labels:
    name: nginx-2
    env: production
spec:
  containers:
    - name: nginx
      image: nginx
```

```sh
kubectl apply -f mypod.yaml
```

## Commands and Arguments

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod
spec:
  containers:
    - name: ubuntu-sleeper
      image: ubuntu-sleeper # entry point: sleep
      args: ["10"] # ~ cmd
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod
spec:
  containers:
    - name: ubuntu-sleeper
      image: ubuntu-sleeper
      command: ["sleep"] # ~ entry point
      args: ["10"] # ~ cmd
```

## ENV variables

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-2
  labels:
    name: nginx-2
    env: production
spec:
  containers:
    - name: nginx
      image: nginx

      envFrom:
        - configMapRef:
            name: env-configmap
        - secretRef:
            name: env-secrets
  env:
    - name: DEBUG
      value: false
    - name:
      valueFrom:
        configMapKeyRef:
    - name:
      valueFrom:
        secretKeyRef:
```

## Init containers

A Pod can have multiple containers running apps within it, but it can also have one or more init containers, which are run before the app containers are started.

Init containers are exactly like regular containers, except:

- Init containers always run to completion.
- Each init container must complete successfully before the next one starts.

If any of the initContainers fail to complete, Kubernetes restarts the Pod repeatedly until the Init Container succeeds.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup mydb; do echo waiting for mydb; sleep 2; done;']
```


## Security context

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
  securityContext: # security in the pod level
    runAsUser: 1000
  containers:
    - name: ubuntu
      image: ubuntu
      command: ["sleep", "1000"]
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
  containers:
    - name: ubuntu
      image: ubuntu
      command: ["sleep", "1000"]
      securityContext: # security in the container level
        runAsUser: 1000
        capabilities:
          add: ["MAC_ADMIN"]
```

## Resource Requirements

Kubernetes defines Limits as the maximum amount of a resource to be used by a container. This means that the container can never consume more than the memory amount or CPU amount indicated. By default K8s sets a limit of one vCPU and 512 Mebibyte to containers.

Requests, on the other hand, are the minimum guaranteed amount of a resource that is reserved for a container. By default k8s assumes that a pod requires: 0.5 CPU, 256 Mi

If you know that your application need more than this. You can modify these values by specifying them in your pod definition.

![](https://user-images.githubusercontent.com/17776979/227699096-f6378c23-a5a4-4eeb-b160-c0c61abaee79.png)

What does `cpu: 1` mean?

- 1 AWS vCPU
- 1 GCP Core
- 1 Azure Core
- 1 Hyperthread

1000 milicores = 1 core
1 core = 1024 shares
100 milicores = 102 shares

Memory

- 1G (Gigabyte) (1.000.000.000 bytes), 1M (Megabyte) (1.000.000 bytes), 1K (Kilobyte) (1.000 bytes)
- 1Gi (Gibibyte) (1.073.741.824 bytes), 1Mi (Mebibyte) (1.048.576 bytes), 1Ki (Kibibyte) (1.024 bytes)

## Useful commands

```sh
kubectl get pods
```

```sh
kubectl port-forward {pod_name} 8080:80
```

```sh
kubectl delete pod {pod_name}
```

```sh
kubectl describe pod {pod_name}
```

```sh
kubectl get pod <pod-name> -o yaml > pod-definition.yaml
```

```sh
kubectl edit pod <pod-name>
```
