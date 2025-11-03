# Network-CA Project Title:

Deployment of an EC2 Instance on AWS Cloud using Terraform and running a Docker application using GitHub Actions. 

## Overview:

This project describes the steps involved in creation of an AWS EC2 instance with the help of terraform and containerization of a web application in docker. And, running that docker file via GitHub action pipeline. 

## Features:
   * Provisioning of an AWS infrastrucure using Terraform.
      * It will create an EC2 instance having all the network conguration.
   * Using Ansible Playbook to Deploy Docker.
   * Creating automated Pipeline with GitHub Actions.
     * Pipeline will run automatically whenever there is a change in the index.html file.

## Tools used in this project:
   * Amazon Web Services (AWS): AWS Cloud provider is well known these days for its services and support. It has many services such as EC2, VPC, IAM, S3 and many more.
   * Terraform: Terraform is an infrastructure as code tool that lets you build, change, and version infrastructure safely and efficiently.
   * Docker: Docker is an open platform to develop and manage any application.
   * GitHub Actions: GitHub Action: GitHub Action is a Continuous Integration and Continuous Delivery (CI/CD) tool through which we can automate the pipeline whenever there is a change in our web application.

## Creation of EC2:

   * Terraform to create the EC2 having all necessary network ans security configuration.
       provider "aws" {
       region = "eu-west-1"
      }

# 1. VPC
resource "aws_vpc" "main_vpc" {
  cidr_block = "10.0.0.0/16"
  tags = {
    Name = "main-vpc"
  }
}

# 2. Subnet
resource "aws_subnet" "main_subnet" {
  vpc_id            = aws_vpc.main_vpc.id
  cidr_block        = "10.0.1.0/24"
  availability_zone = "eu-west-1a"
  tags = {
    Name = "main-subnet"
  }
}

# 3. Internet Gateway
resource "aws_internet_gateway" "main_igw" {
  vpc_id = aws_vpc.main_vpc.id
  tags = {
    Name = "main-igw"
  }
}

# 4. Route Table
resource "aws_route_table" "main_route_table" {
  vpc_id = aws_vpc.main_vpc.id
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main_igw.id
  }
  tags = {
    Name = "main-route-table"
  }
}

# 5. Route Table Association
resource "aws_route_table_association" "main_rta" {
  subnet_id      = aws_subnet.main_subnet.id
  route_table_id = aws_route_table.main_route_table.id
}

# 6. Security Group
resource "aws_security_group" "web_sg" {
  name        = "web-sg"
  description = "Allow SSH and HTTP"
  vpc_id      = aws_vpc.main_vpc.id

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 3000
    to_port     = 3000
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "web-sg"
  }
}

# 7. Key Pair
resource "aws_key_pair" "deployer" {
  key_name   = "my-ec2-key"
  public_key = file("~/.ssh/my-ec2-key.pub")
}


# 8. EC2 Instance
resource "aws_instance" "web-server" {
  ami           = "ami-0025245f3ca0bcc82" # Amazon Linux 2 AMI
  instance_type = "t2.micro"
  subnet_id     = aws_subnet.main_subnet.id
  key_name      = aws_key_pair.deployer.key_name
  vpc_security_group_ids = [aws_security_group.web_sg.id]
  associate_public_ip_address = "true"


  tags = {
    Name = "web-server"
  }
}

## Ansible Playbook to Install and Deploy Docker:

 

 # name: Install and configure Docker on Amazon Linux 2023 

 hosts: web 

 become: true 

 

  tasks: 

   - name: Update all packages 

     dnf: 

       name: "*" 

       state: latest 

       update_cache: yes 

 

   - name: Install Docker 

     dnf: 

       name: docker 

       state: present 

 

   - name: Enable and start Docker service 

     systemd: 

       name: docker 

       enabled: yes 

       state: started 

 

   - name: Add ec2-user to docker group 

     user: 

       name: ec2-user 

       groups: docker 

       append: yes 








