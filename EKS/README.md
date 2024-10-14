# K8s Loki Logging example deployment on EKS

## deploy EKS Cluster

For this example I started with deploying a plain EKS cluster on AWS. 

```
eksctl create cluster -f EKS/cluster.yaml
```

Create an IAM OIDC provider for your cluster

```
cluster_name=eks-loki-cluster
oidc_id=$(aws eks describe-cluster --name $cluster_name --query "cluster.identity.oidc.issuer" --output text | cut -d '/' -f 5)
eksctl utils associate-iam-oidc-provider --cluster $cluster_name --approve
```

Create an IAM role and attach a policy

```
eksctl create iamserviceaccount \
        --name ebs-csi-controller-sa \
        --namespace kube-system \
        --cluster $cluster_name \
        --role-name AmazonEKS_EBS_CSI_DriverRole \
        --role-only \
        --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
        --approve
```


Create Addon

```
eksctl create addon --cluster $cluster_name --name aws-ebs-csi-driver --version latest \
    --service-account-role-arn arn:aws:iam::<account-id>:role/AmazonEKS_EBS_CSI_DriverRole --force
```

Configure Storage Class

```
kubectl apply -f storage-class.yaml
```

The next step is to install the LOKI resources. 

## Links & Documentation

### EKS

* [Creating and Managing Clusters](https://eksctl.io/usage/creating-and-managing-clusters/)
* [Create an IAM OIDC provider for your cluster](https://docs.aws.amazon.com/eks/latest/userguide/enable-iam-roles-for-service-accounts.html)
* [Store Kubernetes volumes with Amazon EBS](https://docs.aws.amazon.com/eks/latest/userguide/ebs-csi.html)
* [Set up driver permissions](https://github.com/kubernetes-sigs/aws-ebs-csi-driver/blob/master/docs/install.md#set-up-driver-permissions)
* [Creating an Amazon EKS add-on](https://docs.aws.amazon.com/eks/latest/userguide/creating-an-add-on.html)
* [aws-ebs-csi-driver: Dynamic Volume Provisioning](https://github.com/kubernetes-sigs/aws-ebs-csi-driver/tree/master/examples/kubernetes/dynamic-provisioning)
