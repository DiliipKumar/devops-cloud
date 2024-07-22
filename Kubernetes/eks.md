## Kubernetes End to End project on EKS | EKS Install and app deploy with Ingress 

[Youtube](https://www.youtube.com/watch?v=RRCrY12VY_s&list=PLdpzxOOAlwvLNOxX0RfndiYSt1Le9azze&index=25)
[github](https://github.com/iam-veeramalla/aws-devops-zero-to-hero/tree/main/day-22)

## Prerequisites

Ensure you have the following installed and configured use choco and chatgtp:

- **kubectl**: Kubernetes command-line tool.
- **eksctl**: CLI tool for creating and managing EKS clusters.
- **AWS CLI**: Command-line tool for interacting with AWS services.

Configuring AWS CLI
```bash
aws configure
```
## Install eks cluster using Fargate


```bash
 eksctl create cluster --name demo-cluster --region us-east-1 --fargate
```
Configuring kubectl for EKS:
```bash
aws eks update-kubeconfig --name your-cluster-name
```

## Delete the cluster


```bash
eksctl delete cluster --name demo-cluster --region us-east-1
```

## Update Your Local Kubernetes Configuration to local terminal of laptop


```bash
aws eks update-kubeconfig --name <cluster-name> --region <region>

```

Please follow the prerequisites document before proceeding.



## Create Fargate profile with namespace game-2048

```bash
eksctl create fargateprofile --cluster demo-cluster  --region us-east-1  --name alb-sample-app  --namespace game-2048
```
## Deploy the Namespace, deployment, service and Ingress with namespace game-2048

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml
```
Concept:-

1.targetPort of svc should be equal containerPort of Deployment

svc  ports:

      targetPort: 80

 deployment ports:

        - containerPort: 80


2.Namespace of Namespace, deployment, service and Ingress should be sample

metadata:  
     namespace: game-2048


3.Deployment ka label match to selector of svc,deployment 

label of deployment:

        app.kubernetes.io/name: app-2048

selector of deployment and svc:


    app.kubernetes.io/name: app-2048

4.Ingress route the traffic inside cruster

5.Ingress controller--> ingress-2048 -->Load balancer--> Target group,port


## IAM OIDC provider configured before Alb controller IMP
Reason:-alb controller is pod if it want to external and

 internal service of aws like  s3,load balancer

# commands to configure IAM OIDC provider 

```
export cluster_name=demo-cluster
```

```
oidc_id=$(aws eks describe-cluster --name $cluster_name --query "cluster.identity.oidc.issuer" --output text | cut -d '/' -f 5) 
```

## Check if there is an IAM OIDC provider configured already

- aws iam list-open-id-connect-providers | grep $oidc_id | cut -d "/" -f4\n 

If not, run the below command

```
eksctl utils associate-iam-oidc-provider --cluster $cluster_name --approve
```
# How to setup alb add on

```bash
 choco install curl
```

Download IAM policy


```bash
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
```

Create IAM Policy

```
aws iam create-policy 
    --policy-name AWSLoadBalancerControllerIAMPolicy 
    --policy-document file://iam_policy.json
```

Create IAM Role

```
eksctl create iamserviceaccount 
  --cluster=<your-cluster-name> 
  --namespace=kube-system 
  --name=aws-load-balancer-controller 
  --role-name AmazonEKSLoadBalancerControllerRole 
  --attach-policy-arn=arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy 
  --approve
```
## Install Helm using Chocolatey
```
choco install kubernetes-helm
```
## Deploy ALB controller


Add helm repo

```
helm repo add eks https://aws.github.io/eks-charts
```

Update the repo

```
helm repo update eks
```

Install

```
helm install aws-load-balancer-controller eks aws-load-balancer-controller            
  -n kube-system 
  --set clusterName=<your-cluster-name> 
  --set serviceAccount.create=false 
  --set serviceAccount.name=aws-load-balancer-controller 
  --set region=<region> 
  --set vpcId=<your-vpc-id>
```

Verify that the deployments are running.

```
kubectl get deployment -n kube-system aws-load-balancer-controller
```