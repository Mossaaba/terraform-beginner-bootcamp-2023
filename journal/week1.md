# Terraform Beginner Bootcamp 2023 - Week 1

## Root Module Structure

Our root module structure is as follows:

```
PROJECT_ROOT
│
├── main.tf                 # everything else.
├── variables.tf            # stores the structure of input variables
├── terraform.tfvars        # the data of variables we want to load into our terraform project
├── providers.tf            # defined required providers and their configuration
├── outputs.tf              # stores our outputs
└── README.md               # required for root modules
```

[Standard Module Structure](https://developer.hashicorp.com/terraform/language/modules/develop/structure)

## Terraform and Input Variables

### Terraform Cloud Variables

In terraform we can set two kind of variables:

- **Enviroment Variables** - those you would set in your bash terminal eg. AWS credentials
- **Terraform Variables** - those that you would normally set in your tfvars file

We can set Terraform Cloud variables to be **sensitive** so they are not shown visibliy in the UI.

### Loading Terraform Input Variables

[Terraform Input Variables](https://developer.hashicorp.com/terraform/language/values/variables)

### var flag
We can use the `-var` flag to set an input variable or override a variable in the tfvars file eg. `terraform -var user_ud="my-user_id"`

### var-file flag

- In Terraform, the **var-file flag** is used to specify an *external variables file* when running Terraform commands. 

This allows to separate your **variable values** from your **Terraform configuration code**, making it easier to manage and share configurations across different environments and collaborators.

To do that : 

1.  **Create** the var-file **json** OR **hcl** 

 ```JSON
    {
    "region": "eu-west-3",
    "instance_type": "t2.micro",
    "terrahouse_id": "terra-12345678"
    }
 ```

2.  **Reference** the var-file -> In **main** Terraform configuration file , define variables as usual but we don't assign values to them. 

```hcl
variable "region" {}
variable "instance_type" {}
variable "ami_id" {}

provider "aws" {
  region = var.region
}

resource "aws_instance" "example" {
  ami           = var.ami_id
  instance_type = var.instance_type
}
```
Instead, we reference the variables from your external variables file using the var function.

3.  **Provide** the path to your external variables file.

```sh
terraform apply -var-file=variables.json
```
### Terraform.tvfars

This is the default file to load in terraform variables in blunk

### auto.tfvars

The **auto.tfvars file** is a special filename that Terraform automatically looks for to load variable values when you run Terraform commands like **terraform apply**, **terraform plan**, or **terraform init**. This file is part of Terraform's **automatic variable** loading mechanism, and it allows you to set **default values** for your variables without explicitly specifying them on the command line.

Example **auto.tfvars**: 
```sh
region = "us-west-2"
instance_type = "t2.micro"
```

In your **main** Terraform configuration file (usually named main.tf), you define your variables as usual but don't assign default values to them. Instead, Terraform will automatically load the values from **auto.tfvars**
Example **main.tf**: 

```sh
variable "region" {}
variable "instance_type" {}
```
### order of terraform variables

In Terraform, variable values can be sourced from multiple places, following a specific order of precedence:

1. **Command-Line Flags:** Values specified through command-line flags take the highest precedence.

2. **Environment Variables:** Values set using environment variables named in a specific format.

3. **Variable Files:** Values specified in files with extensions like `.tfvars`, loaded in alphabetical order with later files overriding earlier ones.

4. **Default Values in Variable Blocks:** Variables can have default values defined within the Terraform configuration.

5. **No Value Specified:** Terraform prompts the user for a value if none is provided through the previous methods.

This hierarchy of sources enables flexibility and customization while managing Terraform variables.


## Dealing With Configuration Drift

## What happens if we lose our state file?

If you lose your **statefile**, you most likley have to tear down all your cloud infrastructure manually.

You can use **terraform import** but it won't for all cloud resources. You need check the terraform providers documentation for which resources support import.

### Fix Missing Resources with Terraform Import

`terraform import aws_s3_bucket.bucket bucket-name`

[Terraform Import](https://developer.hashicorp.com/terraform/cli/import)
[AWS S3 Bucket Import](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/s3_bucket#import)

### Fix Manual Configuration

If someone goes and delete or modifies cloud resource manually through ClickOps. 

If we run **Terraform plan** is with attempt to put our infrstraucture back into the expected state fixing Configuration Drift

If we run Terraform plan is with attempt to put our infrstraucture back into the expected state fixing Configuration Drift

## Fix using Terraform Refresh

```sh
terraform apply -refresh-only -auto-approve
```

## Terraform Modules

### Terraform Module Structure

It is recommend to place modules in a **`modules`** directory when locally developing modules but you can name it whatever you like.

### Passing Input Variables

We can pass input variables to our module.
The **module** has to declare the terraform variables in its own **variables.tf**

```tf
module "terrahouse_aws" {
  source = "./modules/terrahouse_aws"
  user_uuid = var.user_uuid
  bucket_name = var.bucket_name
}
```

### Modules Sources

Using the source we can import the module from various places eg:
- locally
- Github
- Terraform Registry

```tf
module "terrahouse_aws" {
  source = "./modules/terrahouse_aws"
}
```

[Modules Sources](https://developer.hashicorp.com/terraform/language/modules/sources)
