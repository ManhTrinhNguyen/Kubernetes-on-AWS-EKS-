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

<img width="500" alt="Screenshot 2025-02-28 at 14 00 19" src="https://github.com/user-attachments/assets/467ede87-3b9c-4707-af37-fd4c2d0c8b8f" />

<img width="500" alt="Screenshot 2025-02-28 at 14 17 39" src="https://github.com/user-attachments/assets/b20c062d-4d5d-466a-93a2-cebbb0cafab5" />

- I need Infrastructure that run actually runs my Pods and workdloads .

- Same way as ECS . I will create EC2 Instances called Compute-Fleet of multiple virtual Server and then connect them EKS

- In ECS I have EC2 isntances each server has ECS agent installed . This way Control Plane could communicate with individual Nodes

- In EKS each EC2 instances will have Kubernetes Processes so this ways Control Plane can communicate with Worker Node

- I have to manage the EC2 instances myself

- In EKS cluster however a possiblely to choose a semi managed EC2 for my Cluster . I can group my Worker Node into Node Group and Node group will handle heavy lifting for me . Make it easier for me to configure new Worker Node in my Cluser

  -- For example : Worker Nodes in EC2 instances manage by NodeGroup will get all the processes nessesary installed on them . So I don't have to worry about installing container runtime, Kubernetes Worker Processes etc ... for them to make it into Worker Node 

- When working with EKS Cluster it is a good practice to use NodeGroup bcs I can create multiple NodeGroup to group my Worker Nodes in different logical groups.

- Still need to manage other stuff like : Auto Scaling, Creating new EC2 instances ...

 -- However if that also something I want to delegate to AWS I can use Fargate instead . So I have fully managed Control Plane and fully Manage Worker Node

----Or I can use Both EC2 and Fargate and the same time for the same EKS Cluster----

<img width="600" alt="Screenshot 2025-02-28 at 14 20 50" src="https://github.com/user-attachments/assets/c7cf1e43-73b8-4954-bcc6-4778fbaa34c4" />

- From the hosting perspective of where my containers are running having EC2 and Fargate as alternative this part is pretty much the same whether I use EKS or ECS

- And this give me an option of combination of the services if I want to run the containerized Application on AWS using Container Orchestration tool to manage them

**Step to create EKS Cluster**

 <img width="600" alt="Screenshot 2025-02-28 at 14 31 53" src="https://github.com/user-attachments/assets/6007ab37-61c9-4baa-8f58-6924cb98ae90" />

Step 1 : Provision an EKS cluster with Control Plane Nodes .

Step 2 : Create a group of EC2 instances by using NodeGroup . So I don't have to Create EC2 instance individually Maybe I need 5000 virtual machine 

Step 3 : Connect that NodeGroup or multiple NodeGroup to EKS Cluster 

Step 4 : Once it done I can simply connect to the Cluster using Kubectl command and start deploying containers inside the Cluster 

- Now I have NodeGroup and NodeGroup connected and that give me EKS Cluster


## ECR (Elastic Container Registry)

- This is a Repository for Container Images

- Alternative to Dockerhub or Nexus

**Advantage**

- Part of the whole AWS system it intergate well with other services

- Automatically get notified when a new version of the Image get pushed to a Repo and automatically dowload to my EKS or ECS

----How to use----

- Create Private Repo for different Application Images and I can start pushing different version or different tag of the Images 


## Create EKS Cluster (UI)

**Steps to create EKS Cluster**

Step 1 : Create EKS IAM role : Give EKS service certain Permission to create thing in AWS account 

Step 2 : Create VPC for Worker Nodes

Step 3 : Create EKS Cluster (Control Plane )

Step 4 : Connect kubectl with EKS Cluster

Step 5 : Create EC2 IAM Role for Node Group : Give certion Permission to call AWS API and configure and create stuff in AWS

Step 6 : Create NodeGroup and attach to EKS cluster : NodeGroup which is a Group of EC2 instances that are gonna run as Worker Node in Cluster

--- Now I have Worker Node connected to Control Plane Node . 
--- Then I will have complete Kubernetes Cluster with Control Plan Nodes managed that running in an AWS managed account and Worker Node run on my Account

Step 7 : Configure Auto Scaling for K8 Cluster : Based on resources required by Cluster, how many Pod I want to run. How many Applications I want to deploy in the Cluster, the number of EC2 instances or worker Nodes will be intelligenly Scale up and Scale down to match Resources Requirment

Step 8 : Deploy my App on Cluster 

## Step 1 : Create EKS IAM role

