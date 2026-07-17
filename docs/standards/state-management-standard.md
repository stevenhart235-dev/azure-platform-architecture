# State Management Standard

## Purpose

This standard defines operational requirements for Terraform state management
across the Azure Platform Framework.

Terraform state is a critical platform dependency. It records deployed
infrastructure relationships, resource identifiers, provider metadata, and
values that can influence future plans and applies. State must therefore be
remote, protected, access controlled, recoverable, and separated by deployment
boundary and blast radius.

This standard codifies the accepted remote state and root deployment decisions
from ADR 0004 and ADR 0008. It does not create Terraform code, backend
configuration, Azure resources, pipelines, or implementation repositories.

## Scope

This standard applies to:

- `azure-platform-foundation`.
- `azure-platform-connectivity`.
- Future landing-zone deployment repositories.
- Terraform root deployments.
- Bootstrap state.
- Remote backend configuration.
- Terraform state access and recovery practices.

This standard also applies negatively to reusable modules:

- Reusable child modules in `azure-platform-modules` must not configure
  backends.
- Reusable child modules must not own real environment state.

## State Ownership

Root deployment repositories own Terraform state for the resources they deploy.

Ownership model:

- `azure-platform-foundation` owns Foundation state for bootstrap, management
  groups, platform subscription placement, governance, identity, management,
  and observability concerns.
- `azure-platform-connectivity` owns connectivity state for hub networking,
  routing, firewall, DNS, hybrid connectivity, and spoke onboarding concerns.
- Future landing-zone deployment repositories or scopes own their approved
  landing-zone state boundaries.
- `azure-platform-modules` owns reusable module source and tests, not real
  environment state.
- `azure-platform-architecture` owns state standards and decisions, not
  deployable state.

State ownership must align with repository responsibility, deployment identity,
approval path, and blast radius.

## Backend Ownership

Root deployment repositories own backend configuration.

Required rules:

- Root modules configure backends.
- Root modules configure providers.
- Root deployment repositories commit dependency lock files for real root
  deployments.
- Reusable child modules must not include backend configuration.
- Reusable child modules must not include provider configuration.
- Backend configuration must not expose secrets or sensitive environment values
  in Git.

Backend configuration must be reviewable as part of the deployment root that
owns the corresponding state boundary.

## State Isolation Boundaries

State must be separated where a shared state file would create unacceptable
blast radius, ownership coupling, deployment cadence coupling, or recovery
complexity.

Initial state boundaries are intentionally coarse and may be split later based
on:

- Lifecycle.
- Blast radius.
- Ownership.
- Deployment cadence.
- Recovery requirements.
- Environment boundary.
- Regulatory or administrative boundary.
- Network boundary.

Production state must not share a state file with lower environments.

Foundation and connectivity must not share one state file unless a future
accepted architecture decision explicitly changes the repository and state
boundary model.

## Storage Account and Container Expectations

Azure Blob Storage is the authoritative remote backend for initial platform
state.

Required expectations:

- Use the native Terraform `azurerm` backend.
- Use dedicated backend storage for platform state.
- Use private containers for Terraform state.
- Do not permit anonymous container access.
- Disable public blob access.
- Use Microsoft Entra ID authentication and Azure RBAC authorization for normal
  workflows.
- Protect state storage as critical infrastructure.
- Enable recovery protections appropriate to the environment and criticality.

Initial accepted containers from ADR 0004:

- `bootstrap`
- `foundation`
- `connectivity`
- `landing-zones`

Additional containers or storage accounts may be introduced only when justified
by lifecycle, ownership, environment, network, regulatory, or recovery
requirements.

## Backend Key Naming Principles

State keys must be stable, descriptive, and aligned with the deployment
boundary they represent.

Required principles:

- Keys identify the deployment concern and state boundary.
- Keys do not contain secrets or sensitive values.
- Keys avoid personal names and local machine information.
- Keys do not depend on Git branch names as the environment model.
- Keys remain stable across normal configuration changes.
- Key changes are treated as state migration events and reviewed accordingly.

Initial accepted keys from ADR 0004:

```text
bootstrap/state.tfstate
foundation/core.tfstate
connectivity/core.tfstate
```

Future key names must preserve the same clarity: they should describe the
platform concern without encoding volatile implementation details.

## Temporary Local Bootstrap State

Local state is allowed only temporarily during first bootstrap, when the remote
backend does not yet exist.

Requirements:

- Local state may be used only to create the initial backend resources.
- Local state must not be used for normal shared platform deployments.
- Local state files must not be committed.
- Local state backups must not be committed.
- Local plans containing sensitive values must not be committed.
- Bootstrap state must be migrated to Azure Blob Storage after the backend
  exists.
- Local state artifacts must be removed after successful migration.

The temporary local-state allowance must not be reused as a shortcut for later
Foundation, connectivity, or landing-zone deployments.

## Migration Requirements

Bootstrap state migration is mandatory after backend creation.

Migration requirements:

- Migration must move bootstrap state from local state to Azure Blob Storage.
- The migrated state must use the approved backend, container, and key pattern.
- The migration must be documented in the Foundation repository.
- Post-migration initialization must confirm that Terraform uses the remote
  backend.
- Local state artifacts must be removed from the working tree after migration.
- No follow-on shared deployment may rely on local state after migration.

Moving an existing remote state file to another container, key, or storage
account is also a state migration and requires deliberate review.

## Authentication and Authorization

Authentication must use Microsoft Entra ID.

Authorization must use least-privilege Azure RBAC.

Accepted access patterns:

- Human Entra identities for bootstrap, development, troubleshooting, and
  emergency recovery.
