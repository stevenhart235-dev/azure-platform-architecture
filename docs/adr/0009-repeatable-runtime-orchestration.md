# ADR 0009 - Repeatable Runtime Orchestration

## Status

Proposed for M8.1. The historical POC is complete; the workflow described here
is not implemented or claimed as proven.

## Owners

Ordicor Platform

## Context

The [Discovery-to-Runtime POC](../architecture/discovery-to-runtime-poc.md)
proved a manually coordinated slice using existing engines and an external,
connectivity-owned subnet. Repeatability needs a cross-repository contract,
not another AKS implementation. This supplements the integration boundaries
of [ADR 0002](0002-repository-separation.md); it does not replace the four
platform repositories or supersede production state, identity, or network ADRs.

## Proposed Decision

Coordinate existing discovery and platform-breakfix interfaces through the
[M8.1 lifecycle and single-target catalog contract](../roadmap/09-application-landing-zones.md).

- Discovery retains intent, platform selection, binding, and adapter translation.
- Platform-breakfix retains AKS planning, execution, validation, and cleanup.
- Connectivity retains VNet/subnet/peerings; applications never own connectivity.
- Cluster-foundation remains provider-neutral.
- The catalog is platform configuration with exactly one approved mapping,
  not infrastructure IDs embedded in application intent.
- Explicit local/manual review binds approval to target and saved-plan digest.
  Execution preserves that plan; changed plans require new approval.
- Validation, runtime-only destroy, and verify-clean are mandatory stages.
- No duplicate AKS/Terraform implementation, new runtime engine, TTL enforcement,
  or Seneschal dependency is introduced.

Exact identifiers in the linked closure and catalog are owner-requested
historical evidence and contract documentation only. Executable configuration,
plans, and state remain outside this repository; this is not a general waiver
of repository guardrails.

## Alternatives Considered

- Continue manual coordination: already proven, but does not establish one
  repeatable workflow or an explicit repeatable approval boundary.
- Replace engines or build generalized vending: duplicates implementation and
  exceeds the single-target milestone.
- Coordinate existing engines: selected proposal; preserves ownership and
  limits the change to lifecycle handoffs and platform configuration.

## Consequences and Risks

Existing engines retain their responsibilities. Integration must preserve
binding and saved-plan identity across handoffs and record failures even when
cleanup succeeds. Manual approval is not production authorization, and a
single catalog entry does not prove generalized selection. TTL intent does
not ensure cleanup; explicit destroy and verify-clean remain necessary.

Workflow entry point and catalog serialization remain implementation details
to document in the consuming repository. Acceptance requires all linked M8.1
criteria, including rejection cases and two consecutive complete runs.

## Revisit Conditions

Revisit before adding targets, regions, runtimes, automated authorization,
TTL enforcement, or changing network ownership. Firewall, DNS resolver,
remote-state hardening, production hardening, and subscription vending are
outside this milestone.
