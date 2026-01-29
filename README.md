# Terraform AWS VPC Public Network Module

## 🧩 BIG PICTURE

This Terraform module builds:

```
Internet 🌍
   ↓
Internet Gateway
   ↓
Route Table (0.0.0.0/0 → IGW)
   ↓
Public Subnet
   ↓
VPC (your private network in AWS)
```

So you're creating a network that can reach the internet.

## 1️⃣ Variables (inputs to your module)

These are like settings you pass to the network.

| Variable | Meaning |
|----------|---------|
| vpc_name | Name tag of the VPC |
| vpc_cidr | IP range of entire network |
| public_subnet_cidr | Smaller IP range inside VPC |
| availability_zone | Which AWS data center |

**Example:**
```hcl
vpc_cidr = "10.0.0.0/16"
```

**Means:**
👉 Your network can have 65,536 IP addresses

## 2️⃣ aws_vpc — Create the Network

```hcl
resource "aws_vpc" "this" {
  cidr_block = var.vpc_cidr
  
  tags = {
    Name = var.vpc_name
  }
}
```

🧠 **What happens:**

- AWS creates a private network
- Nothing can access internet yet
- Like building a city with no roads outside

## 3️⃣ aws_subnet — Create Public Subnet

```hcl
resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.this.id
  cidr_block              = var.public_subnet_cidr
  availability_zone       = var.availability_zone
  map_public_ip_on_launch = true
  
  tags = {
    Name = "${var.vpc_name}-public-subnet"
  }
}
```

🧠 **What happens:**

- A section of the VPC is created
- This subnet can hold servers (EC2)
- `map_public_ip_on_launch = true` means:
  👉 Servers get public IPs automatically

**Think:**
🏢 VPC = City  
🏠 Subnet = Neighborhood

## 4️⃣ aws_internet_gateway — The Door to Internet

```hcl
resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.this.id
  
  tags = {
    Name = "${var.vpc_name}-igw"
  }
}
```

🧠 **What happens:**

- A gateway device is attached to your VPC
- This is the exit/entry point to the internet
- Without IGW → VPC is isolated forever 🚫🌍

## 5️⃣ aws_route_table — Traffic Rules

```hcl
resource "aws_route_table" "public_rt" {
  vpc_id = aws_vpc.this.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }
  
  tags = {
    Name = "${var.vpc_name}-public-rt"
  }
}
```

🧠 **This is the MOST important part.**

It says:

> "If traffic is going ANYWHERE outside the VPC  
> (0.0.0.0/0 = the whole internet)  
> Send it to the Internet Gateway."

Without this → even with IGW, no traffic flows ❌

## 6️⃣ aws_route_table_association — Connect Subnet to Route Table

```hcl
resource "aws_route_table_association" "public_assoc" {
  subnet_id      = aws_subnet.public.id
  route_table_id = aws_route_table.public_rt.id
}
```

🧠 **What happens:**

- You apply the internet route rules to this subnet
- Now this subnet becomes PUBLIC

## 7️⃣ Outputs — What Terraform Shows After Creation

```hcl
output "vpc_id" {
  description = "The ID of the VPC"
  value       = aws_vpc.this.id
}

output "public_subnet_id" {
  description = "The ID of the public subnet"
  value       = aws_subnet.public.id
}

output "internet_gateway_id" {
  description = "The ID of the Internet Gateway"
  value       = aws_internet_gateway.igw.id
}

output "public_route_table_id" {
  description = "The ID of the public route table"
  value       = aws_route_table.public_rt.id
}
```

These are values you can use later:

| Output | Why useful |
|--------|------------|
| vpc_id | Attach EC2, RDS, etc |
| public_subnet_id | Launch servers |
| internet_gateway_id | Network troubleshooting |

## 🔥 FINAL RESULT

You built a network where:

| Component | Purpose |
|-----------|---------|
| VPC | Your private cloud network |
| Subnet | Where servers live |
| IGW | Internet connection |
| Route Table | Traffic rules |
| Association | Makes subnet public |

## 🚀 When you launch an EC2 here:

1. It gets private IP from subnet
2. Gets public IP automatically
3. Route table sends traffic → IGW
4. Server can reach internet 🌍

## 📋 Complete Module Usage

### Variables

```hcl
variable "vpc_name" {
  description = "Name tag of the VPC"
  type        = string
}

variable "vpc_cidr" {
  description = "IP range of entire network"
  type        = string
}

variable "public_subnet_cidr" {
  description = "Smaller IP range inside VPC"
  type        = string
}

variable "availability_zone" {
  description = "Which AWS data center"
  type        = string
}

variable "student_name" {
  description = "Student's GitHub username"
  type        = string
}
```

### Provider Configuration

```hcl
terraform {
  required_version = ">= 1.9.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.aws_region

  default_tags {
    tags = {
      Environment = "Learning"
      ManagedBy   = "Terraform"
      Student     = var.student_name
      AutoTeardown = "8h"
    }
  }
}
```

### Example Usage

```hcl
module "vpc" {
  source = "./modules/terraform-aws-vpc-public"
  
  vpc_name             = "my-app-vpc"
  vpc_cidr             = "10.0.0.0/16"
  public_subnet_cidr   = "10.0.1.0/24"
  availability_zone    = "us-east-1a"
  student_name         = "your-username"
}

# Launch EC2 in the public subnet
resource "aws_instance" "web" {
  ami           = "ami-12345678"
  instance_type = "t3.micro"
  subnet_id     = module.vpc.public_subnet_id
  
  tags = {
    Name = "web-server"
  }
}
```

## 🎯 This is Cloud Networking 101

Interviewers LOVE asking this.

## 🔄 Next step to become 🔥 DevOps:

👉 Add private subnet + NAT Gateway

**Say "teach NAT" and we go deeper.**

## 🧠 Key Concepts to Remember

1. **VPC = Private Cloud Network**
   - Isolated environment in AWS
   - Controls IP address ranges

2. **Subnet = Network Section**
   - Divides VPC into smaller networks
   - Public vs Private determines internet access

3. **Internet Gateway = Internet Door**
   - Connects VPC to internet
   - Two-way communication

4. **Route Table = GPS for Traffic**
   - Tells packets where to go
   - 0.0.0.0/0 = "send to internet"

5. **Association = Applying Rules**
   - Links subnet to route table
   - Makes subnet behavior official

## ⚠️ Common Mistakes to Avoid

1. **Forgetting Route Table Association**
   - Subnet won't know how to reach internet

2. **Wrong CIDR Blocks**
   - Overlapping IP ranges cause conflicts

3. **Missing map_public_ip_on_launch**
   - EC2 instances won't get public IPs

4. **Internet Gateway Not Attached**
   - VPC remains isolated

## 🔍 Troubleshooting Commands

```bash
# Check VPC details
aws ec2 describe-vpcs --vpc-ids <vpc-id>

# Check subnet details
aws ec2 describe-subnets --subnet-ids <subnet-id>

# Check route table
aws ec2 describe-route-tables --route-table-ids <rt-id>

# Test internet connectivity from EC2
curl -I https://www.google.com
```

This module creates the foundation for all AWS networking. Master this, and you're ready for advanced cloud architecture!
