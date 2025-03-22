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

### Step 1 : Create EKS IAM role

```
  - I will create a Role in my AWS account that will have permissions attached to it and I will give that Role to AWS services outside my own Account, This will be services EKS that managed by AWS to Manage, Create components on my AWS account on my behalf

  - Concept in AWS : I can give Role from my Account to another Account . So that Service on another account can manage components and Services on my Account

  - In AWS Console go to IAM -> Choose Role -> Choose AWS service (Allow AWS services like EC2, Lamdba, or other perform action on AWS account) -> Choose EKS-Cluster (will automatically selected Policy for me) . This is done for Step 1 and 2

  - In Step 3 : Give it a name and there is 2 piceces Information
   -- First one : Which Service is allow to use That Role (Selected Trust entities) . In this case is EKS 
   -- Second one :  What EKS allow to do in that Role (Policy) .

  !!! Note : IAM service is not defined per Region but It is global for my Whole Account 
```

### Step 2 : Create VPC for Worker Nodes

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

**Why do I need another VPC for EKS cluster when I already have existing VPC with all Components? Part 2**

```
 - Using a separate VPC for your Amazon EKS (Elastic Kubernetes Service) cluster instead of the default VPC is generally recommended for several reasons:

  1. Security & Isolation

   - A dedicated VPC ensures that your Kubernetes workloads are isolated from other AWS resources that may exist in the default VPC.
   - You can apply fine-grained network security policies (e.g., private subnets, security groups, and network ACLs) to control access.

  2. Custom Networking Requirements

   - EKS requires specific networking configurations, such as multiple subnets across Availability Zones (AZs) for high availability.
   - You might want to configure private subnets for worker nodes to restrict public internet exposure.

  3. Avoiding Conflicts

   - The default VPC might have overlapping CIDR ranges, which could cause IP address conflicts if you have multiple services running.
   - Creating a separate VPC allows for better IP address planning, avoiding subnet exhaustion issues.

  4. Better Control over VPC Components

   - With a custom VPC, you can configure custom NAT gateways, route tables, security groups, and other networking components tailored for Kubernetes.
   - The default VPC might not have the necessary private/public subnet separation or the correct settings for Kubernetes networking.

  5. Compliance & Governance

   - Some organizations have strict compliance policies requiring Kubernetes clusters to be deployed in dedicated, isolated environments.
   - A separate VPC makes it easier to enforce security and compliance policies.

  6. Enhanced Scalability

   - If your EKS cluster scales significantly, it’s better to have a VPC designed to handle increased network traffic and resources.
   - The default VPC may have limited resources that might not be sufficient for large-scale Kubernetes workloads.

  7. Multi-Environment Deployments

  - If you deploy multiple environments (e.g., Dev, Staging, and Production), using separate VPCs for each cluster helps maintain isolation and prevent interference.
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

### Step 3 : Create EKS Cluster (Control Plane)

- In the UI go to EKS

**Configure Cluster**

```
   - Name -> Role Selection

   - Secrect Encryption: Secrect base64 encoded not very secure . I need to encrypt Secret . To encrypt Secret I need to install addition tools to do that . Also I can encrypt my secrect by using Secret Encryption provided by AWS

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

### Step 4 : Connect kubectl locally with EKS Cluster 

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

### Cluster Overview - Cluster Node 

**Detail Overview**

```
 - API Server endpoint : This is where K8 API Server accessible at

 - Bcs I selected Public I can access this Enpoint from External Client

 ----Cluster IAM Role ARN----

 - Role that I assigned in the Cluster

 - Cluster ARN : EKS Cluster unique ID 
```

**Compute**

```
 - 3 Sections : Nodes, Nodes Group, Fargate profile

 - I can run the Pod or the workload on my Worker Nodes in Kubernetes either using EC2 instances or Fargate

 - Fargate will provison virtual machine in the background that will run the Pod

 - Summary : Nodes group is self-managed . Fargate is managed on top of Managed Control Plane 
```

**Networking**

```
  - I have created and pass it on to EKS Cluster

  - That mean that Control Plane is running it's managed by AWS . and I have to created Worker Node and connects those Worker Node to the Control Plane 
```

### Step 5:  Create EC2 IAM Role for my Node Group 

```
 - I want to create NodeGroup instead of individual Instance Worker Node bcs if I want to create 100 Instances at 1 time it will very cost time .

 - Instead I want to group them , or maybe I want multiple Group are all then connected to Control Plane
```

**Create EC2 Role for Node Group**

