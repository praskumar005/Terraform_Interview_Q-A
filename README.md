# Terraform_Interview_Q-A
All Terraform Topics are covered

# 🚀 Terraform Interview Preparation

> Quick revision notes for DevOps / Cloud DevOps Engineer interviews.

---

## 📑 Table of Contents

1. [What is Terraform?](#1-what-is-terraform)
2. [Terraform vs CloudFormation](#2-terraform-vs-aws-cloudformation)
3. [Terraform File Extension](#3-terraform-file-extension)
4. [Terraform Provider](#4-terraform-provider)
5. [Terraform Resource](#5-terraform-resource)
6. [Terraform Variables](#6-terraform-variables)
7. [Terraform Workflow](#7-terraform-workflow)
8. [Terraform State](#8-terraform-state)
9. [Remote Backend](#9-terraform-remote-backend)
10. [State Locking](#10-terraform-state-locking)
11. [Drift Detection](#11-terraform-drift)
12. [Terraform Refresh](#12-terraform-refresh)
13. [Terraform Import](#13-terraform-import)
14. [Terraform Taint / Replace](#14-terraform-taint--replace)
15. [Terraform Lifecycle](#15-terraform-lifecycle)
16. [Terraform Modules](#16-terraform-modules)
17. [Terraform Provisioners](#17-terraform-provisioners)
18. [Terraform in CI/CD](#18-terraform-in-cicd)
19. [Scenario-Based Questions](#19-scenario-based-questions)
20. [Quick Revision](#20-quick-revision)

---

# 1. What is Terraform?

Terraform is an **Infrastructure as Code (IaC)** tool developed by HashiCorp.

It allows us to provision and manage infrastructure using declarative configuration files written in **HCL (HashiCorp Configuration Language)**.

Instead of manually creating AWS resources through the console, we define them as code.

### Example

```hcl
resource "aws_instance" "web" {
  ami           = "ami-xxxxxxxx"
  instance_type = "t3.micro"
}
```

### Interview Answer

> Terraform is an Infrastructure as Code tool that allows us to provision and manage infrastructure declaratively. I use Terraform to automate AWS resources, version infrastructure through Git, review changes using `terraform plan`, and deploy them consistently using `terraform apply`.

---

# 2. Terraform vs AWS CloudFormation

| Terraform | CloudFormation |
|---|---|
| Multi-provider | AWS-native |
| Uses HCL | Uses YAML/JSON |
| Terraform Modules | Modules/Nested Stacks |
| Explicit state management | AWS-managed stack state |
| `terraform plan` | Change Sets |

### Interview Answer

> Both Terraform and CloudFormation are Infrastructure as Code tools. CloudFormation is AWS-native, whereas Terraform provides a consistent workflow across multiple providers. Terraform's reusable modules, planning workflow, and ecosystem make it useful for building standardized enterprise infrastructure.

---

# 3. Terraform File Extension

Terraform configuration files use:

```text
.tf
```

Common files:

```text
main.tf
provider.tf
variables.tf
outputs.tf
terraform.tfvars
```

---

# 4. Terraform Provider

A **Provider** allows Terraform to communicate with external platforms such as AWS.

### Example

```hcl
provider "aws" {
  region = "us-east-1"
}
```

### Remember

```text
Provider = WHERE infrastructure is created
```

### Interview Answer

> A Terraform provider is a plugin that allows Terraform to communicate with external platforms through their APIs. For example, the AWS provider allows Terraform to manage EC2, VPC, IAM, S3, EKS, and other AWS resources.

---

# 5. Terraform Resource

A resource represents the infrastructure Terraform creates or manages.

### Example

```hcl
resource "aws_instance" "web" {
  ami           = "ami-xxxxxxxx"
  instance_type = "t3.micro"
}
```

### Remember

```text
Resource = WHAT Terraform creates
```

---

# 6. Terraform Variables

Variables prevent hardcoding and make Terraform reusable.

### Example

```hcl
variable "instance_type" {
  type    = string
  default = "t3.micro"
}
```

Use:

```hcl
resource "aws_instance" "web" {
  ami           = "ami-xxxxxxxx"
  instance_type = var.instance_type
}
```

### Remember

```text
Variable = VALUE
```

---

# 7. Terraform Workflow

```bash
terraform init
terraform fmt
terraform validate
terraform plan
terraform apply
```

### Meaning

```text
init      → Initialize Terraform
fmt       → Format code
validate  → Validate configuration
plan      → Preview changes
apply     → Implement changes
destroy   → Remove managed infrastructure
```

---

# 8. Terraform State

Terraform keeps track of managed infrastructure using a state file:

```text
terraform.tfstate
```

Concept:

```text
Terraform Configuration
        ↓
Terraform State
        ↕
AWS Infrastructure
```

### Interview Answer

> Terraform state maintains the mapping between Terraform resource addresses and actual infrastructure resources. In team environments, I prefer storing state in a secure remote backend rather than locally.

---

# 9. Terraform Remote Backend

A remote backend stores Terraform state centrally.

A common AWS solution is **Amazon S3**.

```hcl
terraform {
  backend "s3" {
    bucket       = "company-terraform-state"
    key          = "prod/terraform.tfstate"
    region       = "us-east-1"
    use_lockfile = true
  }
}
```

Benefits:

- Centralized state
- Team collaboration
- CI/CD integration
- State recovery with S3 versioning
- State locking

---

# 10. Terraform State Locking

State locking prevents multiple Terraform operations from modifying the same state simultaneously.

```text
Engineer A
    ↓
Acquire Lock
    ↓
terraform apply
    ↓
Release Lock

Engineer B
    ↓
Cannot acquire same lock
```

### S3 Locking

```hcl
use_lockfile = true
```

Terraform creates an S3 lock object while the lock is held.

Example:

```text
terraform.tfstate
terraform.tfstate.tflock
```

### Verify Locking

Terminal 1:

```bash
terraform apply
```

Terminal 2:

```bash
terraform apply
```

The second conflicting operation should not be able to acquire the same state lock.

---

# 11. Terraform Drift

**Configuration Drift** occurs when actual infrastructure differs from the desired Terraform configuration.

Example:

```text
Terraform Configuration → t3.micro
AWS Actual Resource     → t3.medium

                         ↑
                        DRIFT
```

Detect using:

```bash
terraform plan
```

### Interview Answer

> If I detect production drift, I first determine whether the change was authorized. If it was accidental, I restore the desired configuration through Terraform. If it was intentional, I update the Terraform code through the normal Git review process so that Infrastructure as Code remains the source of truth.

---

# 12. Terraform Refresh

Refresh synchronizes Terraform's state representation with actual infrastructure.

Preferred approach:

```bash
terraform plan -refresh-only
```

Then:

```bash
terraform apply -refresh-only
```

### Remember

```text
Drift   = Difference
Refresh = Synchronization of state
```

---

# 13. Terraform Import

Terraform Import brings an existing infrastructure resource under Terraform management.

Example:

```bash
terraform import aws_instance.web i-0123456789abcdef
```

Then:

```bash
terraform plan
```

### Remember

```text
Existing AWS Resource
        ↓
terraform import
        ↓
Terraform State
        ↓
Terraform Management
```

---

# 14. Terraform Taint / Replace

Historically:

```bash
terraform taint aws_instance.web
```

Modern approach:

```bash
terraform plan -replace="aws_instance.web"
terraform apply -replace="aws_instance.web"
```

### Purpose

Force Terraform to replace a managed resource.

---

# 15. Terraform Lifecycle

Lifecycle controls how Terraform manages resource changes.

### create_before_destroy

```hcl
lifecycle {
  create_before_destroy = true
}
```

Creates the replacement before destroying the existing resource.

### prevent_destroy

```hcl
lifecycle {
  prevent_destroy = true
}
```

Provides protection against Terraform destroying a critical resource.

### ignore_changes

```hcl
lifecycle {
  ignore_changes = [
    tags
  ]
}
```

Tells Terraform not to reconcile selected attributes.

### Interview Tip

> I use `ignore_changes` carefully because excessive use can hide meaningful configuration drift.

---

# 16. Terraform Modules

A Terraform module is a reusable collection of Terraform configuration.

### Structure

```text
terraform-project/
│
├── main.tf
├── variables.tf
│
└── modules/
    └── ec2/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
```

### Call Module

```hcl
module "web_server" {
  source = "./modules/ec2"

  instance_type = "t3.micro"
  environment   = "dev"
}
```

### Interview Answer

> Terraform modules allow platform teams to create standardized and reusable infrastructure components. Instead of duplicating infrastructure code across applications and environments, teams consume approved modules and provide environment-specific inputs.

### Root vs Child Module

```text
Root Module
    ↓
Child Module
```

The directory where Terraform is executed is the **root module**.

Modules called from it are **child modules**.

---

# 17. Terraform Provisioners

Provisioners execute commands or copy files during resource creation/destruction.

Types:

```text
local-exec  → Runs on Terraform machine

remote-exec → Runs on remote machine

file        → Copies files to remote machine
```

### remote-exec Example

```hcl
provisioner "remote-exec" {
  inline = [
    "sudo dnf install httpd -y",
    "sudo systemctl enable --now httpd"
  ]
}
```

### Interview Answer

> Terraform provisioners can execute commands during resource provisioning, but I treat them as a last resort. Where possible, I prefer EC2 user data, cloud-init, configuration-management tools, or immutable images.

---

# 18. Terraform in CI/CD

Typical enterprise workflow:

```text
Developer
    ↓
Git Pull Request
    ↓
terraform fmt
    ↓
terraform validate
    ↓
Security / Policy Check
    ↓
terraform plan
    ↓
Review / Approval
    ↓
terraform apply
    ↓
AWS
```

### Interview Answer

> I would store Terraform code in Git and execute Terraform through CI/CD. Pull requests would run formatting, validation, security or policy checks, and `terraform plan`. Production changes would require approval before `terraform apply`, with state stored securely in a remote backend and state locking enabled.

---

# 19. Scenario-Based Questions

## Q1. Someone manually modifies Production infrastructure. What do you do?

> I first run `terraform plan` to identify the drift and determine whether the manual change was authorized. If accidental, I restore the desired configuration using Terraform. If intentional, I update the Terraform code through Git and the standard review process so IaC remains the source of truth.

---

## Q2. Two Jenkins pipelines run Terraform simultaneously. What happens?

> Both pipelines should use the same remote backend with state locking. One operation acquires the lock, while the other is prevented from concurrently modifying the same state.

---

## Q3. Someone deletes the Terraform state file. What do you do?

> With an S3 remote backend, I would use S3 versioning and recovery controls to restore the appropriate state version. Before running another apply, I would verify that the recovered state correctly maps to the existing infrastructure.

---

## Q4. Multiple teams need the same VPC configuration. What would you do?

> I would create a reusable and versioned Terraform VPC module containing organizational networking and security standards. Teams can consume the module with environment-specific inputs rather than creating independent VPC configurations.

---

## Q5. How would you manage Dev, Test and Production?

```text
             Reusable Modules
                    |
          +---------+---------+
          |         |         |
         DEV       TEST      PROD
          |         |         |
       State A   State B   State C
```

> I would use reusable modules with environment-specific inputs and isolated state for each environment. Production changes would go through Git review, Terraform plan, approval, and controlled CI/CD deployment.

---

## Q6. SSH timeout during remote-exec. What do you check?

Check:

- Security Group
- Port 22
- NACL
- Route Table
- Public/Private IP
- Internet Gateway/NAT/access path
- SSH service

Test:

```bash
nc -vz <IP> 22
```

---

## Q7. SSH authentication fails. What do you check?

Check:

- SSH username
- Private key
- EC2 key pair
- Matching public key
- File permissions

Amazon Linux:

```text
ec2-user
```

Ubuntu:

```text
ubuntu
```

---

## Q8. remote-exec connects but `apt-get` isn't available.

Check:

```bash
cat /etc/os-release
```

Amazon Linux:

```bash
sudo dnf install httpd -y
```

Ubuntu:

```bash
sudo apt-get install apache2 -y
```

---

# 20. Quick Revision

| Concept | Remember |
|---|---|
| Provider | WHERE |
| Resource | WHAT |
| Variable | VALUE |
| State | Tracks managed infrastructure |
| Backend | Where state is stored |
| Locking | Prevent concurrent state modification |
| Drift | Actual ≠ Desired |
| Refresh | Synchronize state view |
| Import | Bring existing resource into state |
| Replace | Recreate resource |
| Lifecycle | Control resource behavior |
| Module | Reusable infrastructure |
| Provisioner | Execute commands/files |
| Plan | Preview changes |
| Apply | Implement changes |

---

# ⭐ 30-Second Terraform Interview Answer

> I use Terraform for Infrastructure as Code to automate and standardize AWS infrastructure provisioning. I'm comfortable with providers, resources, variables, outputs, reusable modules, remote state, state locking, lifecycle rules, imports, drift detection, and CI/CD integration. In team environments, I prefer secured remote state with locking and review infrastructure changes through `terraform plan` before applying them. My focus is on making infrastructure repeatable, version-controlled, secure, and reusable.

---

# 🎯 Interview Keywords

Remember to use these naturally:

`Infrastructure as Code`

`Reusable Modules`

`Remote State`

`State Locking`

`Configuration Drift`

`Source of Truth`

`Least Privilege`

`CI/CD`

`Automation`

`Standardization`

`Developer Self-Service`

`Infrastructure Governance`

`High Availability`

`Environment Isolation`

`Git-based Change Management`

---

**Next Topic:** AWS Interview Preparation — VPC, IAM, EC2, S3, ALB, ASG, Route53, ECR, EKS and CloudWatch.
