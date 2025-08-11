## OIDC Setup

# OIDC builds trust between AWS and K8-RBAC

#Setup Variables

```
region_code=us-east-1
cluster_name=roboshop
ac_id=513993748676
```

# Create an IAM OIDC provider. You can skip this step if you already have one for your cluster.

```
eksctl utils associate-iam-oidc-provider \
    --region $region_code  \
    --cluster $cluster_name \
    --approve
```

# Download an IAM policy for the LBC using one of the following commands

```
curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.13.4/docs/install/iam_policy_us-gov.json
```

# Create an IAM policy named AWSLoadBalancerControllerIAMPolicy. 
If you downloaded a different policy, replace iam-policy with the name of the policy that you downloaded

```
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam-policy.json
```

# Create an IAM role and Kubernetes ServiceAccount for the LBC. Use the ARN from the previous step.

```
eksctl create iamserviceaccount \
--cluster=$cluster_name \
--namespace=kube-system \
--name=aws-load-balancer-controller \
--attach-policy-arn=arn:aws:iam::$ac_id:policy/AWSLoadBalancerControllerIAMPolicy \
--override-existing-serviceaccounts \
--region $region_code  \
--approve

```
# Install Drivers

```
helm repo add eks https://aws.github.io/eks-charts
```

```
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=$cluster_name --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller
```