```
  - I will create a Role in my AWS account that will have permissions attached to it and I will give that Role to AWS services outside my own Account, This will be services EKS that managed by AWS to Manage, Create components on my AWS account on my behalf

  - Concept in AWS : I can give Role from my Account to another Account . So that Service on another account can manage components and Services on my Account

  - In AWS Console go to IAM -> Choose Role -> Choose AWS service (Allow AWS services like EC2, Lamdba, or other perform action on AWS account) -> Choose EKS-Cluster (will automatically selected Policy for me) . This is done for Step 1 and 2

  - In Step 3 : Give it a name and there is 2 piceces Information
   -- First one : Which Service is allow to use That Role (Selected Trust entities) . In this case is EKS 
   -- Second one :  What EKS allow to do in that Role (Policy) .

  !!! Note : IAM service is not defined per Region but It is global for my Whole Account 
```

## Step 2 : Create VPC for Worker Nodes

```
  - In AWS I have default VPC that AWS created for me in every Region

  - And I also have subnet in VPC and Route Table

  ----Why do I need another VPC for EKS cluster when I already have existing VPC with all Components?----

  - EKS Cluster need specific networking Configuration for it to work without problems

  - EKS based on Kunbernetes . Kubernetes has it own networking rules and AWS on top of that has it owns Networking Rules and those 2 have to work together and for that it need to configure correctly

  - When I create VPC in order to create the EKS Cluster this VPC is a network that is going to run my Worker Nodes . So It is not a VPC that I am creating for EKS or the master Nodes, but rather for Worker Nodes

  - For example : Some specific Configuration that Kubernetes or EKS cluster needs are Firewall Rule . Each VPC has Subnets, and Subnets has their own Firewall rules Which are configured by Network ACLs (Access Control List) . Inside Subnet I have EC2 and each EC2 instances can have their own Firewall rules Which is defined by Security Group

  - Bascically, My Worker Nodes which are going to run in my VPC in different Subnets, need to have set of Firewall rules that is nessesary for Control Plane nodes to connect to my Worker Node and also to manage them .

  !!! !!! EKS running on different VPC which is managed by AWS outside of my AWS account . And Worker Node running on VPC that managed by my AWS account . So the communication need to work between Control Plane and Worker Node accross those different VPC

  - And for that communication all the nessesary Port need to configured correctly

  - Another Important Example Specific to EKS is best practice for Creating and Configuring Subnets in my Worker Nodes VPC . That is having Public Subnet and Private Subnet . That mean when I create a Service which is Loadbalancer Type Service, Service get created also automatically a cloud native LoadBalancer get Created . What happen is when I configure Subnet into Private and Public K8 will know to create that external Loadbalancer in Public Subnet bcs I want that Load balancer to be accessible externally . So I want to create it in Public Subnet that allow external connection

 - In Addition to that to giving this infomation to Kubernetes or Control Plane on EKS to my Worker Nodes VPC , I also give Kubernetes permission to change things in my VPC . It happen through a Role and that Permission to open PORT on my behalf in my VPC . For example If I create NodePort Service, that mean my Worker Node need to open a PORT so that directly accessible on that NODE PORT . So Control Plane will do it automatically in my behalf

```

**To Create VPC for EKS by using AWS template (CloudFormation)**

```
 - Create a whole stack, the VPC and all Components, Security Group, Internet Gateway, Subnet ... in CloudForamtion
 
 - In CloudFormation -> Choose Create Stack
 
 - Generally I have defined or pre-defined files with all the configuration nessessary that basically need to describe what needs to create with which Configuration .
 
 - Template also host in S3 buckets . Available on AWS S3 bucker URL
 
 - There is a page in AWS EKS Docs where I can see all these URL  (https://docs.aws.amazon.com/eks/latest/userguide/creating-a-vpc.html)
```

**2 choices to create VPC**

```
 1. Create VPC with all private Subnets

 2. Or Create VPC with all Public Subnets

 3. Or a mixture of those (Recommended by AWS)
```

**Create VPC with Private and Public Subnets**

```
 - URL of Public and Private Network : https://s3.us-west-2.amazonaws.com/amazon-eks/cloudformation/2020-10-29/amazon-eks-vpc-private-subnets.yaml

 - This is content of template using multiple components using one template file

 - In the template : I have VPC itself with Cidr block - Subnet with Cidr block (is IP address range) - IP address range will be use to assign the IP address to Components in Kubernetes Cluster

 - Put Template URL S3 URL

 - Next I have Specific Stack detail - Parameter : I have possibility to configure the IP address ranges before creating the stack

 - Then I will summary and create the Stack

 - After stack created I will go to Output . Now If I go to Output , I am actually going to see 3 Resources or Components that I gonna need as Infomation for EKS Cluster
```

**What is EKS Cluster need to know?**

```
 - EKS Cluster need to know which VPC of my AWS accounts I want to use to deploy Worker Nodes

 - We are also giving infomation about the Subnet ID in that VPC

 - Also the Security Group that I am using in that VPC . I can have Multiple Security Group so I need to tell EKS which one I use for Worker Node Group

 - I also need this Information when creating the EKS cluster (Control Plane)
```

