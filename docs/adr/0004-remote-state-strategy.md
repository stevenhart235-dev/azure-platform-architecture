# ADR 0004 - Remote State Strategy

## Status

Accepted

## Context

ADR 0001 establishes Terraform as the authoritative Infrastructure as Code
engine for the Azure Platform Framework. ADR 0002 establishes separate
repositories for architecture, reusable modules, foundation deployments, and
connectivity deployments. ADR 0003 establishes Terraform `1.15.8` as the
approved execution version and confirms that root deployment repositories own
provider configuration, backend configuration, dependency lock files, and
environment values.

The roadmap requires remote state that is secured, recoverable, access
controlled, and isolated by lifecycle and blast radius. The repository standard
prohibits Terraform state files, state backups, and plan files from being
stored in Git. The versioning standard confirms that roots, not reusable child
modules, own backend configuration and reproducible execution concerns.

This ADR records the initial remote state strategy for the Ordicor Platform Lab
and the Azure Platform Framework. It does not create Terraform code, backend
files, storage resources, workflows, or repositories. Detailed M2 bootstrap,
identity, network, and recovery implementation remains deferred to M2 work and
related implementation documentation.

## Decision Drivers

- Terraform state must not be stored in Git.
- Shared platform deployments require remote, locked, recoverable state.
- The backend must work with Terraform as the authoritative engine.
- The initial lab must be simple enough to bootstrap before the connectivity
  platform exists.
- State access must use Microsoft Entra ID and Azure RBAC.
- Storage account keys, SAS tokens, and long-lived client secrets must not be
  part of the normal workflow.
- Foundation and connectivity concerns require clear state ownership and blast
  radius boundaries.
- Cross-state dependencies must not become hidden integration contracts.
- The strategy must scale from a lab model to enterprise-separated state
  accounts where risk, ownership, network, or regulatory boundaries require it.

## Options Considered

1. Native Terraform `azurerm` backend with Azure Blob Storage.
2. Terraform Cloud or HCP Terraform remote state.
3. Local state for early platform work.
4. Custom or third-party backend.

## Option 1 - Native Terraform azurerm Backend

Use the native Terraform `azurerm` backend with Azure Blob Storage.

Advantages:

- Aligns directly with Azure platform ownership.
- Supports Azure Blob Storage state locking behavior.
- Supports Microsoft Entra ID authentication and Azure RBAC data-plane
  authorization.
- Keeps state in the Azure tenant and subscription model governed by the
  platform.
- Avoids introducing another SaaS control plane for the initial implementation.
- Fits the repository model where root deployment repositories own backend
  configuration.

Disadvantages:

- Backend storage must be bootstrapped before normal remote state use.
- Storage security, recovery, diagnostics, and access controls must be designed
  and operated by the platform team.
- Private-only access depends on connectivity and private-capable runners that
  do not exist at initial lab bootstrap.

## Option 2 - Terraform Cloud or HCP Terraform

Use Terraform Cloud or HCP Terraform for remote state and execution-adjacent
capabilities.

Advantages:

- Provides managed remote state capabilities.
- May integrate with policy, runs, teams, and workspace workflows.
- Reduces direct operation of storage account recovery features.

Disadvantages:

- Adds another platform dependency and operating model before M1 and M2 are
  complete.
- Requires organization, workspace, access, billing, and compliance decisions
  outside the current Azure-native baseline.
- Does not align with the initial lab requirement to keep the backend Azure
  native.
- Is not required for the initial platform implementation.

## Option 3 - Local State

Use local Terraform state for early work and defer remote state.

Advantages:

- Simple for one developer during throwaway experiments.
- Requires no backend bootstrap before the first run.

Disadvantages:

- Unsafe for shared platform deployments.
- Does not provide shared locking, central recovery, or controlled access.
- Risks state loss, drift, and unreviewed local operations.
- Conflicts with the roadmap requirement for secured remote state after
  bootstrap.

## Option 4 - Custom or Third-Party Backend

Use a custom backend or non-Azure third-party storage platform.

Advantages:

- May fit organizations with an existing enterprise state service.
- Could support specialized controls not available in Azure Storage.

Disadvantages:

- Adds avoidable complexity to the initial Azure platform.
- Requires additional support, security, compliance, and recovery decisions.
- Does not provide a clear advantage over Azure Blob Storage for the initial
  lab.

## Decision

Use the native Terraform `azurerm` backend with Azure Blob Storage.

Do not use Terraform Cloud for the initial platform implementation.

The Ordicor Platform Lab may initially access the state storage account through
the public Azure endpoint. Authentication must use Microsoft Entra ID.
Authorization must use Azure RBAC. Storage account keys and SAS tokens are not
part of the normal workflow.

