# How to Use Terraform

[Terraform](https://www.terraform.io/) is used to provide infrastructure as code. AWS is one of the supported resource providers. With a set of terraform files, it's possible to create complex infrastructures, keep track of changes, and destroy the resources again.

In this Guide, I focus on using Terraform to provision AWS infrastructures.

## Installation

In order to work with Terraform and be able to deploy configurations to the cloud, a Linux system is recommended. This guide focuses on Ubuntu. If you are under Windows, you can use a [WSL Ubuntu installation](https://github.com/MichaelBreitung/Development-How-Tos/blob/master/How-To-Setup-WSL-Dev-Env.md).

### Prerequisites

- Ubuntu Installation - e.g. [WSL Ubuntu installation](https://github.com/MichaelBreitung/Development-How-Tos/blob/master/How-To-Setup-WSL-Dev-Env.md)
- [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) has to be installed and [configured](https://aws.amazon.com/getting-started/guides/setup-environment/module-three/)

### Workflow

1. Install unzip:
   ````
   sudo apt install unzip
   ````

3. Install the latest version of terraform - get proper download link from [here](https://releases.hashicorp.com/terraform/). As an example, below is the command to download version 1.4.6:

   ````
   wget https://releases.hashicorp.com/terraform/1.4.6/terraform_1.4.6_linux_amd64.zip
   ````

4. Unzip the downloaded package:

   ````
   unzip terraform_1.4.6_linux_amd64.zip
   ````

5. Move the executable into the */usr/local/bin* folder:
   ````
   sudo mv terraform /usr/local/bin/
   ````

6. Test Terraform:
   ````
   terraform --version
   ````

7. If you are using Visual Studio Code as your IDE, install the [Terraform extension](https://marketplace.visualstudio.com/items?itemName=HashiCorp.terraform).

## Commands

Here are the basic commands you'll use to validate, initialize, plan, provision, and destroy an infrastructure:

````
terraform validate
terraform init
terraform plan -out plan
terraform apply "plan"
terraform destroy
````

## Terraform Files

Below is the list of files you might find inside a terraform repository. Not all of them are mandatory, but it's recommended to provide some separation of concerns by using different files:

- **main.tf** - In the root of your terraform configuration and in each module you define, you'll have to provide a main.tf containing the configuration for your project or module

- **backend.tf** - In the root of your terraform configuration, provide a backend.tf file to specify where the terraform state shall be stored. Here's an example for an AWS S3 bucket available in the EU central region:

  ````
  terraform {
    backend "s3" {
      region = "eu-central-1"
      bucket = "your-bucket-name"
    }
  }
  ````

  Note that you will also have to provide a *key* in above definition. But it's recommended to rather use a [partial configuration](https://developer.hashicorp.com/terraform/language/settings/backends/configuration#partial-configuration) here and supply the key as parameter when calling *terraform init*:

  ````
  terraform init -backend-config="key=terraform/{name of statefile}"
  ````

  This way, you don't have to fear that you accidentally overwrite the state of a production infrastructure. 

- **provider.tf** - Use this file to specify the provider. For AWS:

  ````
  provider "aws" {
    region = var.region
  }
  ````

- **versions.tf** - Specify the required versions for the provider and for terraform. This is important, if you use some of the newer features that are regularly added. Here's an example for the latest available versions from the date this guide was created:

  ````
  terraform {
    required_providers {
      aws = {
        source  = "hashicorp/aws"
        version = "~> 4.62.0"
      }
    }
    required_version = ">= 1.4.6"
  }
  ````

- **variables.tf**  - Define any input variables for your infrastructure here. Where it's appropriate, provide defaults. Use clear naming and if necessary, provide documentation in form of comments. Example:

  ````
  # Every resource within this infrastructure will get a name that is prefixed using this value. In 
  # addition to that, a Tag named "App" will be created with this prefix as a value. 
  variable "resource_name_prefix" {
    default = "dev-my-app"
  }
  ````

- **outputs.tf** - This file is mostly helpful inside of modules, to define output variables that can in turn be consumed by other modules as input variables. Here's an example for the address of a Redis store, whose resource is named *ec_cluster* inside the modules *main.tf*:

  ````
  output "ec_cluster_public_url" {
    value = aws_elasticache_cluster.ec_cluster.cache_nodes[0].address
  }
  ````

- **development.tfvars.json** and **production.tfvars.json** - Those are not standardized files, but they can be used to provide values for the input variables of your infrastructure. Here's an example, defining a different prefix for development and production:

  ````
  # Development
  {
    "resource_name_prefix": "dev-my-app"
  }
  
  # Production
  {
    "resource_name_prefix": "prod-my-app"
  }
  ````

  You can provide those files as input to the *terraform plan* and *terraform apply* commands:
  ````
  # Development
  terraform plan -var-file=development.tfvars.json
  
  # Production
  terraform plan -var-file=production.tfvars.json
  ````

## Modules

To keep your code organized, try to split it into modules. Providing proper input and output variables, make those modules reusable. Inside your root folder, create a modules sub-folder, and inside of it create a folder for each module you want to create. 

Here's an example for a *Security* and a *Redis* module and how they can be connected.

### Security Module (modules/security)

#### main.tf

Note that the *vpc_id* is provided as input variable. I usually create the network as a separate *network* module. 

````
# Further Security Groups defined up here, for example the eb_ec2_security_group that's used below

resource "aws_security_group" "ec_security_group" {
  name        = "${var.resource_name_prefix}-ec-security-group"
  description = "Allows traffic from Elastic Beanstalk EC2."
  vpc_id      = var.vpc_id

  ingress {
    description     = "Redis TCP"
    from_port       = 6379
    to_port         = 6379
    protocol        = "TCP"
    security_groups = [aws_security_group.eb_ec2_security_group.id]
  }

  tags = {
    App : "${var.resource_name_prefix}"
    Name : "${var.resource_name_prefix}-ec-security-group"
  }
}
````

#### outputs.tf

````
output "ec_security_group_id" {
  value = aws_security_group.ec_security_group.id
}
````

#### variables.tf

````
variable "resource_name_prefix" {}
variable "vpc_id" {}
````

### Redis Module (modules/redis)

#### main.tf

Note that subnets are created as part of *network* module and passed in here as input variables.

````
resource "aws_elasticache_subnet_group" "ec_subnet_group" {
  name       = "${var.resource_name_prefix}-ec-subnet-group"
  subnet_ids = [var.public_subnet_a_id, var.public_subnet_b_id]
  tags = {
    App : "${var.resource_name_prefix}"
    Name : "${var.resource_name_prefix}-ec-subnet-group"
  }
}

resource "aws_elasticache_cluster" "ec_cluster" {
  cluster_id           = "${var.resource_name_prefix}-ec-cluster"
  engine               = "redis"
  node_type            = "cache.t3.micro"
  num_cache_nodes      = 1
  parameter_group_name = "default.redis7"
  engine_version       = "7.0"
  port                 = 6379
  security_group_ids   = [var.ec_security_group_id]
  subnet_group_name    = aws_elasticache_subnet_group.ec_subnet_group.name

  tags = {
    App : "${var.resource_name_prefix}"
    Name : "${var.resource_name_prefix}-ec-cluster"
  }
}
````

#### outputs.tf

````
output "ec_cluster_public_url" {
  value = aws_elasticache_cluster.ec_cluster.cache_nodes[0].address
}
````

#### variables.tf

````
variable "resource_name_prefix" {}
variable "ec_security_group_id" {}
variable "public_subnet_a_id" {}
variable "public_subnet_b_id" {}
````

### Root main.tf

Inside the root *main.tf* file, the modules can be combined like this:

````
# Up here, you'd have the network module configuration

module "security" {
  source               = "./modules/security"
  resource_name_prefix = var.resource_name_prefix
  vpc_id               = module.network.vpc_id
}

module "redis" {
  source               = "./modules/redis"
  ec_security_group_id = module.security.ec_security_group_id
  resource_name_prefix = var.resource_name_prefix
  public_subnet_a_id   = module.network.public_subnet_a_id
  public_subnet_b_id   = module.network.public_subnet_b_id
}
````



