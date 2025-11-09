# Installing Terraform

```Bash
# Install Terraform on AWS CLI
sudo yum install -y yum-utils shadow-utils
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/AmazonLinux/hashicorp.repo
sudo yum -y install terraform
```

# Setting up the Infrastructure

1. Set the necessary Terraform providers
```Bash
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "5.46.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"
}
```

<br>

2. Store the necessary startup scripts inside variables

<br>

3. Define the infrastructure's networking setup
```Bash
resource "aws_vpc" "demo-network" {
  cidr_block           = "172.20.0.0/16"
  enable_dns_support   = true
  enable_dns_hostnames = true

  tags = {
    Name = "demo-network"
  }
}

resource "aws_subnet" "demo-subnet" {
  vpc_id                  = aws_vpc.demo-network.id
  cidr_block              = "172.20.73.0/24"
  availability_zone       = "us-east-1a"
  map_public_ip_on_launch = true

  tags = {
    Name = "demo-subnet"
  }
}

# Create Internet Gateway and attach to VPC
resource "aws_internet_gateway" "demo-igw" {
  vpc_id = aws_vpc.demo-network.id

  tags = {
    Name = "demo-internet-gateway"
  }
}

# Create Route Table with default route to IGW
resource "aws_route_table" "demo-public-route" {
  vpc_id = aws_vpc.demo-network.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.demo-igw.id
  }

  tags = {
    Name = "public-route-table"
  }
}

# Associate Route Table with Public Subnet
resource "aws_route_table_association" "demo-public-route-association" {
  subnet_id      = aws_subnet.demo-subnet.id
  route_table_id = aws_route_table.demo-public-route.id
}
```

<br>

4. Define the security groups to assign to our EC2 instance
```Bash
resource "aws_security_group" "allow_all_https_traffic" {
  name        = "allow-all-http-traffic"
  description = "Allow all HTTPS traffic"
  vpc_id      = aws_vpc.demo-network.id
  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

<br>

5. Define our EC2 instance for our Bitwarden service
```Bash
data "http" "bitwarden-server-public-ip" {
  url = "https://icanhazip.com"
}

resource "tls_private_key" "bitwarden-server-rsa-key" {
  algorithm = "RSA"
  rsa_bits  = 4096
}

resource "local_file" "bitwarden-server-private-key" {
  content         = tls_private_key.bitwarden-server-rsa-key.private_key_pem
  filename        = "${path.module}/bitwarden-server-generated-rsa-key.pem"
  file_permission = "0400"
}

resource "aws_key_pair" "bitwarden-server-generated-key-pair" {
  key_name   = "bitwarden-server-private-key"
  public_key = tls_private_key.bitwarden-server-rsa-key.public_key_openssh
}

data "aws_ami" "ubuntu2" {
  most_recent = true
  owners      = ["099720109477"]
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]
  }
}

resource "aws_instance" "bitwarden-server" {
  ami                    = data.aws_ami.ubuntu2.id
  instance_type          = "t2.micro"
  key_name               = aws_key_pair.bitwarden-server-generated-key-pair.key_name
  vpc_security_group_ids = [aws_security_group.allow_all.id]
  subnet_id              = aws_subnet.demo-subnet.id
  private_ip             = "172.20.73.102"

#  Note: bitwarden a été installé manuellement 
#  user_data = var.bitwarden-server-startup-script

  tags = {
    Name = "Serveur bitwarden"
  }
}

# Allocate an Elastic IP (static public IP)
resource "aws_eip" "bitwarden-server-static-ip" {
  domain = "vpc"
  tags = {
    Name = "Static EIP for bitwarden Server"
  }
}

# Associate the Elastic IP with the EC2 instance
resource "aws_eip_association" "bitwarden-server-eip_assoc" {
  instance_id   = aws_instance.bitwarden-server.id
  allocation_id = aws_eip.bitwarden-server-static-ip.id
}

output "bitwarden-server-public-ip" {
  value = aws_eip.bitwarden-server-static-ip.public_ip
}
```

<br>

# Deploy the Infrastructure

1. 
```Bash
terraform init
```

<br>

```Bash
terraform apply --auto-approve
```
