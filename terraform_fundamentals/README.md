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

## Providers

Plugins that Terraform relies on to interact with cloud providers, SaaS providers, and other APIs. Must be declared to be used and may require additional configurations before they can be used.

### Provider Installation

Installed as part of every run. Everytime you initialize a working directory, Terraform CLI finds and installs new providers.

Can specify `plugin_cache_dir` to setup optional plugin cache to save on time and bandwidth.

Can be found in the Terraform Registry, partner sites, or from the community.

### Provider Configuration

The `required_provider` declares what providers are needed:
```
terraform {
  required_providers {
    mycloud = {
      source  = "mycorp/mycloud"
      version = "~> 1.0"
    }
  }
}
```

The `provider` block specifies the provider configurations:
```
provider "google" {
  project = "acme-app"
  region  = "us-central1"
}
```

Can use the `alias` setting to define multiple configurations for the same provider:
```
# The default provider configuration; resources that begin with `aws_` will use
# it as the default, and it can be referenced as `aws`.
provider "aws" {
  region = "us-east-1"
}

# Additional provider configuration for west coast region; resources can
# reference this as `aws.west`.
provider "aws" {
  alias  = "west"
  region = "us-west-2"
}
```
* The block with the `alias` setting is the default
* In the `required_providers` block, specify the `configuration_aliases` setting, which would be specified as `<PROVIDER NAME>.<ALIAS>`

#### Alternate Provider Configuration

To refer to an alternate provider for a resource, specify the alias in the `provider` meta-argument (follows the `<PROVIDER NAME>.<ALIAS>` format):
```
resource "aws_instance" "foo" {
  provider = aws.west

  # ...
}
```

To refer to an alternate provider in a child module, use the `providers` meta-argument:
```
module "aws_vpc" {
  source = "./aws_vpc"
  providers = {
    aws = aws.west
  }
}
```


## Plugins
Terraform is built on a plugin-based architecture.

Two main components of Terraform: Terraform Core and Terraform Plugins.

### Terraform Core
Uses RPC to communicate with Terraform Plugins. Offers multiple ways to discover and load plugins to use.

Written in Go and is open source and found [here](https://github.com/hashicorp/terraform).

This is the CLI tool `terraform` primarily used for:
* IaC
* Resource state management
* Construction of the Resource Graph
* Plan execution
* Communication with plugins over RPC

### Terraform Plugins
Exposes an implementation for a specific service like AWS or provider like bash. Written in Go and invoked by Terraform Core via RPC. All Providers and Provisioners are Plugins. The primary responsibility for both are:
* Provider Plugins
	* Initialization of any included libraries used to make API calls
	* Authentication with the Infrastructure Provider
	* Define resources that map to specific Services
* Provisioner Plugins
	* Executing commands or scripts on the designated Resource after creation or on destruction

## Dependency Lock File
Version constraints determine which dependencies are potentially compatible. After selecting a version for each dependency, Terraform remembers these decisions in the Dependency Lock File so it makes the same decisions in the future.

If there is no existing recorded selection, Terraform will select the newest available that fits within the version constraints and then updates the lock file to include that selection. 

If a selection already exists, Terraform will always reuse that one even if a newer version is available. This can be overrided by adding the `-upgrade` flag to `terraform init`.

Only tracks provider dependencies. Version selections for remote modules are not remembered, so Terraform always uses the newest available version within the specified version constraints.

### Location
Located in the `.terraform` directory in the working directory, the file is called `.terraform.lock.hcl`. Automatically created and updated when `terraform init` is ran.

### Checksum Verification
Intended to represent a trust-on-first-use approach. Terraform will raise an error when it encounters a non-matching package for the same provider version. Two general cases:
* When a provider is installed from an origin repository, Terraform treats all signed checksums as valid as long as one of them is valid. This means that the lock file will include the checksums for both the package you installed for your current platform AND any other packages available for other platforms
	* `terraform init` includes the fingerprint of the key that signed the checksums
* When a provider is installed for the first time using an alternative installation method, Terraform will not be able to verify the checksums for other platforms other than where `terraform init` was ran. This means that it will not record the checksums for other platforms, so the configuration can't be used on other platforms.
	* Can be avoided by pre-populating checksums for other platforms in your lock file using the `terraform providers lock` command.

### Version Control
The Dependency Lock File should be included as part of version control to track changes in the different depency versions. Various scenarios where your version control system shows a change would include:
* Dependency on a new provider
	* Shows the version, constraints, and hashes
* New version of an existing provider
* New provider package checksum
	* Hashing schemes
		* zh: - zip hash, legacy hash format that captures the SHA256 hash of each .zip package indexed in the origin registry
		* h1: - hash scheme 1, captures the SHA256 hash of the contents of the provider distribution package, meaning it can work on the official .zip file, the unpacked directory, or recompressed .zip file.
* Providers that are no longer required