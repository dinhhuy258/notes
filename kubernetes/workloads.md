---
title: Workloads
parent: Kubernetes
nav_order: 1
---

# Workloads

A workload is an application running on Kubernetes. Whether your workload is a single component or several that work together, on Kubernetes you run it inside a set of pods
In Kubernetes, a Pod represents a set of running containers on your cluster.

Kubernetes provides several built-in workload resources:

- Deployment and ReplicaSet
- StatefulSet
- DaemonSet
- Job and CronJob

## 1. Pods

### What is a pod?

Pods are the smallest deployable units of computing that you can create and manage in Kubernetes.

A Pod is a group of one or more containers, with shared storage and network resources, and a specification for how to run the containers.

The shared context of a Pod is a set of Linux namespaces, cgroups, and potentially other facets of isolation - the same things that isolate a Docker container.

In terms of Docker concepts, a Pod is similar to a group of Docker containers with shared namespaces and shared filesystem volumes.

![](../assets/images/kubernetes/pod.png)

### Pod networking

Each Pod is assigned a unique IP address for each address family. Every container in a Pod shares the network namespace, including the IP address and network ports. Inside a Pod (and only then), the containers that belong to the Pod can communicate with one another using `localhost`.
When containers in a Pod communicate with entities outside the Pod, they must coordinate how they use the shared network resources (such as ports).
The containers in a Pod can also communicate with each other using standard inter-process communications
Containers in different Pods have distinct IP addresses and can not communicate by IPC without special configuration. Containers that want to interact with a container running in a different Pod can use IP networking to communicate.

### Pod lifetime

Pods are created, assigned a unique ID (UID), and scheduled to nodes where they remain until termination (according to restart policy) or deletion. If a Node dies, the Pods scheduled to that node are scheduled for deletion after a timeout period.

Pods do not, by themselves, self-heal. If a Pod is scheduled to a node that then fails, the Pod is deleted; likewise, a Pod won't survive an eviction due to a lack of resources or Node maintenance. Kubernetes uses a higher-level abstraction, called a controller, that handles the work of managing the relatively disposable Pod instances.

## 2. ReplicaSet

A ReplicaSet ensures that a specified number of pod replicas are running at any given time.

Example:

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
spec:
  replicas: 2
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: php-redis
        image: gcr.io/google_samples/gb-frontend:v3
```

The `apiVersion`, `kind`, `metadata` fields are mandatory with all k8s objects.

- Line 1: We specified that the `apiVersion` is `apps.v1`
- Line 2-3: The `kind` is `ReplicaSet` and `metadata` has the `name` key set to `frontend`
- Line 5-6: The first field we defined in the `spec` section is `replicas`. It sets the desired number of replicas of the pod. In this case the ReplicaSet should ensure that 2 pods should run concurrently.
- Line 7: The next `spec` section is the `selector`. We use it to select which pods should be included in the ReplicaSet. If pods that match the `selector` exist, ReplicaSet will do nothing. If they don't, it will create as many as pods that match the value of the `replicas` field. Not only that ReplicaSet creates the pods that are missing, but also monitors the cluster and ensures that the desired number of `replicas` is always running.
- Line 8-9: We used `spec.selector.matchLabels` to specify labels. They must match the labels defined in `spec.tempalte`. In our case, ReplicaSet will look for Pods with `tier` set to `frontend`. If pods with this label does not exist, it will create them using the `spec.template` section.
- Line 10-13: The last section of the `spec` field is the `template` and it has the same schema as a Pod specification. At a minimum, the labels of the `spec.template.metadata.labels` section must match those specified in the `spec.selector.matchLabels`.
- Line 14-17: Finally the `spec.template.spec` section the `containers` definition.

![](../assets/images/kubernetes/replica_set.png)

### The process of creating ReplicaSet

1. K8s client (kubectl) sent a request to the API server requesting the creation of ReplicaSet
2. The controller is watching the API server for new events and it detected that there is a new ReplicaSet objecto
3. The controller creates 2 new pod definitions because we have configured replica value as 2 in the above example
4. The scheduler is watching the API server for new events and it detected that there are 2 unasigned pods
5. The scheduler decided which node to assign the Pod and sent that information to the API server
6. Kublet is also watching the API server. It detected that the 2 pods were assigned to the node it is running on
7. Kublet sent request to Docker requesting the creation of the containers that form the Pod.
8. Finally, Kublet sent a request to the API server notifying it that the pods were created successfully

![](../assets/images/kubernetes/replica_set_creation.png)

A ReplicaSet ensures that a specified number of pod replicas are running at any given time. However, a Deployment is a higher-level concept that manages ReplicaSets and provides declarative updates to Pods along with a lot of other useful features. Therefore, k8s recommend using Deployments instead of directly using ReplicaSets, unless you require custom update orchestration or don't require updates at all.

This actually means that you may never need to manipulate ReplicaSet objects: use a Deployment instead, and define your application in the spec section.
