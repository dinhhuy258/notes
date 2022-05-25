# Cohesion and coupling

## Cohesion

Cohesion is the degree to which the elements inside a microservice belong together.

A microservice with high cohesion contains elements that are tightly related to each other and united in their purpose.

## Coupling

Coupling is the degree of interdependence between microservices. When services are loosely coupled, a change to one service should not require a change to another.

### Type of coupling

### Domain coupling

Domain coupling describes a situation in which one microservice needs to interact with another microservice, because the first microservice needs to make use of the functionality that the other microservice provides.

In microservice, this type of interaction is largely unavoidable. A microservice-based system relies on multiple microservices collaborating in order for it to do its work. We still want to keep this to a minimum, though; whnever you see a single micorservice depending on multiple downstream services in this way, it can be a cause for concern.
