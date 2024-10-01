## Security group - All traffic
#### Install Terraform

sudo yum install -y yum-utils shadow-utils

sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/AmazonLinux/hashicorp.repo

sudo yum -y install terraform

#### Version

terraform -v

#### Install aws cli   (not to do this if you have taken linux in instance)

``` yml

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"

unzip awscliv2.zip

sudo ./aws/install

```
### aws configure

#### Make directory

mkdir /project

cd /project

#### Vim file 
#### Naming extension for terraform file is tf
#### Create an user for access key and secret key
#### The region should be of the ami created

vim provider.tf

``` tf   // do this is our aws is not configure in server

provider "aws" {
  region     = "us-west-2"
  access_key = "my-access-key"     #add the keys
  secret_key = "my-secret-key"
}

```

#### Terminal

l.

yum install tree -y

tree -a  (*To check the generated file*)

terraform init

#### Inside vim file 
#### my ec2 code
#### copy ami from other other resources on a different region
#### Give the name of your instance
#### Name of the key should be of new region 
#### Create new key pair from key pair option

``` tf

resource "aws_instance" "this" {
  ami                     = "ami-0dcc1e21636832c5d"
  instance_type           = "t2.micro"
  key_name                = "terraform-key"      # *Give name of the key pair*
  tags = {
    Name = "HelloWorld"
  }
}

```

#### Terraform init

terraform init

terraform validate (*The configuration is valid*)

terraform plan

terraform apply 

#### Connect the new isntance on a new terminal 

#### Security group (SSH, All traffic)

yum install httpd -y

#### Old terminal 
cd /project

aws configure

vim provider.tf (*Backspace the first para*)

terraform destroy

ll
(*After ll you will get a file that wasn't created by you, do cat filename*)
cat terraform.tstate

#### Manage tag new instance

manage tag >> name

terraform fmt

cat provider.tf

terraform apply 

terraform destroy

#### Old terminal 

mkdir /mahek
cd /project
cd provider.tf /mahek
cd /mahek
mv provider.tf my-resource.tf 
vim my-resource.tf

resource "aws_instance" "this" {
  ami                     = "ami-0dcc1e21636832c5d"
  instance_type           = "t2.micro"
  availability_zone       = "us-east-1a"
  key_name                = "terraform-key"      # *Give name of the key pair*
  
}

#### To create an instance from terminal, add security groups, attach abs and volume.
#### Configure aws
#### Create a new key pair in the new region where you want to create a new instance
#### Terraform init, fmt (format), validate, plan, apply

vim security.tf

``` tf


provider "aws" {
  region = "ap-northeast-2"
}

resource "aws_security_group" "my_security_group" {
  name        = "my-sg1"
  description = "Allow SSH and HTTP traffic"

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 80
    to_port     = 80
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

resource "aws_instance" "my_instance" {
  ami                    = "ami-06f73fc34ddfd65c2" (# *Replace with a valid AMI ID for your region*)
  instance_type          = "t2.micro"
  key_name               = "seoul-key"
  vpc_security_group_ids = [aws_security_group.my_security_group.name]

  tags = {
    Name = "MyInstance"
  }
}

resource "aws_ebs_volume" "example" {
  availability_zone = "ap-northeast-2a"
  size              = 40

  tags = {
    Name = "HelloWorld"
  }
}

resource "aws_volume_attachment" "ebs_att" {
  device_name = "/dev/sdh"
  volume_id   = aws_ebs_volume.example.id
  instance_id = aws_instance.my_instance.id     (*Name of the instance created*) // remove this bracket one in while performing
}

```

terraform init 

terraform fmt 

terraform validate 

terraform plan

terraform apply 

#### Go check on aws ,open your instance > storage > volume > open your attached volume.

#### Create vpc and subnet

vim vpc.tf

``` tf
resource "aws_vpc" "main" {

  cidr_block       = "20.0.0.0/16"

  instance_tenancy = "default"
 
  tags = {

    Name = "vpc"

  }

}
 
resource "aws_subnet" "public" {

  vpc_id     = aws_vpc.main.id

  cidr_block = "20.0.0.0/24"

  tags = {

    Name = "vpc-public"

  }

}
 
resource "aws_subnet" "private" {

  vpc_id     = aws_vpc.main.id

  cidr_block = "20.0.1.0/24"

  tags = {

    Name = "vpc-private"

  }

}
 
 
resource "aws_internet_gateway" "gw" {

  vpc_id = aws_vpc.main.id
 
  tags = {

    Name = "internet-gateway"

  }

}
 
resource "aws_route_table" "vpc-route" {

  vpc_id = aws_vpc.main.id

  route {

        cidr_block = "20.0.0.0/16"

        gateway_id = "local"

        }

  route {

    cidr_block = "0.0.0.0/0"

    gateway_id = aws_internet_gateway.gw.id

  }
 
 
  tags = {

    Name = "public"

  }

}
 
resource "aws_route_table_association" "a" {

  subnet_id      = aws_subnet.public.id

  route_table_id = aws_route_table.vpc-route.id

}
 
 
resource "aws_eip" "nat_eip" {

  vpc=true
 
}
 
 
resource "aws_nat_gateway" "example" {

  allocation_id = aws_eip.nat_eip.id

  subnet_id     = aws_subnet.public.id
 
  tags = {

    Name = "gw NAT"

  }
 
  connectivity_type = "public"

}
 
resource "aws_route_table" "private-rout" {

  vpc_id = aws_vpc.main.id
 
  route {

    cidr_block = "0.0.0.0/0"

    nat_gateway_id = aws_nat_gateway.example.id

  }
 
  tags = {

    Name = "private-route-table"

  }

}
 
 
resource "aws_route_table_association" "association-1" {

  subnet_id      = aws_subnet.private.id

  route_table_id = aws_route_table.private-rout.id

}
 
```
