# Kubernetes-on-AWS-EKS

## Overview 

- Container Services on AWS

- Create K8s Cluster with EKS using Managment Console Or UI

- Configure auto scaling for K8 . Which is common practice to save infrastructure cost when running my workflow on AWS

- Create K8s Cluster with EKS CLI

- Build CI/CD Piplines to Deploy to EKS cluster

## Container Services on AWS 

**Overview**

- Elastic Container Service (ECS)

- Elastic Kubernetes Service (EKS)

- EC2 vs Fargate

- Elastic Contaoner Registry (ECR)

**Container Orchestration**

<img width="400" alt="Screenshot 2025-02-27 at 10 39 11" src="https://github.com/user-attachments/assets/0960ef88-b7c0-4cf0-a296-f3b4d582f13f" />

- If I deploy Microservice I would need Container for each . Container scale very quick

----How to manage a lot of Container ?----
----How to know how much resources remain?----
----How to know where to shedule the next Container ?----
----How to know Container not running anymore?----
----How to set up Load Balancer for that ?----

- Container Orchestration tools = Managing, scaling and deploying container

## ECS Elastic Container Service 

- ECS is a Container Orchestration Service

- Manage a whole lifecycle of the Container : Start, re-schedule, loadbalance

 **How does ECS work ?**

 - I create an ECS AWS cluster . ECS cluster will contain all the services that nessesary for managing the individual container

 - ECS cluster represent a Control Plane for all my Virtual Machine that are running Container

 - And the Control Plane,  and the Managing Service in the Control Plane manage these individual containers and do all these things that I mention before like resheduling container, making sure the desire state are the same and essentilly managing the whole lifecycle of the container from being started and sheduled to being removed

 - When I created ECS cluster which is Control Plane and my Container are running on Virtual Machine . That Virtual Machine are EC2 instances . EC2 instances will be connected to this ECS cluster that I created . EC2 will be managed by Control Plane process on ECS Cluster

----Which Services are running on my EC2 instances?----

  - Container runtime : Docker ...

  - ECS Agent installed : This way ECS process Control Plane can communicate with each individual EC2 instances and manage them

  - In the end I have the Control Plane (ECS) that helped me manage all my Container and Automate the complex these that I don't want to do manually .

  - However I have to manage EC2 Virtual Machine .
    I have to create EC2 instances
    I have to join the ECS Cluster
    I have to make sure EC2 have enough Resources for the next Container when I schedule new Container
    I have to Mange OS

**ECS with Fargate**

<img width="400" alt="Screenshot 2025-02-27 at 11 24 43" src="https://github.com/user-attachments/assets/bcbfe33e-26f3-4f8f-8cc6-5bd819237bc1" />

- What if I want to delegate managment of a Infrastructure also to AWS ?

- Container lifecycle manage by AWS

- Hosting Infrastructure manage by AWS

- Everything above called Fargate

- Fargate is a alternative to EC2 Instances . Instead of create and provision EC2 instances and connect to ECS Cluster . I create Fargate instances and connect it to ECS cluster

----How does Fargate work?----

- Fargate is Serverless way to launch Container

- Serverless mean No Server on AWS account . It is managed by AWS

- I take the Container that I want to Deploy on AWS . That I want to manage by AWS services or Container Orchestration tool and I tell AWS here my container please run it . At this point I don't have any Virtual Machine that will run my Container . I haven't provision any server yet . I just have Control Plane ECS and the Fargate interface

- Fargate will then analyze my Container . To see how much Resources my Container need to run ? How much CPU . How much RAM . How much Storage it need to be deploy and to run ?

- And then Fargate itsel will automatically provision a server resources for that specific container . And then it will deploy and run that container on the provisioned server resources

- And everytime I add a new Container to Fargate . Fargate will do the same

----Fargate Advandtage----

- No need to manage and provision servers

- It all happen Per Demand : Everytime I add new Container . It will Allocated and Provisioned for it 

- Need only the amount of the Infrastructure resources to run my Container

- Pay only for What I used

- Easily Scale without fixed resources defined beforehand

- The Price base on how long the Container run and how much CPU and memory was requested for that container

----Fragate is Greate BUT----

- If I need more access to Underline Infrastructure running my Container then I have a flexibility and access with EC2 Instances

**AWS Ecosystem Available**

<img width="600" alt="Screenshot 2025-02-28 at 13 02 43" src="https://github.com/user-attachments/assets/755850a0-1838-45b1-a08a-881fc9842828" />

## Elastic Kubernetes Service (EKS)

- Managing Kubernetes Cluster on AWS infrastructure

- Alternative for ECS

**Advandtage of EKS**

<img width="700" alt="Screenshot 2025-02-28 at 13 19 33" src="https://github.com/user-attachments/assets/ebc443a6-87a3-4438-a3f2-f31e14ff698b" />

- If I am using K8s already and My Project want to deploys their Kubernetes Cluster on AWS Infrastructure and want to keep K8s tools and not use AWS-container orchestration

- If later I decide to migrate to other platform I can move that to other Infrastructure with K8s

  -- Migration will be difficult . Bcs If I am using EKS and use alot of other tools in AWS when I migrate to other platform those tools will not available

- Kubernetes much more popular than ECS . I also have access to all the Kubernetes tools and plugins and things are developing in the Kubernetes ecosystem

**How does EKS work?**

<img width="700" alt="Screenshot 2025-02-28 at 13 37 25" src="https://github.com/user-attachments/assets/31e95509-afe4-494a-9367-eb066b80c25e" />

----Control Plane Node----

- With EKS I create a Cluster with represent the Control Plane

- When I create an EKS services or EKS cluster , AWS will provision the backgroud Kubernetes Control Plane Node that already have all the Control Plane services installed on them

- Bcs EKS managed by AWS Control Plane will Replicated accross multiple Availability Zone

 -- If I am creating EKS in a region that has three AZs, then I will get automatically replication of my control Nodes on all of those AZs

- And I have the Storage which is etcd part of the control plane processes which store the whole Cluster Configuration . Basically the current State of the Cluster, that and storage need to be replicated as well . Bcs Shouldn't lose data

----Worker Node----

<img width="700" alt="Screenshot 2025-02-28 at 14 00 19" src="https://github.com/user-attachments/assets/467ede87-3b9c-4707-af37-fd4c2d0c8b8f" />

- I need Infrastructure that run actually runs my Pods and workdloads .

- Same way as ECS . I will create EC2 Instances called Compute-Fleet of multiple virtual Server and then connect them EKS

- In ECS I have EC2 isntances each server has ECS agent installed . This way Control Plane could communicate with individual Nodes

- In EKS each EC2 instances will have Kubernetes Processes so this ways Control Plane can communicate with Worker Node

- I have to manage the EC2 instances myself

- In EKS cluster however a possiblely to choose a semi managed EC2 for my Cluster . I can group my Worker Node into Node Group and Node group will handle heavy lifting for me . Make it easier for me to configure new Worker Node in my Cluser

  -- For example : Worker Nodes in EC2 instances manage by NodeGroup will get all the processes nessesary installed on them . So I don't have to worry about installing container runtime, Kubernetes Worker Processes etc ... for them to make it into Worker Node 






















































