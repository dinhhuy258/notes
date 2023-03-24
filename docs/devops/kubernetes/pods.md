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
