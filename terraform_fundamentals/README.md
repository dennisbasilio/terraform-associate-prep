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
