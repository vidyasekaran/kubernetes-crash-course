# Kubernetes on AWS and Azure

Highlights
- This is a bonus section added in 6 months after the course release
- RECOMMENDED APPROACH - First Learn with GCP and then come here!
- 99% of things with Kubernetes remain the same in AWS, GCP and Azure.
- ISTIO and HELM would be the same in all clouds. So, we will not be worry about them here!
- Download Github repo again!

## AWS

Prerequisites
- Install and Configure AWS CLI `aws configure`
- Install EKSCTL - https://eksctl.io/introduction/installation/ `eksctl version`
- EKS is not free. t2.medium is not in the free tier!


### Create EKS Cluster

Help for eksctl https://eksctl.io/

```
eksctl create cluster --name in28minutes-cluster --nodegroup-name in28minutes-cluster-node-group  --node-type t2.medium --nodes 3 --nodes-min 3 --nodes-max 7 --managed --asg-access
```

If you get this error
```
AWS::EKS::Cluster/ControlPlane: CREATE_FAILED – "Cannot create cluster 
'in28minutes-dev-cluster' because us-east-1e, the targeted availability zone, does not 
currently have sufficient capacity to support the cluster. Retry and choose from these 
availability zones: us-east-1a, us-east-1b, us-east-1c, us-east-1d, us-east-1f (Service: 
	AmazonEKS; Status Code: 400; Error Code: UnsupportedAvailabilityZoneException; Request 
	ID: a5580928-689d-4558-b3bd-2573131ec69e)
```

Add Availability Zones

```
--zones=us-east-1a,us-east-1b
```

Things to Note
- VPCs and SubNets are Created

### Deploying Basic Applications

```
kubectl create deployment hello-world-rest-api --image=in28min/hello-world-rest-api:0.0.1.RELEASE
kubectl get deployment - deployment is created
kubectl get replicaset - replica set is created

kubectl expose deployment hello-world-rest-api --type=LoadBalancer --port=8080
kubectl get svc

$ kubectl get svc
NAME                   TYPE           CLUSTER-IP       EXTERNAL-IP                                                                PORT(S)          AGE
hello-world-rest-api   LoadBalancer   10.100.126.207   a3b847de9584c4ee19f5b1401fdeebc6-1775864312.ap-south-1.elb.amazonaws.com   8080:31231/TCP   3m29s
kubernetes             ClusterIP      10.100.0.1       <none>                                                                     443/TCP          15m
todowebapp-h2          LoadBalancer   10.100.28.37     ab7f944d59f3e4db4ad45253b4ceba6a-967590409.ap-south-1.elb.amazonaws.com    8080:32560/TCP   6s

http://a3b847de9584c4ee19f5b1401fdeebc6-1775864312.ap-south-1.elb.amazonaws.com:8080/hello-world

kubectl get pods
kubectl create deployment todowebapp-h2 --image=in28min/todo-web-application-h2:0.0.1-SNAPSHOT
kubectl expose deployment todowebapp-h2 --type=LoadBalancer --port=8080
kubectl get svc

http://ab7f944d59f3e4db4ad45253b4ceba6a-967590409.ap-south-1.elb.amazonaws.com:8080/list-todos
username pwd : in28minutes/dummy


cd /Ranga/git/01.udemy-course-repos/kubernetes-crash-course/03-todo-web-application-mysql/backup/02-final-backup-at-end-of-course 
kubectl apply -f mysql-database-data-volume-persistentvolumeclaim-aws.yaml,mysql-deployment.yaml,mysql-service.yaml
kubectl apply -f config-map.yaml,secret.yaml,todo-web-application-deployment.yaml,todo-web-application-service.yaml
echo -n dummytodos | base64
```

### Delete Basic Applications

```
kubectl delete all -l app=hello-world-rest-api
kubectl delete all -l app=todowebapp-h2
kubectl delete all -l io.kompose.service=todo-web-application
kubectl delete all -l io.kompose.service=mysql
```

### Deploy Basic Microservices

```
kubectl apply -f 04-currency-exchange-microservice-basic/deployment.yaml 
kubectl apply -f 05-currency-conversion-microservice-basic/deployment.yaml
```

