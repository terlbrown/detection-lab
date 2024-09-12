# Battle-Ready: How Detection Labs Shape Elite Cyber Defenders

This guide introduces the Terraform configurations used to build a cloud-based penetration testing lab. Each block of code is explained to help participants understand how infrastructure is created in AWS using Terraform.

The Terraform configurations in this repository define a cloud-based penetration testing lab environment. This lab is designed for cybersecurity professionals to test, research, and hone their defensive and offensive skills. The configurations are based on [**Chapter 6** of the GitHub project 'Building and Automating Penetration Testing Labs in the Cloud'](https://github.com/PacktPublishing/Building-and-Automating-Penetration-Testing-Labs-in-the-Cloud/tree/main/ch06).

This project uses **AWS** as the cloud provider and leverages Terraform to automate the creation of various resources such as **VPCs**, **subnets**, **EC2 instances**, and **security groups**. This lab consists of **target environments** (victim) and **attacker environments** (offensive tools), providing a realistic training ground for cybersecurity scenarios.

### Prerequisites

Before applying the Terraform code, make sure to have the following:

- AWS account
- Terraform CLI installed (`>=1.3.9`)
- SSH key pairs for EC2 access

---

## Introduction to the Terraform Configurations

The lab environment simulates real-world attack scenarios and defensive exercises using **AWS** as the cloud provider. Key infrastructure components include Virtual Private Clouds (VPCs), subnets, EC2 instances, and security groups. The lab is divided into two main parts:
1. **Target Environment** (victim machines).
2. **Attacker Environment** (offensive tools).

---

## Detailed Breakdown of the Terraform Configuration

### 1. **Terraform Version Configuration (`versions.tf`)**

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.0"
    }
  }
}
```
This block defines the required version of the AWS provider. It ensures that all users are using a compatible version of the AWS provider (version 4.0 or higher). This is important to avoid compatibility issues between Terraform and AWS.

---

### 2. **Provider Configuration (`provider.tf`)**

```hcl
provider "aws" {
  region = "us-east-1"
}
```
The provider block tells Terraform to interact with AWS resources. The `region` field specifies where the AWS resources will be deployed (in this case, the `us-east-1` region).

---

### 3. **VPC and Network Configuration (`network_01.tf`)**

#### a. **VPC Configuration**

```hcl
resource "aws_vpc" "primary_vpc" {
  cidr_block = "10.0.0.0/16"
  tags = {
    Name = "PrimaryVPC"
  }
}
```
This creates a Virtual Private Cloud (VPC) in AWS with the specified CIDR block (`10.0.0.0/16`). A VPC allows you to launch AWS resources in a virtual network isolated from the public internet.

#### b. **Public Subnet Configuration**

```hcl
resource "aws_subnet" "public_subnet" {
  vpc_id = aws_vpc.primary_vpc.id
  cidr_block = "10.0.1.0/24"
  availability_zone = "us-east-1a"
  map_public_ip_on_launch = true
  tags = {
    Name = "PublicSubnet"
  }
}
```
This block creates a public subnet within the VPC. Instances launched in this subnet will be assigned public IPs, allowing them to communicate with the internet.

#### c. **Internet Gateway**

```hcl
resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.primary_vpc.id
  tags = {
    Name = "InternetGateway"
  }
}
```
The internet gateway allows resources in the public subnet to access the internet. It is attached to the VPC and serves as a doorway for internet traffic.

#### d. **Route Table for Public Subnet**

```hcl
resource "aws_route_table" "public_route_table" {
  vpc_id = aws_vpc.primary_vpc.id
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }
  tags = {
    Name = "PublicRouteTable"
  }
}
```
This block sets up routing rules for the public subnet. It directs all outgoing traffic (`0.0.0.0/0`, meaning all IP addresses) to the internet gateway, allowing instances in the public subnet to access the internet.

---

### 4. **Private Subnet Configuration (`network_02.tf`)**

#### a. **Private Subnet**

```hcl
resource "aws_subnet" "private_subnet" {
  vpc_id = aws_vpc.primary_vpc.id
  cidr_block = "10.0.2.0/24"
  availability_zone = "us-east-1a"
  tags = {
    Name = "PrivateSubnet"
  }
}
```
This creates a private subnet within the VPC. Instances launched in this subnet will not be assigned public IPs and are not directly accessible from the internet, which is important for security-sensitive resources.

#### b. **Security Group for Private Subnet**

```hcl
resource "aws_security_group" "private_security_group" {
  vpc_id = aws_vpc.primary_vpc.id
  description = "Allow internal traffic"

  ingress {
    from_port   = 0
    to_port     = 65535
    protocol    = "tcp"
    cidr_blocks = ["10.0.2.0/24"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
  tags = {
    Name = "PrivateSecurityGroup"
  }
}
```
The security group is a set of firewall rules for the private subnet. It allows internal traffic between resources in the private subnet and restricts access from the outside.

---

### 5. **Attacker Environment (Kali Linux VM)**

#### a. **Kali Linux EC2 Instance**

```hcl
resource "aws_instance" "vm_kali" {
  ami           = data.aws_ami.kali_linux.id
  instance_type = "t3.medium"
  tags = {
    Name = "vm-kali"
  }
  associate_public_ip_address = true
  subnet_id                   = aws_subnet.public_subnet_attacker.id
  vpc_security_group_ids      = [aws_security_group.sg_attacker.id]

  user_data = <<-EOF
    #!/bin/bash
    sudo apt update
    sudo apt install -y kali-tools-post-exploitation
    echo "admin" > /root/usernames.txt
    echo "password" > /root/passwords.txt
    systemctl restart ssh
  EOF
}
```
This block creates a **Kali Linux** virtual machine. This machine is used as the **attacker** in your lab. The user data block installs post-exploitation tools and sets up SSH access to simulate an adversary attempting to exploit systems in the network.

---

### 6. **Target Environment (Victim VMs)**

#### a. **Victim EC2 Instance 1**

```hcl
resource "aws_instance" "vm_target" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t2.micro"
  associate_public_ip_address = true
  subnet_id                   = aws_subnet.public_subnet_target.id
  vpc_security_group_ids = [
    aws_security_group.sg_target.id
  ]
  user_data = <<-EOF
    #!/bin/bash
    echo "FLAG #1" > /root/flag.txt
    NEW_USER=adminuser
    adduser --disabled-password --gecos "" $NEW_USER
    echo "$NEW_USER:password" | chpasswd
    systemctl restart ssh
  EOF
}
```
This creates a victim EC2 instance using an Ubuntu image. The instance is configured with a user (`adminuser`) and a "flag" file that the attacker can try to retrieve. The victim simulates a vulnerable system within the lab.

#### b. **Victim EC2 Instance 2**

```hcl
resource "aws_instance" "vm_target_02" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t2.micro"
  associate_public_ip_address = true
  subnet_id                   = aws_subnet.public_subnet_target.id
  vpc_security_group_ids = [
    aws_security_group.sg_target_02.id
  ]
  user_data = <<-EOF
    #!/bin/bash
    echo "FLAG #2" > /root/flag.txt
    NEW_USER=adminuser2
    adduser --disabled-password --gecos "" $NEW_USER
    echo "$NEW_USER:password" | chpasswd
    systemctl restart ssh
  EOF
}
```
This block sets up a second victim machine with similar configurations, simulating another vulnerable system. Both victim instances are targets for the attacker to exploit.

---

### 7. **VPC Peering Configuration**

```hcl
resource "aws_vpc_peering_connection" "peering_connection" {
  peer_vpc_id = aws_vpc.vpc_attacker.id
  vpc_id      = aws_vpc.vpc_target.id
  auto_accept = true
}
```
This block creates a peering connection between the **attacker VPC** and the **target VPC**, allowing traffic between the two networks. This is crucial for simulating **lateral movement** within the lab.

---

### Conclusion

These Terraform configurations automate the creation of a complex, multi-environment penetration testing lab. By using **infrastructure as code**, this setup can be deployed quickly and repeatedly, allowing for continuous testing and skill refinement.

---
