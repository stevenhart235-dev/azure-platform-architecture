# Repository Standard

## Purpose

This standard defines repository engineering requirements for the Azure Platform
Framework.

Consistent repository structure, ownership, workflows, and controls are required
for a maintainable enterprise platform. The platform spans architecture,
reusable Terraform modules, foundation deployments, and connectivity
deployments. Without common repository expectations, teams will create different
branching models, review gates, file conventions, release practices, and
security controls. That inconsistency increases operational risk and makes the
platform harder to review, recover, audit, and extend.

This standard provides the common contract for the four Azure platform
repositories. It supports the roadmap goals of configuration-driven
deployments, reusable and versioned Terraform modules, clear separation between
modules and environment deployments, controlled CI/CD, state isolation by blast
radius, and recoverable infrastructure changes.

## Repository Responsibilities

The Azure Platform Framework uses four repositories.

| Repository | Responsibility |
| --- | --- |
| `azure-platform-architecture` | Architecture, standards, ADRs, diagrams, roadmap, and implementation guidance |
| `azure-platform-modules` | Reusable and independently versioned Terraform modules |
| `azure-platform-foundation` | Management groups, platform subscriptions, governance, identity, management, and observability deployments |
| `azure-platform-connectivity` | Enterprise hubs, routing, firewalls, DNS, hybrid connectivity, and spoke onboarding deployments |

### `azure-platform-architecture`

Belongs here:

- Roadmap documents.
- Architecture documents.
- ADRs.
- Engineering standards.
- Diagrams.
- Implementation guidance.
- Decision backlog and milestone definitions.

Must not be stored here:

- Terraform root modules.
- Reusable Terraform module source.
- Terraform state files or state backups.
- Terraform plan files.
- Provider lock files for deployable root modules.
- Azure tenant IDs, subscription IDs, client secrets, private keys, or
  environment-specific deployment configuration.
- CI/CD workflows that apply Azure infrastructure.
- Generated deployment artifacts.

Architecture decisions and standards belong in this repository. This repository
defines what the platform will do and why. It must not become a hidden
deployment repository.

### `azure-platform-modules`

Belongs here:

- Reusable Terraform modules.
- Module examples.
- Module tests.
- Module documentation.
- Module release notes.
- Semantic version tags for module releases.
- Azure Verified Module wrappers or compositions where approved.

Must not be stored here:

- Root deployments for real platform environments.
- Environment-specific tenant IDs, subscription IDs, regions, address ranges,
  resource names, or tags.
- Remote state backend configuration for real platform deployments.
- Terraform state files or state backups.
- Terraform plan files.
- Long-lived credentials, secrets, private keys, tokens, or pipeline
  environment secrets.
- Foundation or connectivity environment configuration.
- Hard-coded enterprise IP allocations.

Reusable Terraform modules belong in this repository only. Modules must be
consumed by deployment repositories through immutable released versions.

### `azure-platform-foundation`

Belongs here:

- Terraform root deployments for management groups.
- Platform subscription placement and enrollment configuration.
- Governance and Azure Policy assignments.
- Identity and RBAC assignments.
- Management and observability resources.
- Foundation environment configuration.
- Foundation deployment documentation and runbooks.
- Foundation state boundary definitions after the state ADR is completed.

Must not be stored here:

- Reusable module source that belongs in `azure-platform-modules`.
- Enterprise hub, firewall, routing, DNS, hybrid connectivity, or spoke
  onboarding deployments unless a documented dependency requires coordination.
- Terraform state files or state backups.
- Terraform plan files except short-lived CI artifacts managed outside Git.
- Secrets, private keys, client secrets, or local developer overrides.

Environment-specific configuration for foundation deployments belongs in this
repository, subject to the state, identity, naming, and deployment architecture
ADRs.

### `azure-platform-connectivity`

Belongs here:

- Terraform root deployments for enterprise network hubs.
- Routing, firewall, firewall policy, DNS, private resolver, and hybrid
  connectivity deployments.
