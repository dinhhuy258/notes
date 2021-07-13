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

Pods are the smallest deployable units of computing that you can create and manage in Kubernetes.

A Pod is a group of one or more containers, with shared storage and network resources, and a specification for how to run the containers.

![](../assets/images/kubernetes/pod.png)
