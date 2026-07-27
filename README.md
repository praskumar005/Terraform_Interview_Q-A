# Terraform_Interview_Q-A
All Terraform Topics are covered

Terraform Interview Questions & Strong Answers
1. What is Terraform?

Terraform is an Infrastructure as Code tool developed by HashiCorp that allows us to provision and manage infrastructure declaratively using HCL. Instead of manually creating resources through the AWS Console, we define the desired infrastructure as code, version-control it in Git, review changes using terraform plan, and provision them using terraform apply.

Example:

resource "aws_instance" "web" {
  ami           = "ami-xxxx"
  instance_type = "t3.micro"
}

Key words: IaC, declarative, automation, repeatability, version control.

2. Why Terraform over AWS CloudFormation?

Both are strong IaC solutions. CloudFormation is AWS-native, whereas Terraform provides a consistent workflow across AWS and many other providers. Terraform also has strong module reusability and an explicit planning and state-management workflow. In an enterprise environment, reusable Terraform modules can help standardize infrastructure across multiple teams and environments.

Don't say:

"Terraform is better than CloudFormation."

Say:

"The choice depends on the organization's requirements."

That sounds more experienced.

3. What extension do Terraform files use?

Terraform configuration files normally use the .tf extension and are primarily written in HCL.

Common structure:

main.tf
provider.tf
variables.tf
outputs.tf
terraform.tfvars
4. What is a Terraform Provider?

A provider is a plugin that allows Terraform to communicate with an external platform through its API. For example, the AWS provider enables Terraform to manage AWS resources such as EC2, VPC, S3, IAM, and EKS.

Example:

provider "aws" {
  region = "us-east-1"
}

Remember:

Provider = WHERE

5. What is a Terraform Resource?

A resource block represents an infrastructure object that Terraform creates or manages. For AWS, resources could include EC2 instances, S3 buckets, VPCs, Security Groups, IAM roles, or Load Balancers.

Example:

resource "aws_instance" "web" {
  ami           = "ami-xxxx"
  instance_type = "t3.micro"
}

Remember:

Resource = WHAT

6. What are Terraform Variables?

Variables allow us to parameterize Terraform configurations instead of hardcoding values. This makes the same Terraform code reusable across environments such as Development, Test, and Production.

Example:

variable "instance_type" {
  type    = string
  default = "t3.micro"
}

Use:

instance_type = var.instance_type
7. Explain the Terraform workflow.

One of the most important questions.

terraform init
terraform fmt
terraform validate
terraform plan
terraform apply

Answer:

I first run terraform init to initialize the working directory and download providers and modules. I use terraform fmt for formatting and terraform validate to validate the configuration. Then terraform plan allows me to review proposed infrastructure changes before I run terraform apply.

For cleanup:

terraform destroy
8. What does terraform init do?

terraform init initializes the Terraform working directory. It downloads required providers and modules and initializes the configured backend. It is normally the first command I run after cloning or creating a Terraform project.

9. What is terraform plan?

terraform plan compares the desired Terraform configuration with Terraform's current understanding of the infrastructure and displays the changes Terraform intends to make. It allows us to review additions, modifications, replacements, or deletions before applying them.

Very important in production.

In CI/CD, I would generate and review the plan before allowing production infrastructure changes.

10. What is Terraform State?

Terraform state is Terraform's record of the infrastructure resources it manages and the mapping between Terraform resource addresses and real infrastructure objects. By default, it is stored in terraform.tfstate.

Example:

Terraform Code
      ↓
Terraform State
      ↕
AWS Infrastructure

Important:

State can contain sensitive information, so it should be secured appropriately and should not normally be committed to Git.

11. Why shouldn't terraform.tfstate be stored in Git?

State may contain sensitive infrastructure information and changes frequently. Git also doesn't provide Terraform's state-locking semantics. For team environments, I prefer a secured remote backend such as S3 with encryption, access controls, versioning, and locking.

Excellent interview point.

12. What is a Terraform Remote Backend?

A remote backend allows Terraform state to be stored centrally rather than on an individual engineer's machine. In AWS, S3 is commonly used as a remote backend. This enables collaboration between engineers and CI/CD pipelines while providing centralized state management.

