# AWS Load Balancer Controller Installation Guide

## Prerequisites
Before proceeding, ensure you have:
- AWS CLI installed and configured
- `eksctl` installed
- `kubectl` installed
- `helm` installed
- An existing EKS cluster

## Step 1: Download IAM Policy
```sh
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.11.0/docs/install/iam_policy.json
```

## Step 2: Create IAM Policy
```sh
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json
```

## Step 3: Create IAM Service Account
```sh
eksctl create iamserviceaccount \
  --cluster=viral-cluster \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::<your-AWS-ACC-ID>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
```

### Override Existing Service Account (if needed)
```sh
eksctl create iamserviceaccount \
  --cluster=viral-cluster \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::<your-AWS-ACC-ID>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve \
  --override-existing-serviceaccounts
```

## Step 4: Install AWS Load Balancer Controller using Helm
### Add and Update Helm Repository
```sh
helm repo add eks https://aws.github.io/eks-charts
helm repo update eks
```

### Install Controller
```sh
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=viral-cluster \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=<your-REGION> \
  --set vpcId=<your_VPCID>
```

## Step 5: Verify Installation
```sh
kubectl get deployment -n kube-system aws-load-balancer-controller
kubectl get pods -n kube-system
kubectl get events -n kube-system
```

## Step 6: Validate IAM Service Account
```sh
kubectl describe serviceaccount aws-load-balancer-controller -n kube-system
```

### If Service Account is Not Created, Create It Manually
```sh
kubectl create serviceaccount aws-load-balancer-controller -n kube-system
```

## Step 7: Annotate Service Account
```sh
kubectl annotate serviceaccount aws-load-balancer-controller \
  -n kube-system \
  eks.amazonaws.com/role-arn=arn:aws:iam::<your-AWS-ACC-ID>:role/AmazonEKSLoadBalancerControllerRole
```

## Step 8: Restart Deployment
```sh
kubectl rollout restart deployment aws-load-balancer-controller -n kube-system
```

## Step 9: Debugging and Logs
### Check Pods
```sh
kubectl get pods -n kube-system
```

### Describe Specific Pod
```sh
kubectl describe pod aws-load-balancer-controller-68c6ff5b9b-scqr8 -n kube-system
```

### View Logs
```sh
kubectl logs aws-load-balancer-controller-68c69-scqr8 -n kube-system
```

## Conclusion
Following these steps, you should have successfully installed and configured AWS Load Balancer Controller in your EKS cluster.