Private-only access may be introduced later when the connectivity platform and
private-capable runners exist.

## Rationale

Azure Blob Storage with the native Terraform `azurerm` backend is the simplest
backend that satisfies the platform's early requirements without introducing an
additional control plane. It keeps Terraform state in Azure, supports Entra ID
and Azure RBAC, and aligns with future foundation and connectivity deployment
repositories that own their root backends.

The initial lab must be able to bootstrap before private networking and
private-capable CI runners exist. Allowing public endpoint access for the lab
keeps the bootstrap path practical while still requiring identity-based access,
RBAC scoping, disabled public blob access, and a path toward stronger network
controls later.

Coarse initial state boundaries reduce early operational overhead while the
platform is still proving its bootstrap, foundation, and connectivity patterns.
The ADR deliberately defines criteria for later splits instead of guessing a
large state matrix before the implementation exists.

## Security Model

The state storage account is a critical platform dependency.

Required security model:

- Authentication uses Microsoft Entra ID.
- Authorization uses Azure RBAC data-plane roles.
- Public blob access is disabled.
- Storage account keys are not used in normal operation.
- SAS tokens are not used in normal operation.
- Long-lived client secrets are prohibited for normal state access.
- State access permissions are scoped to the minimum required state operation.
- State permissions and Azure deployment permissions are separate concerns.
- Break-glass access is documented, tightly controlled, time bounded where
  practical, and auditable.

The state storage account must eventually use:

- Blob versioning.
- Blob soft delete.
- Container soft delete.
- Accidental deletion protection.
- Azure RBAC data-plane authorization.
- Diagnostic logging where justified.
- Public blob access disabled.
- Infrastructure protection appropriate for critical state.

Network access:

- The lab may initially use the public Azure endpoint.
- Private-only access is deferred until the connectivity platform and
  private-capable runners exist.
- Introducing private endpoints, firewall restrictions, private DNS, or
  runner-network requirements is future implementation work and must not be
  hidden inside this ADR.

## State Layout

The initial lab uses:

- One dedicated resource group.
- One dedicated storage account.
- Separate containers by platform concern.

Initial containers:

- `bootstrap`
- `foundation`
- `connectivity`
- `landing-zones`

Initial state keys:

```text
bootstrap/state.tfstate
foundation/core.tfstate
connectivity/core.tfstate
```

The `landing-zones` container is reserved for future landing-zone state
boundaries. This ADR does not define final landing-zone keys.

State files should group resources normally planned, applied, changed, and
recovered together.

The platform begins with coarse state boundaries and splits states later based
on:

- Lifecycle.
- Blast radius.
- Ownership.
- Deployment cadence.
- Recovery requirements.

Likely future splits may include management groups, policy, identity,
management, observability, regional connectivity, DNS, spoke onboarding, or
landing-zone boundaries, but this ADR does not decide those implementation
details.

## Cross-State Dependency Strategy

Cross-state dependencies must be explicit and reviewed. Terraform state outputs
must not become an undocumented integration contract.

Prefer dependency patterns in this order:

1. Shared configuration in Git.
2. Explicit outputs passed through deployment automation.
3. Azure data sources for stable existing resources.
4. `terraform_remote_state` only by exception.

`terraform_remote_state` is not the default integration pattern.

Any use of `terraform_remote_state` must be documented as an intentional
dependency. The documentation must identify:

- Consuming state.
- Producing state.
- Outputs consumed.
- Reason shared configuration, automation outputs, or Azure data sources are
  insufficient.
- Operational impact if the producing state changes, is unavailable, or is
  recovered.

## Access Model

Initial local access:

- Uses the authenticated Microsoft Entra user through Azure CLI.
- The user receives scoped `Storage Blob Data Contributor` access for required
  state operations.
- Access is scoped to the state storage account, container, or narrower scope
  where practical.

Future CI/CD access:

- Uses workload identity federation.
- Avoids long-lived client secrets.
- Separates plan and apply identities where practical.
- Grants state permissions separately from Azure deployment permissions.
- Scopes each identity to only the state containers and Azure scopes it needs.

Break-glass access:

- Must be documented before production use.
- Must be tightly controlled and auditable.
- Must define who may approve access, how access is granted, how long access
  lasts, and how access is reviewed afterward.

## Bootstrap Considerations

The backend resources must be created through a controlled bootstrap process.

Accepted bootstrap principles:

- Bootstrap may begin with local state.
- After backend creation, bootstrap state must be migrated to Azure Storage.
- Bootstrap must be repeatable and documented.
- Bootstrap must not leave normal operations dependent on local state.
- Bootstrap must not require storage account keys or SAS tokens for the normal
  workflow.
