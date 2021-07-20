# Ingress

## 1. What is a ingress?

Ingress is an object that allows access to the k8s services within the cluster from outside. Traffic routing is controlled by rules defined on the Ingress Controller.

![](../../assets/images/kubernetes/ingress.png)

An ingress may be configurated to make services externally-reachable URLs, load balance traffic, terminate SSL/TLS, and offer name-based virtual hosting.

You must have an Ingress Controller to satisfy an Ingress. Only creating an Ingress resource has no effect.

Example:

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: minimal-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
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

## 2. Ingress rules

Each HTTP each contains the following information:

- An optional host. In the above example, no host is specified, so the rule applies to all inbound HTTP traffic throught the IP adress specified. If a host is provided (eg: dinhhuy258.com) the rules apply to that host.
- A list of paths (Eg: `/testpath`), each of which has an associated backend defined with a `service.name` and `service.port.name` or `service.port.number`. Both the host and path must match the content of an incomming request before the load balancer directs traffic to the referenced Service.

## 3. Ingress path types

Each path in Ingress is required to have a corresponding path type. There are three supported path types:

- `ImplementationSpecific`: With this path type, matching is up to the IngressClass. Implementation can treat this as a separate `pathType` or treat it identically to `Prefix` or `Exact` path types.
- `Exact`: Match the URL path exactly and with case sensitive.
- `Prefix`: Match based on a URL path prefix split by `/`

**Note:** If the last element of the path is a substring of the last element in the request path, it is not match (Eg: `/foo/bar` matches `/foo/bar/baz` but does not match `/foo/barbaz`)

## 4. TLS

You can secure an Ingress by specifying a Secret that contains a TLS private key an certificate.

Eg:

```
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

```
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

## 5. Load balancing

An Ingress Controller is bootstrapped with some load balancing policy settings that it applies to all Ingress, such as the load balancing algorithm, backend weight scheme, and others. More advanced load balancing concepts (eg: persistent session, dynamic weights) are not yet exposed through the Ingress. You can instead get these features through the load balancer used for a Service.
