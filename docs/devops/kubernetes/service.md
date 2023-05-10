# Service

K8s services enable communication between various components within and outside of the application.

Kubernetes pods are created and destroyed to match the state of your cluster. If you use Deployment to run your app, it can create and destroy pods dynamically.

Each pods get it own IP address, however in Deployment, the set of pods running in one moment in time could be different from the set of pods running that application a moment later.

It leads to a problem that if set of pods (backend) provides functionality for other pods (frontend) inside your cluster, how do the frontend can find out and keep track which ip address to connect to?

Kubernetes Services provides addresses through which associated pods can be accessed.

![](https://user-images.githubusercontent.com/17776979/237433000-8193e2e5-b572-460a-9ce5-e0a59794f33d.png)

## 1. Define a service

For example, suppose you have a set of pods where each listens on TCP port `9376` and contains a label `app=MyApp`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```

This specification creates a new Service object named `my-service` which targets TCP port `9376` on any Pod with the `app=MyApp` label.

Kubernetes assigns this Service an IP address (sometimes called the "cluster IP"), which is used by the Service proxies.

The controller for the Service selector continuously scans for Pods that match its selector, and then POSTs any updates to an Endpoint object also named `my-service`.

## 2. Services without selectors

Services most commonly abstract access to Kubernetes Pods, but they can also abstract other kinds of backends. For example:

- You want to have an external database cluster in production, but in your test environment you use your own databases.
- You want to point your Service to a Service in a different Namespace or on another cluster.

Eg:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```

Because this Service has no selector, the corresponding Endpoints object is not created automatically. You can manually map the Service to the network address and port where it's running, by adding an Endpoints object manually:

```yaml
apiVersion: v1
kind: Endpoints
metadata:
  name: my-service
subsets:
  - addresses:
      - ip: 192.0.2.42
    ports:
      - port: 9376
```

## 3. Type of service

Kubernetes Services allow you to specify what kind of Service you want. The default is `ClusterIP`.

### ClusterIP

Exposes the service on cluster internal IP. Choosing this makes the Service only reachable from within the cluster.

![](https://user-images.githubusercontent.com/17776979/237435236-7ad3791e-716d-44ce-8deb-3f6ea66cd982.png)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  selector:
    app: MyApp
  type: ClusterIP
  ports:
    - targetPort: 80
      port: 80
```

### NodePort

Service helps us map a port on the node to a port on the pod. NodePort exposes the service on each Node'IP at a static port. A `ClusterIP` is automatically created. You will be able to reach the NodePort service from outside the cluster, by requesting <NodeIP>:<NodePort>

![](https://user-images.githubusercontent.com/17776979/237435397-387fbc7c-84da-4839-a6d3-707de5fdf4c5.png)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  type: NodePort
  ports:
    - targetPort: 80
      port: 80
      nodePort: 30008
```

### LoadBalancer

A LoadBalancer service is based on the NodePort service, and adds the ability to configure external load balancers in public and private clouds. It exposes services running within the cluster by forwarding network traffic to cluster nodes.

![](https://user-images.githubusercontent.com/17776979/237435539-3c8cfc8d-b43f-4b3e-b5c6-fa8b95533761.png)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-loadbalancer-service
spec:
  type: LoadBalancer
  clusterIP: 10.0.160.135
  loadBalancerIP: 168.196.90.10
  selector:
    app: nginx
  ports:
    — name: http
      protocol: TCP
      port: 80
      targetPort: 8080
```

### ExternalName

An ExternalName service maps the service to a DNS name instead of a selector. You define the name using the spec:externalName parameter. It returns a CNAME record matching the contents of the externalName field (for example, my.service.domain.com), without using a proxy.

This type of service can be used to create services in Kubernetes that represent external components such as databases running outside of Kubernetes. Another use case is allowing a pod in one namespace to communicate with a service in another namespace—the pod can access the ExternalName as a local service.

![](https://user-images.githubusercontent.com/17776979/237435688-c050dfb5-a3a2-48f4-9041-e838216fe675.png)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-externalname-service
spec:
  type: ExternalName
  externalName: my.database.domain.com
```

## Useful commands

```sh
kubectl get services
```
