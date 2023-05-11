# Ingress

![](https://user-images.githubusercontent.com/17776979/237624351-834335c2-ff65-4101-9901-bb9ba8b2b8a2.png)

## What is a ingress?

In Kubernetes, an Ingress is an object that allows access to Kubernetes services from outside the Kubernetes cluster. The external traffic could be via HTTP or HTTPS to a service running within your Kubernetes cluster.

Ingress in Kubernetes offers more advanced features for managing and routing traffic to APIs compared to NodePort and LoadBalancer.

- It allows you to direct requests to different services based on domain names, paths, or headers
- Ingress also provides built-in support for SSL/TLS encryption and load balancing
- It simplifies the configuration and centralizes the management of traffic routing rules

## How Does Kubernetes Ingress work

There are 2 concepts in k8s ingress:

1. **Kubernetes Ingress Resource:** Kubernetes Ingress Resource is responsible for storing DNS routing rules in the cluster. It specifies how incoming requests should be handled, such as domain-based routing or SSL/TLS termination.
2. **Kubernetes Ingress Controller:** Kubernetes ingress controllers (Nginx/HAProxy etc.) are responsible for implementing the actual traffic routing based on the rules defined in the **Ingress Resource**. It configures the underlying load balancers or reverse proxies to direct the traffic accordingly.

### Kubernetes Ingress Resource

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: test-ingress
  namespace: dev
spec:
  rules:
  - host: test.apps.example.com
    http:
      paths:
      - backend:
          serviceName: hello-service
          servicePort: 80
```

The above declaration means, that all calls to `test.apps.example.com` should hit the service named `hello-service` residing in the `dev` namespace.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: minimal-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
    # - host: dinhhuy258.com
    # http:
    - http:
        paths:
          - path: /testpath
            pathType: Prefix
            backend:
              service:
                name: test
                port:
                  number: 80
```

**Annotations**

Ingress frequently uses annotations to configure some options depending on the Ingress controller, an example of which is the `rewrite-target` annotation. Different Ingress controllers support different annotations.

**Ingress rules**

Each HTTP each contains the following information:

- An optional host. In the above example, no host is specified, so the rule applies to all inbound HTTP traffic throught the IP adress specified. If a host is provided (eg: dinhhuy258.com) the rules apply to that host.
- A list of paths (Eg: `/testpath`), each of which has an associated backend defined with a `service.name` and `service.port.name` or `service.port.number`. Both the host and path must match the content of an incomming request before the load balancer directs traffic to the referenced Service.

**Ingress path types**

Each path in Ingress is required to have a corresponding path type. There are three supported path types:

- `ImplementationSpecific`: With this path type, matching is up to the IngressClass. Implementation can treat this as a separate `pathType` or treat it identically to `Prefix` or `Exact` path types.
- `Exact`: Match the URL path exactly and with case sensitive.
- `Prefix`: Match based on a URL path prefix split by `/`

**Note:** If the last element of the path is a substring of the last element in the request path, it is not match (Eg: `/foo/bar` matches `/foo/bar/baz` but does not match `/foo/barbaz`)

**TLS**

You can secure an Ingress by specifying a Secret that contains a TLS private key an certificate.

Eg:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: testsecret-tls
  namespace: default
data:
  tls.crt: base64 encoded cert
  tls.key: base64 encoded key
type: kubernetes.io/tls
```

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-example-ingress
spec:
  tls:
  - hosts:
      - https-example.foo.com
    secretName: testsecret-tls
  rules:
  - host: https-example.foo.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
   apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-example-ingress
spec:
  tls:
  - hosts:
      - https-example.foo.com
    secretName: testsecret-tls
  rules:
  - host: https-example.foo.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: service1
            port:
              number: 80
         name: service1
            port:
              number: 80
```

### Kubernetes Ingress Controller

An ingress controller is typically a reverse web proxy server implementation in the cluster. In kubernetes terms, it is a reverse proxy server deployed as kubernetes deployment exposed to a service type Loadbalancer. (Nginx is one of the widely used ingress controllers)

Key things to understand about ingress objects.

1. An ingress object requires an ingress controller for routing traffic.
2. And most importantly, the external traffic does not hit the ingress API, instead, it will hit the ingress controller service endpoint configured directly with a load balancer.

Follow [this](https://devopscube.com/setup-ingress-kubernetes-nginx-controller/) tutorial to learn how to setup an ingress controller in k8s.