### Adding Ingress - little messy the below method you can use below kalyans method to create ingress controller.
https://github.com/vidyasekaran/aws-eks-kubernetes-masterclass/tree/master/08-ELB-Application-LoadBalancers/08-01-ALB-Ingress-Install
https://github.com/vidyasekaran/aws-eks-kubernetes-masterclass/tree/master/08-ELB-Application-LoadBalancers/08-03-ALB-Ingress-ContextPath-Based-Routing

- https://docs.aws.amazon.com/eks/latest/userguide/alb-ingress.html
- Check out sample application - https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.4/docs/examples/2048/2048-ingress.yaml


```

```

### Adding Application Insights

- https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Container-Insights-setup-EKS-quickstart.html
- Attach CloudWatchAgentServerPolicy to the nodes

### Exploring Cluster Auto Scaler

- https://docs.aws.amazon.com/eks/latest/userguide/cluster-autoscaler.html

Testing auto scaler
```
kubectl create deployment autoscaler-demo --image=nginx
kubectl scale deployment autoscaler-demo --replicas=50
```

### Clean up the Cluster

https://docs.aws.amazon.com/eks/latest/userguide/delete-cluster.html

Delete all LoadBalancer services
```
kubectl get svc --all-namespaces
kubectl delete svc service-name
```

Delete the cluster
```
eksctl delete cluster --name in28minutes-cluster
```

## Azure

### Create AKS Cluster

Let's use Azure Cloud Shell

```
# az login # az cli

az group create --name kubernetes-resource-group --location westeurope

az ad sp create-for-rbac --skip-assignment --name kubernetes-cluster-service-principal

az aks create --name in28minutes-cluster --node-count 4 --enable-addons monitoring --resource-group kubernetes-resource-group --vm-set-type VirtualMachineScaleSets --load-balancer-sku standard --enable-cluster-autoscaler  --min-count 1 --max-count 7  --generate-ssh-keys --service-principal <<appId>> --client-secret  <<password>>

az aks get-credentials --resource-group kubernetes-resource-group --name in28minutes-cluster
```

### Deploying Basic Applications

```
kubectl create deployment hello-world-rest-api --image=in28min/hello-world-rest-api:0.0.1.RELEASE

kubectl expose deployment hello-world-rest-api --type=LoadBalancer --port=8080

kubectl create deployment todowebapp-h2 --image=in28min/todo-web-application-h2:0.0.1-SNAPSHOT

kubectl expose deployment todowebapp-h2 --type=LoadBalancer --port=8080

cd kubernetes-crash-course/03-todo-web-application-mysql/backup/02-final-backup-at-end-of-course 
kubectl apply -f mysql-database-data-volume-persistentvolumeclaim.yaml,mysql-deployment.yaml,mysql-service.yaml

#WAIT FOR MYSQL TO BE UP AND RUNNING

kubectl apply -f config-map.yaml,secret.yaml,todo-web-application-deployment.yaml,todo-web-application-service.yaml
```

### Delete Basic Applications

```
kubectl delete all -l app=hello-world-rest-api
kubectl delete all -l app=todowebapp-h2
kubectl delete all -l io.kompose.service=todo-web-application
kubectl delete all -l io.kompose.service=mysql
```

### Deploy Microservices

```
kubectl apply -f 04-currency-exchange-microservice-basic/deployment.yaml 
kubectl apply -f 05-currency-conversion-microservice-basic/deployment.yaml
# watch -n 0.1 curl
# Explore Logs
```

### Create Ingress

Install Ingress Controller
```
helm repo add stable https://kubernetes-charts.storage.googleapis.com/

helm install stable/nginx-ingress --namespace default --set controller.replicaCount=1 --set controller.nodeSelector."beta\.kubernetes\.io/os"=linux --set defaultBackend.nodeSelector."beta\.kubernetes\.io/os"=linux --generate-name

kubectl get service -l app=nginx-ingress
```

More details - https://docs.microsoft.com/en-us/azure/aks/ingress-tls

Setup Ingress
```
kubectl apply -f 05-currency-conversion-microservice-basic/ingress_azure.yaml
```
### Exploring Cluster Auto Scaler

Testing auto scaler
```
kubectl create deployment autoscaler-demo --image=nginx
kubectl scale deployment autoscaler-demo --replicas=5000
```

### Clean up the cluster

Delete all LoadBalancer services
```
kubectl get svc --all-namespaces
kubectl delete svc service-name
```

Delete the cluster
```
az aks delete --name in28minutes-cluster --resource-group kubernetes-resource-group
```
