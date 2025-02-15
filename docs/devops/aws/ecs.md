# AWS Elastic Container Service (ECS)

## Amazon Elastic Container Registry

Amazon Elastic Container Registry (ECR) is an AWS managed Docker container registry service. ECR provides a secure and scalable repository to store, manage, and deploy Docker images. ECR offers both public and private registries that can allow multiple repositories in the registries.

## Amazon Elastic Container Service (ECS)
 
![](https://user-images.githubusercontent.com/17776979/197403659-ac68aef5-55f7-4e0a-be95-d2ac4b242544.png)

Amazon ECS is a fully managed service by AWS that helps you deploy, manage, and scale containerized applications. It uses Docker containers to run your apps and is divided into three key layers: Provisioning, Controller, and Capacity.

### 1. Provisioning

This layer is responsible for the tools that help you deploy and manage your containers. You can interact with ECS using:

- AWS SDKs (for integration into custom apps)
- Copilot (a command-line tool to simplify containerized app deployment)
- AWS CLI (Command Line Interface)
- ...

### 2. Controller

The controller layer is responsible for managing containers and their configurations. Here’s how it works:

- **Task Definition**: Think of this as a blueprint for your container. It defines the container's image, CPU, memory, IAM role and other settings.
- **Task/ Service**: A task is simply an instance of a task definition (i.e., a running container).
- **Cluster**: A cluster is a group of tasks or services that run on shared infrastructure. You can run many services and tasks in the same cluster.

### 3. Capacity

This layer handles the infrastructure where containers are actually running. ECS supports three types of infrastructure:

- **On-premises**: Containers can run on your local infrastructure.
- **EC2 Instances**: Containers are deployed on AWS EC2 virtual machines.
- **AWS Fargate**: A serverless option where AWS handles all the infrastructure for you, so you only focus on your containers.

![imgur.png](https://i.imgur.com/tNBbZlD.png)

## EC2 Launch Type

### EC2 Launch Type

![](https://user-images.githubusercontent.com/17776979/202229455-41c553c8-0df8-4c27-bfa8-8cfca724ffa1.png)

### Fargate

![](https://user-images.githubusercontent.com/17776979/202230078-c3f886f4-a31c-4d6c-b17c-f56be983fc07.png)

## Amazon EKS

Amazon Elastic Kubernetes Service (Amazon EKS) is a managed service that you can use to run Kubernetes on AWS without needing to install, operate, and maintain your own Kubernetes control plane or nodes. EKS offers different compute services, from serverless Fargate to EC2 to offer high availability, security, and seamless integration with other AWS services for enhanced development and operational efficiency.

### How EKS works

An EKS cluster consists of two major components: Control plane and Nodes. When provisioning an EKS cluster to run an application, the control plane is hosted on the AWS-managed VPC, whereas the worker nodes and other infrastructure are hosted in the customer-managed VPC.

The worker nodes may communicate with the control plane through the managed API endpoint in the control plane. The worker nodes are connected to the control plane through the API or an EKS-managed ENI, placed in the customer subnets.

![imgur.png](https://i.imgur.com/TGramhN.png)
