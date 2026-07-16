# ADR 0006 - Deployment Identity Strategy

## Status

Accepted

## Context

The Azure Platform Framework requires a deployment identity model that supports
secure bootstrap, local development, CI/CD planning, controlled applies,
state access, troubleshooting, and emergency recovery.

ADR 0001 establishes Terraform as the authoritative Infrastructure as Code
engine. ADR 0002 establishes separate deployment repositories for foundation
and connectivity, with future landing-zone implementation separated by
responsibility. ADR 0003 establishes the Terraform toolchain baseline. ADR
0004 establishes Azure Blob Storage remote state with Microsoft Entra ID and
Azure RBAC. ADR 0005 establishes the management group hierarchy and platform
ownership model.

This ADR records the accepted identity strategy for human access, automation
access, repository-owned deployment identities, plan/apply separation, RBAC,
bootstrap, and break-glass access. It does not create app registrations,
managed identities, federated credentials, Terraform code, Azure resources, or
workflows.

## Decision Drivers

- Human identities must not become embedded automation credentials.
- Routine platform deployments must use automation identities.
- Authentication must use Microsoft Entra ID.
- Long-lived credentials must be avoided.
- GitHub Actions must be able to authenticate without stored secrets.
- Foundation and connectivity repositories need independent identity
  ownership, audit boundaries, and RBAC scopes.
- Plan and apply operations have different risk profiles.
- State access and Azure deployment permissions are separate concerns.
- Bootstrap must be possible before automation identities exist.
- Break-glass access must be human, documented, auditable, and outside CI/CD.

## Options Considered

1. Human-only deployments.
2. Shared service principal for all deployment repositories.
3. Repository-owned Entra applications with workload identity federation.
4. Managed identities for all CI/CD deployment operations.

## Option 1 - Human-Only Deployments

Use named human identities for bootstrap, plans, applies, troubleshooting, and
routine deployments.

Advantages:

- Simple during early manual lab work.
- Uses existing Entra authentication and Azure RBAC.
- Requires no CI/CD identity setup before initial bootstrap.

Disadvantages:

- Poor audit separation between human intent and automation.
- Does not support repeatable routine deployment workflows.
- Makes deployment behavior dependent on individual user access.
- Encourages broad standing privileges.
- Does not meet the platform goal of controlled CI/CD deployments.

## Option 2 - Shared Service Principal

Use one shared service principal or application identity across foundation,
connectivity, and future landing-zone deployments.

Advantages:

- Fewer Entra applications to manage.
- Simple initial setup.
- Easier to grant broad access once.

Disadvantages:

- Creates a large blast radius.
- Blurs repository ownership and auditability.
- Makes least privilege difficult.
- Couples foundation, connectivity, and landing-zone permissions.
- Makes credential rotation or trust-condition changes riskier.
- Conflicts with repository separation and state boundary goals.

## Option 3 - Repository-Owned Entra Applications With Workload Identity Federation

Each deployment repository owns an Entra application. GitHub Actions
authenticates through GitHub OIDC and Microsoft Entra Workload Identity
Federation. Each repository uses separate federated credentials for plan and
apply workflows, with different trust conditions and Azure role assignments.

Advantages:

- Avoids long-lived secrets.
- Aligns identity ownership with repository boundaries.
- Supports least privilege per repository and workflow type.
- Keeps plan and apply permissions separable.
- Improves auditability and blast-radius control.
- Scales cleanly to future deployment repositories.

Disadvantages:

- Requires more Entra application and federated credential governance.
- Requires careful trust-condition design.
- Requires CI/CD and Azure RBAC implementation discipline.
- May need additional operational runbooks for troubleshooting identity
  federation.

## Option 4 - Managed Identities for All CI/CD Deployment Operations

Use managed identities for all automation, including GitHub Actions deployment
workflows.

Advantages:

- Strong Azure-native identity model where runners execute inside Azure.
- Avoids service principal secrets.
- Can pair well with private runners and private backend access.

Disadvantages:

- Requires Azure-hosted or private-capable runners before it can be the primary
  pattern.
- Does not fit initial GitHub-hosted runner assumptions as directly as OIDC
  federation.
- Adds runner infrastructure decisions before M1 and M2 are complete.
- May still require repository-level trust and RBAC separation.

## Decision

Adopt repository-owned Microsoft Entra applications with GitHub OIDC and
Microsoft Entra Workload Identity Federation for CI/CD deployment
authentication.

