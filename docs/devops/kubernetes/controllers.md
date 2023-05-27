# Controllers

Controller are using to monitor k8s objects and respond accordingly.

## ReplicaSet (old term ReplicationController)

- High availability
- Load balancing + Scaling

A ReplicaSet ensures that a specified number of pod replicas are running at any given time.

Example:

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
spec:
  replicas: 2 # number of replicas
  selector:
    matchLabels:
      tier: frontend
  template:
    # same as pod definition - begin
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
        - name: php-redis
          image: gcr.io/google_samples/gb-frontend:v3
    # same as pod definition - end
```

The selector in the ReplicaSet definition helps the ReplicaSet identify which pods belong to it. The selector enables the ReplicaSet to manage pods that were not created as part of the ReplicaSet creation.

For example, if there were pods created before the creation of the ReplicaSet that matched the labels specified in the selector, the ReplicaSet would also consider those pods when creating replicas.

![](https://user-images.githubusercontent.com/17776979/222917753-7b914c80-7f9f-40f4-b56f-e0db6271a1b4.png)

### The process of creating ReplicaSet

1. K8s client (kubectl) sent a request to the API server requesting the creation of ReplicaSet
2. The controller is watching the API server for new events and it detected that there is a new ReplicaSet object
3. The controller creates 2 new pod definitions because we have configured replica value as 2 in the above example
4. The scheduler is watching the API server for new events and it detected that there are 2 unasigned pods
5. The scheduler decided which node to assign the Pod and sent that information to the API server
6. Kublet is also watching the API server. It detected that the 2 pods were assigned to the node it is running on
7. Kublet sent request to Docker requesting the creation of the containers that form the Pod.
8. Finally, Kublet sent a request to the API server notifying it that the pods were created successfully

![](https://user-images.githubusercontent.com/17776979/222917855-040c27d8-2d46-4d1a-b87f-14219a751074.png)

## Deployments

A ReplicaSet ensures that a specified number of pod replicas are running at any given time. However, a Deployment is a higher-level concept that manages ReplicaSets and provides declarative updates to Pods along with a lot of other useful features. Therefore, k8s recommend using Deployments instead of directly using ReplicaSets, unless you require custom update orchestration or don't require updates at all.

Deployment manages the overall lifecycle of an application, including rolling updates and rollbacks, while a ReplicaSet ensures that a specified number of identical replicas of the application are running at any given time. Deployments use ReplicaSets to manage scaling and ensure the desired state is achieved.

This actually means that you may never need to manipulate ReplicaSet objects: use a Deployment instead, and define your application in the spec section.

### Define a zero-downtime deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: go-demo-2-api
spec:
  replicas: 3
  selector:
    matchLabels:
      type: api
      service: go-demo-2
  minReadySeconds: 1
  progressDeadlineSeconds: 60
  revisionHistoryLimit: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  template:
    metadata:
      labels:
        type: api
        service: go-demo-2
        language: go
    spec:
      containers:
        - name: api
          image: vfarcic/go-demo-2
          env:
            - name: DB
              value: go-demo-2-db
          readinessProbe:
            httpGet:
              path: /demo/hello
              port: 8080
            periodSeconds: 1
          livenessProbe:
            httpGet:
              path: /demo/hello
              port: 8080
```

- `spec.minReadySeconds`: defines the number of seconds before k8s starts considering the Pod healthy. The default value is `0`, meaning that the Pods will be considered available as soon as they are ready and, when specified `livenessProbe` returns OK.
- `spec.revisionHistoryLimit`: defines the number of old `ReplicaSet` we can rollback (default value is `10`)
- `spec.strategy.type`: can be either `RollingUpdate` or `Recreate` type.

### Deployment strategies

#### Recreate

The recreate strategy is a dummy deployment which consists of shutting down version A then deploying version B after version A is turned off. This technique implies downtime of the service that depends on both shutdown and boot duration of the application.

![](https://user-images.githubusercontent.com/17776979/223171910-eefe6615-e6fa-4230-b1ef-9270ce0dbf57.gif)

Pros:

- Easy to setup
- Application state entirely renewed.

Cons:

- High impact on the user, expect downtime that depends on both shutdown and boot duration of the application.

#### Rolling update

The rolling update deployment strategy consists of slowly rolling out a version of an application by replacing instances one after the other until all the instances are rolled out.

![](https://user-images.githubusercontent.com/17776979/223172285-eeb87a2c-c1e8-434c-ae3e-725b143e39c6.gif)

Pros:

- Easy to setup
- Version is slowly released across instances
- Convenient for stateful applications that can handle rebalancing of the data.

Cons:

- Rollout/rollback can take time.
- Supporting multiple APIs is hard.
- No control over traffic.