- Spoke onboarding configuration.
- Connectivity environment configuration.
- Connectivity deployment documentation and runbooks.
- Connectivity state boundary definitions after the state ADR is completed.

Must not be stored here:

- Reusable module source that belongs in `azure-platform-modules`.
- Management group hierarchy, global policy baseline, or platform identity
  deployments unless explicitly documented.
- Terraform state files or state backups.
- Terraform plan files except short-lived CI artifacts managed outside Git.
- Secrets, private keys, client secrets, or local developer overrides.

Environment-specific configuration for connectivity deployments belongs in this
repository, subject to the state, networking, DNS, IPAM, identity, naming, and
deployment architecture ADRs.

### Terraform State

Terraform state must never be stored in any Git repository.

State must be remote, secured, recoverable, access-controlled, and isolated by
ownership, lifecycle, and blast radius. The final backend, container, key,
access, and recovery strategy is deferred to the remote state ADR and related
deployment architecture standards.

## Common Repository Requirements

Every repository must include files that create clear ownership, contribution,
security, and maintenance expectations. Files should be required because they
provide operational value, not because they are fashionable.

Mandatory for all repositories:

- `README.md`: explains repository purpose, ownership, major workflows, and
  links to relevant standards.
- `CONTRIBUTING.md`: defines contribution expectations, pull request process,
  validation expectations, and review requirements.
- `CODEOWNERS`: maps paths or repository scope to responsible reviewer roles.
- `.gitignore`: excludes local state, plan files, override files, credentials,
  generated artifacts, and tool caches.
- `.editorconfig`: normalizes whitespace, line endings, indentation, and final
  newline behavior.
- Pull request template: captures purpose, scope, validation evidence, risk,
  and reviewer notes.
- Security guidance: documents how to report security issues and what must not
  be committed.

Repository-specific files:

- Issue templates are required where the repository accepts work intake through
  issues. They are optional for repositories managed entirely through an
  external backlog.
- `LICENSE` is required where organizational policy requires explicit licensing
  metadata. Internal-only repositories may use the enterprise standard license
  approach.
- Changelog or release notes are required for `azure-platform-modules` and
  recommended for deployment repositories. They are optional for
  `azure-platform-architecture` unless standards or ADR baselines are released
  by tag.
- Module release notes are required for reusable modules.
- Deployment runbooks are required for foundation and connectivity before
  production use.

## Branching Strategy

`main` is the authoritative protected branch for every repository.

Required rules:

- All changes must be made through pull requests.
- Direct changes to `main` are prohibited.
- Feature branches must be short-lived.
- Long-lived environment branches are prohibited.
- Force pushes to protected branches are prohibited.
- Branch deletion protection must be enabled for protected branches where the
  platform supports it.
- Emergency changes must still use pull requests, required checks, and
  emergency approvers unless a documented break-glass process is invoked.

Infrastructure environments must not be represented solely by Git branches.
Branches are a source-control workflow, not an environment model. Production,
nonproduction, region, and platform concerns must be represented through
configuration, state boundaries, and deployment controls, not by maintaining
long-lived branches that drift from one another.

Emergency changes:

- Must be limited to the smallest safe change.
- Must reference the incident, risk, or operational need.
- Must include validation evidence appropriate to the repository.
- Must receive approval from the responsible CODEOWNERS or documented emergency
  approver role.
- Must be followed by normal documentation, review, and corrective work if any
  standard process was bypassed.

## Pull Request Standards

Every pull request must be reviewable without requiring undocumented context.

Required pull request content:

- Clear purpose.
- Scope of change.
- Related issue, ADR, roadmap milestone, or work item where applicable.
- Validation evidence.
- Risk and rollback notes when the change affects deployable infrastructure or
  module contracts.
- Documentation updates when contracts, behavior, ownership, or operational
  procedures change.
- Reviewer approval before merge.

Additional requirements:

- Deployment repositories must include a Terraform plan summary for changes
  that affect deployable infrastructure.
