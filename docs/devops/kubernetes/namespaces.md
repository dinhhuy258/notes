# Namespaces

In Kubernetes, namespaces provides a mechanism for isolating groups of resources within a single cluster.

![](https://user-images.githubusercontent.com/17776979/224064740-988a4e05-23af-43fa-86d5-c317565f63be.png)

## Initial namespaces

Kubernetes starts with four initial namespaces:

- **default** Kubernetes includes this namespace so that you can start using your new cluster without first creating a namespace.

- **kube-node-lease** This namespace holds Lease objects associated with each node. Node leases allow the kubelet to send heartbeats so that the control plane can detect node failure.

- **kube-public** This namespace is readable by all clients (including those not authenticated). This namespace is mostly reserved for cluster usage, in case that some resources should be visible and readable publicly throughout the whole cluster.

- **kube-system** The namespace for objects created by the Kubernetes system.

## Working with Namespaces

```sh
# list namespaces
kubectl get ns
kubectl get namespaces

# create namespace
kubectl create namespace <namespace-name>

# list all objects from default, dev1 & dev2 namespaces
kubectl get all -n default
kubectl get all -n dev1
kubectl get all -n dev2

# deploy all k8s objects
kubectl apply -f kube-manifests/
kubectl apply -f kube-manifests/ -n dev1
kubectl apply -f kube-manifests/ -n dev2

# switch namespace
kubectl config set-context $(kubectl config current-context) --namespace=dev
```

## Namespaces and DNS

When you create a Service, it creates a corresponding DNS entry. This entry is of the form `<service-name>.<namespace-name>.svc.cluster.local`, which means that if a container only uses <service-name>, it will resolve to the service which is **local** to a namespace. As a result, all namespace names must be valid [RFC 1123 DNS labels](https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#dns-label-names).
