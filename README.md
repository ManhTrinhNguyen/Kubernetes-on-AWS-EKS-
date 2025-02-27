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

 - When I created ECS cluster which is Control Plane and my Container are running on Virtual Machine . That Virtual Machine are EC2 instances . Instead of me manage and create EC2 but EC2 instances will be connected to this ECS cluster that I created . EC2 will be managed by Control Plane process on ECS Cluster

----Which Services are running on my EC2 instances?----

  - Container runtime : Docker ...

  - ECS Agent installed : This way ECS process Control Plane can communicate with each individual EC2 instances and manage them
































