
# Terraform Script to Create EC2 Instances

We will use a Terraform script to create five EC2 instances. Instance names will be `seun1`, `seun2`, `seun3`, `seun4`, and `bastion`.

To configure the virtual machines on different network layers, we will create two subnets within a VPC. One will have a private subnet, and the other will have a public subnet with an internet gateway associated with it.

## Summary of the Provided Terraform Script

### Provider Configuration
- **Provider**: AWS
- **Region**: `ca-central-1`

### VPC and Subnets
- **VPC**: A single VPC (`main_vpc`) with a CIDR block of `10.0.0.0/16`.
- **Public Subnet**: `public_subnet` with a CIDR block of `10.0.1.0/24` in availability zone `ca-central-1a`. It has `map_public_ip_on_launch` set to true, ensuring instances in this subnet receive public IP addresses.
- **Private Subnet**: `private_subnet` with a CIDR block of `10.0.2.0/24` in availability zone `ca-central-1a`.

### Internet Gateway and Route Table
- **Internet Gateway**: `main_igw`, attached to the `main_vpc`.
- **Route Table**: `public_route_table`, associated with the `public_subnet`, which routes `0.0.0.0/0` through the internet gateway (`main_igw`).

### Security Groups
- **bastion_sg**: Allows SSH (port 22) inbound traffic from anywhere.
- **SG1**: Allows inbound traffic on port 8888, SSH (port 22), and ICMP (all types and codes) from anywhere.
- **SG2**: Allows inbound traffic on port 9999 from anywhere.
- **private_sg**: Allows SSH (port 22) inbound traffic from instances in the `bastion_sg` security group.

### EC2 Instances
- **Bastion Host**: An EC2 instance named `bastion` in the `public_subnet` with `bastion_sg` security group, public IP assigned.
- **Public Instances**:
  - `seun1`: In the `public_subnet` with `SG1` security group, public IP assigned.
  - `seun2`: In the `public_subnet` with `SG1` security group, public IP assigned.
- **Private Instances**:
  - `seun3`: In the `private_subnet` with `SG2` and `private_sg` security groups.
  - `seun4`: In the `private_subnet` with `SG2` and `private_sg` security groups.

