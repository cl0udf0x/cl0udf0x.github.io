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

4. DESTROY 

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