```
 - Worker Nodes run Workers processes . Kublet is one of the Processes

 - Kubelet is the main Processes that responsible for scheduling Pods , getting all Resources from Servers like RAM, CPU ... assigning to the Pod . But also communicate with other Services in AWS and K8's Control Plane .

 - Basically Kubelet make various API calls, including to AWS services on my behalf .

 - So I need to give Kubelet permissions to execute all of those Tasks . (Give Kubelet a IAM Roles)

  - Step 1 : I need to create a Role for NodeGroup that allow Worker Processes to perform certain Action 

   -- Go to IAM Roles -> Create Role for EC2 -> so Processes on EC2 will then able to do stuff or interact with other AWS Services -> Then Choose AmazonEKSWorkerNodePolicy, ContainerRegistryPolicyReadOnly, Amazon EKS_CNI Policy

   -- AmazonEKSWorkerNodePolicy : Include access to EC2 and EKS

   -- ContainerRegistryPolicyReadOnly : Include Permission for ECR (Elastic Container Registry) . This will be important relevant when we connect our Worker Nodes with the Container registry , so they can pull the Container Registry so they can pull Images from there with new Versions .

    !!! Note : AWS services intergrate well together . As long as we give them permission to interact with each other we can Configure communication between them .

   -- Amazon_eks_CNI policy : This has access to EC2 services . All Actions can perform on that Services . Baciscally CNI is a Container Network Interface. CNI is internal network in Kubernetes . So that Pod in different Servers can communicate with each other . No matter they are on Worker Nodes or Control Plane Nodes or which of those Node they are located . I am giving that role access to the CNI policy so that it can interact with other Worker Nodes 

```

### Step 6 : Add NodeGroup to EKS Cluster 

```
 - Step 1: Goto EKS Compute -> Add Node Group -> Select the Role that will give Node Group and EC2 Instaces in that Node Group Permission for interacting with other AWS Services

 - Step 2 : Set compute and scaling configuration

   -- AMI Type

   -- Capacity Type : On demand (You pay for what you use) .

   -- Instance Type : t3.small . The Type have CPU, Memory ...

   -- Disk size : The Storage

   -- NodeGroup Scaling Infomation configuration : Desired size , Minimum Size, Maximum Size .
     -- For example : If my Desired size : 3, Minium Size : 2 , Desire Size : 4 . So It will launch with 3 Intances and it will scale up or down base on the Work Load it will run on the Cluster

   -- NodeGroup Update configuration : NodeGroup will update by AWS  to a latest Version when they are released . I can define how many Node it should taken offline during this update process

 - Step 3 : Specify Network Configuration

  -- Our Worker Node need to run in VPC that I have created with these specific configuration . So that Control Plan which is running on another VPC on another AWS Account can connect to that VPC and can Managed the Worker Node  running in the different Subnet within that VPC.

  -- So I am configuring which VPC and Subnet our NodeGroups is going to launch

  -- I have specified 2 EC2 instances for this NodeGroup right now . And I have 4 Subner . Which mean NodeGroup can launch EC2 instances in different Subnet . Eventhough right now I have just 2 if I add additional EC2 instances later If I scale it up to 10 EC2 Instances then it will be distributed among different Subnet 

  -- Enable SSH that allowing access to Worker Node

```

**Wrap up**

```
 - A part of initialization of these EC2 instances is installing all the nessesary processes on these Servers . Bcs In order to run Pods I need Container runtime, Kubelet, KubeProxy . All of those things get installed by create Node Group
```

**Cost Effiency**

```
 - Set Desire Node, Minimum Node, Maximum Node as a Minimum and then Configure Cluster auto Scaler to manage scale up and down of EC2 Instances for me based on how much Resources my pod requires . Big Advandtage is Saving cost of Infrastructure 
```

# Virtual Private Cloud (VPC)

**What is VPC**

```
 - Virtual Private CLoud is my own Isolated Network In the Cloud (Isoliated my Network from others. Bcs others might use the name Network from that Data center)

 - Whenever I create account I will have Default VPC in each Region

 - VPC span all Availability Zone (Subnet) in that Regions

 !!! NOTE : My Resources have to be alway running in the VPC

 - In the bigger Company they could use multiple VPC in multiple different Region

 - VPC is like a Virtual Representation of Physical Network Infrastructure: Server set up , Network configuration moved into the Cloud
```

**Subnet 1**

```
 ----Subnet----

 - Subnet is a Range IP Address in my VPC

 - It is a Private Network inside Network

 - VPC span the whole Region . Subnet span in each Availability Zone so I have Subnet for each Availability Zone .

 ----Private and Public Subnet----

 - Base on my Configuration I can have Private and Public subnet 
```

**Subnet part 2**

```
 - In VPC I have a Range of Private IP Address (Internal IP Address) . That Range defined by default and I can change its Range

 - When I create a new resources such as an EC2 instances, an IP Address is assigned within this subnet's IP range

 - This a allow communication inside the VPC

 - This Private IP Address only for communication inside the VPC . If I want accessible from the outside I need to assign the Public IP Address

 - Public IP Address Configure in VPC Service . For allowing Internet Connectivity with my VPC I use the Internet Gateway

```

