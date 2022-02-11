# Terraform Notes

## Work Flow

1. **WRITE** - Use VCS (recommended)
2. **PLAN** - Review the changes the code will make (cycle between write and plan)

- Reads the code and shows a plan of execution. 
- It makes API calls to the Cloud provider.
- While this is effectively a RO command, it requires auth creds to connect to the Cloud provider.

3. **APPLY** - Provision the real infrastructure
   
- Deploys the instructions and statements in the code.
- Updates the tracking mechanism file, AKA the state file (terraform.tfstate local or remote)

4. **DESTROY**

- Use with caution
- Destroys all the resources that are being tracked by the state file.

### Initialising the Terraform Work Flow
`terraform init`
- downloads modules and plugins (also known as providers).
- caches modules and plugins, but will download the latest by default.
- sets up the backed for storing the state file.
- Its a critical command!

## Resource Addressing

### **_provider block_**

```
# code fetches a provider (keyword btw), in this case AWS
provider "aws" {
    region ="us-east-1"
}
```

### **_resource block_**
```
# resource - keyword
# aws_instance - derived from the provider (see above)
# web - arbitrary user provided name
# resource config args inside curly braces
resource "aws_instance" "web"{
    ami =
    instance_type = t2.micro
}
```
To reference use the resource address:
```
aws_instance.web
```    
### **_data source block_**
Fetches/tracks data of an already created resource
```
# data - keyword
# aws_instance - derived from the provider (see above)
# my_vm - arbitrary user provided name
data "aws_instance" "my_vm" {
    instance_id = i-asdasdasd
}
```
To reference use the resource address:
```
data.aws_instance.my-vm
```

## Terraform code behavior
- Terraform executes code  in files with the .tf extension.
- by default looks in the terraform providers registry - registry.terraform.io but this can also be sourced locally/internally, and you can even write your own. 

## Installing Terraform

1. Download the zip binary, unzip and place it on the system path where it can be invoked.
2. Setup the Terraform repo for you Linux flavour and use pkg manager to install and it will be setup ready to use.


## Terraform Providers

- These abstract the integrations with API control layer of the cloud provider.
- Every cloud/infra has its own provider.
- By default, Terraform looks in the providers registry.
- Providers can also be sourced locally/internally (on-prem e.g. Artifactory)
- Providers are plugins and have their own release cycle and own version number.
- Essentially, a provider is a predefined chunk of code that enables Terraform to interact with a given Cloud provider.
- Its possible to write you own custom provider.

- Terraform finds and installs providers when initialising the working dir (see `terraform init`).
- Providers should be pegged to a specific version as changes to the version may break your deployment.
- Providers are downloaded to Terraform.

## Terraform State file

- Resource tracking - Terraform can keeps tabs on what has been deployed.
- Critical to Terraform functionality.  Do resources need to created, modified or destroyed?
- The state file is a JSON dump of all the resources that have been deployed and is stored as a flat file.
- Helps Terraform calculate deployment deltas and create new plans.
- plan is compared to the state file to calculate delta.
- The state file may contain sensitive data so don't let it fall into the wrong hands! 

## Variables and Output

Example:
```
# variable - keyword 
# var_name - arbitrary user provided name
# args     - inside curly braces

variable "var_name" {
    description
    type
    default
}
```

Its also possible to declare a variable as follows, but values would have to passed via OS ENV var or cmd line input.

```
variable "var_name" = {}
```

To reference a variable:
```
var.my-var
```

Its best practice to gather variables in the `terraform.tfvars` file.

### Variable validation

```
variable "var_name" {
    description
    type
    default
    validation {
        condition = length(var.my-var) > 4
        error_message = "the string must be 4 chars"
    }
}
```

Use a config parameter called sensitive to prevent vars to be hidden.
```
variable "var_name" {
    description
    type
    sensitive = true
}
```

### Base types

- string
- number
- bool

### Complex types

- list, set, map, object, tuple

Which [variables are given what precedence](https://www.terraform.io/language/values/variables#variable-definition-precedence)?

### Output values

```
# output      - keyword
# instance_ip - arbitrary user provided name
# args        - inside curly braces

output "instance_ip"{
    description = "Instance IP"
    value = aws_instance.my-vm.private_ip 
}
```
- Output variables are shown on the shell after running terraform apply.
- Output variables are like return values that you want to track after a successful Terraform deployment.

## Terraform Provisioners 
- The Terraform way of bootstrapping custom scripts against resources.
- These can be run locally or remotely on resources spun up through Terraform deployment.
- A provisioner is attached to a Terraform resource and allows custom connection parameters, which can be passed to the remote resources via SSH or WINRM and carry out arbitrary commands against that resource.
- 2 types: Creation-Time and Destroy-Time
- If a command within the provisioner returns non-zero, it is considered failed and the resource will be tainted. 
- Use `self.id` when referencing from within resource.