**Wrap Up**

```
 - So now I have a Role that will let EKS do things in my behalf in my AWS account to bacsically manage my Worker Nodes and other Resources.

 - And I have VPC in that my Worker Node will run

 - With these 2 informations now I can Create EKS Cluster
```

## Step 3 : Create EKS Cluster (Control Plane)

- In the UI go to EKS

**Configure Cluster**

```
   - Name -> Role Selection

   - Secrect Encryption: Secrect base64 encoded not very secure . I will encrypt my secrect by using Secret Encryption provided by AWS

   - AWS has a service for Encryption . In the Manage K8 Services , I basically have an option bcs AWS services they are all Intergrated, tightly intergrated with each other
```

**Specify Networking**

<img width="600" alt="Screenshot 2025-03-03 at 14 17 15" src="https://github.com/user-attachments/assets/cca45451-448a-46bc-8c81-8f4a04e9270d" />

```
 - I need to Specify or give information to EKS about my Networking . Where are my Worker will run

 - I will select the VPC I created for Worker Node in step 2

 - Also I will select the Security Group the one in VPC for Worker Node

 ----Cluster Access Enpoint----

 - Configure access to the Kubernetes API server endpoint

 - API server is a entry point to my Cluster.

 - Also API server is one of the Control Plane Processes so It is going to deployed and running inside the EKS VPC . AWS managed VPC . 

 - Public endpoint : If I want to Connect or talk to K8 Cluster externally, Access API Server or the Cluster from Local Laptop by using CLI, Kubectl (Outside the both VPC) I need to enable Public Endpoint

 - Private enpoint : Enable Worker Nodes to communicate or connect to the Cluster Endpoint within my VPC network . So it enalbe the Control Plane and the Worker Node to talk to each other through my VPC . So the Network interface will get created in my VPC that would enable the traffic to go from my Worker Node through our VPC directly to the Control Plane Node

 - I can choose Public and Private mode to have Both
```

**Configure logging**

```
 - This is a Configuration Logging for my Control Plane

 - I able to decide if I want to see the Logs that come from the Master Processes (Logging on the Control Plane):

   -- All the Request API Server, Authentication Event, What the Controller Manager does, Sheduler does

   -- Managed by AWS
```

**AWS add-on**

```
  - Core-DNS : Kubernetes Process is going to add DNS services to my EKS Cluster . This way I will able to use DNS name inside the Cluster to communicate instead of using the IP address

  - Kube-Proxy : Manage Networking tasks like routing in my Cluster . Kube-proxy will run on all of My Nodes and forward all the traffics between Services and Pods using these Routing Rules and make sure all the part of the CLuster can Communicate

  - Amazon VPC CNI(Container Network Interface ) :

   -- Enable easier communication my My Cluster . It will create Network Layers within the Cluster, so that all the Pods running on all these different Nodes will communicate with each other as if they were running on the same Server (The same local network) .

   -- CNI Plugin will basically allocate the Pods's the IP addresses from the IP address Range allocated to the Cluster through the AWS VPC . So this way my Pods able to communicate to any Resources if needed

 - Amazon GuardDuty : Can detect any Security Threats and make them Available in another AWS service 
```

**Summary and Create EKS Cluster**

## Step 4 : Connect kubectl locally with EKS Cluster 

- Eventhough I don't have Worker Nodes yet . I can still can talk to the API Server bcs Control Plane is running

- They way to connect is create kubeconfig file for newly created EKS Cluster

**Configure Kubectl to connect to EKS Cluster**

```
 Step 1: To see AWS configure detail : `aws configure list`

 Step 2: To create kubeconfig file locally : `aws eks update-kubeconfig --name <cluster-name>`

   -- --name : Connection info for Cluster . Which is the Cluster Name
```

 - After Created kubeconfig file my Local machine already connected to AWS K8 Cluster. The file will store in .kube/config

 - **To read this kubeconfig file** : `cat .kube/config`
   
<img width="500" alt="Screenshot 2025-03-04 at 13 29 40" src="https://github.com/user-attachments/assets/7f6d724a-10c3-4743-a432-0224ce367664" />

 - In this Kubeconfig file I can have multiple Cluster Credentials for multiple different K8's Cluster .
 
   - Context : For each of those Cluster, It will actually create a the context entry in this file and to know which CLuster I need to connect to anytime I use kubectl command .
  
   - Current Context : Since Kubectl command by default will use this kubeconfig file, in this default location . The current context is a set in this file that kubectl will use as a current Cluster context to connect to that Cluster
  

 - **To get Worker Node** : `kubectl get nodes`

 - **To get Namespaces** : `kubectl get ns`

 - **To see Info of K8 Cluster**: `kubectl cluster-info` -> I Will have the Cluster endpoint . Which is the same endpoint in Detail (UI)

## Cluster Overview - Cluster Node 












