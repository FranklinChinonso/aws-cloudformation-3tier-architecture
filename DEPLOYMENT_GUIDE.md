# 3-Tier AWS Architecture - Deployment Guide

##  Prerequisites

Before deploying this CloudFormation stack, ensure you have:

1. **AWS Account** with appropriate permissions (EC2, VPC, RDS, S3, CloudFormation)
2. **EC2 Key Pair** created in your target AWS region
3. **AWS CLI configured** (optional, for CLI deployment)
4. **Sufficient service quotas** (especially for RDS and EC2 in your region)

---

##  Deployment Steps

### Step 1: Prepare Your Environment

1. **Create an EC2 Key Pair** (if you don't have one):
   - Go to AWS Console  EC2  Key Pairs
   - Click \"Create key pair\"
   - Name it (e.g., `my-keypair`)
   - Download the `.pem` file and store it securely

2. **Choose your AWS Region**:
   - Recommended: `eu-central-1` (Frankfurt) or your preferred region
   - **Important**: Update the AMI ID in the template if using a different region

### Step 2: Deploy via AWS Console

1. **Navigate to CloudFormation**:
```
   AWS Console  CloudFormation  Create stack  With new resources
```

2. **Upload Template**:
   - Choose \"Upload a template file\"
   - Select your `template.yml` file
   - Click \"Next\"

3. **Specify Stack Details**:
   - **Stack name**: `my-3tier-stack` (or your preferred name)
   - **Parameters**:
     - KeyName: `my-keypair` (select from dropdown)
     - EnvironmentType: `dev`
     - DBName: `mydatabase`
     - DBUser: `admin`
     - DBPassword: `Password123` (use a strong password in production!)

4. **Configure Stack Options** (Step 3):
   - Tags (optional): Add `Project: CloudFormationTest`
   - Permissions: Leave as default
   - Advanced options: Leave as default
   - Click \"Next\"

5. **Review and Create**:
   - Review all settings
   - Check the box: \"I acknowledge that AWS CloudFormation might create IAM resources\"
   - Click \"Submit\"

6. **Monitor Stack Creation**:
   - Watch the \"Events\" tab for real-time updates
   - Status will change: CREATE_IN_PROGRESS  CREATE_COMPLETE
   - **Time estimate**: 10-15 minutes (RDS takes the longest)

### Step 3: Verify Deployment

Once status shows `CREATE_COMPLETE`:

1. **Check the Outputs Tab**:
   - VPCId
   - EC2InstanceId
   - EC2PublicIP
   - S3BucketName
   - RDSEndpoint
   - WebsiteURL

2. **Test EC2 Instance**:
```bash
   # Copy the WebsiteURL from Outputs
   curl http://<EC2-Public-IP>
   
   # Should display: \"Hello from dev Environment!\"
```

3. **Verify Resources Created**:
   - VPC: Go to VPC Console  Your VPCs
   - Subnets: VPC Console  Subnets (should see 2)
   - EC2: EC2 Console  Instances (should see 1 running)
   - RDS: RDS Console  Databases (should see 1 available)
   - S3: S3 Console  Buckets (should see your bucket)

---

##  Stack Update Task

### Updating EnvironmentType from 'dev' to 'prod'

1. **Navigate to CloudFormation**:
   - Select your stack (`my-3tier-stack`)
   - Click \"Update\"

2. **Select Update Method**:
   - Choose \"Use current template\"
   - Click \"Next\"

3. **Modify Parameters**:
   - Change **EnvironmentType** from `dev` to `prod`
   - Keep all other parameters the same
   - Click \"Next\"

4. **Review Changes**:
   - CloudFormation will show you the change preview
   - Expected change: EC2 Instance tags will be updated
   - Click \"Next\"

5. **Execute Update**:
   - Review the change set
   - Click \"Submit\"

6. **Monitor Update**:
   - Watch the \"Events\" tab
   - Status: UPDATE_IN_PROGRESS  UPDATE_COMPLETE
   - **Time estimate**: 2-5 minutes

7. **Verify Update**:
   - Go to EC2 Console  Instances
   - Check the \"Name\" tag: Should now show `MyInstance-prod`
   - Check CloudFormation Outputs tab: EC2InstanceName should show `MyInstance-prod`

---

##  Screenshots to Take

### 1. Initial Stack Creation
- **File**: `stack_create_complete.png`
- **Show**: Stack status with CREATE_COMPLETE, all resources listed

### 2. Initial Outputs
- **File**: `stack_outputs_dev.png`
- **Show**: Outputs tab with all 7 outputs visible

### 3. Stack Update Complete
- **File**: `stack_update_complete.png`
- **Show**: Stack status with UPDATE_COMPLETE in Events tab

### 4. Updated Outputs
- **File**: `stack_outputs_prod.png`
- **Show**: Outputs tab showing updated EC2InstanceName (MyInstance-prod)

---

##  Cleanup (Delete Stack)

**Important**: Always clean up resources to avoid charges!

1. **Empty S3 Bucket First**:
```bash
   aws s3 rm s3://<your-bucket-name> --recursive
```

2. **Delete Stack**:
   - CloudFormation Console  Select stack  Delete
   - Confirm deletion
   - Monitor Events tab until DELETE_COMPLETE

3. **Verify Deletion**:
   - Check that all resources are removed:
     - EC2 instances
     - RDS instances
     - S3 buckets
     - VPC resources

---

##  Troubleshooting

### Common Issues

**Issue 1: Stack creation fails due to AMI not found**
- **Solution**: Update the AMI ID in the template for your region
- Find AMI: AWS Console  EC2  AMIs  Public images  Search \"Amazon Linux 2023\"

**Issue 2: Key pair not found**
- **Solution**: Ensure you've created a key pair in the same region as your stack

**Issue 3: RDS creation takes too long**
- **Expected**: RDS instances typically take 10-15 minutes to create
- **Wait patiently**: Check Events tab for progress

**Issue 4: S3 bucket already exists**
- **Solution**: Bucket names must be globally unique
- Update stack name or add a random suffix

**Issue 5: Insufficient permissions**
- **Solution**: Ensure your IAM user/role has permissions for:
  - cloudformation:*
  - ec2:*
  - rds:*
  - s3:*
  - iam:CreateServiceLinkedRole (for RDS)

---

**Author**: Franklin Chinonso Osuji  
**Date**: November 2025  
**Purpose**: CloudFormation Test Task - 3-Tier Architecture
