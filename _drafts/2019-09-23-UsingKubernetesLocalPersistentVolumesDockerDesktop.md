---
layout: post
title:  "Using Kubernetes Local Persistent Volumes on Docker-Desktop"
date:   2019-09-23 13:37:00
comments: true
modified: 2019-09-23
---

In this blog post, I'll be walking through how to create a persistent volume in Kubernetes. There are sevearl volume types in Kubernetes, but to get start I'll be using the local volume type. A local volume represents a mounted local storage device such as a disk, partition or directory. I'll be using a Kubernetes cluster running within docker-desktop. Because of this, there isn't an easy way (at lest that I've found) to access the node running in the docker-desktop instance that hosts the Kubernetes cluster. Because of that the maintenance of the voulmes is a bit different that you'd expect. Before we dive in first let's answer the question of __why would you create a persistent volume?__

If you've been working with virtual machines most of your career this isn't something you've given much thought. The reasons for that is most virtualized environments are static. You create a virtual machine and make changes to that virtual machine. You only delete it when it is time to decomission it. Which we all know is a very unlikely scenario. When administering containers on the other hand the deletion of a container happens all the time. Everytime you want to update the container, you delete it and replace it with a new one. With that in mind how do we keep the data the container wrote? Persisteted volumes answer that question. The example I'll be using in this blog post is persisting data stored by mysql for hosting a TeamCity database. 

* TOC
{:toc}

### Methods for Creating Persistent Volumes

There are two options you have use when creating persistent volumes. You can create the volume as a pod volume. Which means that you define the volume and mount it within a single Kubernets pod spec. While this is the simpliest option, but it also had a draw back. By defining the volume in the Pod spec you couple the data layer in with your application. This will make scaling and managment of the data layer more difficult. It also requires that you have a good understanding of the data layer. The other option you have is to create a persistent volume and persistent volume claim. By doing so you decouple the data layer and the application layer. Making the management of each simpler. In this blog post I'll be using a persistent volume and persistent volume claim.

Two Options
* Pod Volume
* Persistent Volume & Persistent Volume Claim

### Creating a Local PersistentVolume

You can think of a persistent volume as a datastore to steal some vmWare terminology. Persistent volumes are provisioned by cluster administrators or dynamically provisioned using storage classes. For now, let's keep it simple and provision it by hand. You'll have to define a Kubernetes manifest with the kind of `PersistentVolume`. The manifest requires a few details about the storage that will be provisioned. You'll have to give it a name, storageClassName, specify the capcacity, access modes and path since I'm using the local storage type. These options will change with the type of storage you choose to use. 

There are two important peices of information that are worth explaining a bit more. Capacity and accessModes are used by the persistent volume claims. Capactiy is straight forward. It's the amount of space available for a claim to take. Claims use this information to determine which persistent volumes it can bind to. accessModes define what operations can be performed on the volume. There are a few different accessModes; ReadWriteOnce, ReadOnlyMany and ReadWriteMany. These define how the pods can read and write data to the volume. Claims use capacity and access mode to determine which volumes to bind to do fulfill the claim. 

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Delete
  hostPath:
    path: "/tmp/teamcitydata"
```

```
#save above as mysql-pv.yaml
kubectl apply -f mysql-pv.yaml

#Get pv (persistentVolume)
kubectl get pv
```



### Creating a Persistent Volume Claim

Storage is now available and ready to be claimed. Next, is to create a persistent volume claim. This claim will be used by the pod spec and mounted to the container. As I mentioned previously the volume needs to define the capacity and access mode. The volume claim also needs to specify this information because it uses those to match up the claim with the volume. 


```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

```
#save above as mysql-pvc.yaml
kubectl apply -f mysql-pvc.yaml

#get pvc (persistentVolumeClaims)
kubectl get pvc
```

### Using Persistent Volume Claims with Deployments

At this point, a persistent volume and a persistent volume claim have been created. The persistent volume claim has been bound to the persistent volume by matching the access modes and capacity. You can confirm this by running the `kubectl get pv` command and looking at the _STATUS_ column of the `mysql-pv-volume`. The claim is now ready to be used by a pod or deployment spec. To demonstate this I'll use a mysql deployment that creates a service for mysql and runs a single mysql container. Pay close attention to the volumes and volumeMounts sections. The volumeMounts section of the container spec specifies the name of the volume and the mounthPath inside the conatiner. The volumes section creates uses the persistent volume claim as it's mapping to an actual volume that can be used by the container.

```
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  ports:
  - port: 3306
  selector:
    app: mysql
  clusterIP: None
---
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: mysql
spec:
  selector:
    matchLabels:
      app: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - image: mysql:5.6
        name: mysql
        env:
          # Use secret in real usage
        - name: MYSQL_ROOT_PASSWORD
          value: password
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: mysql-pv-claim
```

```
#save the above as mysql-deployment.yaml
kubectl apply -f mysql-deployment.yaml

#get pods (confirm mysql container is running)
kubectl get po -l app=mysql
```

### Staging a Database

With a running pod I can now stage some configuration for my TeamCity database. To do this I'm going to cheat and load up a side-car container with mysql-client on it. Once I'm connected to the mysql database I'm going to create a database for Teamcity and provision a user account for TeamCity to use when connecting to the database. I'll also grant that user permissions to the new database. After you issued the mysql commands you can check the mounted folder on your local operating system to verify if the data has been written or not. In my case I'm using docker-desktop for mac and my shared path is `/tmp/teamcitydata`.

```
#run side-car mysql-client container
kubectl run -it --rm --image=mysql:5.6 --restart=Never mysql-client -- mysql -h mysql -ppassword

#mysql commands to run
create database teamcity collate utf8_bin;
create user teamcity identified by 'password';
grant all privileges on teamcity.* to teamcity;
grant process on *.* to teamcity;

#list contents of mysql volume
ls /tmp/teamcitydata
```

### Deleting Data Within a PersistentVolume

talk about persistentVolumeReclaimPolicy just deletes the volume not the data 
recycle option
talk abut shared paths & using /tmp


### Sources

[run-single-instance-stateful-application](https://kubernetes.io/docs/tasks/run-application/run-single-instance-stateful-application/)

[Setting up a MySQL External Database for TeamCity](https://www.jetbrains.com/help/teamcity/setting-up-an-external-database.html?_ga=2.213872598.374019039.1565610915-964155662.1565610915#SettingupanExternalDatabase-MySQL)