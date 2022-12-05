# F5-XC-AWS-VPC-SNODE-SNIC

This repository consists of Terraform templates to bring up a f5-xc-aws-vpc-snode-snic object.

## Usage

- Clone this repo with: `git clone --recurse-submodules https://gitlab.com/volterra/solution/f5-xc-terraform/f5-xc-aws-vpc-snode-snic`
- Enter repository directory with: `cd f5-xc-aws-vpc-snode-snic`
- Obtain F5XC API certificate file from Console and save it to `cert` directory
- Pick and choose from below examples and add mandatory input data and copy data into file `main.tf.example`.
- Rename file __main.tf.example__ to __main.tf__ with: `rename main.tf.example main.tf`
- Initialize with: `terraform init`
- Apply with: `terraform apply -auto-approve` or destroy with: `terraform destroy -auto-approve`

## f5-xc-aws-vpc-snode-snic module usage example

````hcl
variable "project_prefix" {
  type        = string
  description = "prefix string put in front of string"
  default     = "f5xc"
}

variable "project_suffix" {
  type        = string
  description = "prefix string put at the end of string"
  default     = "01"
}

variable "f5xc_api_p12_file" {
  type    = string
}

variable "f5xc_api_url" {
  type    = string
}

variable "f5xc_api_token" {
  type    = string
}

variable "f5xc_tenant" {
  type    = string
}

variable "f5xc_namespace" {
  type    = string
  default = "system"
}

variable "f5xc_aws_cred" {
  type    = string
  default = "ck-aws-01"
}

variable "f5xc_aws_region" {
  type    = string
  default = "us-east-2"
}

variable "f5xc_aws_availability_zone" {
  type    = string
  default = "a"
}

variable "owner_tag" {
  type    = string
  default = "c.klewar@f5.com"
}

variable "ssh_public_key_file" {
  type    = string
}

locals {
  aws_availability_zone = format("%s%s", var.f5xc_aws_region, var.f5xc_aws_availability_zone)
  custom_tags = {
    Owner        = var.owner_tag
    f5xc-tenant  = var.f5xc_tenant
    f5xc-feature = "f5xc-aws-vpc-site"
  }
}

provider "volterra" {
  api_p12_file = var.f5xc_api_p12_file
  url          = var.f5xc_api_url
  alias        = "default"
}

provider "aws" {
  region = var.f5xc_aws_region
  alias  = "us-east-2"
}


module "f5xc_aws_vpc_single_node_single_nic_new_vpc_new_subnet" {
  source                    = "./modules/f5xc/site/aws/vpc"
  f5xc_api_url              = var.f5xc_api_url
  f5xc_api_token            = var.f5xc_api_token
  f5xc_namespace            = var.f5xc_namespace
  f5xc_tenant               = var.f5xc_tenant
  f5xc_aws_region           = var.f5xc_aws_region
  f5xc_aws_cred             = var.f5xc_aws_cred
  f5xc_aws_vpc_owner        = var.owner
  f5xc_aws_vpc_site_name    = format("%s-vpc-sn-snic-nvpc-nsnet-%s", var.project_prefix, var.project_suffix)
  f5xc_aws_vpc_primary_ipv4 = "192.168.168.0/21"
  f5xc_aws_ce_gw_type       = "single_nic"
  f5xc_aws_vpc_az_nodes     = {
    node0 = { f5xc_aws_vpc_local_subnet = "192.168.168.0/26", f5xc_aws_vpc_az_name = local.aws_availability_zone },
  }
  f5xc_aws_default_ce_os_version       = true
  f5xc_aws_default_ce_sw_version       = true
  f5xc_aws_vpc_no_worker_nodes         = true
  f5xc_aws_vpc_use_http_https_port     = true
  f5xc_aws_vpc_use_http_https_port_sli = true
  f5xc_labels                          = { "aws-env" = "shared" }
  ssh_public_key                       = file(var.ssh_public_key_file)
  custom_tags                          = local.custom_tags
  providers                            = {
    aws      = aws.us-east-2
    volterra = volterra.default
  }
}
````
# Trigger Pipeline

```bash
curl -X POST \
     --fail \
     -F token=<REPLACE-WITH-OWN-PIPELINE-TRIGGER-TOKEN> \
     -F ref=main \
     -F "variables[F5XC_CERT_NAME]=playground.staging" \
     https://gitlab.com/api/v4/projects/41613940/trigger/pipeline
```
