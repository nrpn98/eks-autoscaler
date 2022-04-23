In this repo you can find how to create an AWS EKS cluster using eksctl and enabling autoscaler

## Prerequisites
AWS account 
Create a User -> raju@abc.com

Create Role -> AWSServiceRoleForAmazonEKS

SSH Key pair -> eks-raj

Install following in your PC
	
	awscli
		https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html
	eksctl
		https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html
	kubectl
		https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#install-kubectl-binary-with-curl-on-linux
		
Test that your installation was successful with the following command.
	aws --version
  
  eksctl version
  
  kubectl version --short --client			


Setup your PC with your aws account
	aws configure
	aws sts get-caller-identity # to know who is currently login

## Create EKS cluster using eksctl
In this example I have create the EKS cluster in existing VPC.
```bash
time eksctl create cluster -f cluster-create.yml
```

## applying the autoscaler
Once you created the cluster with autoScaler: true apply following yaml to set the autscaler
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/autoscaler/master/cluster-autoscaler/cloudprovider/aws/examples/cluster-autoscaler-autodiscover.yaml
```
Once the autoscare created we need to create the annoation, doing this the autoscaler wont evict it own pod
```bash
kubectl -n kube-system annotate deployment.apps/cluster-autoscaler cluster-autoscaler.kubernetes.io/safe-to-evict="false"
kubectl -n kube-system annotate deployment.apps/cluster-autoscaler cluster-autoscaler.kubernetes.io/safe-to-evict="false" --overwrite  #check this in prodction

kubectl -n kube-system deployment.apps/cluster-autoscaler | grep -i 'safe-to-evict'
```

Then go to following link and find the cluster autoscaler image version for your kubernetes cluster
https://github.com/kubernetes/autoscaler/releases?page=2

https://github.com/kubernetes/autoscaler/releases/tag/cluster-autoscaler-1.21.2

Once you get the image version type below and ammend the image version along with clustername

```bash
kubectl -n kube-system edit deployment.apps/cluster-autoscaler

kubectl get pod -n kube-syste
```

## test the autoscaler
```bash
kubectl apply -f nginx-deployment.yaml
kubectl scale --replica=5 deployment/nginx-deployment
kubeclt get pod -o wide
watch kubectl get node -o wide
watch kubectl get pod -o wide
kubectl scale --replica=1 deployment/nginx-deployment
kubeclt get pod -o wide
watch kubectl get node -o wide
watch kubectl get pod -o wide
```


## view cluster autoscaler logs
```bash
kubectl -n kube-system logs deployment.app/cluster-autoscaler | grep -A5 "Expanding Node Group"
kubectl -n kube-system logs deployment.app/cluster-autoscaler | grep -A5 "node removed by cluster autoscaler"
```