Example:

terraform {
  backend "s3" {
    bucket       = "company-terraform-state"
    key          = "prod/terraform.tfstate"
    region       = "us-east-1"
    use_lockfile = true
  }
}
13. How would you secure an S3 Terraform backend?

A very good senior-level follow-up.

I would block public access, enable encryption, enable S3 versioning for state recovery, restrict bucket access through least-privilege IAM policies, enable state locking, and separate state appropriately by application and environment.

14. What is Terraform State Locking?

State locking prevents multiple Terraform operations from modifying the same state simultaneously. When one process acquires the lock, another conflicting operation cannot modify that state until the lock is released. This protects against concurrent updates and possible state corruption.

Think:

Engineer A → Lock acquired → Apply
Engineer B → Cannot acquire same lock
15. What is use_lockfile = true?

For the S3 backend, use_lockfile = true enables S3-based state locking. Terraform creates a lock object while a state-changing operation holds the lock and removes it when the lock is released.

Example:

backend "s3" {
  bucket       = "terraform-state"
  key          = "prod/terraform.tfstate"
  region       = "us-east-1"
  use_lockfile = true
}
16. How do you verify state locking is actually working?

This is an excellent practical interview question.

I can start a Terraform operation against a particular state from one terminal and then attempt another conflicting operation against the same state from another terminal or pipeline. The second process should be unable to acquire the lock. With S3 lockfile-based locking, I can also inspect the backend while the lock is held and observe the .tflock object.

This demonstrates hands-on understanding.

17. What is Terraform Drift?

Configuration drift occurs when the real infrastructure differs from the desired Terraform configuration, usually because someone modified a Terraform-managed resource outside the normal Terraform workflow.

Example:

Terraform:

t3.micro

Someone manually changes AWS:

t3.medium

Now you have:

Terraform → t3.micro
AWS       → t3.medium
             ↑
            Drift
18. How do you detect Terraform drift?

I normally use terraform plan. Terraform refreshes its view of managed resources and compares the actual infrastructure against the desired configuration. If there is a difference, the plan shows what Terraform proposes to change.

19. What would you do if you detected drift in Production?

This answer can impress an interviewer:

I wouldn't immediately run terraform apply. First, I would determine why the change occurred and whether it was authorized. If it was accidental, I would use the approved Terraform workflow to restore the desired configuration. If the infrastructure change was intentional, I would update the Terraform configuration through version control and review so that IaC remains the source of truth.

Excellent phrase:

"IaC should remain the source of truth."

20. What is Terraform Refresh?

Refresh synchronizes Terraform's state representation with the actual infrastructure. The standalone terraform refresh command is deprecated, so I would use terraform plan -refresh-only to review refresh-only changes and terraform apply -refresh-only when I intentionally need to update the state without changing remote resources.

21. Difference between Drift and Refresh?

Drift is the difference between desired configuration and actual infrastructure. Refresh is the process Terraform uses to update its state representation based on actual infrastructure.

Simple:

Drift   = Difference
Refresh = Synchronization of state
22. What is terraform import?

Terraform import is used to associate an existing infrastructure resource with a Terraform resource address so that Terraform can begin managing it.

Example:

An EC2 was manually created:

i-0123456789

Define:

resource "aws_instance" "web" {
}

Import:

terraform import aws_instance.web i-0123456789

Then:

terraform plan
23. Import vs Refresh?

Import brings an existing resource under Terraform state management. Refresh synchronizes Terraform's state representation for resources it already manages with their actual remote values.

Remember:

Import  → Existing resource → Terraform
Refresh → Existing managed resource → Update state view
24. What is Terraform Taint?

Historically, terraform taint was used to mark a resource for replacement even when its configuration hadn't changed. The command is deprecated, and the preferred approach today is -replace.

Example:

terraform plan -replace="aws_instance.web"
terraform apply -replace="aws_instance.web"

Use case:

"If a VM has become corrupted and needs replacement, I can explicitly request Terraform to replace that managed resource."

25. What is Terraform Lifecycle?

