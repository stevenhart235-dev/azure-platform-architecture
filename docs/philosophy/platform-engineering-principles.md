# Platform Engineering Principles

> These principles emerged while designing and implementing the Azure Platform Framework.
>
> They describe the engineering philosophy behind the architecture, repository structure, automation, governance, and deployment model.
>
> Each principle is backed by accepted architectural decisions (ADRs), engineering standards, or implementation within this framework. They are intended to evolve alongside the platform.

---

# 1. Build Trust Before Deployment

## Principle

> Automation should earn deployment permissions through proven engineering quality, not receive them by default.

## Rationale

Infrastructure automation is one of the most privileged systems in an organization.

Before allowing automation to modify cloud infrastructure, establish confidence in the engineering process itself.

## Implementation

Validation capability is introduced in deliberate stages:

1. Terraform formatting
2. Terraform validation
3. Native Terraform tests
4. Static analysis
5. Security scanning
6. Authenticated planning
7. Controlled deployment

GitHub Actions validates infrastructure long before it is trusted to deploy it.

---

# 2. Architecture Precedes Implementation

## Principle

> Every significant implementation should follow an accepted architectural decision.

## Rationale

Infrastructure often outlives individual engineers.

Documenting decisions first makes implementation intentional, repeatable, reviewable, and easier to evolve.

## Implementation

The framework separates architectural decisions from implementation.

- Architecture repository owns ADRs.
- Implementation repositories consume those decisions.
- Architectural changes occur before implementation changes.

---

# 3. Separate Decisions from Deployments

## Principle

> Architecture, reusable modules, and deployment repositories solve different problems and should evolve independently.

## Rationale

These artifacts have different consumers, owners, and release cadences.

Treating them as independent products improves governance and reduces coupling.

## Implementation

```
Architecture
        │
        ▼
Reusable Modules
        │
        ▼
Deployment Repositories
```

---

# 4. Infrastructure Must Be Reproducible

## Principle

> Deployment repositories should consume immutable infrastructure dependencies.

## Rationale

Infrastructure should always be explainable.

Every deployment should identify the exact module version used to produce it.

## Implementation

Preferred:

```hcl
source = "git::https://github.com/...//modules/resource-group?ref=resource-group-v0.1.0"
```

Avoid mutable branches or local filesystem references for committed deployment code.

---

# 5. Repository Boundaries Should Reflect Ownership

## Principle

> Repository structure should model organizational responsibility rather than technology categories.

## Rationale

Ownership drives maintainability.

Repositories should answer:

> "Who owns this?"

before

> "What Azure service is inside it?"

## Implementation

```
azure-platform-architecture
azure-platform-modules
azure-platform-foundation
azure-platform-connectivity
azure-platform-landingzones
```

Each repository has a single, clearly defined responsibility.

---

# 6. Reusable Logic Lives Once

## Principle

> Write infrastructure once. Configure it many times.

## Rationale

Duplicated Terraform roots inevitably drift.

Environment differences belong in configuration—not copied implementation.

## Implementation

```
platform/
    bootstrap/
    governance/

environments/
    lab/
    nonprod/
    prod/
```

One implementation.

Multiple environments.

---

# 7. Identity Should Be Ephemeral

## Principle

> Authentication should be temporary. Permissions should be least privilege.

## Rationale

Long-lived credentials become operational liabilities.

Short-lived identities reduce operational risk and credential management overhead.

## Implementation

The framework adopts:

- GitHub OIDC
- Microsoft Entra Workload Identity Federation
- No client secrets
- No stored credentials
- Repository-owned deployment identities
- Least-privilege RBAC

---

# 8. Validate Continuously. Deploy Deliberately.

## Principle

> Validation should be automatic. Deployment should be intentional.

## Rationale

Finding problems early is inexpensive.

Changing cloud infrastructure is not.

## Implementation

Every pull request validates engineering quality automatically.

Deployment remains an explicit engineering decision until confidence has been established.

---

# 9. Platform Engineering Is Systems Engineering

## Principle

> Great platforms emerge from well-designed engineering systems rather than isolated technology choices.

## Rationale

Terraform, Azure, GitHub Actions, and Microsoft Entra are tools.

The real product is the engineering system created by combining them through consistent architecture, governance, automation, and operational discipline.

## Implementation

The Azure Platform Framework standardizes:

- Repository structure
- Engineering validation
- Deployment boundaries
- Versioning
- Identity
- Governance
- Module reuse

---

# 10. Standardize Before You Scale

## Principle

> Scaling inconsistent practices only produces larger inconsistencies.

## Rationale

Small investments in standards compound over time.

A documented approach reduces onboarding time, improves review quality, and minimizes architectural drift.

## Implementation

Before expanding platform capability, establish:

- ADRs
- Engineering standards
- Repository standards
- Validation workflows
- Versioning strategy
- Ownership boundaries

Implementation follows standards—not the other way around.

---

# 11. Keep Primitive Modules Boring

## Principle

> Primitive modules should remain small, explicit, and environment-neutral.

## Rationale

The first released modules proved that simple building blocks are easier to
validate, version, release, and compose than modules that try to own environment
policy.

Primitive modules should expose clear resource contracts. They should not hide
environment naming, tag policy, backend wiring, dependency orchestration, or
deployment sequencing.

## Implementation

Primitive modules own focused Azure resource behavior.

Composition layers and root deployments own:

- Environment policy
- Naming decisions
- Effective tags
- Dependency wiring
- State boundaries
- Provider and backend configuration

This keeps reusable modules environment-neutral while allowing Foundation,
connectivity, and future landing-zone deployments to apply platform policy
through configuration.

---

# Living Document

These principles are intentionally incomplete.

Additional principles will emerge as the framework evolves through future work, including:

- Remote state
- Bootstrap automation
- Management Groups
- Azure Policy
- Networking
- Subscription vending
- Identity
- Governance
- Landing Zones
- Operational excellence

The philosophy should evolve only when supported by real implementation experience rather than hypothetical design.
