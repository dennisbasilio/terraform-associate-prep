# terraform-associate-prep

The information within this repo will be based off of the study guide for the Terraform Associate Certification (003) found [here](https://developer.hashicorp.com/terraform/tutorials/certification-003/associate-study-003).

Even if I do not plan on going for the certification, this should serve as a good foundation for getting into Terraform. The goal is to also make this publicly available for those who may also be learning how to use Terraform.

## Exam Objectives
1. [Understand infrastructure as code (IaC) concepts](https://github.com/dennisbasilio/terraform-associate-prep/tree/main/iac_basics#infrastructure-as-code-iac-basics)
	* Explain what IaC is
	* Describe advantages of IaC patterns
2. [Understand the purpose of Terraform (vs other IaC)](https://github.com/dennisbasilio/terraform-associate-prep/tree/main/iac_basics#infrastructure-as-code-iac-basics)
	* Explain multi-cloud and provider-agnostic benefits
	* Explain the benefits of state
3. [Understand Terraform basics](https://github.com/dennisbasilio/terraform-associate-prep/tree/main/terraform_fundamentals#terraform-fundamentals)
	* Install and version Terraform providers
	* Describe plugin-based architecture
	* Write Terraform configuration using multiple providers
	* Describe how Terraform finds and fetches providers
4. Use Terraform outside of core workflow
	* Describe when to use `terraform import` to import existing infrastructure into your Terraform state
	* Use `terraform state` to view Terraform state
	* Describe when to enable verbose logging and what the outcome/value is
5. Interact with Terrform modules
	* Contrast and use different module source options including the public Terraform Module Registry
	* Interact with module inputs and outputs
	* Describe variable scope within modules/child modules
	* Set module version
6. Use the core Terraform workflow
	* Describe Terraform workflow (Write -> Plan -> Create)
	* Initialize a Terraform working directory (`terraform init`)
	* Validate a Terraform configuration (`terraform validate`)
	* Generate and review an execution plan for Terraform (`terraform plan`)
	* Execute changes to infrastructure with Terraform (`terraform apply`)
	* Destroy Terraform managed infrastructure (`terraform destroy`)
	* Apply formatting and style adjustments to a configuration (`terraform fmt`)
7. Implement and maintain state
	* Describe default `local` backend
	* Describe state locking
	* Handle backend and cloud integration authentication methods
	* Differentiate remote state back end options
	* Manage resource drife and Terraform state
	* Describe `backend` block and cloud integration in configuration
	* Understand secret management in state files
8. Read, generate, and modify configuration
	* Demonstrate use of variables and outputs
	* Describe secure secret injection best practice
	* Understand the use of collection and structural types
	* Create and differentiate `resource` and `data` configuration
	* Use resource addressing and resource parameters to connect resources together
	* Use HCL and Terraform functions to write configuration
	* Describe built-in dependency management (order of execution based)
9. Understand Terraform Cloud capabilities
	* Explain how Terraform Cloud helps to manage infrastructure
	* Describe how Terraform Cloud enables collaoration and governance
