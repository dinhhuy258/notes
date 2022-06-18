# Service Mesh

A service mesh is a dedicated infrastructure layer that adds features to a network between services:
- Service to service communication (service discovery and encryption)
- Observability (monitoring and tracking)
- Resiliency (circuit breakers and retries)

![](../assets/images/distributed-system/service_mesh.png) 

**Without a service mesh**: each microservice implements business logic and cross cutting concerns (CCC) by itself.
**With a service mesh**: many CCCs like traffic metrics, routing, and encryption are moved out of the microservice and into a proxy.

## Service Mesh Implementations

- Istio
- Linkerd
- Consul
