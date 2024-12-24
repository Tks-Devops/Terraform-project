Terraform Project for AWS Infrastructure

Step 1: Install Terraform

Download and install Terraform from terraform.io.

Verify installation:

terraform --version

Step 2: Create Terraform Files

Create a new directory for Terraform:

mkdir terraform_project && cd terraform_project

Create main.tf:

nano main.tf

Paste the following configuration:

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

terraform init
terraform apply
Terraform Project for AWS Infrastructure

main.tf

provider "aws" {
  region = "us-east-1"
}

resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_subnet" "public" {
  count             = 3
  vpc_id            = aws_vpc.main.id
  cidr_block        = cidrsubnet(aws_vpc.main.cidr_block, 8, count.index)
  map_public_ip_on_launch = true
  availability_zone = element(["us-east-1a", "us-east-1b", "us-east-1c"], count.index)
}

resource "aws_subnet" "private" {
  count             = 3
  vpc_id            = aws_vpc.main.id
  cidr_block        = cidrsubnet(aws_vpc.main.cidr_block, 8, count.index + 3)
  availability_zone = element(["us-east-1a", "us-east-1b", "us-east-1c"], count.index)
}

resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.main.id
}

resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id
}

resource "aws_route" "public_route" {
  route_table_id         = aws_route_table.public.id
  destination_cidr_block = "0.0.0.0/0"
  gateway_id             = aws_internet_gateway.igw.id
}

resource "aws_route_table_association" "public_association" {
  count          = 3
  subnet_id      = aws_subnet.public[count.index].id
  route_table_id = aws_route_table.public.id
}

resource "aws_eip" "nat" {
  count = 3
}

resource "aws_nat_gateway" "nat" {
  count         = 3
  allocation_id = aws_eip.nat[count.index].id
  subnet_id     = aws_subnet.public[count.index].id
}

resource "aws_route_table" "private" {
  count  = 3
  vpc_id = aws_vpc.main.id
}

resource "aws_route" "private_route" {
  count                   = 3
  route_table_id          = aws_route_table.private[count.index].id
  destination_cidr_block  = "0.0.0.0/0"
  nat_gateway_id          = aws_nat_gateway.nat[count.index].id
}

resource "aws_route_table_association" "private_association" {
  count          = 3
  subnet_id      = aws_subnet.private[count.index].id
  route_table_id = aws_route_table.private[count.index].id
}