- GitHub OIDC with Microsoft Entra Workload Identity Federation for automation
  where implemented.
- Azure RBAC data-plane roles for state access.

Normal workflows must not depend on:

- Storage account keys.
- SAS tokens.
- Client secrets.
- Certificate credentials.
- Long-lived service principal credentials.
- Credentials committed to Git.

## RBAC Expectations

State access and Azure resource deployment permissions are separate concerns.

RBAC expectations:

- Grant state access at the smallest practical scope.
- Prefer container-scoped or storage-account-scoped data-plane access where it
  satisfies the workflow.
- Separate plan and apply permissions where practical.
- Grant read/update state permissions only to identities that need them.
- Grant Azure resource permissions only at scopes required by the deployment
  operation.
- Document bootstrap, plan, apply, read-only, and break-glass access.
- Review and reduce bootstrap human access after automation is established.

Plan identities may need to read state and Azure resources. Apply identities
may update state and modify Azure resources. Those permissions must be
understood and reviewed separately.

## Encryption and Transport Requirements

State must be protected in transit and at rest.

Required expectations:

- Use Azure Storage encryption at rest.
- Use HTTPS/TLS for backend access.
- Do not transmit state through insecure channels.
- Do not store backend credentials in source control.
- Do not expose state through public or anonymous blob access.

Customer-managed keys, private endpoints, firewall restrictions, and private
DNS may be introduced when justified by enterprise requirements and when the
connectivity and runner model can support them.

## State Locking

Terraform state locking is required for shared remote state.

Operational expectations:

- Use the native locking behavior of the Terraform `azurerm` backend.
- Do not bypass locks during normal operations.
- Treat stale lock intervention as an operational recovery action.
- Document lock handling before production use.
- Validate state health before resuming normal operations after manual lock
  intervention.

## State Artifact Handling

State artifacts must be handled as sensitive operational material.

Prohibited in Git:

- `.tfstate` files.
- `.tfstate.backup` files.
- Terraform plan files containing sensitive values.
- Backend secrets.
- Storage account keys.
- SAS tokens.
- Client secrets.
- Local override files containing environment values.

Generated state or plan artifacts must be short-lived, protected, and stored
only in approved locations when required for review or recovery.

## Prohibited Practices

The following practices are prohibited:

- Committing Terraform state to any Git repository.
- Committing state backup files to any Git repository.
- Using local state for shared platform deployments after bootstrap migration.
- Sharing production state files with lower environments.
- Configuring backends inside reusable child modules.
- Configuring providers inside reusable child modules.
- Using storage account keys, SAS tokens, or client secrets as normal state
  authentication mechanisms.
- Allowing anonymous access to state containers.
- Treating `terraform_remote_state` as the default integration mechanism.
- Depending on undocumented state outputs as cross-repository contracts.
- Moving backend keys or containers without a reviewed migration plan.

## Recovery and Break-Glass Expectations

State recovery must be documented before production use and tested before
production readiness.

Recovery expectations:

- State version recovery is documented.
- Blob soft-delete recovery is documented where enabled.
- Container soft-delete recovery is documented where enabled.
- Lock recovery is documented.
- Lost access recovery is documented.
- Break-glass access is human, auditable, controlled, and not used by CI/CD.
- Recovery actions validate Terraform state health before normal operations
  resume.

Break-glass access must define:

- Who may approve access.
- Which roles may be granted.
- Which scopes may be used.
- How long access may last.
- How activity is audited.
- How access is removed.
- How follow-up review is performed.

## Validation Requirements

State management is compliant when:

- Root deployment repositories own backend configuration.
- Reusable modules contain no backend configuration.
- Azure Blob Storage is used as the authoritative remote backend.
- Local state is used only for first bootstrap.
- Bootstrap state is migrated to Azure Blob Storage after backend creation.
- State containers are private and do not permit anonymous access.
- Normal authentication uses Entra ID.
- Normal authorization uses least-privilege Azure RBAC.
- Storage account keys, SAS tokens, and client secrets are absent from normal
  workflows.
- State files, backup state files, sensitive plans, and backend secrets are not
  committed.
- Production and lower environments do not share a state file.
- Cross-state dependencies are documented when approved by exception.
- Recovery and break-glass procedures exist before production use.

## Compliance Checklist

Use this checklist during repository setup, bootstrap review, and deployment
pull request review:

- [ ] The state boundary belongs to the repository making the change.
- [ ] Backend configuration exists only in a root deployment.
- [ ] Reusable child modules do not configure backends.
- [ ] The backend uses Azure Blob Storage with the Terraform `azurerm`
      backend.
- [ ] The state container is private.
- [ ] Anonymous access is disabled.
- [ ] Authentication uses Microsoft Entra ID.
- [ ] Authorization uses least-privilege Azure RBAC.
- [ ] Storage account keys, SAS tokens, and client secrets are not normal
      workflow dependencies.
- [ ] Temporary local state is limited to first bootstrap.
- [ ] Bootstrap state is migrated to remote state after backend creation.
- [ ] Local state artifacts are removed after migration.
- [ ] `.tfstate`, `.tfstate.backup`, sensitive plan files, backend secrets,
      and local override files are not committed.
- [ ] Production state is separate from nonproduction and lab state.
- [ ] State keys are stable, descriptive, and free of sensitive values.
- [ ] Any `terraform_remote_state` use is documented as an approved exception.
- [ ] Recovery and break-glass expectations are documented for production use.
- [ ] State access and Azure deployment permissions are reviewed separately.

## Definition of Done

State management is production-quality when another qualified platform engineer
can identify state ownership, backend location, access model, migration status,
recovery path, and blast-radius boundary without relying on undocumented
knowledge or local state artifacts.
