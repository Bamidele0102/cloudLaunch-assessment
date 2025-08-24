# CloudLaunch AWS Deployment – Semester 3 Assessment

This repository documents the deployment of the CloudLaunch product on AWS, showcasing skills in S3, IAM, and VPC.

---

## Table of Contents

- [CloudLaunch AWS Deployment – Semester 3 Assessment](#cloudlaunch-aws-deployment--semester-3-assessment)
  - [Table of Contents](#table-of-contents)
  - [Overview](#overview)
  - [Task 1: S3 Static Website \& IAM Access](#task-1-s3-static-website--iam-access)
    - [Objectives](#objectives)
    - [Implementation](#implementation)
      - [S3 Buckets](#s3-buckets)
      - [CloudFront Distribution (Bonus)](#cloudfront-distribution-bonus)
      - [IAM User \& Policy](#iam-user--policy)
    - [Access Links](#access-links)
    - [IAM Policy](#iam-policy)
  - [Task 2: Secure VPC Network Design](#task-2-secure-vpc-network-design)
    - [Objectives](#objectives-1)
    - [Implementation](#implementation-1)
      - [VPC Foundation](#vpc-foundation)
      - [Subnet Architecture](#subnet-architecture)
      - [Routing](#routing)
      - [Security Groups](#security-groups)
      - [IAM Integration](#iam-integration)

---

## Overview

This project demonstrates AWS deployment best practices, including secure S3 hosting, granular IAM permissions, and multi-tier VPC architecture.

---

## Task 1: S3 Static Website & IAM Access

### Objectives

- Host a public static website on S3.
- Secure private S3 buckets with least-privilege IAM policies.

### Implementation

#### S3 Buckets

- **cloudlaunch-site-bucket**: Public static website hosting with an index.html. Public `s3:GetObject` permission via bucket policy.
- **cloudlaunch-private-bucket**: All public access blocked. Only a dedicated IAM user can access contents programmatically.
- **cloudlaunch-visible-only-bucket**: All public access blocked. IAM user can list the bucket but cannot access objects.

#### CloudFront Distribution (Bonus)

- Configured with Origin Access Identity (OAI) for `cloudlaunch-site-bucket`.
- S3 bucket policy restricts access to CloudFront OAI.
- HTTP redirected to HTTPS.

#### IAM User & Policy

- Created `cloudlaunch-user` with console access and forced password reset.
- Attached custom IAM policy for least-privilege access.

### Access Links

- **S3 Website**: [http://cloudlaunch-site-bucket-01.s3-website-eu-west-1.amazonaws.com/](http://cloudlaunch-site-bucket-01.s3-website-eu-west-1.amazonaws.com/)

### IAM Policy

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "ListAllBuckets",
            "Effect": "Allow",
            "Action": "s3:ListAllMyBuckets",
            "Resource": "*"
        },
        {
            "Sid": "ListObjectsInBuckets",
            "Effect": "Allow",
            "Action": "s3:ListBucket",
            "Resource": [
                "arn:aws:s3:::cloudlaunch-site-bucket",
                "arn:aws:s3:::cloudlaunch-private-bucket",
                "arn:aws:s3:::cloudlaunch-visible-only-bucket"
            ]
        },
        {
            "Sid": "ReadWritePrivateBucket",
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:PutObject"
            ],
            "Resource": "arn:aws:s3:::cloudlaunch-private-bucket/*"
        },
        {
            "Sid": "ReadPublicSiteBucket",
            "Effect": "Allow",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::cloudlaunch-site-bucket/*"
        },
        {
            "Sid": "DenyVisibleBucketAccess",
            "Effect": "Deny",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::cloudlaunch-visible-only-bucket/*"
        },
        {
            "Sid": "VPCReadOnly",
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeVpcs",
                "ec2:DescribeSubnets",
                "ec2:DescribeRouteTables",
                "ec2:DescribeSecurityGroups",
                "ec2:DescribeInternetGateways"
            ],
            "Resource": "*"
        }
    ]
}
```

---

## Task 2: Secure VPC Network Design

### Objectives

- Design a secure, multi-tier VPC for future application deployment.

### Implementation

#### VPC Foundation

- **VPC Name**: `cloudlaunch-vpc`
- **CIDR Block**: `10.0.0.0/16`
- **Internet Gateway**: `cloudlaunch-igw` attached for public access.

#### Subnet Architecture

- **Public Subnet**: `10.0.1.0/24` (for load balancers, public resources)
- **Application Subnet**: `10.0.2.0/24` (for EC2 application servers)
- **Database Subnet**: `10.0.3.0/28` (for RDS, private databases)

#### Routing

- **Public Route Table**: `cloudlaunch-public-rt` routes `0.0.0.0/0` to Internet Gateway.
- **Private Route Tables**: `cloudlaunch-app-rt`, `cloudlaunch-db-rt` have no internet routes.

#### Security Groups

- **cloudlaunch-app-sg**: Allows HTTP (port 80) only from within VPC (`10.0.0.0/16`).
- **cloudlaunch-db-sg**: Allows MySQL (port 3306) only from Application Subnet (`10.0.2.0/24`).

#### IAM Integration

- `cloudlaunch-user` policy includes VPC read-only permissions for viewing/troubleshooting.