- Deployment repository pull requests must identify the target environment,
  state boundary, and expected blast radius when applicable.
- Reusable module pull requests must declare whether the change is breaking,
  backward-compatible functionality, or a patch.
- Reusable module pull requests must include migration guidance for breaking
  changes.
- Architecture pull requests must identify impacted ADRs, standards, diagrams,
  or implementation repositories when applicable.
- Pull requests must not mix unrelated changes unless the coupling is
  intentional and explained.

## Commit Standards

Commit history should describe meaningful platform changes. Commits should be
concise, reviewable, and grouped by coherent intent.

Use conventional-style prefixes:

- `docs:` documentation, standards, ADRs, diagrams, or roadmap changes.
- `feat:` new platform capability, module feature, or deployment capability.
- `fix:` defect correction.
- `refactor:` internal restructuring without intended behavior change.
- `test:` test additions or corrections.
- `chore:` maintenance tasks that do not affect behavior.
- `ci:` automation, validation, or pipeline-related changes.

Examples:

- `docs: add repository standard`
- `feat: add private dns resolver module`
- `fix: correct policy assignment scope`
- `test: add module validation fixture`
- `ci: add terraform validation workflow`

Commit messages must not include secrets, credentials, tenant-specific
information, or sensitive operational details.

## Repository Protection

Every repository must protect `main`.

Required controls:

- Protected `main` branch.
- Required status checks before merge.
- Required pull request reviews.
- CODEOWNERS approval for protected paths.
- Prevention of force pushes to protected branches.
- Prevention of protected branch deletion.
- Secret scanning enabled where supported.
- Dependency scanning enabled where dependencies exist.
- Administrative bypass limited to approved platform administrators.

Repository-specific controls:

- Module repository releases must require passing validation before tags are
  created.
- Deployment repositories must require plan validation before apply approval.
- Deployment repositories must use controlled apply authorization.
- Architecture repository checks should focus on documentation quality, links,
  formatting, and standards consistency.

Signed commits are optional unless organizational policy requires them. If
required, the rule must be applied consistently across the platform
repositories.

## Access Control and Ownership

Repository permissions must align with repository responsibility and least
privilege.

Ownership roles:

- Platform architecture maintainers own roadmap, architecture documents, ADRs,
  standards, and diagrams.
- Terraform module maintainers own reusable module interfaces,
  implementation, tests, documentation, and releases.
- Foundation maintainers own management group, policy, identity, RBAC,
  management, observability, and foundation deployment changes.
- Connectivity maintainers own hub, routing, firewall, DNS, hybrid
  connectivity, spoke onboarding, and connectivity deployment changes.
- Security reviewers review changes that affect identity, RBAC, policy,
  network exposure, secrets, encryption, logging, or compliance posture.

Expectations:

- Do not assign ownership to named individuals in standards. Assign ownership
  to roles or groups.
- CODEOWNERS must reflect repository contracts and protected paths.
- Review requirements must match the risk of the change.
- Apply permissions must be separate from general repository write access.
- Break-glass access must be documented, limited, and auditable.

## Versioning and Releases

Release expectations differ by repository.

### `azure-platform-architecture`

The architecture repository is the source of truth for decisions and standards.
It does not produce deployable artifacts.

Expected release behavior:

- Changes are merged through reviewed pull requests.
- ADRs and standards may be referenced by commit SHA or tag when implementation
  repositories need a fixed baseline.
- The repository may use documentation baseline tags if required by governance.
- Not every documentation commit is a platform release.

### `azure-platform-modules`

The modules repository produces reusable Terraform module releases.

Required release behavior:

- Modules must use immutable semantic version tags.
- Released module versions must not be moved or rewritten.
- Breaking changes require major version increments.
- Backward-compatible features require minor version increments.
- Backward-compatible fixes require patch version increments.
- Release notes must describe changes, breaking impacts, and migration steps.
- Deployment repositories must consume released module versions, not mutable
  branches.

### `azure-platform-foundation`

