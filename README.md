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

## Configuration of AWS account:

  We will use this "aws configure" in the VS code terminal, then we will take these input from console,
AWS Access Key ID [****************INOB]: "aws access ID"
AWS Secret Access Key [****************ZgAM]: "aws secret access key"
Default region name [eu-west-1]: "region"
Default output format [json]: "json"  

## Creation of EC2:

   * Terraform to create the EC2 having all necessary network ans security configuration.
       provider "aws" {
       region = "eu-west-1"
      }

* 1. VPC
resource "aws_vpc" "main_vpc" {
  cidr_block = "10.0.0.0/16"
  tags = {
    Name = "main-vpc"
  }
}

* 2. Subnet
resource "aws_subnet" "main_subnet" {
  vpc_id            = aws_vpc.main_vpc.id
  cidr_block        = "10.0.1.0/24"
  availability_zone = "eu-west-1a"
  tags = {
    Name = "main-subnet"
  }
}

* 3. Internet Gateway
resource "aws_internet_gateway" "main_igw" {
  vpc_id = aws_vpc.main_vpc.id
  tags = {
    Name = "main-igw"
  }
}

* 4. Route Table
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

* 5. Route Table Association
resource "aws_route_table_association" "main_rta" {
  subnet_id      = aws_subnet.main_subnet.id
  route_table_id = aws_route_table.main_route_table.id
}

* 6. Security Group
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

* 7. Key Pair
resource "aws_key_pair" "deployer" {
  key_name   = "my-ec2-key"
  public_key = file("~/.ssh/my-ec2-key.pub")
}


* 8. EC2 Instance
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
  * After saving the above code we will use below mentioned command;
    ** terraform plan: it will initialize a working directory containing Terraform configuration files.
    ** terraform init: creates an execution plan.
    ** terraform apply: apply the commands and create the infrastructure.

  
## Ansible Playbook to Install and Deploy Docker:


 name: Install and configure Docker on Amazon Linux 2023            

 hosts: web 

 become: true 

  tasks: 

   - name: Update all packages                                       

     dnf: 

       name: "*" 

       state: latest 

       update_cache: yes 

   - name: Install Docker                                              --> Docker will install by this command

     dnf: 

       name: docker 

       state: present 

   - name: Enable and start Docker service                             --> This command will enable and start the Docker services

     systemd: 

       name: docker 

       enabled: yes 

       state: started 

   - name: Add ec2-user to docker group                                --> this will add the ec2-user to docker group, so we can use the Docker module

     user: 

       name: ec2-user 

       groups: docker 

       append: yes 


## Creation of the Web page:

    <!DOCTYPE html> 

<html> 

<head> 

 <title>My Simple App</title> 

</head> 

<body> 

 <h1>Hello, world!</h1> 

</body> 

</html> 


## Creation of the Docker File:

FROM nginx:alpine                                                               --> We are using nginx web server page

COPY index.html /usr/share/nginx/html/index.html                                --> this will copy the index.html file to nginx server

EXPOSE 80                                                                       --> application will run port on 80

## Ansible Playbook to start the Docker and building the Docker Image


- name: Deploy simple HTML Docker container 

 hosts: web 

 become: true 

 
 vars: 

   app_name: simple-html-app 

   app_port: 3000 

   docker_image: simple-html-app 

   docker_container: simple-html-app 


 tasks: 

   - name: Ensure Docker is installed 

     package: 

       name: docker 

       state: present 

 
   - name: Start and enable Docker service 

     systemd: 

       name: docker 

       enabled: true 

       state: started 


   - name: Create app directory 

     file: 

       path: /opt/{{ app_name }} 

       state: directory 

       mode: '0755' 


   - name: Copy index.html to server 

     copy: 

       src: index.html 

       dest: /opt/{{ app_name }}/index.html 

 
   - name: Copy Dockerfile to server 

     copy: 

       src: Dockerfile 

       dest: /opt/{{ app_name }}/Dockerfile 


   - name: Build Docker image 

     docker_image: 

       name: "{{ docker_image }}" 

       source: build 

       build: 

         path: /opt/{{app_name}} 


   - name: Run Docker container 

     docker_container: 

       name: "{{ docker_container }}" 

       image: "{{ docker_image }}" 

       state: started 

       restart_policy: always 

       published_ports: 

         - "{{ app_port }}:80" 

## Automated Pipeline using GitHub Actions:

We have created a workflow file to automate the actions, whenever there is a change in index.html file. 

 * It will build a Docker image. 

 * Save the Docker image as tar file. 

 * Copy the Docker image and run the container on the EC2 machine. 





