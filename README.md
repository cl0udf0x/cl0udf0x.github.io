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

## Terraform State in-depth
- Maps real-world resources to Terraform configuration
- By default its stored locally in a file called `terraform.tfstate`.
- Prior to any modification operation, Terraform refreshes the state file.
- Resource dependency metadata is also tracked via the state file.
- Helps boost deployment performance by caching attribute resources for subsequent use.

### Terraform state file command

Terraform command utility for manipulating and retrieving the state file
- advanced state management
- manually remove a resource from state file so its not managed by terraform.
- Listing out tracked resources and their details (via state and list commands)
  
```
terraform state list # List all resources tracked by the Terraform state file
terraform state rm   # Delete a resource from the Terraform state file
terraform state show # Show details of a resource tracked in the state file
```

### Local state storage
- Saves state locally on your system
- default behavior.
- saves a backup of your last known state file after successful apply.

### Remote state storage

- AWS s3, google cloud, and more can be used.
- Allows sharing state file between distributed teams.
- Secure and highly available. 
- Allows locking state so parallel executions don't coincide (AWS s3, Hashicorp but not supported by all vendors)
- Enables sharing output values with other Terraform configuration or code.
  
Persisting state in AWS S3.

```
terraform {
    backend = s3{
        region = 
        key    = 
        bucket = 
    }
}
```

## Modules

### Accessing and Using

- just a collection of terraform code
- groups multiple resources that are used together
- essentially makes code reusable 
- Every Terraform configuration has at least one module, called the `root` module, which consists of code files in your main working directory
- When you use modules within the root module these are known as child modules.

Modules can be downloaded from:
- Terraform Public registry
- A private registry
- Your local system

```
module "my-vpc-module" {
    source = "./modules/vpc"
    version = "0.0.5"
    region = var.region
}
```
Always use a specific version of a module

Other allowed parameters
- count
- for_each
- providers
- depends_on

Can optionally take input and provide outputs to plug back to your main code.

### Accessing module outputs in your code

```
resource aws_instance "my_vpc_module" {
    # other args
    subnet_id = "module.my-vpc-module.subnet_id"
}
```

### Interacting with module inputs and outputs

Module inputs are arbitrarily named parameters that pass inside the module block.
These inputs can used as variables inside the module code.

module my-vpc-module {
    source = ./modules/vpc
    server-name = us-east-1 # input parameters for module
}

reference inside module as var.server-name

Outputs declared inside the Terraform module code can be fed back into the root module or your main code.

Example

module.<name of module>.<name-of-output>

```
output ip_address {
    value = "aws_instance.private_ip"
}
```

reference outside of module using `module.myvpc-module.ip_address`

## Built in Functions and Dynamic blocks

### Built in Functions

Terraform comes pre-packaged with functions to help transform and combine values.

User defined  functions are not allowed - only built in ones.

General syntax is `function_name(arg1, arg2)`

Built in functions are extremely useful in making Terraform code dynamic and flexible.

`Join` function example, result terraform-prod

```
variable "project-name" {
    type = string
    default = "prod"
}
```
```
resource "aws_vpc" "my-vpc" {
    cidr_block = "10.0.0.6/16"
    tags {
        Name = join("-", ["terraform", var.project-name])
    }
}
```

List of [Terraform functions](https://www.terraform.io/language/functions)

## Using the Console to test

Test functions

`terraform console`

`> max(1,4,6,7)`

`> timestamp()`

`> join("-", "me", "you"`

`> contains(["one","two","three"], "one")`

## Type Contraints

- Type contraints control the type of variable.
- Primitive e.g. `number`, `string`, `bool`
- Complex e.g multiple types in a single variable e.g `list`, `tuple`, `map`, `object`

### Collection

- Allow multiple values of one primitive type to grouped together.
- `constructors` include `list()`, `map()`, `set()`

### Structural

- Allow multiple types of different primitive types to be grouped together.
- `constructors` include `object()`, `tuple()`, `set()`

```
variable "instructor" {
    type = object({
        name = string
        age = number
    })
}
```

### Any

- is a placeholder for a primitive type yet to be decided
- Actual type will be determined at runtime.

```
# Terraform recognises all values as numbers in one variable.

variable "data" {
    type = list(any)
    default = [1,2,3]
}
```

## Dynamic blocks

Dynamical constructs repeatable nested config blocks inside Terraform resources
Supported within 
- resource
- data
- provider 
- provisioner

Used to make you code look cleaner

```
resource "aws_security_group" "my-sg" {
    name = "my-aws-security-group"
    vpc_id = aws_vpc.my-vpc.id
    ingress {
        from_port   = 22
        to_port     = 22
        protocol    = "tcp"
        cidr_blocks = ["1.2.3.4/32"]
    }
    ingress {
        ... #more ingress rules
    }
}
```

VS
```
resource "aws_security_group" "my-sg" {
    name = "my-aws-security-group"
    vpc_id = aws_vpc.my-vpc.id
    dynamic "ingress" {            # dynamic block keyword
        for_each = var.rules       # Complex variable to iterate over
        content {                  # "content" block defines the body of each generated block using the variable
          from_port   = ingress.value["port"]
          to_port     = ingress.value["port"]
          protocol    = ingress.value["proto"]
          cidr_blocks = ingress.value["cidrs"]
        }
    }
}

variable "rules" {
    default = [
        {
            port = 80
            proto = "tcp"
            cidr_blocks = ["0.0.0.0/0"]

        },
        {
            port = 22
            proto = "tcp"
            cidr_blocks = ["1.2.3.4/32"]

        },

    ]
}
```

- Dynamic blocks expect a complex type to iterate over.
- They act like for loop and outputs a nested block for each element in our variable.
- Caution! Be careful not to overuse in main code as they can be hard to read and maintain.
- Only use dynamic blocks when you need to hide detail in order to build a cleaner user interface when writing reusable modules.




























