#  AWS 3-Tier Architecture with CloudFormation

[![AWS](https://img.shields.io/badge/AWS-CloudFormation-FF9900?style=flat&logo=amazon-aws)](https://aws.amazon.com/cloudformation/)
[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Infrastructure](https://img.shields.io/badge/Infrastructure-as--Code-green.svg)](template.yml)

A production-ready 3-tier AWS architecture deployed using CloudFormation YAML template, demonstrating best practices for cloud infrastructure automation.

##  Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Features](#features)
- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [Deployment Options](#deployment-options)
- [Project Structure](#project-structure)
- [Intrinsic Functions Demonstrated](#intrinsic-functions-demonstrated)
- [Security Features](#security-features)
- [Cost Estimation](#cost-estimation)
- [Troubleshooting](#troubleshooting)
- [License](#license)

##  Overview

This project implements a complete 3-tier AWS architecture using CloudFormation Infrastructure as Code (IaC). It includes:

- **Presentation Tier**: EC2 web server in public subnet
- **Application Tier**: EC2 instance with application logic capability
- **Data Tier**: RDS MySQL database and S3 object storage in private subnet

The template demonstrates advanced CloudFormation features including intrinsic functions, parameter validation, and proper resource dependencies.

##  Architecture

![Architecture Diagram](architecture_diagram.html)

### Network Design
```
VPC (10.0.0.0/16)

 Public Subnet (10.0.1.0/24)
    Internet Gateway
    Route Table  0.0.0.0/0  IGW
    EC2 Instance (t2.micro)
        Security Group (SSH + HTTP)
        Public IP (Auto-assigned)

 Private Subnet (10.0.2.0/24)
     RDS MySQL (db.t3.micro)
        Security Group (MySQL from EC2 only)
     S3 Bucket (AES-256 encrypted)
```

##  Features

### CloudFormation Best Practices
-  Parameterized template for reusability
-  Comprehensive input validation
-  Proper resource dependencies
-  Intrinsic functions (`!Ref`, `!Sub`, `!GetAtt`, `!Select`, `!GetAZs`)
-  Detailed outputs for easy access
-  Extensive inline documentation

### Security Features
-  Private subnet for database isolation
-  Security groups with least privilege access
-  S3 bucket with public access blocked
-  RDS encryption at rest (AES-256)
-  No hard-coded credentials
-  VPC with DNS support

### Operational Features
-  CloudWatch integration ready
-  Comprehensive resource tagging
-  Easy environment switching (dev/prod)
-  Automated Apache installation via UserData
-  Multi-region compatible

##  Prerequisites

### Required
- AWS Account with appropriate permissions
- EC2 Key Pair in target region
- AWS CLI configured (for CLI deployment)

### IAM Permissions Needed
```json
{
  \"Version\": \"2012-10-17\",
  \"Statement\": [
    {
      \"Effect\": \"Allow\",
      \"Action\": [
        \"cloudformation:*\",
        \"ec2:*\",
        \"rds:*\",
        \"s3:*\",
        \"iam:CreateServiceLinkedRole\"
      ],
      \"Resource\": \"*\"
    }
  ]
}
```

##  Quick Start

### Option 1: AWS Console (Easiest)

1. **Create EC2 Key Pair**:
```
   AWS Console  EC2  Key Pairs  Create key pair
   Name: my-keypair
```

2. **Deploy Stack**:
```
   AWS Console  CloudFormation  Create stack
    Upload template.yml
    Follow the wizard
```

3. **Access Web Server**:
```bash
   # Get Public IP from Outputs tab
   curl http://<EC2-Public-IP>
```

### Option 2: AWS CLI
```bash
# Validate template
aws cloudformation validate-template --template-body file://template.yml

# Create stack
aws cloudformation create-stack \
  --stack-name my-3tier-stack \
  --template-body file://template.yml \
  --parameters file://parameters.json \
  --region eu-central-1

# Monitor status
aws cloudformation describe-stacks \
  --stack-name my-3tier-stack \
  --query 'Stacks[0].StackStatus'

# View outputs
aws cloudformation describe-stacks \
  --stack-name my-3tier-stack \
  --query 'Stacks[0].Outputs'
```

##  Project Structure
```
.
 template.yml                  # Main CloudFormation template
 parameters.json               # Parameter values for CLI
 architecture_diagram.html     # Interactive architecture diagram
 README.md                     # This file
 screenshots/
     stack_create_complete.png
     stack_outputs_dev.png
     stack_update_complete.png
     stack_outputs_prod.png
```

##  Cost Estimation

### Monthly Cost Breakdown (us-east-1)

| Service | Resource | Monthly Cost |
|---------|----------|--------------|
| EC2 | t2.micro (1 instance) | ~$8.50 |
| RDS | db.t3.micro (1 instance) | ~$15.00 |
| S3 | Standard (first 50 TB) | ~$0.023/GB |
| VPC | Internet Gateway | Free |
| Data Transfer | First 1 GB/month | Free |
| **Total** | **(excluding data)** | **~$23.50** |

*Note: Costs vary by region and usage. This is an estimate.*

##  Troubleshooting

### Issue: AMI Not Found
**Error**: `The image id '[ami-xyz]' does not exist`

**Solution**: Update the AMI ID in `template.yml` for your region

### Issue: Key Pair Not Found
**Error**: `The key pair 'my-keypair' does not exist`

**Solution**: Create a key pair in your target region first

### Issue: S3 Bucket Already Exists
**Error**: `Bucket already exists`

**Solution**: S3 bucket names must be globally unique. Change stack name or add a suffix.

##  Cleanup

**Important**: Always clean up resources to avoid charges!
```bash
# Empty S3 bucket first
aws s3 rm s3://<your-bucket-name> --recursive

# Delete stack
aws cloudformation delete-stack --stack-name my-3tier-stack
```

##  License

This project is licensed under the MIT License.

##  Author

**Franklin Chinonso Osuji**
- GitHub: [@yourusername](https://github.com/yourusername)
- LinkedIn: [your-linkedin](https://linkedin.com/in/your-profile)

---

** Disclaimer**: This template is for educational and testing purposes. For production use, implement additional security measures, monitoring, backups, and high availability features.
