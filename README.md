# 2048Game_EKSAWSDeploy
Practice project to deploy a real-time 2048 game application and expose it to outside world using Ingress and ALB Ingress Controller. 

Prerequisites:
kubectl
eksctl
AWS CLI

Step 1:
Configuring AWS CLI Credentials:
aws configure

Step 2:
Creating EKS cluster using Fargate:
eksctl create cluster --name demo-cluster --region us-east-1 --fargate

Step 3:
Configuring kubectl for EKS Cluster:
aws eks update-kubeconfig --name your-cluster-name

Step 4:
Creating Fargate profile:
eksctl create fargateprofile \
    --cluster demo-cluster \
    --region us-east-1 \
    --name alb-sample-app \
    --namespace game-2048

Step 5: Deploy namespace.yml, deployment.yml, service.yml, ingress.yml
kubectl apply -f deploy.yml

Step 6: Check Pods are running in the namespace
kubectl get pods -n namespacename

Step 7: Configure OIDC provider to associate EKS cluster and identity provider:
eksctl utils associate-iam-oidc-provider --cluster $cluster_name --approve

Step 8: Installing the AWS Load Balancer Controller

https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html

a) download IAM policy:
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json

b)Create IAM Policy:
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json

c)Create IAM Role
Creating service account which is required for ALB controller to create AWS resource like application load balancer.
eksctl create iamserviceaccount \
  --cluster=my-cluster \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::111122223333:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve

Step 9: Installing ALB controller using helm
  helm repo add eks https://aws.github.io/eks-charts
  helm repo update eks
  helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=my-cluster \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller 
  --set region=region-code \
  --set vpcId=vpc-xxxxxxxx

Step 10: 
Check ALB controller has created load balancer and linked its DNS name to ingress resource.
Access the application using load balancer DNS-name.



  



