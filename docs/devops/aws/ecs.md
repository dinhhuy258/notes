# AWS ECS

AWS ECS (Elastic Container Service) is a managed container orchestration platform that enables fast deployment and scaling of containerized workloads.

Docker containers management on AWS

- Amazon Elastic Container Service (Amazon ECS)
- Amazon Elastic Kubernetes Service (Amazon EKS)
- AWS Fargate (serverless)
- Amazon ECR (store container images)

## Amazon ECS cluster

An Amazon ECS cluster is a logical grouping of tasks or services. Your tasks and services are run on infrastructure that is registered to a cluster. The infrastructure capacity can be provided by:

- AWS Fargate, which is serverless infrastructure that AWS manages
- Amazon EC2 instances that you manage
- On-premise server or virtual machine (VM) that you manage remotely

A cluster may contain a mix of tasks hosted on AWS Fargate, Amazon EC2 instances, or external instances

![](https://user-images.githubusercontent.com/17776979/197403659-ac68aef5-55f7-4e0a-be95-d2ac4b242544.png)

### EC2 Launch Type

The EC2 launch type allows you to run your containerized applications on a cluster of Amazon EC2 instances that you manage.

![](https://user-images.githubusercontent.com/17776979/202229455-41c553c8-0df8-4c27-bfa8-8cfca724ffa1.png)

![](https://user-images.githubusercontent.com/17776979/197403774-66b78b9f-2b09-460c-a439-82825e117cf8.png)

[AWS ECS Cluster using the EC2 Launch Type](https://aws.plainenglish.io/aws-ecs-cluster-using-the-ec2-launch-type-cb5ae2347b46)

### Fargate Launch Type

The Fargate launch type allows you to run your containerized applications without the need to provision and manage the backend infrastructure. Just register your task definition and Fargate launches the container for you. (serverless)

- You just create task definitions
- AWS just run ECS tasks for you based on the CPU/RAM you need
- To scale, just increase the number of tasks.

![](https://user-images.githubusercontent.com/17776979/202230078-c3f886f4-a31c-4d6c-b17c-f56be983fc07.png)

### Load Balancer Integrations

![](https://user-images.githubusercontent.com/17776979/202230775-c198c3e9-a903-482b-9eef-688dba95f515.png) -

[How to route traffic to your Docker container in AWS ECS using an Application Load Balancer](https://appfleet.com/blog/route-traffic-to-aws-ecs-using-application-load-balancer/)

**EC2 Launch Type**

- We get a Dynamic Host Port Mapping if you define only the container port in the task definition
- ALB finds the right port on your EC2 instance
- We must allow on EC2 instance's Security Group **any port** from the ABL's Security Group

**Fargate**

- Each task has a unique private IP
- Only define the container port (host port is not applicable)

### Data Volume

### ECS Service Auto Scalling

### Task definition

Task definition are metadata in JSON form to tell ECS how to run a Docker container.

It contains crucial information, such as:

- Image name
- Port binding for Container and Host
- Memory and CPU required
- Environment variables
- Networking information
- IAM role
- Logging configuration (CloudWatch)

We can define up to 10 contains in a Task Definition

## Amazon ECR

Amazon ECR is a fully managed container registry offering high-performance hosting, so you can reliably deploy application images and artifacts anywhere.

## Amazon EKS

Amazon Elastic Kubernetes Service (Amazon EKS) is a managed service that you can use to run Kubernetes on AWS without needing to install, operate, and maintain your own Kubernetes control plane or nodes.

![](https://user-images.githubusercontent.com/17776979/202235496-3b49368c-1fa5-48f1-b366-35d9ead27a2d.png)
