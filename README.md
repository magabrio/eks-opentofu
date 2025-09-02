# EKS OpenTofu

A simple repository showing configuration to deploy a minimal EKS cluster using OpenTofu (Terraform fork).

## ⚠️ **IMPORTANT DISCLAIMERS**

- **💰 COSTS MONEY**: This deployment will incur AWS charges (approximately $50-100/month)
- **🎓 EDUCATIONAL ONLY**: This is for learning purposes, NOT suitable for production
- **🧹 CLEAN UP**: Always run `tofu destroy` when done to avoid ongoing charges
- **🔒 NO PRODUCTION**: Missing security hardening, monitoring, backup, and other production requirements

## Overview

This project creates a complete EKS cluster with:
- VPC using the official AWS VPC module
- EKS cluster with managed node group
- Proper IAM roles and policies
- Security groups for cluster and nodes

## Prerequisites

1. **OpenTofu installed** - Download from [OpenTofu releases](https://github.com/opentofu/opentofu/releases)
2. **AWS CLI configured** - Run `aws configure` with your credentials
3. **AWS account** with appropriate permissions for EKS, VPC, IAM, and EC2

## Quick Start

1. **Clone and navigate to the repository:**
   ```bash
   git clone https://github.com/magabrio/eks-opentofu.git
   cd eks-opentofu
   ```

2. **Copy the example variables file:**
   ```bash
   cp terraform.tfvars.example terraform.tfvars
   ```

3. **Edit `terraform.tfvars` with your desired configuration:**
   ```hcl
   aws_region = "us-east-1"
   cluster_name = "my-eks-cluster"
   # ... other variables
   ```

4. **Initialize OpenTofu:**
   ```bash
   tofu init
   ```

5. **Plan the deployment:**
   ```bash
   tofu plan -var-file="terraform.tfvars"
   ```

6. **Apply the configuration:**
   ```bash
   tofu apply -var-file="terraform.tfvars"
   ```

7. **Configure kubectl:**
   ```bash
   aws eks update-kubeconfig --region <your-region> --name <your-cluster-name>
   ```

8. **Verify the deployment:**
   ```bash
   # Check cluster status and nodes
   kubectl get nodes
   kubectl get pods --all-namespaces
   
   # Verify cluster info
   kubectl cluster-info
   
   # Test with a simple deployment
   kubectl run test-pod --image=nginx --restart=Never
   kubectl get pods
   kubectl delete pod test-pod
   ```

## Configuration

### Key Variables

- `aws_region`: AWS region for deployment (default: us-east-1)
- `cluster_name`: Name of the EKS cluster (default: sample-eks-cluster)
- `kubernetes_version`: Kubernetes version (default: 1.33)
- `vpc_cidr`: VPC CIDR block (default: 10.0.0.0/16)
- `node_desired_size`: Desired number of worker nodes (default: 2)
- `node_instance_types`: EC2 instance types for nodes (default: ["t4g.small"] - ARM instances)

**Note**: Availability zones and subnet CIDRs are calculated automatically:
- **Availability Zones**: First 2 available zones in the selected region
- **Subnet CIDRs**: Calculated dynamically using `cidrsubnet()` function

### Network Architecture

- **VPC**: 10.0.0.0/16 (configurable via `vpc_cidr` variable)
- **Public Subnets**: 10.0.1.0/24, 10.0.2.0/24 (calculated using `cidrsubnet()`)
- **Private Subnets**: 10.0.10.0/24, 10.0.20.0/24 (calculated using `cidrsubnet()`)
- **Availability Zones**: First 2 available zones in the region (dynamic)
- **NAT Gateway**: Single NAT gateway

## Outputs

After deployment, you'll get important information like:
- Cluster endpoint
- Certificate authority data
- VPC and subnet IDs
- IAM role names

## Cleanup

**⚠️ IMPORTANT: Always destroy resources to avoid ongoing charges!**

To destroy all resources:
```bash
tofu destroy -var-file="terraform.tfvars"
```

**Cost breakdown if left running:**
- EKS Cluster: ~$73/month
- EC2 instances (2x t4g.small): ~$15/month  
- NAT Gateway: ~$45/month
- **Total: ~$133/month if not cleaned up**

## Security Notes

**⚠️ NOT PRODUCTION READY - Educational purposes only!**

- Worker nodes are deployed in private subnets
- Cluster endpoint has both public and private access
- Security groups are configured for basic communication
- IAM roles follow basic least privilege principle
- **Missing**: Network policies, pod security standards, RBAC hardening, encryption at rest, audit logging, etc.

## Cost Optimization

- Uses **t4g.small ARM instances** by default (more cost-effective than x86)
- **Single NAT Gateway** instead of multiple (significant cost savings)
- CloudWatch logs retention is set to 7 days
- Uses AWS VPC module for optimized networking setup
- Dynamic subnet calculation reduces configuration complexity

## Troubleshooting

1. **Permission errors**: Ensure your AWS credentials have sufficient permissions
2. **Resource limits**: Check AWS service limits in your region
3. **Network issues**: Verify VPC and subnet configurations
4. **Node group issues**: Check IAM role policies and instance types
5. **AMI type errors**: If using ARM instances (t4g, m6g, etc.), ensure `ami_type = "AL2023_ARM_64_STANDARD"` is set
6. **Instance type compatibility**: ARM instances require ARM AMIs, x86 instances require x86 AMIs
7. **Version mismatch**: If node group version doesn't match cluster, run: `tofu apply -replace="aws_eks_node_group.main" -var-file="terraform.tfvars"`
8. **Timeout issues**: EKS operations can take 15-30 minutes, timeouts are configured in the code

## Next Steps

After deployment, you can:
1. Install AWS Load Balancer Controller
2. Deploy applications to your cluster
3. Set up monitoring and logging
4. Configure ingress controllers
5. Implement backup strategies
