## kops-kubernetes-cluster-configuration
## Landmark Technologies,  -    Landmark Technologies 
## Tel: +1 437 215 2483,   -     +1 437 215 2483 
## mylandmarktech@gaIL.com,  -    www.mylandmarktech.com 

## Setting up Kubernetes (K8s) Cluster on AWS Using KOPS

1.kops is a software use to create production ready k8s cluster in a cloud provider like AWS.

2. kOPS SUPPORTS MULTIPLE CLOUD PROVIDERS

3. Kops compete with managed kubernestes services like EKS, AKS and GKE

4. Kops is cheaper than the others.

5. Kops create production ready K8S.

6. KOPS create resources like: LoadBalancers, ASG, Launch Configuration, woker node Master node (CONTROL PLANE.

7. KOPS is IaaC

#!/bin/bash
## 1) Create Ubuntu EC2 instance in AWS

## 2a) create kops user and add kops user to sudoers group
``` sh
 sudo adduser kops
 sudo echo "kops  ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/kops
 sudo su - kops
 ```
 ##  2a) install AWSCLI using the apt package manager
  ```sh
 sudo apt install awscli -y 
 ```
 ## or 2b) install AWSCLI using the script below
 ```sh
sudo apt install unzip tree nano vim -y
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
 ```
## 3) Install kops software on an ubuntu instance by running the commands below:
 	sudo apt install wget -y
 	sudo wget https://github.com/kubernetes/kops/releases/download/v1.22.0/kops-linux-amd64
 	sudo chmod +x kops-linux-amd64
 	sudo mv kops-linux-amd64 /usr/local/bin/kops
 
## 4) Install kubectl kubernetes client if it is not already installed
```sh
 sudo curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
 sudo chmod +x ./kubectl
 sudo mv ./kubectl /usr/local/bin/kubectl
```
## 5) Create an IAM role from AWS Console or CLI with the below Policies. 

	AmazonEC2FullAccess 
	AmazonS3FullAccess
	IAMFullAccess 
	AmazonVPCFullAccess

Then Attach IAM role to ubuntu server from Console Select KOPS Server --> Actions --> Instance Settings --> Attach/Replace IAM Role --> Select the role which
You Created. --> Save.

## 6) create an S3 bucket
## Execute the commands below in your KOPS control Server. use unique s3 bucket name. If you get bucket name exists error.
	aws s3 mb s3://class30kops
	aws s3 ls # to verify
	
 ## 6b) create an S3 bucket    
	Expose environment variable:
    # Add env variables in bashrc
    
       vi .bashrc
	# Give Unique Name And S3 Bucket which you created.
	export NAME=class35.k8s.local
	export KOPS_STATE_STORE=s3://class35akops
 
      source .bashrc  
	
### 7) Create sshkeys before creating cluster
```sh
ssh-keygen -t rsa -b 4096
```

# 8) Create kubernetes cluster definitions on S3 bucket
```sh
kops create cluster --zones us-east-1a --networking weave --master-size t2.medium --master-count 1 --node-size t2.medium --node-count=2 ${NAME}
# copy the sshkey into your cluster to be able to access your kubernetes node from the kops server
kops create secret --name ${NAME} sshpublickey admin -i ~/.ssh/id_rsa.pub
```
# 9) Initialise your kops kubernetes cluser by running the command below
```sh
kops update cluster ${NAME} --yes
```
# 10a) Validate your cluster(KOPS will take some time to create cluster ,Execute below commond after 3 or 4 mins)

kops validate cluster
	   
	   Suggestions:
 * validate cluster: kops validate cluster --wait 10m
 * list nodes: kubectl get nodes --show-labels
 * ssh to the master: ssh -i ~/.ssh/id_rsa ubuntu@api.class.k8s.local
 * the ubuntu user is specific to Ubuntu. If not using Ubuntu please use the appropriate user based on your OS.
 * read about installing addons at: https://kops.sigs.k8s.io/operations/addons.

## 10b - Export the kubeconfig file to manage your kubernetes cluster from a remote server. For this demo, Our remote server shall be our kops server 
```sh
 kops export kubecfg $NAME --admin
```
## 11a) To list nodes and pod to ensure that you can make calls to the kubernetes apiSAerver and run workloads
	  kubectl get nodes 

### 11b) Alternative you can ssh into your kubernetes master server using the command below and manage your cluster from the master
    sh -i ~/.ssh/id_rsa ubuntu@ipAddress
    ssh -i ~/.ssh/id_rsa ubuntu@18.222.139.125
    ssh -i ~/.ssh/id_rsa ubuntu@172.20.58.124

### 11b. Alternative, Enable PasswordAuthentication in the master server and assign passwd
```sh
sudo sed -i "/^[^#]*PasswordAuthentication[[:space:]]no/c\PasswordAuthentication yes" /etc/ssh/sshd_config
sudo service sshd restart
sudo passwd ubuntu
```

### 11c) To list nodes

	  kubectl get nodes 
 
## 12) To Delete Cluster

   kops delete cluster --name=${NAME} --state=${KOPS_STATE_STORE} --yes  
   
====================================================================================================


13 # IF you want to SSH to Kubernetes Master or Nodes Created by KOPS. You can SSH From KOPS_Server

sh -i ~/.ssh/id_rsa ubuntu@ipAddress
ssh -i ~/.ssh/id_rsa ubuntu@3.90.203.23
  
``





Horizontal Cluster Autoscaler  - CAS :
  Cluster Autoscaler is a standalone program that adjusts the size of a kubernetes cluster to meet the current needs.
  Cluster autoscaler increases the size of the cluster by adding nodes when there are pods that failed to schedule on any current nodes due to insufficient capacity.
  Cluster autoscaler decreases the size of the cluster when nodes are underutilized for a significant perirod of time. A node is only removed when it has low untilization and all of its system critical pods can be moved elsewhere.


  NOTE: It is your responsibility to ensure such labels and/or taints are applied via the node's kubelet configuration at startup. Cluster Autoscaler will not set the node taints for you.

Recommendations:
  Tag the identifeied AS

It is recommended to use a second tag like k8s.io/cluster-autoscaler/<cluster-name> when k8s.io/cluster-autoscaler/enabled is used across many clusters to prevent ASGs from different clusters having conflicts.
An ASG must contain at least all the tags specified and as such secondary tags can differentiate between different clusters ASGs.
Commands
--------
kubectl get nodes

curl https://raw.githubusercontent.com/helm/helm/master/scripts/get > get_helm.sh

chmod 700 get_helm.sh

./get_helm.sh

kubectl -n kube-system create serviceaccount tiller

kubectl create clusterrolebinding tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller

helm init --service-account tiller

kubectl get pods --namespace kube-system

helm repo update

helm install --name autoscaler stable/cluster-autoscaler --namespace kube-system --set autoDiscovery.clusterName=<YourEKSClusterName> --set awsRegion=us-west-2  --set rbac.create=true --set cloudProvider=aws --set sslCertPath=/etc/kubernetes/pki/ca.crt

kubectl run nginx --image=nginx --port=80 --replicas=50

kubectl delete deployment nginx


Useful Links
------------

https://github.com/kubernetes/autoscaler

https://helm.sh/docs/using_helm/#installing-helm

https://github.com/helm/helm/blob/master/docs/rbac.md








Configure a Metrics Server on our Cluster4??
===========================================
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

wget https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml


https://github.com/LandmakTechnology/metric-server
git clone https://github.com/LandmakTechnology/metric-server
kubectl apply -f metric-server/metrics-server-deploy.yml
=====
$ kubectl apply -f metric-server/metrics-server-deploy.yml
serviceaccount/metrics-server created
clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader created
clusterrole.rbac.authorization.k8s.io/system:metrics-server created
rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader created
clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator created
clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server created
service/metrics-server created
deployment.apps/metrics-server created
apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created

kubernetes objects used in deploying metric server:
 1. RBAC = role base access control
    serviceaccount = metrics-server
    clusterrole [ pods/nodes = get/watch/list ]
    clusterrolebinding:

    role
    rolebinding
2. application [third party apps and addons]
   deployment
3. Service ServiceDiscovery
   service = service/metrics-server


NAME     CPU(cores)   MEMORY(bytes)
webapp   2m           145Mi

    resources:
      requests:
        memory: "128Mi"
        cpu: "500m"
      limits:
        memory: "128Mi"
        cpu: "500m"
 The pod will suffer from oomKilled

RBAC objects:
  serviceaccount metrics-server
     - user
     - groups
     - pods
  clusterrole
     - pods/nodes [get/watch/list]
  clusterrolebinding
     -
  rolebinding
Deployment with HPA
==================
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hpadeployment
spec:
  replicas: 2
  selector:
    matchLabels:
      name: hpapod
  template:
    metadata:
      labels:
        name: hpapod
    spec:
      containers:
        - name: hpacontainer
          image: k8s.gcr.io/hpa-example
          ports:
          - name: http
            containerPort: 80
          resources:
            requests:
              cpu: "100m"
              memory: "64Mi"
            limits:
              cpu: "100m"
              memory: "256Mi"
---
apiVersion: v1
kind: Service
metadata:
  name: hpaclusterservice
  labels:
    name: hpaservice
spec:
  ports:
    - port: 80
      targetPort: 80
  selector:
    name: hpapod
  type: ClusterIP

# HPA For above deployment(hpadeployment)
apiVersion: autoscaling/v2beta1
kind: HorizontalPodAutoscaler
metadata:
  name: hpadeploymentautoscaler
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: hpadeployment
  minReplicas: 1
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50


kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10
kubectl autoscale deployment hpadeployment --cpu-percent=50 --min=1 --max=10

-# Create temp POD using below command interatively and increase the
-# load on demo app by accessing the service.

kubectl run -i --tty load-generator --rm  --image=busybox /bin/sh

-# Access the service to increase the load.

while true; do wget -q -O- http://hpaclusterservice; done