The lifecycle block allows us to customize how Terraform handles creation, replacement, destruction, and selected changes for a resource.

Important options:

lifecycle {
  create_before_destroy = true
}
lifecycle {
  prevent_destroy = true
}
lifecycle {
  ignore_changes = [
    tags
  ]
}
26. Explain create_before_destroy.

It tells Terraform to create the replacement resource before destroying the existing resource when replacement is required, where the resource type and constraints permit it. This can help reduce downtime.

27. Explain prevent_destroy.

prevent_destroy causes Terraform to reject a plan that would destroy the protected resource while that lifecycle rule is present. I would consider it as an additional safeguard for critical resources, but it doesn't replace proper IAM controls, backups, and change management.

That last sentence is a strong interview point.

28. Explain ignore_changes.

ignore_changes tells Terraform not to reconcile specified attributes after resource creation. It can be useful when an attribute is intentionally managed by another system.

Example:

lifecycle {
  ignore_changes = [tags]
}

But mention:

I use it carefully because excessive ignore_changes can hide meaningful drift.

Excellent interview point.

29. What are Terraform Modules?

One of the most important questions for Southwest.

A Terraform module is a reusable collection of Terraform configuration that encapsulates infrastructure patterns. Instead of duplicating VPC or EC2 definitions for every application or environment, we create standardized modules and expose configurable inputs and outputs.

Example:

module "web_server" {
  source = "./modules/ec2"

  instance_type = "t3.micro"
  environment   = "dev"
}
30. Root Module vs Child Module?

The configuration from which Terraform is executed is the root module. A module called by another module is a child module.

Root Module
    ↓
VPC Module
    ↓
EC2 Module
31. Why are modules important in an enterprise?

This is very relevant to Southwest's role.

Modules allow platform teams to encode organizational standards into reusable infrastructure components. For example, instead of every developer independently configuring an EC2 instance, networking, tags, encryption, and security controls, a platform team can provide an approved module. Developers supply only the required inputs while the module enforces common standards.

Then connect it to platform engineering:

"This improves developer productivity while maintaining governance and consistency."

That's an excellent phrase for this job.

32. What is a Terraform Provisioner?

Provisioners allow Terraform to execute commands or transfer files during resource creation or destruction. Common provisioners include local-exec, remote-exec, and file. However, Terraform recommends treating provisioners as a last resort.

33. local-exec vs remote-exec?

local-exec

Runs on:

Terraform execution machine

remote-exec

Runs on:

Remote resource

Example:

provisioner "remote-exec" {
  inline = [
    "sudo dnf install httpd -y"
  ]
}
34. Why should Terraform Provisioners be avoided where possible?

Provisioners introduce imperative configuration into a declarative infrastructure workflow and can be difficult to make reliably idempotent. Where possible, I prefer mechanisms such as cloud-init/user data, immutable images, configuration-management tools, or provider-native resources.

That's a strong answer.

35. What happens when remote-exec fails?

Terraform reports the provisioning failure and the apply fails by default. For a creation-time provisioner failure, Terraform can consider the resource incomplete and plan replacement on a subsequent run. I would troubleshoot the underlying connectivity, authentication, or script problem rather than simply bypassing the failure.

36. remote-exec says SSH timeout. What would you check?

Your actual troubleshooting scenario.

I would check whether port 22 is reachable from the Terraform runner, then verify the Security Group, NACL, subnet route table, public/private addressing, and SSH service. If it's a private instance, I'd also verify the intended access path such as a bastion, VPN, or SSM-based approach.

Example:

nc -vz <IP> 22
37. SSH connection works but authentication fails. What do you check?

If TCP/22 is reachable but SSH authentication fails, I would verify the SSH username, private key, EC2 key pair, file permissions, and whether the matching public key was installed on the instance.

Amazon Linux:

ec2-user

Ubuntu:

ubuntu

This is exactly the issue you encountered.

38. remote-exec connects but apt-get isn't found. Why?

That indicates an OS/package-manager mismatch. For example, apt-get is used by Debian/Ubuntu, whereas Amazon Linux 2023 uses dnf. I would first verify the operating system using /etc/os-release and then use the correct package manager and service names.