**Internet Gateway (Router)**

```
 - Allow me to connect the VPC or its Subnet to the Internet(Outside Network)
```

**Controll Access - Security - Firewall Rule**

```
 - I need to control what traffic Enter my VPC

 - I need to control what traffic access to my Individual Instances

 - And I need to control what traffic goes out

 ----Network ACL (Access Control Lists)----

 - Control access on Subnet Level

 ----Security Group----

 - Control Traffic on Instances Level
```

## Configure Auto Scaling in EKS Cluster  

- Docs (https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/aws/README.md)

**Create OIDC Provider**

 - OIDC authentication allow Services to assume an IAM role and interact with AWS services without having to store credentials as environment variables .

 - To Create OIDC Provider Docs and use for AutoCluser : (https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/aws/CA_with_AWS_IAM_OIDC.md)

 - In the Docs in Section B I create IAM Policy for Autoscaling is :

 ```
     {
      "Version": "2012-10-17",
      "Statement": [
          {
              "Effect": "Allow",
              "Action": [
                  "autoscaling:DescribeAutoScalingGroups",
                  "autoscaling:DescribeAutoScalingInstances",
                  "autoscaling:DescribeLaunchConfigurations",
                  "autoscaling:DescribeScalingActivities",
                  "ec2:DescribeImages",
                  "ec2:DescribeInstanceTypes",
                  "ec2:DescribeLaunchTemplateVersions",
                  "ec2:GetInstanceTypesFromInstanceRequirements",
                  "eks:DescribeNodegroup"
              ],
              "Resource": [
                  "*"
              ]
          },
          {
              "Effect": "Allow",
              "Action": [
                  "autoscaling:SetDesiredCapacity",
                  "autoscaling:TerminateInstanceInAutoScalingGroup"
              ],
              "Resource": [
                  "*"
              ]
          }
      ]
  }
  ```

**How Auto Scaling work in EKS Cluster**

```
 - In the detail section of NodeGroup Configuration -> Check Autoscaling Group name

 - Autoscaling Group was created in AWS .

 - I can see Autoscaling Group Component in NodeGroup's Detail they have a Autoscaling Group URL
```

**What Autoscaling Group Component Does ?**

``` 
 - Autoscaling Group Component logically group a number of EC2 instances together and has a Configuration of Minimum number of EC2 intances to scale down to and Maximum to scale up to. However the Autoscaling group there just to group these Instances It doesn't actually automatically scale our Resources

 - AWS don't do it for me . I need to configure Autoscaler EKS component that is running inside the Kubernetes Cluster that will use this AWS Autoscaling Group will able to scale up or scale down the EC2 instances for me .

 ----Example When it should happen ? How does it work ?----

 - If I have 10 EC2 Instances in my Cluster and the Cluster Auto Scaler Componet inside the Cluster see that most of these EC2 Instances underutilized it will take a Pods from 2 EC2 Instances and distribute them on the rest of them and stop those 2 EC2 Instances and this will save Infrastructure cost in AWS bcs now I have 2 Instances less running bcs I don't need that much Resources

 - OR the opposite When I want to schedule new pod but EC2 Intances has no left Resources so It will take the Maximum size from Autoscaling Group and automatically create new EC2 Instances to schedule the new Pod . Maximum size is a number of instances the group allowed to scale out to 
```

**The reason Why defining Max and Min number**

```
 ----Max----

 - To Save Cost :

  -- Maybe I don't want to pay more certain amount of Servers .

  -- Avoid unexpected and un sustainable costs, especially during sudden traffic spikes or unanticipated high demand

 - Security Reason :

  -- Indication of security or performance issues

  -- Maybe there is a leak in the application that is suddenly using up so much resources, so instead of spinning up more Instances I kept it at certion amount of Instances

  -- I may want to check before allowing more resources

  ----Min----

 - Min number of Instancews should maintain to keep running at all time

 - Ensure Application maintain baseline of Availability, Performance and Reliability

 - Fault Tolerance : Reduce risk of downtime caused by unpredictabke failure, such as hardware

 - Avoid Scaling delay : Scaling out take time, especially if instances need to be initialized and configured 
```

**Things need to do in order to configure Autoscaler**

```
 - First: Autoscaling Group (Can change Min and Max anytime)

 - Second : Create a Role for my NodeGroup . For the autoscaling to work, I need to give the EC2 Instances inside the Worker Node certain permission to make AWS API calls

 - Third : Deploy Cluster AutoScaler 
```

**Create custom Policy for my NodeGroup**

```
 - IAM -> go to Policy -> go to Create Policy -> Choose JSON then paste the custom list of Policy in there -> Then give it a name and create it

 - Custome list is the one that has a list of all the Permissions that we need to give NodeGroup IAM role for the autoscaling to work

 - After Policy created . Attach it to a WorkerNode IAM Role 
```

