# Terraform Project for AWS Infrastructure

## Step 1: Install Terraform

Download and install Terraform from [terraform.io](https://www.terraform.io/downloads.html).

Verify installation:
```bash
terraform --version

Step 2: Create Terraform Files

Create a new directory for Terraform:

mkdir terraform_project && cd terraform_project

Create main.tf:

nano main.tf

Paste the following configuration:
HCL

provider "aws" {
  region = "us-east-1"
}

resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_subnet" "public" {
  count = 3
  vpc_id            = aws_vpc.main.id
  cidr_block        = cidrsubnet(aws_vpc.main.cidr_block, 8, count.index)
  map_public_ip_on_launch = true
  availability_zone = "us-east-1a"
}

resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
}

# Add other resources as described in your task

Initialize and apply:
bash

terraform init
terraform apply