The foundation repository deploys platform foundation state. It is not a
general-purpose reusable software release repository.

Expected release behavior:

- Track deployed revisions.
- Track consumed module versions.
- Record plan/apply history through the approved deployment process.
- Use deployment notes or release records for meaningful production changes.
- Do not pretend every commit is a reusable software release.

### `azure-platform-connectivity`

The connectivity repository deploys enterprise connectivity state. It is not a
general-purpose reusable software release repository.

Expected release behavior:

- Track deployed revisions.
- Track consumed module versions.
- Record plan/apply history through the approved deployment process.
- Use deployment notes or release records for meaningful production changes.
- Do not pretend every commit is a reusable software release.

## Cross-Repository Dependencies

Cross-repository dependencies must be explicit, reviewed, and traceable.

Required practices:

- Deployment repositories must reference reusable modules by immutable version.
- Deployment repositories must not reference mutable module branches such as
  `main`.
- Module upgrade pull requests must be explicit and must summarize changed
  module versions, expected plan impact, and required migration steps.
- Implementation repositories must link to authoritative standards rather than
  copying standards locally.
- ADRs and standards that change implementation contracts must create follow-up
  work in affected implementation repositories.
- Dependency ownership must be documented. Each dependency must have an owning
  repository and responsible maintainer role.
- Cross-state or cross-repository outputs must not become undocumented
  integration contracts.

Architecture changes propagate into implementation work through:

- ADRs that record decisions.
- Standards updates that define engineering requirements.
- Roadmap work items that identify affected milestones.
- Implementation issues or pull requests that apply the new contract.
- Module version releases consumed by deployment repositories.

## Configuration and Secret Handling

Repositories must not contain:

- Terraform state.
- Terraform state backups.
- Secrets.
- Client secrets.
- Private keys.
- Real credentials.
- Unapproved sensitive outputs.
- Local override files.
- Developer-specific configuration.
- Personal access tokens.
- Generated plan files committed to Git.
- Tool cache directories or local runtime artifacts.

Example configuration:

- Must use placeholder values.
- Must not use real tenant IDs, subscription IDs, client IDs, private IP
  allocations, production names, cost centers, owners, or support groups.
- Must clearly distinguish example values from deployable environment values.
- Must avoid values that look real enough to be copied into production without
  review.

Local files:

- Local override files must be excluded by `.gitignore`.
- Developer-specific settings must remain outside Git.
- Temporary plan files and generated artifacts must remain outside Git.

Secrets must be stored and accessed through approved secret management systems,
identity federation, managed identity, or platform-approved credential
mechanisms. They must not be committed, even temporarily.

## Environment Configuration

Environment-specific configuration belongs in the deployment repositories:

- Foundation configuration belongs in `azure-platform-foundation`.
- Connectivity configuration belongs in `azure-platform-connectivity`.

Principles:

- Environment and region boundaries must be clear.
- Production and nonproduction state must not share a blast radius.
- Reusable Terraform logic must remain in `azure-platform-modules`.
- Deployment repositories must consume reusable logic through released module
  versions.
- Environment values must be supplied as configuration, not hard-coded in
  modules.
- Terraform implementations must not be copied repeatedly for each
  environment.
- Configuration should drive environment differences such as region,
  subscription, management group placement, address allocation, policy scope,
  and feature enablement.
- Root deployments must align with state isolation, identity, and deployment
  architecture decisions.

This standard does not prescribe the final deployment repository directory
structure. Final structure is intentionally deferred until the remote state,
deployment identity, and deployment architecture ADRs are completed.

## Automation Expectations

Repositories must include automation appropriate to their responsibility. This
standard defines required categories only; it does not implement pipelines.

Required automation categories:

- Formatting.
- Validation.
- Linting.
- Security scanning.
- Documentation checks.
- Tests where applicable.
- Terraform plans where applicable.
- Controlled applies where applicable.

Repository-specific expectations:

- `azure-platform-architecture`: Markdown formatting, link checks, diagram
  checks where practical, and documentation consistency checks.
