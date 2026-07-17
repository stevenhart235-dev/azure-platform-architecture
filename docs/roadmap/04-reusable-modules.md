# M3 - Reusable Module Platform

## Objective

Create a reusable library of Terraform modules that contain no
environment-specific configuration and can be consumed by deployment
repositories through immutable versions.

## Current Status

The initial reusable module work has produced three released modules:

- `resource-group-v0.1.0`
- `storage-account-v0.1.0`
- `storage-container-v0.1.0`

These modules support the next Foundation bootstrap milestone by providing the
resource group, storage account, and storage container building blocks required
for the initial Azure Blob Storage remote state backend.

## Work Items

- [x] Create module repository baseline
- [x] Build Resource Group module
- [x] Build Storage Account module
- [x] Build Storage Container module
- [ ] Continue module catalog expansion as later milestones require additional
      capabilities
- [ ] Build networking modules when the connectivity milestone begins

## Exit Criteria

- [x] Initial Foundation bootstrap modules are reusable
- [x] Initial Foundation bootstrap modules are versioned
- [x] Initial Foundation bootstrap modules have validation evidence
- [ ] Future modules continue to meet the Terraform module, versioning, and
      engineering validation standards
