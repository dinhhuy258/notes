# Cluster Architecture

![](https://user-images.githubusercontent.com/17776979/205487822-cef86459-1813-4703-939b-719a3c3028c6.png)

Kubernetes Architecture has the following main components:

- Master nodes
- Worker/Slave nodes
- Distributed key-value store (etcd)

## Master Node

It is the entry point for all administrative tasks which is responsible for managing the Kubernetes cluster. There can be **more than one** master node in the cluster to check for fault tolerance. More than one master node puts the system in a High Availability mode, in which one of them will be the main node which we perform all the tasks.

For managing the cluster state, it uses `etcd` in which all the master nodes connect to it.

Master node consists of 4 components:

**API server**

API server is the central management entity that receives all REST requests for modifications (to pods, services, replication sets/controllers and others), serving as frontend to the cluster. Also, this is the only component that communicates with the etcd cluster, making sure data is stored in etcd and is in agreement with the service details of the deployed pods.

**Scheduler**

Scheduler helps schedule the pods (a co-located group of containers inside which our application processes are running) on the various nodes based on resource utilization. It reads the service’s operational requirements and schedules it on the best fit node.

**Controller manager**

Controller manager runs a number of distinct controller processes in the background (for example, replication controller controls number of replicas in a pod, endpoints controller populates endpoint objects like services and pods, and others) to regulate the shared state of the cluster and perform routine tasks. When a change in a service configuration occurs (for example, replacing the image from which the pods are running, or changing parameters in the configuration yaml file), the controller spots the change and starts working towards the new desired state.

**etcd**

etcd is a simple, distributed key value storage which is used to store the Kubernetes cluster data (such as number of pods, their state, namespace, subnets, confimaps, secrets, etc), API objects and service discovery details. It is only accessible from the API server for security reasons.

etcd can be part of the Kubernetes Master, or, it can be configured externally.

## Nodes

Node is a worker machine and that is where containers will be launched by k8s.

**Container runtime**

Container runtime helps to run and manage the container life cycle.

Some of the container runtime examples are:

- containerd
- CRI-O
- Docker

**Kubelet**

It's a agent which communicates with the master node and executes on nodes or worker nodes. It is the main service on a node, regularly taking in new or modified pod specifications (primarily through the kube-apiserver) and ensuring that pods and their containers are healthy and running in the desired state. This component also reports to the master on the health of the host where it is running.

**Kube-proxy**

K8s cluster can have multiple worker nodes and each node has multiple pods running, so if one has to access this pod, they can do so via Kube-proxy.

In order to access the pod via k8s services, there are certain network policies, that allow network communication to your Pods from network sessions inside or outside of your cluster. These rules are handled via kube-proxy
