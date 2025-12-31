# my-terraform


# Terraform Comprehensive Cheatsheet (AWS-Focused)

This cheatsheet is a **practical + reference-style guide** to Terraform, with emphasis on **AWS deployments**. It covers syntax, workflow, core concepts, and links to official documentation for deeper study.

---

## 1. What Terraform Is

Terraform is an **Infrastructure as Code (IaC)** tool by HashiCorp that lets you:

* Declare infrastructure in **HCL (HashiCorp Configuration Language)**
* Version-control infrastructure
* Provision, update, and destroy resources safely

Official Docs: [https://developer.hashicorp.com/terraform/docs](https://developer.hashicorp.com/terraform/docs)

---

## 2. Terraform Core Concepts

### 2.1 Providers

Providers are plugins that talk to APIs (AWS, Azure, GCP, etc.).

```hcl
provider "aws" {
  region = "us-east-1"
}
```

AWS Provider Docs: [https://registry.terraform.io/providers/hashicorp/aws/latest/docs](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)

---

### 2.2 Resources

Resources are **infrastructure objects** (EC2, VPC, ECS, S3, IAM, etc.).

```hcl
resource "aws_s3_bucket" "example" {
  bucket = "my-bucket"
}
```

Resource Docs: [https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources)

---

### 2.3 Data Sources

Used to **read existing infrastructure**.

```hcl
data "aws_vpc" "default" {
  default = true
}
```

Data Source Docs: [https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources)

---

### 2.4 State

Terraform keeps a **state file** that maps configuration → real resources.

* Default: `terraform.tfstate` (local)
* Production: remote backend (S3 + DynamoDB)

State Docs: [https://developer.hashicorp.com/terraform/language/state](https://developer.hashicorp.com/terraform/language/state)

---

## 3. Terraform File Structure

Typical project layout:

```text
.
├── main.tf          # Core resources
├── variables.tf     # Input variables
├── outputs.tf       # Outputs
├── providers.tf     # Provider config
├── versions.tf      # Terraform/provider versions
├── terraform.tfvars # Variable values
```

---

## 4. Terraform Language (HCL) Syntax

### 4.1 Blocks

```hcl
block_type "label1" "label2" {
  argument = value
}
```

---

### 4.2 Arguments

```hcl
instance_type = "t3.micro"
count         = 2
enabled       = true
```

---

### 4.3 Expressions

```hcl
name = "${var.env}-app"
```

Expression Docs: [https://developer.hashicorp.com/terraform/language/expressions](https://developer.hashicorp.com/terraform/language/expressions)

---

### 4.4 Variables

#### Declare

```hcl
variable "region" {
  type    = string
  default = "us-east-1"
}
```

#### Use

```hcl
region = var.region
```

Variable Docs: [https://developer.hashicorp.com/terraform/language/values/variables](https://developer.hashicorp.com/terraform/language/values/variables)

---

### 4.5 Outputs

```hcl
output "vpc_id" {
  value = aws_vpc.main.id
}
```

Output Docs: [https://developer.hashicorp.com/terraform/language/values/outputs](https://developer.hashicorp.com/terraform/language/values/outputs)

---

### 4.6 Locals

```hcl
locals {
  app_name = "my-app"
}
```

Local Docs: [https://developer.hashicorp.com/terraform/language/values/locals](https://developer.hashicorp.com/terraform/language/values/locals)

---

## 5. Terraform Workflow (Terminal Commands)

### 5.1 Initialize Project

```bash
terraform init
```

* Downloads providers
* Configures backend

Docs: [https://developer.hashicorp.com/terraform/cli/commands/init](https://developer.hashicorp.com/terraform/cli/commands/init)

---

### 5.2 Validate Configuration

```bash
terraform validate
```

---

### 5.3 Format Code

```bash
terraform fmt
```

---

### 5.4 Plan Changes

```bash
terraform plan
```

With variables:

```bash
terraform plan -var-file=terraform.tfvars
```

Docs: [https://developer.hashicorp.com/terraform/cli/commands/plan](https://developer.hashicorp.com/terraform/cli/commands/plan)

---

### 5.5 Apply Infrastructure

```bash
terraform apply
```

Auto-approve:

```bash
terraform apply -auto-approve
```

Docs: [https://developer.hashicorp.com/terraform/cli/commands/apply](https://developer.hashicorp.com/terraform/cli/commands/apply)

---

### 5.6 Destroy Infrastructure

```bash
terraform destroy
```

Docs: [https://developer.hashicorp.com/terraform/cli/commands/destroy](https://developer.hashicorp.com/terraform/cli/commands/destroy)

---

## 6. AWS Authentication Methods

Terraform uses the AWS SDK.

### Supported Methods

* AWS CLI credentials (`~/.aws/credentials`)
* Environment variables
* IAM roles (EC2, ECS, EKS)
* OIDC (GitHub Actions)

Auth Docs: [https://registry.terraform.io/providers/hashicorp/aws/latest/docs#authentication](https://registry.terraform.io/providers/hashicorp/aws/latest/docs#authentication)

---

## 7. AWS Core Resources (Most Used)

### 7.1 VPC

```hcl
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}
```

Docs: [https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/vpc](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/vpc)

---

### 7.2 Subnets

```hcl
resource "aws_subnet" "public" {
  vpc_id     = aws_vpc.main.id
  cidr_block = "10.0.1.0/24"
}
```

---

### 7.3 EC2

```hcl
resource "aws_instance" "web" {
  ami           = "ami-0abcdef"
  instance_type = "t3.micro"
}
```

Docs: [https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/instance](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/instance)

---

### 7.4 IAM Role

```hcl
resource "aws_iam_role" "ecs_role" {
  name = "ecsTaskExecutionRole"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"
      Principal = { Service = "ecs-tasks.amazonaws.com" }
      Action = "sts:AssumeRole"
    }]
  })
}
```

Docs: [https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_role](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_role)

---

### 7.5 ECR

```hcl
resource "aws_ecr_repository" "repo" {
  name = "my-app"
}
```

Docs: [https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/ecr_repository](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/ecr_repository)

---

### 7.6 ECS Cluster

```hcl
resource "aws_ecs_cluster" "cluster" {
  name = "my-cluster"
}
```

Docs: [https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/ecs_cluster](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/ecs_cluster)

---

## 8. Modules (Reusable Infrastructure)

### 8.1 Module Structure

```text
modules/vpc/
├── main.tf
├── variables.tf
├── outputs.tf
```

### 8.2 Use Module

```hcl
module "vpc" {
  source = "./modules/vpc"
  cidr   = "10.0.0.0/16"
}
```

Module Docs: [https://developer.hashicorp.com/terraform/language/modules](https://developer.hashicorp.com/terraform/language/modules)

---

## 9. Backends (Remote State for AWS)

### S3 Backend (Recommended)

```hcl
terraform {
  backend "s3" {
    bucket         = "terraform-state-bucket"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}
```

Backend Docs: [https://developer.hashicorp.com/terraform/language/settings/backends/s3](https://developer.hashicorp.com/terraform/language/settings/backends/s3)

---

## 10. Meta-Arguments

### count

```hcl
count = 3
```

### for_each

```hcl
for_each = toset(["a", "b", "c"])
```

### depends_on

```hcl
depends_on = [aws_iam_role.ecs_role]
```

Docs: [https://developer.hashicorp.com/terraform/language/meta-arguments](https://developer.hashicorp.com/terraform/language/meta-arguments)

---

## 11. Lifecycle Rules

```hcl
lifecycle {
  prevent_destroy = true
  create_before_destroy = true
}
```

Docs: [https://developer.hashicorp.com/terraform/language/meta-arguments/lifecycle](https://developer.hashicorp.com/terraform/language/meta-arguments/lifecycle)

---

## 12. Workspaces (Multiple Environments)

```bash
terraform workspace new dev
terraform workspace select prod
```

Docs: [https://developer.hashicorp.com/terraform/cli/workspaces](https://developer.hashicorp.com/terraform/cli/workspaces)

---

## 13. Best Practices (AWS)

* Use **remote state (S3 + DynamoDB)**
* Use **IAM roles**, not access keys
* Keep modules small & reusable
* Never commit `.tfstate`
* Use `terraform plan` in CI

---

## 14. Official References (Bookmark These)

* Terraform Language: [https://developer.hashicorp.com/terraform/language](https://developer.hashicorp.com/terraform/language)
* AWS Provider: [https://registry.terraform.io/providers/hashicorp/aws/latest/docs](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
* Terraform CLI: [https://developer.hashicorp.com/terraform/cli](https://developer.hashicorp.com/terraform/cli)
* Registry Modules: [https://registry.terraform.io/](https://registry.terraform.io/)

---

## 15. Next Logical Steps

* ECS Fargate deployment
* EKS cluster with Terraform
* CI/CD with GitHub Actions + Terraform
* OIDC authentication

(You can ask for **deep-dive cheatsheets** on any of these.)