Human identities are used for bootstrap, development, troubleshooting, and
emergency operations. Routine platform deployments are performed by automation
identities. All authentication uses Microsoft Entra ID. Least privilege is the
default design principle.

Initial deployment repositories:

- `azure-platform-foundation`
- `azure-platform-connectivity`

Future deployment repositories:

- Landing-zone deployment repositories or deployment scopes when implemented.

Each deployment repository owns its own Entra application.

Each deployment repository uses:

- A separate federated credential for plan.
- A separate federated credential for apply.

Plan and apply federated credentials may share the same Entra application for a
repository, but they must use different trust conditions and Azure role
assignments.

Do not use client secrets, certificates, stored credentials, or long-lived
service principal secrets for normal CI/CD authentication.

## Rationale

The platform needs an identity model that is secure enough for enterprise
operation but still practical for the Ordicor Platform Lab. Human identities
are appropriate for initial bootstrap and controlled local work because the
automation identities and backend infrastructure do not exist yet. Once
bootstrap establishes the deployment identities and state backend, routine
deployments should transition to automation.

GitHub OIDC with Entra Workload Identity Federation avoids long-lived
credentials while giving the platform clear trust conditions tied to repository
and workflow context. Repository-owned Entra applications align with ADR 0002:
foundation and connectivity have different ownership, blast radius, state
boundaries, and deployment permissions.

Plan and apply separation reflects their different risk profiles. Plans need to
read state and Azure resources. Applies can change Azure resources and update
state. Keeping those permissions separable reduces the impact of compromised or
misconfigured plan workflows and supports stronger approval gates for apply.

## Identity Architecture

Initial local development uses:

```text
Developer
  -> Azure CLI
    -> Microsoft Entra ID
      -> Azure RBAC
```

Human identities may perform:

- Bootstrap.
- Local validation.
- Controlled lab deployments.
- Troubleshooting.
- Emergency recovery.

Human identities must not be embedded in automation.

CI/CD authentication uses:

```text
GitHub Actions
  -> GitHub OIDC
    -> Microsoft Entra Workload Identity Federation
      -> Azure
```

Repository identity model:

- `azure-platform-foundation` owns a foundation Entra application.
- `azure-platform-connectivity` owns a connectivity Entra application.
- Future landing-zone deployment repositories or scopes receive their own
  Entra applications.
- Each repository has separate plan and apply federated credentials.
- Plan and apply credentials use different trust conditions.
- Plan and apply credentials receive different Azure role assignments.

This ADR does not define exact Entra application names, object IDs, federated
credential subjects, branch filters, environment names, or GitHub workflow
structure. Those details belong to M1/M2 implementation and repository setup.

## RBAC Strategy

Assign permissions at the smallest practical scope.

Preferred scope order:

1. Resource group.
2. Subscription.
3. Management group.

Grant higher-scope permissions only when the deployment action requires that
scope.

State permissions and Azure deployment permissions are separate concerns:

- State permissions authorize reading, locking, and updating Terraform state in
  the backend.
- Azure deployment permissions authorize reading or modifying Azure resources.

Plan identities:

- Read state.
- Acquire backend lease as required.
- Read Azure resources.
- Produce Terraform plans.

Apply identities:

- Update state.
- Modify Azure resources.
- Execute approved infrastructure changes.

Plan/apply separation is preferred where practical. If a temporary exception is
required during early lab implementation, it must be documented and revisited
before production use.

## Bootstrap Considerations

Initial bootstrap uses the authenticated Microsoft Entra user.

Bootstrap creates or enables the minimum required foundation for routine
automation, including:

- Workload identity federation.
- Deployment identities.
- Backend infrastructure.

After bootstrap, routine deployments transition to automation identities.

Bootstrap must be controlled, documented, repeatable, and reviewed. Bootstrap
must not leave long-lived local credentials, storage account keys, or shared
deployment accounts as normal operating dependencies.

This ADR does not define exact bootstrap commands, Terraform roots, role
assignments, naming, federated credential definitions, or workflow files. Those
details are deferred to M2 bootstrap and implementation work.

## Security Model

Security principles:

- No long-lived secrets.
- No client secrets for normal CI/CD.
- No certificate credentials for normal CI/CD.
- No stored credentials in GitHub or repository configuration.
- No storage account keys in normal state workflows.
- No shared deployment accounts across deployment repositories.
- Least privilege by default.
- Identity separation by repository and workflow type.
- Repository ownership of deployment identities.
- Separate state permissions from Azure deployment permissions.
- Human identities are not embedded in automation.
- Break-glass access is human only and not used by CI/CD.
- Authentication and authorization must be auditable.
- Deployment identity setup must be repeatable.

All authentication uses Microsoft Entra ID. Authorization uses Azure RBAC at
the smallest practical scope.

## Break-Glass Access

Break-glass access is for emergency human operations only.

Break-glass access must be:

- Human only.
- Protected by MFA.
- Documented.
- Audited.
- Tightly controlled.
- Not used by CI/CD.
- Reviewed after use.
- Time bounded where practical.

Break-glass procedures must define who can approve access, what roles may be
granted, what scopes may be used, how activity is audited, how access is
removed, and what follow-up review is required.

## Enterprise Scaling

The model scales by giving each additional deployment repository its own Entra
application and the same plan/apply federation pattern.

Future deployment repositories receive:

- One repository-owned Entra application.
- A plan federated credential.
- An apply federated credential.
- Plan RBAC scoped to read state and read required Azure resources.
- Apply RBAC scoped to update state and modify only the required Azure
  resources.
- Repository-specific trust conditions.
- Repository-specific ownership and audit trail.

This keeps new deployment repositories from inheriting broad shared access and
allows each repository to evolve its permissions according to its state
boundaries, management group placement, subscription scope, and operational
risk.

## Consequences

- Human access is allowed for bootstrap, local development, troubleshooting,
  controlled lab deployments, and emergency recovery.
- Routine platform deployment uses automation identities.
- GitHub Actions uses OIDC federation to Entra ID.
- Long-lived service principal secrets are not part of normal CI/CD.
- Foundation and connectivity start with separate repository-owned Entra
  applications.
- Future landing-zone deployment repositories receive separate Entra
  applications.
- Plan and apply trust conditions and RBAC are separated where practical.
- RBAC design must consider state access and Azure resource access separately.
- M1/M2 implementation must document exact identity, RBAC, and workflow
  details.

## Risks

- Federated credential trust conditions may be misconfigured and allow broader
  access than intended.
- Overly broad RBAC assignments can undermine repository and plan/apply
  separation.
- Early lab shortcuts may persist if exceptions are not revisited.
- Bootstrap may grant excessive human access if cleanup is not documented.
- Plan identities may need more permissions than expected to produce accurate
  Terraform plans.
- Apply identities may require management-group scope for some foundation
  operations, increasing blast radius.
- Multiple Entra applications increase governance and operational overhead.
- Break-glass roles may become standing privileged access if not time bounded
  and reviewed.

## Validation Criteria

This decision is valid when:

- Human identities are used only for bootstrap, development, troubleshooting,
  controlled lab deployments, and emergency recovery.
- Human identities are not embedded in automation.
- GitHub Actions authenticates through GitHub OIDC and Microsoft Entra
  Workload Identity Federation.
- Normal CI/CD uses no client secrets, certificates, stored credentials, or
  long-lived service principal secrets.
- Foundation and connectivity deployment repositories each have their own
  Entra application when implemented.
- Future landing-zone deployment repositories receive their own Entra
  application when implemented.
- Each deployment repository has separate plan and apply federated credentials.
- Plan and apply credentials have distinct trust conditions and role
  assignments.
- RBAC is assigned at the smallest practical scope.
- State permissions and Azure deployment permissions are documented separately.
- Bootstrap creates or enables automation identities and transitions routine
  deployments away from human identities.
- Break-glass access is human only, MFA protected, documented, audited, and not
  used by CI/CD.

## Revisit Conditions

Revisit this decision if:

- GitHub Actions is no longer the CI/CD platform.
- Enterprise policy requires managed identities, another workload identity
  pattern, or a different CI/CD trust model.
- Workload identity federation cannot meet required audit or security
  controls.
- Plan/apply separation proves impractical for required Terraform workflows.
- Deployment repositories need stronger identity isolation than one Entra
  application with separate federated credentials.
- Azure RBAC scope requirements become too broad for the accepted repository
  model.
- Private runners or Azure-hosted runners make managed identities a better
  primary automation pattern.
- A security incident or access review identifies unacceptable risk in
  federation, RBAC, bootstrap, or break-glass controls.