**Configure Tags on Autoscaling Group . Automate created by AWS**

```
 - Tags are also use in order for different Services or Component to Read and Detect some Information from each other . This is one of the case where we have Tags that auto scaler that we will deploy inside Kubernetes will require to auto Discover Autoscaling group in the AWS account . So the Cluster Autoscaler Component, which I gonna deploy inside Kubernetes Cluster, needs to communicate with auto scaling group . For this communication to happen the Cluster auto Scaler first needs to detect the auto scaling group from AWS . And it happen by using these 2 tags : k8s.io/cluster-autoscaler/eks-cluster-test, k8s.io/cluster-autoscaler/enable
```

**Deploy Cluster Autoscaler**

```
 - Using : kubectl apply -f <Autoscaler-Deployment-file> : ( https://raw.githubusercontent.com/kubernetes/autoscaler/master/cluster-autoscaler/cloudprovider/aws/examples/cluster-autoscaler-autodiscover.yaml )

 - In that Yamlfile :

  -- In the command part I have the Configuration where NodeGroup auto discover is configured using these tags : --node-group-auto-discovery=asg:tag=k8s.io/cluster-autoscaler/enabled,k8s.io/cluster-autoscaler/<YOUR CLUSTER NAME>

 - To check my Deployment : kubectl get deployment -n kube-system cluster-autoscaler

 - I want to change something in the Deployment file I can use :  kubectl edit deployment -n kube-system cluster-autoscaler

   - Edit annotation : cluster-autoscaler.kubernetes.io/safe-to-evict: "false"

   - In the command part change <Your Cluster Name> to actual cluster name

   - Change Image version : Image version have to match with EKS Cluster Version

   - In the Command level add : --balance-similar-node-groups and --skip-nodes-with-system-pods=false

 - To check pods in kube-system namespace : kubectl get pod -n kube-system

  - In Kube-system For each Node I have these processes : KubeProxy (One of Worker Proccess) and awsNodes proccesses run on each Nodes, , DNScore no need to sit on every Nodes . Also I can see autoScaler pod run in one of Intances as well

  - If I add another Node these awsNoes and KubeProxy Pods will get scheduled on the new Node as well

 - To see Which Instances run the Autoscaler : `kubectl get pods -n kube-system <autoscaler-pod-name> -o wide`, then I get the Node name so I can see its logs : `kubectl logs <Node-name>` .
 - To make a logs easier to read : `kubectl logs <Node-name> > as-logs.txt`

  - In the Logs I can see couple of time of Caculating no need Node, Scale down started , No candidate for Scale Down

  - To test it I change the Autoscaling Group min to 1 . Once the NodeGroup Configuration get Updated and the Autoscaler run again to reevalute the status . I can see that NodeGroup will scale to down to just 1 Instances . Now that Autoscaler has run again with a new configuration .

   - After the Configuration changed to Autoscaler started scale down processes . Here I see autoscaler will check out the Nodes in the AutoScaling Group and as part of the Scale down processes, it will actually analyze when the Instaces were last used . After this evaluation, the Preparing for removing this Nodes will actually get started . And I can see a Auto Scaling Pod from my Cluster is being moved to another Node. So Autoscaler decided lets move the Node right here, which happen to be the First one of the two and the AutoScaler itself was scheudeded in that Node so it need to move that Pod to another Node

   - Also at the begining each EC2 Instances or Worker Node has its own default process running there . These processes are there when EC2 initilized and these are Kubernetess Processes

   - Toward down the part of Logs . The pod are actually get deleted as part of the process so I can see this Clean up message

 ----Advantage----

 - Save cost when I am not using using the capacity of EC2 Insanctes it will get removed

 - However the trade-of of that whenever I need a new EC2 instances It will take time to shedule new Pod
```