- Bootstrap access must be reviewed and reduced after remote state is
  established.

Deferred M2 implementation details include:

- Exact bootstrap repository or directory structure.
- Exact storage account name, resource group name, region, and tags.
- Exact role assignment scopes and identities.
- Exact backend configuration files or partial backend configuration pattern.
- Exact commands for state migration.
- Exact private endpoint, firewall, DNS, and runner-network implementation.
- Exact operational runbooks.

## Recovery Requirements

Remote state recovery must be designed and tested before production readiness.

The platform must define procedures for:

- State version recovery.
- Blob soft-delete recovery.
- Container soft-delete recovery.
- Terraform lock handling.
- Corruption response.
- Lost access recovery.
- Break-glass access.
- Restoring access without exposing storage account keys as a normal workflow.
- Validating the recovered state with Terraform before normal operations
  resume.
- Communicating recovery risk, impact, and rollback status to platform
  maintainers.

Recovery testing must be completed before production readiness. Tests should
prove that the platform can recover state versions, recover from accidental
deletion where supported, handle stale locks safely, and restore access through
documented emergency procedures.

## Enterprise Scaling

The Ordicor Platform Lab uses one state storage account for simplicity.

Enterprise implementations may separate state storage accounts based on:

- Environment.
- Regulatory boundary.
- Administrative ownership.
- Network boundary.
- Business criticality.
- Recovery boundary.

Splitting storage accounts can improve isolation and recovery clarity, but it
also increases operational overhead, access management, private networking,
diagnostics, and bootstrap complexity. Future enterprise separation must be
driven by explicit risk, ownership, compliance, or recovery requirements.

## Consequences

- Terraform root deployments use the native `azurerm` backend.
- Terraform Cloud is not part of the initial platform backend strategy.
- Root deployment repositories own backend configuration.
- State storage is Azure-native and must be protected as critical
  infrastructure.
- The lab starts with one dedicated resource group, one dedicated storage
  account, and concern-based containers.
- Initial state boundaries are intentionally coarse.
- State boundary splits require documented rationale.
- Cross-state consumption through `terraform_remote_state` requires an
  exception and documentation.
- M2 must implement a controlled bootstrap process and state migration path.
- Production readiness must include recovery testing.

## Risks

- Public endpoint access during the lab may be broader than the eventual
  enterprise target.
- One lab storage account creates a shared dependency across platform concerns.
- Coarse state boundaries may become too broad as resources and teams grow.
- Deferring private-only access may require later migration of backend network
  controls and runner placement.
- Misconfigured RBAC could allow excessive state access.
- Lost Entra access or incorrect role assignments could block state
  operations.
- State corruption, accidental deletion, or stale locks could interrupt
  platform delivery if recovery procedures are not tested.
- Cross-state dependencies may drift into hidden coupling if exceptions are
  not reviewed.
- Bootstrap may become a one-off manual process unless repeatability is
  enforced during M2.

## Validation Criteria

This decision is valid when:

- No Terraform state files are committed to Git.
- The initial backend uses the native Terraform `azurerm` backend with Azure
  Blob Storage.
- Terraform Cloud is not required for initial platform state.
- The state account uses Microsoft Entra authentication and Azure RBAC
  authorization for normal workflows.
- Storage account keys and SAS tokens are absent from normal state workflows.
- Initial containers exist for `bootstrap`, `foundation`, `connectivity`, and
  `landing-zones`.
- Initial state keys follow the accepted coarse boundary model.
- Root deployment backend configuration is owned by deployment repositories,
  not reusable child modules.
- `terraform_remote_state` use is absent by default or documented as an
  approved exception.
- Bootstrap state is migrated from local state to Azure Storage after backend
  creation.
- Recovery procedures are documented and tested before production readiness.
- Break-glass access is documented and controlled before production use.

## Revisit Conditions

Revisit this decision if:

- Enterprise policy requires Terraform Cloud, HCP Terraform, or another remote
  state platform.
- The lab moves from public endpoint access to private-only backend access.
- Connectivity and private-capable runners are available and ready to support
  private backend access.
- State boundaries become too large for safe planning, applying, rollback, or
  recovery.
- Foundation, connectivity, or landing-zone ownership requires separate
  storage accounts.
- Regulatory, environment, administrative, network, business criticality, or
  recovery boundaries require stronger state separation.
- `terraform_remote_state` exceptions become common enough to indicate a poor
  integration model.
- Recovery testing identifies unacceptable gaps in versioning, soft delete,
  lock handling, or break-glass procedures.
- A production incident shows that state layout, access, or recovery design is
  insufficient.
