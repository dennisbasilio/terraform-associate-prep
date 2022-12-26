# Terraform Fundamentals

This section will focus on exam objective 3.

## State

Required for Terraform to function. 

### Mapping

It is used to map configurations to resources. 

It also expects that each remote object is bound to only one resource instance to avoid ambiguous configurations.

### Metadata

Includes various information showing the relationship between various components such as resource dependencies.

A copy of the most recent dependencies is maintained within the state. This is important especially in deleting a resource because of the dependency order. A resource may need to be taken down a certain way, and if a configuration is deleted, the most recent copy is in the state.

Another similar metadata includes a pointer to the provider configuration that was most recently used (especially helpful where multiple aliased providers are present).

### Performance

Stores a cache of the attribute values for all resources. Completely optional.

Default behavior for when `terraform plan` and `terraform apply` is ran is to sync all resources in your state. This is slow in larger environements.

The cached state can be used as a source of truth.

### Syncing

States are stored locally in the current working directory where Terraform is ran.

In larger environments, a remote state is recommended, especially for remote locking to prevent multiple runnings of Terraform.

Ensures that Terraform runs with the most recent updated state.

## Terraform Settings

The `terraform` configuration block type contains configurations for Terraform itself.

### Terraform Block Syntax

```
terraform {
  # ...
}
```
* Only constant values can be used within this block
* Arguments may not refer to named objects such as resources, input variables, etc, as well as built-in functions
The following are options supported in the `terraform` block.

#### Configuring Terraform Cloud

The `cloud` block configures Terraform Cloud and enables its CLI-driven run workflow. Only needed when you want to use the CLI to interact with Terraform Cloud.
```
terraform {
  cloud {
    organization = "example_corp"

    workspaces {
      tags = ["app"]
    }
  }
}
```
* Cannot use CLI integration and state backend in the same configuration since they are mutually exclusive. That because Terraform Cloud already manages state

#### Configuring Terraform Backend

The `backend` block configures which state Terraform should use.
```
terraform {
  backend "remote" {
    organization = "example_corp"

    workspaces {
      name = "my-app-prod"
    }
  }
}
```
* There are various types of backends you can choose (remote in this example)
* Access credentials should NOT be included in the backend configuration and should instead be in credential files or environmental variables depending on the type of backend used
* Default backend is `local`.
* When changes are made, you must run `terraform init` to validate and configure the backend
	* Optionally you can migrate the current state to the new backend
	* You can manually backup the state as well by copying the `terraform.tfstate` file elsewhere
* A partial configuration is where some or all of the arguments are omitted
	* Desirable when some other automation script provides these arguments
	* Ways to supply remaining arguments include:
		* File - Passed as an option when running `terraform init` using `-backend-config=PATH` 
			* File must be local
			* Contains content of `backend` block without the need to wrap it in another `terraform` or `backend` block
			* Recommended naming pattern
				* \*.backendname.tfbackend
		* CLI key/value pair - Passed as an option when running `terraform init` using `-backend-config="KEY=VALUE"`
			* Not recommended for secrets due to CLI history
		* Interactively - Terraform prompts for required values unless disabled
			* No prompt for optional values
	* If settings come from multiple places, command-line options override settings in the main configuration, and command-line options are processed in order, overriding values from previous command-line options
		* The final merged configration is stored in the `.terraform`  directory, which should be ignored from version control
			* Sensitive information will be ignored by version control but present in plain text on local disk
	* Requres an empty backend configuration when using partial configuration
```
terraform {
  backend "consul" {}
}
```
* Changing or unconfiguring a backend can be done at anytime and does a reinitialization
	* Can migrate state when changing backend to new one
		* If reconfiguring the same backend, no need to do this
	* Unconfiguring a backend will ask to migrate current state down to normal local state

#### Specifying a Required Terraform Version

The `required_version` setting accepts a version constraint string.

Specifies which versions of Terraform to use with your configuration. If the version doesn't match, Terraform produces an error and exits. Only applies to the version of Terraform CLI.

Example:
```
version = ">= 1.2.0, < 2.0.0"
```
* = - Specifies exact version to use
* != - Excludes an exact version
* >, <, >=, <= - Comparison operators
* ~> - Allows the rightmost version to increment (if ~> 1.0.4, allows 1.0.5 and  1.0.10 but now 1.1.0)
	* Usually called the pessimistic operator

Child modules can specify their own version number and must all be satisfied.

#### Specifying Provider Requirements

The `required_providers` block specifies all of the required providers by the current module.

Maps local provider name to source address and version constraint:
```
terraform {
  required_providers {
    aws = {
      version = ">= 2.7.0"
      source = "hashicorp/aws"
    }
  }
}
```
* aws in this case is the local name

#### Experimental Language Features

Can test new language features by opting in. Can be enabled on a per-module basis by using the `experiments` setting.
```
terraform {
  experiments = [example]
}
```
* The above opts in to the `example` experiment 

#### Passing Metadata to Providers

The `provider_meta` block allows the provider to receive module-specific information.

## Manage Terraform Versions