**Deploy Nginx Application**

 - Now I have my EKS Cluster Up and Running .

 - Step 1 : Create Deployment and Service for Nginx

   - Image name: nginx
     
   - containerPort : 80
     
   - Service Type: LoadBalancer . I am creating external Service in Kubernetes Cluster that will have AWS LoadBalancer attached to it bcs everytime I create a service type of Loadbalancer, Kubernetes spin up a native Load balancer from the Cloud Platform depend on which Platform I use . So that LoadBalancer will become a EntryPoint

 - Step 2 : `kubectl apply -f <Configuration-yaml file>`

 - Step 3 : Check that Pod is running fine or not : `kubectl get pods`

   - If there is an Error when starting up the Pods I can use : `kubectl logs <pods-name>`
 
   - To see processes of the Pods I can use : `kubectl describe pods <Pods-name>`

 - Step 4 : Check Service : `kubectl get svc`

   - Bcs of the Service type is LoadBalancer so K8 will automatic create external IP (external access URL) and internal IP also a NodePort (Range : 30000 - 32722)
  
     <img width="500" alt="Screenshot 2025-03-13 at 15 19 26" src="https://github.com/user-attachments/assets/63e2d09d-a2b5-46f9-8731-01dd572e117b" />

   - In the Configuration Port in AWS UI . Port 80 forwarding to Port 32088
    
     - Port 80 : is a Default http Port . This is a Port where LoadBalancer accessible at and then it does a internal forwarding to a NodePort 
    
     - Port 32088 : is a NodePort sit on every Worker Node . And then from here it will go to Service Port (which is port 80 again I configured) . and then it will go to container Port (Which is port 80 again) 
    

   - Nodeport will sit on top of every Worker Node . However bcs I am not using the external IP of the Worker Nodes . I can directly call it use DNS name and it will then in the background forwarding between those Ports 
 
   - In the EC2 isntances in the UI . I go to loadbalancer section . A Loadbalancer get automatically created when a Service with loadbalancer type got Created . 

   - Another Point : At the beginning when I define VPC for my Worker Node I have defined 2 Public and 2 Private Subnet and a Loadbalancer is mean to be accessible from external Request so It will be schedule in Public Subnet
  
   - !!! NOTE : External Component like LoadBalancer they will got Public and Private IP address . Private for internal communication and Public for external communication
  
   - Another point : In Load Balancing Configuration I can see EC2 Instances that this Load Balancer load balances traffic to . In Instances ID I can see all the EC2 
Instances that is Worker Nodes behinds this load balancer

   - And in AWS there is a requirement for loadbalancer to have at least one EC2 in at least 2 different AZ 

## Create EKS Cluster with Fargate Profile

**Introduce to Fargate**

 - I have run my Application on EC2 instances manage by NodeGroup that is connected to a Cluster. As a alternative I can also work with Fargate

 - Fargate is a Serverless way of deploying containers and creating Pods in the Cluster . Fargate AWS will provision and run, manage the Application for me in AWS managed account

**Improtant note different between Fargate and EC2**

 - When we create Pods using Fargate . Fargate will provision its own virtual machine for each Pod, So we will have as many Virtual Machine as we have a Pod . On EC2 Instances however I can have multiple Pod running. And this different or this characteristic of Fargate actually have some limitation for what we can deploy through Fargate . For example deploying Statefull Application or Daemon Set that are basically applications that will run on every Node are not gonna be possible through Fargate .

### Create Fargate

 - I can have both Worker Node and Fargate attach to one Cluster

**Create IAM Role for Fargate**

 - When my Cluster create Pods on AWS Fargate, the Kubelet on the Virtual Machine will provision for us, will need to make call to a AWS APIs on my behalf. It will need to connect to my VPC, provision some Resources, Virtual Machine, pull the container images from ECR . So I will create a role that will permit Virtual Machine provision by Fargate to do all this stuff .

 - Go to IAM -> Create Role -> Choose EKS -> Choose EKS - Fargate Pod -> Give it a name

**Create Fargate Profile**

 - The way it work is Fargate profile create a rule. Whenever I tell Fargate here is the Pods configuration, Fargate base on the profile rules and what I defined wheather that Pod should be sheduler through Fargate .

 - So Fargate Profile lets us define the criteria to decide, how the new Pod should be schedule

 - Go to Fargate Profile in EKS cluster .

  - **Profile Configuration** : Create name, Attach role for Fargate, And also Subnets of VPC I defined for Worker Node

    - !!! Note : If Fargate is going to provision virtual machine on AWS managed Account and I am not gonna see any EC2 or Virtual Machine on our Account . Why do I need to provide my own VPC ? The reason is the Pod that will get schedule using Fargate will get an IP Address and that IP Address will be from the range that our Subnet have . So our VPC and Subnet networking Infomation will actually be used to schedule the Pod.
  
    - !!! Note : Also AWS removed the Public Subnet bcs Fargate will need to create a Pod only in a Private Subnet . I have the Private Subnet and Public Subnet in 2 different AZ so external Components like external Load Balancer belonging to AWS can be create in Public Subnet and internal Component like Pod can be create in Private Subnet.
 
  - **Configure Pod Selection** : We need sometype of Selectors that will tell Fargate this Pod is meant to be schedule by Fargate .

    - Using Namespace infomation in the Configuration Yaml I can create Fargate rule or Fargate profile that defines a Namespace selectors for a Pods . If I type the name in Namespace Fargate will read the configuration when I try to deploy it using Kubectl apply and it will check wheather the Pods configuration has a namespace defined . If it does then Fargate will decide this is a Pod I need to schedule through Fargate . If not it will handover to be schedule on one of the EC2 instances in my NodeGroup
 
    - Another thing I need to configure is Label Selectors .
 
     - In the Labels Section I can add multiple key:value pair lables 
 
    ----Examole Use Case - having both NodeGroup and Fargate----
 
    - I could decide to split a DEV and TEST environment inside the same EKS Cluster . So I can create Test environment in Node Group and Dev Environment in Fargate . In this case I can tag Pod with the respective Selector or Lables or Namespace so that Pod has Namespace Dev will schedule on Fargate, and one not have Dev Namespace will be schedule in Node Group.
 
    - Another use case is Fargate has Limitation in term of what Kubernetes component can deploy (Can not deploy Deamon Set for Logging, or Statefull App). In this case it might make sense to have NodeGroup or EC2 instances for Stateful Apps and use Fargate profile for Stateless Apps


