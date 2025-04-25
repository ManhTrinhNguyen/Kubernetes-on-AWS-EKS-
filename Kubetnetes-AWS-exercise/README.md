- [Deploy Mysql and phpmyadmin](Deploy-Mysql-and-phpmyadmin)
  
# AWS-EKS 

## Create EKS Cluster . 

#### Connect kubectl locally with EKS Cluster

Eventhough I don't have Worker Nodes yet . I can still can talk to the API Server bcs Control Plane is running

They way to connect is create kubeconfig file for newly created EKS Cluster

Configure Kubectl to connect to EKS Cluster

Step 1: To see AWS configure detail : `aws configure list`

Step 2: To create kubeconfig file locally : `aws eks update-kubeconfig --name <cluster-name>`

-- --name : Connection info for Cluster . Which is the Cluster Name

 - After Created kubeconfig file my Local machine already connected to AWS K8 Cluster. The file will store in `.kube/config`

## Deploy Mysql and phpmyadmin 

#### Deploy Mysql 

To deploy Mysql I will use Helm to make the process more efficent  

Helm is a Package Manager of Kubernetes

Step 1 : Install helm `brew install helm` 

Step 2 : Bitnami is a provider Helm Charts . They also Provide and Maintain MySQL DB Helm chart . To get Bitnami Repo : helm repo add bitnami https://charts.bitnami.com/bitnami

!!! Note : When I execute Helm Command it will execute against the Cluster I connected to

Step 3 : To search for Bitnami Repo : `helm search repo binami/mysql`

Step 4 : Create Mysql values files . So I can override the value that I need for my MySQL 

```
architecture: replication
auth:
  rootPassword: secret-root-pass
  database: my-app-db
  username: my-user
  password: my-pass

# enable init container that changes the owner and group of the persistent volume mountpoint to runAsUser:fsGroup
volumePermissions:
  enabled: true

primary:
  persistence:
    enabled: false

secondary:
  # 1 primary and 2 secondary replicas
  replicaCount: 2
  persistence:
    enabled: false

    # Storage class for EKS volumes
    # storageClass: gp2
    # accessModes: ["ReadWriteOnce"]
```

Step 5: To install MySQL Helm Charts from Bitnami : `helm install <release-name> --values <mysql-helm-value-yaml-file> bitnami/mysql` 

Step 6 : After MySQL DB started . To debug if something wrong with the pods : kubectl logs <pods-name>

Step 7 : I Can also get inside the pod to see mysql pod ENV : kubectl exec -it <pod-name> -- bin/bash Or kubectl describe pod <pod-name>

#### Deploy Mysql for Production 

My values file : 

```
architecture: replication

auth:
  rootPassword: rootpassword  # ✅ use a Kubernetes Secret in real deployments
  database: my-app-db
  username: tim
  password: mypassword     # ✅ use a Kubernetes Secret in real deployments

volumePermissions:
  enabled: true  # ✅ Keep this enabled to ensure volume permissions are correct

primary:
  persistence:
    enabled: true
    storageClass: gp2                     # ✅ Use AWS EBS gp2 or gp3
    accessModes: ["ReadWriteOnce"]
    size: 20Gi                            # ✅ Adjust as per data needs

secondary:
  replicaCount: 2
  persistence:
    enabled: true
    storageClass: gp2
    accessModes: ["ReadWriteOnce"]
    size: 20Gi
```

EKS need certain Perrmission to allow Kubernetes to provision EBS Volumns dynamically when using `storgeClass: gp2 or gp3`

I need to install add-on EBS CSI Driver installed (Must be installed in my Cluster)

  - This driver actually talks to AWS EBS API to Create Volume, Attach them to Pod, Resize, delete them

EBS CSI Driver need an IAM Role with these Permission in AWS-managed policy `AmazonEBSCSIDriverPolicy`

Step 1 : Creates the IAM Role and links it to the Kubernetes service account via an OIDC provider.

```
eksctl create iamserviceaccount \
  --name ebs-csi-controller-sa \
  --namespace kube-system \
  --cluster $CLUSTER_NAME \
  --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
  --approve \
  --role-name AmazonEKS_EBS_CSI_DriverRole
```

  - Service Account is Application User . As the same way for human User I can link Service Account to Role or ClusterRole with RoleBinding or ClusterRoleBinding, And with binding service account or the Application that is behind that Service account will get Permission that are defined in the Role or Cluster Role

Step 2 : Install the EBS CSI driver as an EKS Add-on and attach the IRSA role

```
eksctl create addon \
  --name aws-ebs-csi-driver \
  --cluster $CLUSTER_NAME \
  --service-account-role-arn arn:aws:iam::<YOUR_AWS_ACCOUNT_ID>:role/AmazonEKS_EBS_CSI_DriverRole \
  --force
```

Step 3 : Confirm the EBS CSI driver is installed : `kubectl get pods -n kube-system -l app.kubernetes.io/name=aws-ebs-csi-driver` 

  - I should see something like
    
  ```
  ebs-csi-controller-xxxx        Running
  ebs-csi-node-xxxxx             Running
  ```
**Wrap up**

Every application (pod) runs under a ServiceAccount. If I want it to do something — inside the cluster or outside (like AWS APIs) — I need to give that ServiceAccount the right permissions or attach a role

If I use **Terraform** I can use this : `enable_ebs_csi_driver = true`


## Deploy Java Application 

I have Java Image in my ECR Private Repo

To pull Image From ECR are I need to create Secret Component containe access token and crenditals to my ECR 
 
Configure Deployment to use that Secret using Attribute called `imagePullSecrets` 

#### To Create Secret Component 

```
kubectl create secret docker-registry <my-secrect-name> \
--docker-server=https://565393037799.dkr.ecr.us-west-1.amazonaws.com
--docker-username=AWS
--docker-password=aws ecr get-login-password --region us-west-1
```

#### ENV For Container to connect to DB 

I create Secret Component to store DB_USER, DB_NAME 

```
apiVersion: v1
kind: Secret
metadata: 
  name: java-secret
type: Opaque 
data: 
  DB_USER: dGlt
  DB_NAME: bXktYXBwLWRi
```

And I create Configmap to store DB_URL_SERVER

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: java-configmap
data: 
  database_server: "mysql-primary-0:mysql-primary-headless"
```


















