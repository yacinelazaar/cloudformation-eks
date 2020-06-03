# EKS Cloudformation template

This VPC architecture is considered the best practice for common Kubernetes workloads on AWS. In this configuration, nodes are instantiated in the private subnets and ingress resources (like load balancers) are instantiated in the public subnets. This allows for maximum control over traffic to the nodes and works well for a majority of Kubernetes applications.


Check these first:  
[Managed node groups](https://docs.aws.amazon.com/eks/latest/userguide/managed-node-groups.html)  
[Creating a VPC for your Amazon EKS cluster](https://docs.aws.amazon.com/eks/latest/userguide/create-public-private-vpc.html)  
[VPC tagging requirement + VPC considerations](https://docs.aws.amazon.com/eks/latest/userguide/network_reqs.html#vpc-subnet-tagging)  
[Amazon EKS security group considerations](https://docs.aws.amazon.com/eks/latest/userguide/sec-group-reqs.html)
[Pod networking (CNI)](https://docs.aws.amazon.com/eks/latest/userguide/pod-networking.html)  
[Alternate compatible CNI plugins](https://docs.aws.amazon.com/eks/latest/userguide/alternate-cni-plugins.html)

Resources needed to build an EKS cluster:  
- EKS Cluster (Duh)  
- VPC  
- IGW and attachement  
- Subnet(tagged properly and ditributed in the right fashion across AZs)  
- NatGateway and their EIPs  
- Route tables and their associations  
Optional:
- Security group(s) for the contorl plane
- Security group(s) for the workers


To launch the stack:  
```
aws cloudformation package \
--template-file base.yml \
--output-template-file packaged-template.yml \
--s3-bucket yassine-cfn-templates \
--s3-prefix eks-cluster
```

To launch the stack:  
```
aws cloudformation deploy \
--template-file packaged-template.yml \
--stack-name eks-cluster \
--parameter-overrides $(jq -r '.[] | [.ParameterKey, .ParameterValue] | join("=")' parameters.json) \
--capabilities CAPABILITY_IAM CAPABILITY_AUTO_EXPAND
```

Estimate stack monthly cost (Doesnt fucking work)
```
aws cloudformation estimate-template-cost \
--template-body file://packaged-template.yml \
--parameters | cat parameters.json
```

To get the kubeconfig (NB: this will override the current config)
```
aws eks update-kubeconfig --name <EKS_CLUSTER_NAME>
```