### Deploy Pod through Fargate 

 - Create namespace for Fargate : `kubectl create ns <namespace-name>`

 - Then apply pod : `kubectl apply -f <yaml-pod-config>`

 - Wait until pod ready : `kubectl get pods -n <namspace-name> -w`

   - -w : Wait until pod ready
  
 - To check Nodes of Fargate : `kubectl get nodes -n dev`

   - The IP address is the part of the VPC IP Address range that I created
  
   - So Fargate is a Virtual Machine provisioned outside our account, we don't see that and don't even have access to it bcs it completely manage by AWS   


## EKS Cluster with eksctl CLI 

**eksctl**

 - CLI tools for working with EKS Cluster that automates many individual Tasks

**Install eksctl and connect to AWS account**

 - Install eksctl

 ```
  brew tap weaveworks/tap
  brew install weaveworks/tap/eksctl
 ```

 - When eksctl installed I have to configure AWS Credentials : `aws configure list`


**Create EKS Cluster**

 - Create EKS by using CLI

 ```
 eksctl create cluster \
 --name eks-cli
 --version 1.32 \
 --region us-west-1 \
 --nodegroup-name demo-nodes \
 --node-type t2.micro \
 --nodes 2 \
 --nodes-min 1 \
 --nodes-max 3
 ```

   - This command will take sometime 

 - Alternative way Create yaml config file . The advantage of using Yaml file is I will have history of what I created I can just repeat it instead of just saving the command somewhere . If the Configuration complex and big it is way easier to configure in Yaml file

 - Eksctl is not just for create a Cluster . It is also for Managing and Configuring after Cluster created 

 ```
 apiVersion: eksctl.io/v1alpha5
 kind: ClusterConfig
 
 metadata:
   name: basic-cluster
   region: eu-north-1
 
 nodeGroups:
   - name: ng-1
     instanceType: m5.large
     desiredCapacity: 10
   - name: ng-2
     instanceType: m5.xlarge
     desiredCapacity: 2
 ```

**Review created EKS Cluster**

 - Cluster was created

 - Deploying Demo Cluster

 - Deploying NodeGroup Stack

 - Kubeconfig automatically configure and saved in `kube/config`

 - kubectl installed

**What get created in AWS Account**

 - IAM Roles :

   - EKS-Cluster-NodeGroup role : AmazonEC2ContainerRegistryReadOnly, AmazonEKS_CNI_Policy, AmazonEKSWorkerNodePolicy, AmazonSSMManagedInstanceCore
  
   - EKS-Cluster-Service role : This is a Policy give AWS Managed Account permission to do stuff in my AWS Account . AmazonEKSClusterPolicy, AmazonEKSVPCResourceController,
  
   - I have AWS managed policy which let AWS manage some of EKS network resources on my behalf
  
   - I have AWSServiceRoleforAutoScaling and AWSServiceRoleForAmazonEKS
  
   - NOTE : eksctl does a lot of stuff in the background . Configuring some of additional stuff
  
 - VPC :
 
   - Public and Private Subner get created
  
 - EKS :

   - 2 Node got listed
  
   - We don't see the Control Plane Nodes AWS Managed
  
   - Processes are running on those Worker Nodes is : coreDNS, Kubeproxy , awsNode processes, kubelet, AWS VPC CNI
  
     - aws-nodes process: Let the Worker Nodes communicate with the Control Plane Nodes on AWS account
    
     - kubelet (in the OS level of every Nodes): Communicate with API-server, Start the shedule Pod, continusly monitor Nodes'heath , manage lifecycle Pod
    
     - AWS VPC CNI : Manage Networking . Assign Elastic Nerwork Interface and IP Address to Pods . Ensure network communication betweens pods and external Service
    
     - kubeproxy and core DNS are nessesary to run schedule Pod to configure Services etc ...

   - I also have Public endpoint of EKS Cluster 

## Deploy to EKS Cluster from Jenkins Pipeline 

