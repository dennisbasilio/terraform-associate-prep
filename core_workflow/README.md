# Core Workflow

This section will focus on exam objective 6.

## Core Terraform Workflow

Basic three steps:
1. Write - Author IaC
2. Plan - Preview changes before applying
3. Apply - Provision reproducible infrastructure

Workflow varies depending if working individually or as a team or in Terraform Cloud.

###  Individual

#### Write
Set up initial repository and configurations.
```
# Create repository
$ git init my-infra && cd my-infra

Initialized empty Git repository in /.../my-infra/.git/

# Write initial config
$ vim main.tf

# Initialize Terraform
$ terraform init

Initializing provider plugins...
# ...
Terraform has been successfully initialized!
```
Continue to edit and run plans to flush out syntax errors and everything works as expected.
```
# Make edits to config
$ vim main.tf

# Review plan
$ terraform plan

# Make additional edits, and repeat
$ vim main.tf
```

#### Plan
Commit work to repository.
```
git add main.tf
git commit -m 'Managing infrastructure as code!'
```
Apply to display plan to conduct final review.
```
terraform apply
```

#### Apply
Once done reviewing, just enter yes to perform actions.
```
Do you want to perform these actions?

  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.
  Enter a value: yes

# ...

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.

```
From here, you can store configuration in remote repository
```
git remote add origin https://github.com/*user*/*repo*.git
git push origin main
```

### Team
New steps are added to each part to ensure everyone works together smoothly.

#### Write
Create separate working branches where each person can work on their own version. Ideally there would be some kind of CICD pipeline..

#### Plan
Plan outputs allow gor team members to review each other's work. Done in pull requests where an individual proposes a merge of their working branch to the main branch. Should include the output of the plan in the PR for others to review

#### Apply 
Once the PR is approved and merged, a final concrete plan should be ran against the shared team branch and the latest state file to review any potential differences that may be caused by issues like merge order or recent infrastructure changes.

Also good to watch the apply output as it is happening. Can be done by sharing a screen or watching the build log

### Terraform Cloud
Meant to make it easier for teams and organizations to collaborate.

#### Write
Terraform Cloud provides a centralized and secure location for storing input variables and state whil also having a tight feedback loop. To interact with Terraform Cloud, use the following configuration:
```
terraform {
  cloud {
    organization = "my-org"
    hostname = "app.terraform.io" # Optional; defaults to app.terraform.io

    workspaces {
      tags = ["networking", "source:cli"]
    }
  }
}
```
All that is needed is an API key.

#### Plan
Plans are automatically ran when the PR is created. Once completed, the status update indicates whether there were any changes in the speculative plan. For deeper review, you can review and analyze the full plan details

#### Apply
After merging, Terraform Cloud presents the concrete plan for final review and approval. Once confirmed, the progress is displayed live for anyone to watch.

## Command: init
Used to initialize a working directory containing Terraform configuration files.
* First command that should be run after writing a new Terraform file or cloning an existing one from version control.
* Safe to run multiple times.

### Usage
`terraform init [options]`

#### General Options
* `-input=true` - Ask for input if necessary. If false, will error if input was required.
* `-lock=false` - Disable locking of state files during state-related operations.
* `-lock-timeout=<duration>` - Override the time Terraform will wait to acquire a state lock. Default is `0s` which causes immediate failure if the lock is alrady held by another process.
* `-no-color` - Disable color codes in the command output.
* `-upgrade` - Opt to upgrade modules and plugins as part of their respective installation steps.

#### Copy a Source Module
By default, `terraform init` assumes configuration files are already in the working directory and will attempt to initialize it.

