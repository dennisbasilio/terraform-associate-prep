# Infrastructure as Code (IaC) Basics

This section will focus on exam objectives 1 and 2.

## What is IaC?

Infrastructure that is defined as code in definition files that is both human and machine readable.

Standardizes workflow across different providers (AWS, Azure, GCP, etc.) using common syntax. Can span multiple files to allow for better organization based off the intent

### Infrastructure Lifecycle
IaC can be applied throughout the infrastructure lifecycle from the initial build through the life of the infrastructure.
* Day 0 Activities - Provisions and configures initial infrastructure
* Day 1 Activities - OS and application configurations applied after the initial build of the infrastructure
 
### Reliability
IaC allows for idempotent, consistent, repeatable, and predictable provisioning of infrastructure. You can test code and review it prior to it being applied to the environment. Version controlling allows you to review how the infastructure changes over time.

### Manageability
IaC allows for scaling up or down an environment depending on the needs of the application or infrastructure. Observing the current state of an environment when compared to the desired state determines if changes need to be made in the infrastructure. 

## Terraform
Hashicorp's IaC offering. Creates and manages resources (on-prem or in the cloud) and services via their respective APIs.

### Workflow
Three stages:
1. Write - Define infrastructure in configuration files
2. Plan - Review the changes that will be made
3. Apply - Provision your infrastructure and update the state file

### Key Features
* Management
	* Find Providers for various platforms and services
	* Can look in [Terraform Registry](https://registry.terraform.io/)
* Tracking
	* Track current state of infrastructure in state file
	* Used to determine what changes need to be made in infrastructure to match configuration
* Automation
	* Configuration files are declarative describing desired state of environment
	* Terraform handles underlying logic to get to that state
	* Builds a resource graph to determine resource dependencies and creates or modifies non-dependent resources in parallel
* Standardization
	* Uses modules (reusable configuration components) to define collections of infrastructure
	* Saves time and encourages best practices
* Collaboration
	* Do version controlling of infrastructure
	* Use Terraform Cloud to manage workflows
		* Provides secure access to shared state and secret data, role-based access controls, a private registry for sharing both modules and providers, and more

### Use Cases
More info on the below use cases found [here](https://developer.hashicorp.com/terraform/intro/v1.1.x/use-cases):
* Multi-cloud Deployment
* Application Infrastructure Deployment, Scaling, and Monitoring Tools
* Self-Service Clusters
* Policy Compliance and Management
* PaaS Application Setup
* Software Defined Networking
* Kubernetes
* Parallel Environments
* Software Demos
