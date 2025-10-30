---
title: Terraform Best Practices
inclusion: fileMatch
fileMatchPattern: '**/*.tf,**/*.tfvars,**/*.hcl,**/terraform.lock.hcl'
---

# Terraform Best Practices

## Code Organization and Structure
- Use consistent file naming: `main.tf`, `variables.tf`, `outputs.tf, versions.tf`
- Group related resources in separate files by function
- Use modules for reusable infrastructure components
- Keep root modules simple, delegate complexity to child modules
- Use meaningful resource and variable names
- Use `deploy/terraform/` directory as the entry point for all files in the repo.

```hcl
# Good: Organized file structure
# main.tf - primary resources
# variables.tf - input variables
# outputs.tf - output values
# versions.tf - provider requirements
# modules/
#   vpc/
#     main.tf
#     variables.tf
#     outputs.tf
```

## State Management
- Always use remote state backends (S3, Azure Storage, GCS)
- Enable state locking to prevent concurrent modifications
- Use separate state files for different environments
- Never commit state files to version control
- Use workspaces or separate configurations for environments

```hcl
# Good: Remote state with locking
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}
```

## Variables and Configuration
- Use descriptive variable names and descriptions
- Set appropriate variable types and validation rules
- Use default values for optional parameters
- Group related variables using objects
- Use locals for computed values and DRY principles

```hcl
# Good: Well-defined variables
variable "environment" {
  description = "Environment name (dev, staging, prod)"
  type        = string
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}

variable "vpc_config" {
  description = "VPC configuration settings"
  type = object({
    cidr_block           = string
    enable_dns_hostnames = bool
    enable_dns_support   = bool
  })
  default = {
    cidr_block           = "10.0.0.0/16"
    enable_dns_hostnames = true
    enable_dns_support   = true
  }
}

locals {
  common_tags = {
    Environment = var.environment
    Project     = var.project_name
    ManagedBy   = "terraform"
  }
}
```

## Resource Management
- Use consistent naming conventions with prefixes/suffixes
- Apply comprehensive tagging strategy for all resources
- Use data sources to reference existing resources
- Implement proper resource dependencies with `depends_on`
- Use lifecycle rules to prevent accidental deletions

```hcl
# Good: Consistent naming and tagging
resource "aws_instance" "web_server" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = var.instance_type
  
  tags = merge(local.common_tags, {
    Name = "${var.project_name}-${var.environment}-web"
    Role = "web-server"
  })
  
  lifecycle {
    prevent_destroy = true
    ignore_changes  = [ami]
  }
}

data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"] # Canonical
  
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]
  }
}
```

## Modules and Reusability
- Create modules for commonly used infrastructure patterns
- Use semantic versioning for module releases
- Document module inputs, outputs, and usage examples
- Keep modules focused on single responsibilities
- Use module composition for complex infrastructure

```hcl
# Good: Module usage with version pinning
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 3.0"
  
  name = "${var.project_name}-${var.environment}"
  cidr = var.vpc_cidr
  
  azs             = data.aws_availability_zones.available.names
  private_subnets = var.private_subnet_cidrs
  public_subnets  = var.public_subnet_cidrs
  
  enable_nat_gateway = true
  enable_vpn_gateway = false
  
  tags = local.common_tags
}
```

## Security and Compliance
- Never hardcode sensitive values in configuration
- Implement least privilege access for Terraform execution
- Use provider-specific security best practices
- Enable encryption for all applicable resources

```hcl
# Good: Using sensitive variables and data sources
variable "database_password" {
  description = "Database master password"
  type        = string
  sensitive   = true
}

resource "aws_db_instance" "main" {
  allocated_storage    = 20
  storage_encrypted    = true
  engine              = "mysql"
  engine_version      = "8.0"
  instance_class      = "db.t3.micro"
  db_name             = var.database_name
  username            = var.database_username
  password            = var.database_password
  
  # Security best practices
  backup_retention_period = 7
  backup_window          = "03:00-04:00"
  maintenance_window     = "sun:04:00-sun:05:00"
  
  tags = local.common_tags
}
```

## Testing and Validation
- Use `terraform plan` before applying changes
- Implement automated testing with tools like Terratest
- Use `terraform validate` and `terraform fmt` in CI/CD
- Test modules independently before integration
- Use `terraform refresh` to sync state with reality

```bash
# Good: Validation and formatting pipeline
terraform fmt -check -recursive
terraform validate
terraform plan -out=tfplan
terraform apply tfplan
```

## Performance and Operations
- Use `terraform import` for existing resources
- Implement proper error handling and retry logic
- Use parallelism settings for large infrastructures
- Monitor Terraform execution time and optimize accordingly
- Use `terraform refresh` sparingly in automation

```hcl
# Good: Provider configuration with retry logic
provider "aws" {
  region = var.aws_region
  
  default_tags {
    tags = local.common_tags
  }
  
  retry_mode      = "adaptive"
  max_retries     = 3
}
```