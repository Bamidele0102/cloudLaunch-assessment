CloudLaunch AWS Deployment - Semester 3 Assessment
This repository contains the configuration and documentation for deploying the CloudLaunch product on AWS, demonstrating core competencies in S3, IAM, and VPC.

Task 1: Static Website Hosting & Secure S3 Access with IAM
Objective
Host a public static website on S3 and configure secure, granular access to private storage buckets using a dedicated IAM user with a least-privilege policy.

Implementation Summary
S3 Buckets Configuration:

cloudlaunch-site-bucket: Configured for public static website hosting. A bucket policy grants public s3:GetObject permission. An index.html file was uploaded as the default document.

cloudlaunch-private-bucket: Blocked all public access. Only the designated IAM user has programmatic access to its contents.

cloudlaunch-visible-only-bucket: Blocked all public access. The IAM user can list the bucket's existence but is explicitly denied access to its contents.

CloudFront Distribution (Bonus):

A CloudFront distribution was created with an Origin Access Identity (OAI) for the cloudlaunch-site-bucket.

The S3 bucket policy was automatically updated to only allow access from the CloudFront OAI, enhancing security and enabling HTTPS.

The distribution is configured to redirect HTTP to HTTPS for secure browsing.

IAM User and Policy:

Created a user cloudlaunch-user with console access and a forced password reset on first login.

Attached a custom, granular IAM policy that strictly follows the principle of least privilege, allowing only the required actions on the specified resources.

Access Links
S3 Static Website Endpoint: http://cloudlaunch-site-bucket.s3-website-us-east-1.amazonaws.com

CloudFront Distribution (HTTPS): https://d3q8vq8sukzzcloudfront.net (Example URL)

IAM User Policy (JSON)
The following custom policy is attached to the cloudlaunch-user:

json
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
Task 2: Secure VPC Network Design
Objective
Design and implement a secure, multi-tiered Virtual Private Cloud (VPC) network architecture to provide a logically isolated environment for future application deployment.

Implementation Summary
VPC Foundation:

Created the VPC cloudlaunch-vpc with the CIDR block 10.0.0.0/16.

Created and attached an Internet Gateway (cloudlaunch-igw) to facilitate internet access for public resources.

Subnet Architecture:

Created three subnets with exact, specified CIDR blocks for clear separation of layers:

Public Subnet (10.0.1.0/24): For public-facing resources like Load Balancers. Associated with a public route table.

Application Subnet (10.0.2.0/24): For private application servers (e.g., EC2 instances). Associated with a private route table.

Database Subnet (10.0.3.0/28): For private database instances (e.g., RDS). Associated with a separate private route table for maximum isolation.

Routing:

Public Route Table (cloudlaunch-public-rt): Contains a route sending 0.0.0.0/0 traffic to the Internet Gateway.

Private Route Tables (cloudlaunch-app-rt, cloudlaunch-db-rt): Have no route to the internet, ensuring their resources remain private.

Security Groups (Virtual Firewalls):

cloudlaunch-app-sg: Configured to allow HTTP (port 80) traffic only from within the VPC (10.0.0.0/16).

cloudlaunch-db-sg: Configured to allow MySQL (port 3306) traffic only from the Application Subnet (10.0.2.0/24), enforcing strict database access controls.

IAM Integration:

The cloudlaunch-user policy was updated with a VPCReadOnly statement, granting the ability to list and describe VPC components for viewing and troubleshooting without modification rights.

Submission Details
AWS Account ID: 123456789012

IAM User Name: cloudlaunch-user

AWS Console Sign-In URL: https://123456789012.signin.aws.amazon.com/console

User Password: Enforced change on first login. Temporary password shared via secured .csv file.
