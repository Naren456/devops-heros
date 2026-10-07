Session 18: Terraform & Infrastructure as Code

Task 1: Terraform S3 Demo

Create a Terraform project:
terraform-s3-demo/
├── main.tf
├── variables.tf
├── outputs.tf
├── provider.tf
├── terraform.tfvars
└── README.md

Create an AWS S3 bucket using Terraform.

Perform:
terraform init
terraform fmt
terraform validate
terraform plan
terraform apply
terraform show
terraform output
terraform destroy
Document the complete workflow in README.md.


Task 2: AWS Services Research

Learn about the following AWS services and create a separate README.md for each.

01. IAM - Governance
Learn and document:
What is IAM?
Users
Groups
Roles
Policies
Permissions
Least privilege
IAM best practices
Common use cases
02. EC2 - Compute
Learn and document:
What is EC2?
AMI
Instance types
Key pairs
Security Groups
EBS
Public vs private IP
Instance lifecycle
Common use cases
03. S3 - Storage
Learn and document:
What is S3?
Buckets
Objects
Storage classes
Versioning
Lifecycle policies
Encryption
Bucket policies
Common use cases
04. VPC - Networking
Learn and document:
What is VPC?
CIDR
Subnets
Route tables
Internet Gateway
NAT Gateway
Security Groups
Network ACLs
Public vs private subnet
05. DynamoDB & RDS - Database Services
Learn and document:

DynamoDB
NoSQL
Tables
Items
Attributes
Partition key
Sort key
Use cases
RDS
Relational database
Supported engines
DB instances
Security
Backups
Multi-AZ
Read replicas
Use cases
Deliverables
terraform-s3-demo/
aws-services/
├── 01-iam/
│   └── README.md
├── 02-ec2/
│   └── README.md
├── 03-s3/
│   └── README.md
├── 04-vpc/
│   └── README.md
└── 05-dynamodb-rds/
    └── README.md