- `azure-platform-modules`: Terraform formatting, Terraform validation, TFLint,
  security scanning, module tests, example validation, documentation checks, and
  release validation.
- `azure-platform-foundation`: Terraform formatting, validation, linting,
  security scanning, plan generation, drift detection where approved, and
  controlled apply workflow.
- `azure-platform-connectivity`: Terraform formatting, validation, linting,
  security scanning, plan generation, drift detection where approved, and
  controlled apply workflow.

Terraform automation must use Terraform as the authoritative engine, per
ADR 0001. OpenTofu validation is not required unless a future ADR changes the
platform engine decision.

## Documentation Expectations

Every repository must have a `README.md` that explains:

- Repository purpose.
- Repository ownership roles.
- What belongs in the repository.
- What must not be committed.
- How to propose changes.
- Required validation before pull request review.
- Links to relevant roadmap, ADRs, and standards.

Repository-specific documentation:

- `azure-platform-architecture` must document roadmap navigation, ADR process,
  standards ownership, and diagram conventions.
- `azure-platform-modules` must document module layout, release process,
  versioning, testing expectations, and module consumption guidance.
- `azure-platform-foundation` must document deployment scope, state boundaries
  when defined, environment configuration approach, plan/apply process,
  consumed module versions, and operational runbooks before production use.
- `azure-platform-connectivity` must document deployment scope, network
  configuration approach, state boundaries when defined, plan/apply process,
  consumed module versions, and operational runbooks before production use.

Documentation must be kept current when repository contracts, module
interfaces, deployment behavior, state boundaries, or ownership change.

## Repository Review Checklist

Use this checklist for repository setup reviews and pull request reviews:

- [ ] The change belongs in this repository according to the repository
      responsibility contract.
- [ ] The change does not introduce prohibited content for the repository.
- [ ] No Terraform state, plan files, secrets, private keys, or local override
      files are committed.
- [ ] Environment-specific configuration is stored only in the appropriate
      deployment repository.
- [ ] Reusable Terraform code is stored only in `azure-platform-modules`.
- [ ] Architecture decisions and standards are stored only in
      `azure-platform-architecture`.
- [ ] The pull request has a clear purpose and scope.
- [ ] Related issue, ADR, roadmap item, or milestone is linked where
      applicable.
- [ ] Required validation evidence is included.
- [ ] Deployment repository changes include a Terraform plan summary where
      applicable.
- [ ] Module repository changes declare version impact and breaking changes.
- [ ] Documentation is updated when contracts or behavior change.
- [ ] Required reviewers and CODEOWNERS approved the change.
- [ ] Cross-repository dependencies use immutable versions or authoritative
      links.
- [ ] No mutable branch reference such as `main` is used for reusable module
      consumption.
- [ ] Security-sensitive changes received security review where required.
- [ ] Emergency changes include incident context and follow-up work if any
      normal control was bypassed.

## Definition of Done

A production-quality repository is complete when it:

- Has a clear responsibility contract.
- Contains only content appropriate to that contract.
- Has a useful `README.md`, `CONTRIBUTING.md`, `CODEOWNERS`, `.gitignore`,
  `.editorconfig`, pull request template, and security guidance.
- Uses `main` as the protected authoritative branch.
- Requires pull requests for all changes.
- Requires status checks and CODEOWNERS review before merge.
- Blocks force pushes and protected branch deletion.
- Enables secret scanning and dependency scanning where applicable.
- Uses role-based ownership rather than named individual ownership.
- Protects secrets, state, plan files, local overrides, and developer-specific
  configuration from Git.
- Uses Terraform as the authoritative engine for Terraform validation and
  deployment workflows.
- References shared standards instead of copying them.
- Manages cross-repository dependencies explicitly.
- Tracks releases or deployed revisions in a way appropriate to the repository
  responsibility.
- Provides enough documentation for another qualified platform engineer to
  review, operate, recover, and extend the repository without undocumented
  knowledge.