**Steps I need to Configure**

 - Step 1 : Install kubectl inside Jenkins Container

 - Step 2 : Install aws-iam-authenticator tool inside Jenkins Container

   - When I try to connect to the Kubernetes Cluster on AWS I need to authenticate not only with Kubernetes cluster, but also with AWS bcs the Cluster is running on AWS account
  
 - Step 3 : Create Kubeconfig file to connect to EKS Cluster

   - This is a alternative to creating credentials inside Jenkins, now I go inside Jenkins Container directly and we gonna create a Config file that will contain all the infomation it needs in order to authenticate with AWS account using aws-iam-authenticator also EKS cluster I have created
  
   - This kubeconfig file is the same file that we are using locally to execute kubectl commands. On local machine when I created EKS Cluster this `kube/config.json` file automatically got generated this config file contain all the infomation that needs to authenticate to EKS
  
   - When I install EKS by using eksctl tools it automatically installed aws-iam-authenticator
  
 - Step 4 : Add AWS credentials on Jenkins for AWS account authentication

   - I have AWS User that I use to create a Cluster, and each User on AWS has an access key ID and Secret access key, I need these credentials in order to connect to our Cluster .
  
 - Step 5 : Adjust Jenkinsfile to configure EKS cluster deployment


### Step 1 : Install kubectl inside Jenkins Container

 - Connect to a Server : `ssh root@...`

 - Check running Container : `docker ps`

 - Go inside Jenkins container as a Root User bcs Jenkins doesn't admin permission : `docker exec -it -u 0 <container-id> bash`

 - Inside Jenkins container install kubectl, make it executable and move it to /usr/local/bin/kubectl : `curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl; chmod +x ./kubectl; mv ./kubectl /usr/local/bin/kubectl`

   - As I learned in Jenkins Moudule . Whenever I need some CLI tools execute commands with inside our Pipelines I can install them directly on a Jenkins Server or inside a Jenkins container and they will be available as Linux Command inside the Pipeline I can execute them pretty simply .
  

### Step 2 : Install AWS IAM Authenticator 

 - This is Specific to AWS . When I created EKS Cluster I got a Kubeconfig which contain for the Secret and all the certificate for authenticating and connecting, it also container the infomation to a Cluster on AWS specificly to EKS Cluster . So I will provide all the credentials, however I need kubectl and aws-iam-authenticator both to acctually connect to EKS cluster and authenticate to it from Jenkins

 - Install aws-iam-authentocator, make it executable and move it to /usr/local/bin :

 ```
 curl -Lo aws-iam-authenticator https://github.com/kubernetes-sigs/aws-iam-authenticator/releases/download/v0.6.11/aws-iam-authenticator_0.6.11_linux_amd64

 chmod +x ./aws-iam-authenticator

 mv ./aws-iam-authenticator /usr/local/bin
 ```

### Step 3 : Create Kubeconfig file for Jenkins

 - I don't have the editor inside the Jenkins Container bcs it is a lightweight container . I will create the file outside on the Host and I just simple copy the file into Jenkin Container

 - Basic config file contents

 ```
 apiVersion: v1
 kind: Config
 clusters:
   - name: aws
     cluster:
       server: <Server API-Endpoint>
       certificate-authority-data: <base64-encoded-ca-cert>
 users:
   - name: arn:aws:eks:us-west-2:123456789012:cluster/my-cluster
     user:
       exec:
         apiVersion: client.authentication.k8s.io/v1beta1
         command: /usr/bin/aws-iam-authenticator
         args:
           - "eks"
           - "get-token"
           - "--cluster-name"
           - "my-cluster" 
 contexts:
   - name: aws
     context:
       cluster: kubernetes
       user: aws
 ```

 - I need to put in Cluster name

 - I need to put in Server API Endpoint : I can get this from EKS overview in Management Console

 - The other thing I need to chagne is certificate-authority-data : This is something that get generated in EKS Cluster when its gets created and I have that file available on local in `kube/config`

 - When I have the file ready I go inside Jenkins and create kube folder : `docker exec -it <container-id> bash` then `mkdir kube`

 - Then copy config file from host to Jenkins : `docker cp config container-id:/var/jenkins_home/.kube/`


### Step 4 : Create AWS Credentials 

 - Need Credentials for AWS Users . Locally I have work with the Admin User which is AWS User with its own secret key ID and access key . I need to configure the same for Jenkins

 - NOTE : I don't have to use Admin User to execute command on Jenkins . Best practice would be Create AWS IAM Jenkins User for different Services that Jenkins connect to and needs to authenticate to including Docker Repos, AWS, Kubernetes and so on. And I can give that User Limited Permission for security Reason

 - Inside Jenkins UI . Inside mutiple Branches Pipeline -> Go to Credentials -> Add Credentials -> Choose Secret Text

   - I will create 2 Credentials : access_key_id and secret_key_id

   - Locally my Credentials live here : `.aws/credentials`
  
