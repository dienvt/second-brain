
https://github.com/88labs/andpad-terraform

Continueous Deploying
https://www.runatlantis.io/

# How terraform works
defined `main.tf` describe the final services will be apply. So that it will not change if we run `terraform apply` multiple times

# Syntax
## Struct
### 1.`locals`

The `locals` keyword allows you to create local variables within your configuration. These are useful for intermediate values and to simplify complex expressions.

**Example:**

```
locals {
	instance_name = "example-instance" 
}  

resource "aws_instance" "example" {   
	ami           = "ami-12345678"   
	instance_type = "t2.micro"   
	tags = {     
		Name = local.instance_name
	} 
}
```

### 2. `data`

The `data` keyword in Terraform is used to access information defined outside of Terraform or managed by other services. This is often used to **query existing resources**.
```
data "aws_ami" "example" {
  most_recent = true
  owners      = ["self"]

  filter {
    name   = "name"
    values = ["my-ami"]
  }
}

```
### 3. `variable`

The `variable` keyword is used to define **input variables**, which allow you to parameterize your Terraform configurations. This makes configurations more flexible and reusable.

**Example:**
```
variable "instance_type" {
  description = "Type of instance to create"
  default     = "t2.micro"
}

resource "aws_instance" "example" {
  ami           = "ami-12345678"
  instance_type = var.instance_type
}
```


### 4. `output`

The `output` keyword allows you to define **output values** that are displayed when `terraform apply` or `terraform output` is run. These outputs can be used to display useful information or to be used in other configurations.

**Example:**
```
output "instance_ip" {
	value = aws_instance.example.public_ip 
}
```

### 5. `resource`

The `resource` keyword is used to define components that make up your infrastructure, such as virtual machines, storage, or databases. Each resource type has its own set of arguments.

**Example:**
```
resource "aws_instance" "example" {
	ami           = "ami-12345678"   
	instance_type = "t2.micro" 
}
```
### 6. `module`
The `module` keyword is used to call a module, which is a container for multiple resources that are used together. Modules are a way to organize and encapsulate code for reusability and to simplify management.

**Example:**
```
module "network" {
	source = "./modules/network"    
	vpc_id = aws_vpc.main.id 
}
```


### Compare locals and variables
https://www.reddit.com/r/Terraform/comments/12tob5q/when_to_use_locals_vs_variables/

### Compare data vs variables
| Data                                            | Variables                            |
| ----------------------------------------------- | ------------------------------------ |
| Actively get data from another resource, module | Passive receive data from the caller |
|                                                 |                                      |
|                                                 |                                      |

## [Built-in Functions](https://developer.hashicorp.com/terraform/language/functions)
### 1. max




# Services
* AMIs (Amazone Machine Images)
* CloudWach Alarm : Monitor resourses
* Dynamodb: Database
* ECR: Elastic Container Registry
* IAM (Identity and Accessibility Manager)
* KMS (Key Manager Service): 
* PubSub (sns & sqs): 
* S3_Bucket: 
* Slack_Alert_Channel: 
* SNS_Platform_Application
* WAF (Web Application Firewall)
# How to create terraform

## Step 1: Define `main.tf`


## Step 2: Run cmd `terraform init`


## Step 3: Apply change

### `terraform plan`

- Generate and show the execution plan in the currently directory:
    terraform plan

- Show a plan to destroy all remote objects that currently exist:
    terraform plan -destroy

- Show a plan to update the Terraform state and output values:
    terraform plan -refresh-only

- Specify values for input variables:
    terraform plan -var 'name1=value1' -var 'name2=value2'

- Focus Terraform's attention on only a subset of resources:
    terraform plan -target resource_type.resource_name[instance index]

- Output a plan as JSON:
    terraform plan -json

- Write a plan to a specific file:
    terraform plan -no-color > path/to/file
