# M2 - Bootstrap and State

## Purpose

Milestone 2 establishes the minimum trusted Azure resources and state controls
required before normal Foundation deployment can begin.

Bootstrap creates the remote state backend used by the platform. It starts from
a constrained temporary local-state process only because the remote backend does
not exist yet. After the backend exists, bootstrap state is migrated to Azure
Blob Storage and local state is no longer an approved operating mode.

This milestone prepares the `azure-platform-foundation` repository to consume
released reusable modules and manage Foundation state through the accepted
remote state, identity, repository, and root deployment structure decisions.

## Scope

In scope:

- Foundation bootstrap documentation and state setup expectations.
- Consumption of immutable released modules from `azure-platform-modules`.
- Creation of the initial state resource group.
- Creation of the initial state storage account.
- Creation of the private Terraform state container.
- Temporary local-state use during first bootstrap only.
- Migration of bootstrap state to Azure Blob Storage.
- Authentication and authorization expectations for state access.
- Security expectations for the initial state backend.
- Prohibition of local state artifacts after migration.

Out of scope:

- Deploying management groups.
- Deploying Azure Policy assignments.
- Deploying platform subscriptions.
- Deploying observability resources.
- Deploying connectivity, hub networking, DNS resolver, firewall, routing, or
  spoke onboarding.
- Creating workload landing zones or subscription vending.
- Final production recovery testing.
- Private endpoint-only backend access before connectivity and private-capable
  runners exist.

## Prerequisites

Required architecture decisions and standards:

- [ADR 0001 - Infrastructure as Code Engine](../adr/0001-iac-engine.md)
- [ADR 0002 - Repository Separation](../adr/0002-repository-separation.md)
- [ADR 0003 - Terraform Toolchain Baseline](../adr/0003-terraform-toolchain-baseline.md)
- [ADR 0004 - Remote State Strategy](../adr/0004-remote-state-strategy.md)
- [ADR 0006 - Deployment Identity Strategy](../adr/0006-deployment-identity-strategy.md)
- [ADR 0008 - Root Deployment Repository Structure](../adr/0008-root-deployment-repository-structure.md)
- [State Management Standard](../standards/state-management-standard.md)
- [Repository Standard](../standards/repository-standard.md)
- [Versioning Standard](../standards/versioning-standard.md)

Required module releases:

- `resource-group-v0.1.0`
- `storage-account-v0.1.0`
- `storage-container-v0.1.0`

Required repository boundary:

- Bootstrap implementation belongs in `azure-platform-foundation`.
- Reusable module source remains in `azure-platform-modules`.
- This architecture repository remains documentation-only.

## Released Modules Consumed

Foundation bootstrap consumes immutable released module versions:

| Module | Version | Bootstrap purpose |
| --- | --- | --- |
| `resource-group` | `resource-group-v0.1.0` | Creates the resource group that contains the state backend resources. |
| `storage-account` | `storage-account-v0.1.0` | Creates the storage account used by the Terraform `azurerm` backend. |
| `storage-container` | `storage-container-v0.1.0` | Creates the private blob container used for Terraform state. |

Foundation must reference these modules by immutable tag or approved commit SHA.
Mutable branch references such as `main` are not acceptable for committed
bootstrap deployment code.

## Bootstrap Sequence

The accepted bootstrap sequence is:

1. Foundation consumes immutable released modules from
   `azure-platform-modules`.
2. Bootstrap creates the resource group for state backend resources.
3. Bootstrap creates the storage account for the Terraform `azurerm` backend.
4. Bootstrap creates the private state container.
5. Initial bootstrap may use local Terraform state because the remote backend
   does not exist yet.
6. After backend creation, bootstrap state is migrated to Azure Blob Storage.
7. Authentication uses Microsoft Entra ID, Azure RBAC, and workload identity
   federation for automation where available.
8. Storage account keys, SAS tokens, client secrets, and long-lived credentials
   are not normal authentication mechanisms.
9. Local state artifacts are removed after migration and prohibited from Git.

## Temporary Local-State Allowance

Temporary local state is allowed only for the first bootstrap operation that
creates the remote backend.