### Step 5 : Create Jenkins file 

<img width="600" alt="Screenshot 2025-03-21 at 10 56 21" src="https://github.com/user-attachments/assets/c59995c3-dc95-43b8-8c82-6ab3dbe0d125" />

 - I can execute kubectl bcs bcs I have it installed inside my Jenkins Container . With kubectl execution aws-iam-authenticator also execute in the background

 - Before kubectl execute success I need to set or export ENV that will be use in that connection . I am setting those 2 ENV as a Context for the kubectl command to execute bcs in the background IAM will be executed an that command will need Access Credentials to connect to AWS  . Bcs I don't have .aws/config so I have to config like this . S

    ```
     AWS_ACCESS_KEY_ID

     AWS_SECRECT_ACCESS_KEY

    ----Sumarize----

    - Kubectl get executed which will use kubeconfig files created in .kube/config and inside that config file it is configure that aws-IAM-authenticator need to be use in order to authenticate with AWS account . And when aws-iam-authenticator command get trigger in the background, it need AWS_ACCESS_KEY and AWS_SECRET_ACCESS_KEY (AWS Credentials)
    ```

    - !!! NOTE : This part of authentication where I need the aws-iam-authenticator and setting AWS-crenditals in addition to Kubernetes Authentication it acctually specific to AWS . Other platform will have different way to authenticate 

## Deploy to LKE Cluster from Jenkins Pipelines

**Steps need to configure**

- Step 1 : Having Kubectl command line tools inside Jenkins Container

- Step 2 : Install Jenkins plugin that will make it possible for us to execute Kubectl with kubeconfig credentials

 - Instead of create kubeconfig of Linode Kubernetes Cluster inside the Jenkins container . I will configure it as a credentials inside Jenkins bcs it is a different type of Config file we can actually use that as a credentials, unlike AWS config file, which was more reliant on the authenticator in addition .

 - Once we have that Credentials of Kube config file , I will use the Plugin inside my Jenkinsfile so using plugin I can execute the Kubectl command that will deploy nginx deployment and pod inside the Linode Cluster

**Create Kubernetes Cluster on Linode and connect to it**

 - The reason why K8 Cluster on Linode so easy to create and get start with and also so fast is that AWS has much more granular control of the infrastructure, AWS has much more thing that I can able to configure when creating instances and create K8 on AWS . For example the VPC concepts and whole networking and private, public subnets

 - Step 1 : Go to Linode and Create K8 Cluster

 - Step 2: Once Cluster created I have Kubeconfig file that I can download . This Kubeconfig file will help me to connect to Cluster using Kubectl Command

**Add LKE credentials on Jenkins**

 - Go to multiple Branch Pipeline -> Credentails -> Choose Secret File -> Download the kubeconfig file to it

**Install Kubernetes CLI Plugin on Jenkins**

 - Install Jenkins Plugin that allow me to connect this kubectl to an actual Cluster

 - In Jenkins Plugin -> Available Plugin -> Kubernetes CLI

 - And some of dependency plugin also need to installed first, before installation can complete . So I will need to restart my Jenkins

**Configure Jenkinsfile to deploy to LKE cluster**

  - Create another branch to deploy to LKE Cluster : `git checkout -b deploy-to-LKE`

  - Now we have the Credentials for Kubeconfig and we have kubectl command available . How do I tell kubectl how to connect to LKE Cluster or which Cluster to connect to ? . And that come from kubectl CLI plugin that I installed

  - To use that Plugin : `withKubeConfig([credentialsId: 'credentials-id', serverUrl: 'LKE Cluster Enpdpoint']) { sh 'kubectl create deployment'}` .
  
  - serverUrl : Is tell kubectl where to connect to


## Credentials on Jenkins 


 - 1 thing from configuring credentials in Jenkins . As I saw in our example I have configured credentials for EC2 instances inside Jenkins so I could deploy on EC2 Server . I have have also configure Kubernetes credentials as kubeconfig file to connect  to K8 Cluster

 - Better Alternative is Create Jenkins user on EC2 server or whatever remote server, and for that Jenkins server there will be a Linux Server on that Instance, and for that Jenkins User basically to create credentials with its private key .

 - Same way created Kubernetes credential using Kubeconfig that Kubernetes admin user is using . So basically have all the Permission to execute stuff and configure stuff in Kubernetes . An Alternative here would be to create a Jenkins user in Kubernetes . In Kubernetes user are called Service Account . So basically create a Jenkins Service account and give it only the Permission that Jenkins needs which is just deploying a new Application , create deployment kind and maybe some other stuff . For example if it doesn't need to delete anything I may limit permission of the Jenkins service account and then I would create Credentials for Jenkins user inside K8 Cluster with user token 





