### Detailed Terraform Script
```hcl
provider "aws" {
  region = "ca-central-1"
}

# Create a single VPC
resource "aws_vpc" "main_vpc" {
  cidr_block = "10.0.0.0/16"
}

# Create Internet Gateway
resource "aws_internet_gateway" "main_igw" {
  vpc_id = aws_vpc.main_vpc.id
}

# Create Subnets
resource "aws_subnet" "public_subnet" {
  vpc_id                = aws_vpc.main_vpc.id
  cidr_block            = "10.0.1.0/24"
  availability_zone     = "ca-central-1a"
  map_public_ip_on_launch = true
}

resource "aws_subnet" "private_subnet" {
  vpc_id            = aws_vpc.main_vpc.id
  cidr_block        = "10.0.2.0/24"
  availability_zone = "ca-central-1a"
}

# Create Route Table for public subnet
resource "aws_route_table" "public_route_table" {
  vpc_id = aws_vpc.main_vpc.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main_igw.id
  }
}

# Associate Route Table with public subnet
resource "aws_route_table_association" "public_subnet_association" {
  subnet_id      = aws_subnet.public_subnet.id
  route_table_id = aws_route_table.public_route_table.id
}

# Create Security Groups
resource "aws_security_group" "bastion_sg" {
  name        = "bastion_sg"
  description = "Allow SSH inbound traffic"
  vpc_id      = aws_vpc.main_vpc.id

  ingress {
    from_port   = 22
    to_port     = 22
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

resource "aws_security_group" "SG1" {
  name        = "SG1"
  description = "Allow inbound on port 8888, SSH (22), and ICMP"
  vpc_id      = aws_vpc.main_vpc.id

  ingress {
    from_port   = 8888
    to_port     = 8888
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = -1
    to_port     = -1
    protocol    = "icmp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_security_group" "SG2" {
  name        = "SG2"
  description = "Allow inbound on port 9999"
  vpc_id      = aws_vpc.main_vpc.id

  ingress {
    from_port   = 9999
    to_port     = 9999
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

resource "aws_security_group" "private_sg" {
  name        = "private_sg"
  description = "Allow SSH traffic from bastion host"
  vpc_id      = aws_vpc.main_vpc.id

  ingress {
    from_port       = 22
    to_port         = 22
    protocol        = "tcp"
    security_groups = [aws_security_group.bastion_sg.id]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# Create Bastion Host in Public Subnet
resource "aws_instance" "bastion" {
  ami                      = "ami-0c4596ce1e7ae3e68"
  instance_type            = "t2.micro"
  key_name                 = "project-key"
  subnet_id                = aws_subnet.public_subnet.id
  vpc_security_group_ids   = [aws_security_group.bastion_sg.id]
  associate_public_ip_address = true
  tags = {
    Name = "bastion"
  }
}

# Create EC2 Instances in Public Subnet
resource "aws_instance" "seun1" {
  ami                      = "ami-0c4596ce1e7ae3e68"
  instance_type            = "t2.micro"
  key_name                 = "project-key"
  subnet_id                = aws_subnet.public_subnet.id
  vpc_security_group_ids   = [aws_security_group.SG1.id]
  associate_public_ip_address = true
  tags = {
    Name = "seun1"
  }
}

resource "aws_instance" "seun2" {
  ami                      = "ami-0c4596ce1e7ae3e68"
  instance_type            = "t2.micro"
  key_name                 = "project-key"
  subnet_id                = aws_subnet.public_subnet.id
  vpc_security_group_ids   = [aws_security_group.SG1.id]
  associate_public_ip_address = true
  tags = {
    Name = "seun2"
  }
}

# Create EC2 Instances in Private Subnet
resource "aws_instance" "seun3" {
  ami                   = "ami-0c4596ce1e7ae3e68"
  instance_type         = "t2.micro"
  key_name              = "project-key"
  subnet_id             = aws_subnet.private_subnet.id
  vpc_security_group_ids = [aws_security_group.SG2.id, aws_security_group.private_sg.id]
  tags = {
    Name = "seun3"
  }
}

resource "aws_instance" "seun

4" {
  ami                   = "ami-0c4596ce1e7ae3e68"
  instance_type         = "t2.micro"
  key_name              = "project-key"
  subnet_id             = aws_subnet.private_subnet.id
  vpc_security_group_ids = [aws_security_group.SG2.id, aws_security_group.private_sg.id]
  tags = {
    Name = "seun4"
  }
}
```

This script sets up a basic AWS infrastructure including a VPC, subnets, internet gateway, route tables, security groups, and EC2 instances. The instances are divided into public and private subnets, with appropriate security groups to manage inbound and outbound traffic. The bastion host in the public subnet allows SSH access, facilitating secure access to instances in the private subnet.

### Next Steps: Copy the Private Keys for SSH Access into the Servers
```bash
scp -i "C:\Users\balog\.ssh\project-key-aug.pem" "C:\Users\balog\.ssh\project-key-aug.pem" ubuntu@35.182.125.175:/home/ubuntu/.ssh
scp -i "C:\Users\balog\.ssh\project-key-aug.pem" "C:\Users\balog\.ssh\project-key-aug.pem" ubuntu@35.183.106.246:/home/ubuntu/.ssh
scp -i "C:\Users\balog\.ssh\project-key-aug.pem" "C:\Users\balog\.ssh\project-key-aug.pem" ubuntu@35.183.20.237:/home/ubuntu/.ssh
```

### IP Addresses of All Instances
- **Bastion**: 35.182.125.175
- **Seun1**: 35.183.106.246
- **Seun2**: 35.183.20.237
- **Seun3**: 10.0.2.141 (Private IP)
- **Seun4**: 10.0.2.226 (Private IP)

Log on to `seun1` and `seun2` and run ping/ssh commands across each server.

### From the Bastion Host, SSH into the Private Instance
```bash
ssh -i /home/ubuntu/.ssh/project-key-aug.pem ubuntu@10.0.2.141

## Analyze network traffic using packet-capturing tools like Wireshark or tcpdump
Run below commands to collect packets from the server.

```
sudo tcpdump -i enX0 -w /home/ubuntu/seun2_icmp.pcap
sudo tcpdump -i enX0 icmp -w /home/ubuntu/seun2_icmp.pcap
```

## Copy out the output files `seun2_icmp.pcap` and `seun2_icmp.pcap` for wireshark analysis:

```
scp -i "C:\Users\balog\.ssh\project-key-aug.pem" -r ubuntu@35.183.20.237:/home/ubuntu/seun2.pcap "C:\Users\balog\Desktop\"
```