This allowance exists because Terraform cannot use a backend before the backend
resources exist. It must not become a normal operating pattern.

Requirements:

- Local state may contain only the bootstrap resources required to create the
  backend.
- Local state must not be committed.
- Local state backups must not be committed.
- Local plans that contain sensitive values must not be committed.
- After backend creation, bootstrap state must be migrated to Azure Blob
  Storage.
- After successful migration, local state artifacts must be removed from the
  working tree.

## Remote Backend Creation

The initial remote backend uses the native Terraform `azurerm` backend with
Azure Blob Storage.

Bootstrap creates:

- A dedicated resource group for state backend resources.
- A dedicated storage account for Terraform state.
- A private blob container for state.

The architecture does not define implementation-specific values such as real
subscription IDs, tenant IDs, resource group names, storage account names,
regions, or tag values. Those values belong in `azure-platform-foundation`
environment configuration.

## State Migration Path

The migration path is:

1. Run the first bootstrap with temporary local state.
2. Create the backend resource group, storage account, and private container.
3. Configure the bootstrap root backend according to the accepted backend
   configuration pattern.
4. Migrate bootstrap state from local state to Azure Blob Storage.
5. Confirm subsequent Terraform initialization uses the remote backend.
6. Remove local state artifacts.
7. Treat Azure Blob Storage as the authoritative state location for bootstrap
   and later Foundation operations.

No shared Foundation deployment should continue to rely on local state after
the backend exists.

## Authentication Expectations

Authentication uses Microsoft Entra ID.

Accepted patterns:

- Initial human bootstrap through an authenticated Entra user and Azure CLI.
- Future automation through GitHub OIDC and Microsoft Entra Workload Identity
  Federation.
- Azure RBAC for state data-plane access.

Normal workflows must avoid:

- Storage account keys.
- SAS tokens.
- Client secrets.
- Certificate credentials.
- Long-lived service principal credentials.
- Credentials committed to Git or stored in repository files.

## Security Expectations

State storage is a critical platform dependency.

Required expectations:

- State containers do not permit anonymous access.
- Public blob access is disabled.
- Authentication uses Entra ID.
- Authorization uses least-privilege Azure RBAC.
- Storage account keys and SAS tokens are not normal workflow dependencies.
- Bootstrap state is migrated to remote state after backend creation.
- State files, state backups, sensitive plan files, backend secrets, and local
  override files are not committed.
- State access is scoped separately from Azure resource deployment permission.
- State must be recoverable through documented storage protection and recovery
  practices.

The lab may initially use the public Azure endpoint for state access, as
accepted by ADR 0004. Private-only backend access is deferred until
connectivity and private-capable runners exist.

## Non-Goals

This milestone does not:

- Complete the full Foundation platform.
- Complete production readiness.
- Define real environment values.
- Create implementation code in this repository.
- Deploy governance, observability, management groups, or connectivity.
- Decide unresolved subscription vending, IPAM, DNS implementation, or policy
  rollout details.
- Require private endpoint-only state access before the platform can support
  it operationally.

## Exit Criteria

M2 is complete when:

- `azure-platform-foundation` has a bootstrap deployment unit aligned with ADR
  0008.
- Bootstrap consumes immutable released versions of the resource group,
  storage account, and storage container modules.
- The backend resource group exists.
- The backend storage account exists.
- The private state container exists.
- Initial local bootstrap state has been migrated to Azure Blob Storage.
- Local bootstrap state artifacts have been removed and remain prohibited.
- Backend access uses Entra ID and Azure RBAC for normal operation.
- Storage account keys, SAS tokens, and client secrets are not normal workflow
  dependencies.
- Backend configuration is owned by the Foundation root deployment.
- Reusable modules do not configure backends.
- Documentation identifies state location, access model, migration status, and
  recovery expectations without exposing sensitive environment values.

## Accomplished Looks Like

Foundation can continue implementation with secured remote state, controlled
identity-based state access, immutable module consumption, and no dependency on
local Terraform state.

Another qualified platform engineer can understand the bootstrap path, verify
where state lives, confirm how access is authorized, and proceed with later
Foundation deployment work without inventing a backend or state migration
pattern.
