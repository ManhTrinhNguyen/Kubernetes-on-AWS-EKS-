# AWS-EKS 

## Create EKS Cluster . 

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

## Deploy Java Application 



