Amazon Linux:

sudo dnf install httpd -y
sudo systemctl enable --now httpd

Ubuntu:

sudo apt-get install apache2 -y

This is a very good real troubleshooting example for you to mention.

39. How would you manage Terraform across Dev, Test and Production?

Very important.

I would keep reusable infrastructure logic in version-controlled modules and provide environment-specific configuration through variables or separate root configurations. Each environment should have isolated state, appropriate IAM permissions, and controlled CI/CD workflows. Production changes should go through pull-request review, terraform plan, approval gates, and then terraform apply.

Architecture:

Reusable Modules
       │
 ┌─────┼─────┐
 ↓     ↓     ↓
DEV   TEST  PROD
 ↓     ↓     ↓
Separate Terraform State
40. How would you implement Terraform in CI/CD?

Strong Southwest answer:

I would store Terraform code in Git and trigger a pipeline through pull requests. The pipeline would run formatting, validation, security/policy checks, and terraform plan. The plan would be reviewed, and production changes would require an approval gate before terraform apply. State would be maintained in a secured remote backend with locking enabled.

Conceptually:

Developer
   ↓
Git Pull Request
   ↓
terraform fmt
   ↓
terraform validate
   ↓
Security / Policy Checks
   ↓
terraform plan
   ↓
Code Review / Approval
   ↓
terraform apply
   ↓
AWS
5 Scenario Questions I Expect for Southwest

These are the questions I'd particularly prepare.

Scenario 1: "Five development teams need VPCs with the same security standards. How would you implement this?"

I would build a reusable, versioned Terraform VPC module containing the organization's approved networking, tagging, logging, and security defaults. Teams would consume the module and provide only approved inputs. This gives developers self-service capabilities while the platform team maintains governance and consistency.

This connects directly with their Golden Path/platform engineering philosophy.

Scenario 2: "An engineer manually changes Production infrastructure. What would you do?"

I would use terraform plan to identify the drift, investigate whether the change was authorized, and avoid blindly applying the plan. If accidental, I would restore the desired configuration through Terraform. If intentional, I would update the IaC through Git and the normal review process so Terraform remains the source of truth.

Scenario 3: "Two Jenkins pipelines execute Terraform against Production simultaneously."

They should use the same remote backend with state locking. One operation should acquire the lock while the other is prevented from concurrently modifying the same state. I would also design the CI/CD process to serialize production infrastructure deployments where appropriate.

Scenario 4: "Someone accidentally deletes your Terraform state."

With an S3 backend, I would rely on S3 versioning and appropriate backup/recovery controls to recover a previous state version. Before performing further applies, I would verify that the recovered state accurately maps to the existing infrastructure.

Scenario 5: "You have 100 developers. Would you allow all of them to write Terraform?"

I would provide reusable, approved Terraform modules and developer workflows rather than requiring every developer to understand every infrastructure detail. Developers could consume standardized modules through templates or Golden Paths, while the platform team owns the underlying standards, security controls, and module lifecycle.

That answer is particularly strong for this Southwest Platform & AI Accelerators position.

Your 30-Second Terraform Summary

If the interviewer simply says:

"Tell me about your Terraform experience."

You can answer:

"I use Terraform for Infrastructure as Code to automate and standardize AWS infrastructure provisioning. I'm comfortable working with providers, resources, variables, outputs, reusable modules, remote state, state locking, lifecycle rules, imports, drift detection, and Terraform integration with CI/CD. I prefer reusable modules and environment-specific configuration rather than duplicated infrastructure code. For team environments, I use secured remote state with locking and review infrastructure changes through terraform plan before applying them. My focus is on making infrastructure repeatable, version-controlled, secure, and easy for development teams to consume."

That is the answer I'd practice until you can deliver it naturally without memorizing word-for-word.

For this Southwest role, your next preparation priority should be AWS VPC → IAM → EC2/ALB/ASG → S3 → ECR/EKS → Jenkins CI/CD → Docker → Kubernetes → Platform Engineering/Backstage. Terraform is only one major part of the interview; the JD strongly suggests they'll test how you combine these technologies into an enterprise platform.