Use the  `-from-module=MODULE-SOURCE` option to copy the module to the working directory before initialization. This is done for two use cases:
1. When given a version control source, can serve as a shorthand for checking out from version control and initializing the working directory (not recommended and should use the version control system's own commands)
2. The source refers to example configurations to be used as a basis for a new configuration

#### Backend Initialization
The chosen backend is initialized using the configuration settings from the root configuration directory. Re-running init with an already initialized backen will update the working directory to use the new backend settings.
* To update the backend configuration, use either the `-reconfigure` or `-migrate-state` options
	* `-migrate-state`  attempts to copy existing state to the new backend. Depending on what is changed, this may result in interactive prompts to confirm migration of workspace states
		* Use `-force-copy` to suppress these prompts and answer "yes" to all of them
	* `-reconfigure` disregards any existing configuration, preventing migration of any existing state
To skip backend configuration, use `-backend=false`. Some other init states require an initialized backend, so this should only be used on an already initialized working directory.

In situations where backend settings are dynamic or sensitive (cannot be statically specified in the config), use the `-backend-config=...` for partial backend configuration

#### Child Module Installation
During init, the configuration is searched for `module` blocks, and the source code for the referenced modules is retrieved from the locations given in their `source` arguments.
Re-running init will install sources for any modules added since the last init but will not change already installed modules. The `-upgrade` option overrides this.

To skip child module installation, use `-get=false`. Note that some other init steps can only be completed when the module tree is completed, so only use this option when the working directory was already previously initialized with its child modules.

#### Plugin Installation
During init, direct and indirect references to providers are searched for to attempt to install the plugins for those providers. If the provider is published in the public Terraform Registry or in a third-party registry, `terraform init` will automatically find, download, and install it.

After successful installation, the information about the selecgted providers is written to the dependency lock file. This should be commited to version control to ensure that when `terraform init` is ran in the future, it will select exactly the same provider versions. The `-upgrade` option ignores the dependency lock file and installs the newer versions

General options include:
* `-upgrade` - Noted above
* `-get-plugins=false` -Skip plugin installation (superseded by the provider_installation and plugin_cache_dir settings and will be removed in Terraform 0.15)
* `-plugin-dir=PATH` - Force plugin installation to read plugins only from the specified directory as if it had been configured as a `filesystem_mirror`
* `-lockfile=MODE` - Set a dependency lock file mode
	* The valid value for the lock file mode is readonly, which suppresses the lockfile changes, but verifies checksums against the information already recorded. Conflicts with `-upgrade`. Useful  if the lock file is updated with third-party dependency management tools.

#### Running `terraform init` in Automation
There are special considersations for automation such as optionally making plugins available locally to avoid repeated re-installation. More info [here](https://developer.hashicorp.com/terraform/tutorials/automation/automate-terraform).

#### Passing a Different Configuration Directory
In v0.13 and earlier, you used to be able to specify a directory path in place of the plan file argument to `terraform apply`, which would cause Terraform to use that directory as the root module instead of the current working directory.

This will be removed in v0.15. If overriding the root module directory is still needed, the `-chdir` global option can be used across all commands and makes Terrform consistently look in the given directory for all files it would normally look for in the current working directory.

If also relying on Terraform writing the `.terraform` subdirectory into the current working directory even though the root module directory was overridden, the `TF_DATA_DIR` environment variable can specify where to write the `.terraform` directory.

## Command: validate
Validates the configuration files in a directory, referring only to the configuration and not accessing any remote services such as remote state, provider APIs, etc.

Verifies whether a configuration is syntactically correct and internall consistent, regardless of variables or existing state. Safe to run automatically, but requires an initialized working directory with any referenced plugins and modules installed.

### Usage
`terraform validate [option]`

### General Options
* `-json` - Produce output in JSON format (always disables color).
* `-no-color` - If specified, output won't contain any colors.

### JSON Output Format
Using the `-json` option could potentially produce errors, which should be taken into account for external software and treated as a generic error case.

The `format_version` starts at version 1.0. The semantics are:
* Minor upgrades (1.1) are for backward-compatible changes or additions. Ignores any project properties with unrecognized names to remain forward-compatible with future minor versions.
* Major upgrades (2.0) are for changes that are not backward-compatible. Rejects any input which reports an unsupported major version.

Normally output is printed to standard output stream. The top-level JSON object contains the following properties:
* `valid` (boolean) - `true` if configuration is valid and `false` if any errors detected
* `error_count` (number) - Count of errors. Always 0 when `valid` is `true`.
* `warning_count` (number) - Count of warnings. Configuration is not considered invalid, but could indicate potential caveats that the user should consider and potentially resolve.
* `diagnostics` (array of objects) - Contains nested object that describes each error and warning.

The `diagnostics` has the following properties:
* `severity` (string) - Valid entries are `error` and `warning`. Future versions could introduce new severities, so those should be taken into account.
* `summary` (string) - Description of the problem.
* `detail` (string) - Gives more details of the problem.
* `range` (object) - Optional object referencing a portion of the config source code that the diagnostic message relates to.
	* For errors, typically indicate the bounds of the specific block header, attribute, or expression which was detected as invalid.
	* A source range is an object with a property `filename` which has the filename given as a relative path from the current working directory.
		* The `start` and `end` properties are objects describing source positions.
	* Not all diagnostic messages are connected with specific portions of the configuration
		* `range` will be omitted or `null`.
	* The source position object has the following properties
		* `byte` (number) - A zero-based byte offset into the indicated file.
		* `line` (number) - A one-based line count for the line containing the relevant position in the indicated file.
		* `column` (number) - A one-based count of _Unicode characters_ from the start of the line indicated in `line`.
* `snippet` (object) - Optional object that includes an excerpt of the config source code the diagnostic message relates to. Includes:
	* `context` (string) - Optional summary of the root context of the diagnostic. Could be the resource block containing the expression which triggered the diagnostic.
	* `code` (string) - A snippet of Terraform configuration including the source of the diagnostic.
	* `start_line` (number) - Line number where the `code` line starts. Could be different from the `range.start.line` as `code` can include lines before source of diagnostic for context.
	* `highlight_start_offset` (number) - Points at the start of the expression which triggered the diagnostic.
	* `highlight_end_offset` (number) - Points at the end of the expression which triggered the diagnostic.
	* `values` (array of objects) - Zero or more expression values that can help in understanding the source of a diagnostic in a complex expression. Expression values has two properties:
		* `traversal` (string) - An HCL-like traversal string, such as `var.instance_count`. Complex index key values may be elided, so this will not always be valid, parseable HCL.
		* `statement` (string) - Describes the value of the expression when the diagnostic was triggered.

## Command: plan
Creates an execution plan, which lets you preview the changes that Terraform plans to make to your infrastructure. Does not actually carry out proposed changes.

When a plan is created:
* Reads the current state of any already-existing remote objects to make sure that the Terraform state is up-to-date.
- Compares the current configuration to the prior state and noting any differences.
- Proposes a set of change actions that should, if applied, make the remote objects match the configuration.
## Command: apply
## Command: destroy
## Command: fmt