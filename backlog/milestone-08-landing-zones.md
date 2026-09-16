# M8 - Application Landing Zones Backlog

## Completed Vertical Slice

- [x] Discovery-to-Runtime Landing Zone POC: **POC COMPLETE**.
- [x] Record supplied plan, saved-plan continuity, runtime, workload, drift,
  ownership, and teardown evidence in the
  [durable closure](../docs/architecture/discovery-to-runtime-poc.md).

This closes only the proven slice, not broader enterprise M8 outcomes.

## Next Milestone: M8.1 - Repeatable Runtime Orchestration

Status: Defined; implementation not started by this documentation change.

Goal: turn the manually coordinated slice into one repeatable, controlled
workflow using existing engines. The
[milestone contract and acceptance criteria](../docs/roadmap/09-application-landing-zones.md)
define this increment.

| Work item | Responsible boundary | Completion evidence |
| --- | --- | --- |
| Define entry point and stage handoffs | Discovery/adapter coordination with platform-breakfix | One lifecycle runbook; no new runtime engine |
| Add single-target platform catalog | Platform configuration consumed by discovery | Exact mapping; negative resolution checks; no infrastructure IDs in application intent |
| Preserve binding through adapter and plan | application-discovery-engine | Target traceable through payload and existing adapter |
| Enforce review and plan continuity | Coordination and platform-breakfix | Digest/target approval; rejection paths block provision |
| Record validation and drift | platform-breakfix with sample workload coordination | Cilium, readiness, DNS/HTTP, and zero-change drift results |
| Handle failure, destroy, and verify-clean | platform-breakfix | Runtime-only cleanup; retained connectivity; failures visible |
| Demonstrate repeatability | Existing engine integration | Two consecutive approved complete runs; evidence per run |

All implementation items remain open. No Azure execution, engine changes,
deployable catalog, or orchestration code is part of this repository change.

## Guardrails

Preserve connectivity ownership and provider-neutral `cluster-foundation`.
No duplicate AKS/Terraform, new engine, TTL enforcement, Seneschal, firewall,
DNS resolver, multi-region, remote-state hardening, production hardening, or
generalized subscription vending.

## Exit Criteria

- [ ] Every linked M8.1 acceptance criterion has evidence.
- [ ] Repeatability and approval rejection cases are demonstrated in the
  implementation repositories without expanding the single-target scope.
- [ ] Closure distinguishes observed results from deferred capabilities.
