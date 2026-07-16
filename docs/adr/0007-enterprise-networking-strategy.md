# ADR 0007 - Enterprise Networking Strategy

## Status

Accepted

## Context

The Azure Platform Framework requires an enterprise networking strategy that
supports secure landing zones, centralized inspection, private connectivity,
shared DNS, hybrid connectivity, and repeatable regional expansion.

ADR 0001 establishes Terraform as the authoritative Infrastructure as Code
engine. ADR 0002 establishes the four-repository model, including a dedicated
`azure-platform-connectivity` repository for enterprise hubs, routing,
firewalls, DNS, hybrid connectivity, and spoke onboarding. ADR 0003 establishes
the Terraform toolchain baseline. ADR 0004 establishes remote state boundaries
that include connectivity state. ADR 0005 establishes management group
placement for connectivity under the platform hierarchy. ADR 0006 establishes
the deployment identity strategy used by future connectivity deployment
automation.

This ADR records the accepted enterprise networking strategy. It does not
create Terraform code, virtual networks, peerings, firewalls, gateways, route
tables, DNS zones, or Azure resources.

## Decision Drivers

- Optimize for enterprise architecture first.
- Allow the Ordicor Platform Lab to scale down without changing the target
  architecture.
- Use Azure-native implementation patterns.
- Preserve cloud-agnostic architectural principles where practical.
- Centralize routing, inspection, and DNS ownership.
- Keep workload landing zones separate from platform connectivity ownership.
- Support future hybrid connectivity through VPN and ExpressRoute.
- Support regional expansion without restructuring the network model.
- Avoid hard-coded enterprise CIDR ranges in reusable modules.
- Leave room for future enterprise IPAM integration.

## Options Considered

1. Hub-and-spoke reference architecture.
2. Azure Virtual WAN as the default architecture.
3. Flat network model with direct workload networking.
4. Full mesh spoke-to-spoke networking.

## Option 1 - Hub-and-Spoke Reference Architecture

Use one platform-owned hub network per Azure region, with workload spokes
connected to the regional hub.

Advantages:

- Familiar enterprise Azure pattern.
- Clear separation between platform connectivity and workload networks.
- Supports centralized routing, inspection, DNS, and hybrid connectivity.
- Gives each region an independent deployment and recovery unit.
- Scales by adding regional hubs without redesigning the topology.
- Works well for lab implementation with a single initial region.

Disadvantages:

- Requires the platform to operate hub routing and firewall policy.
- Multi-region routing and transitive connectivity require deliberate design.
- Can become complex if every exception is handled with custom routes.

## Option 2 - Azure Virtual WAN as the Default Architecture

Use Azure Virtual WAN as the default enterprise networking platform.

Advantages:

- Azure-native managed wide-area networking service.
- Strong fit for organizations with broad branch, VPN, ExpressRoute, and
  multi-region connectivity needs.
- Can reduce some hub operations when the organization's requirements align
  with Virtual WAN.

Disadvantages:

- Introduces a different operating model and control plane from traditional
  hub-and-spoke.
- May be more than the lab or initial platform needs.
- Can constrain routing, inspection, and integration patterns differently than
  hub-and-spoke.
- Requires organization-specific evaluation before becoming the default.

## Option 3 - Flat Network Model

Allow workload networks to connect directly to required services without a
platform hub.

Advantages:

- Simple for isolated small workloads.
- Reduces initial platform networking components.

Disadvantages:

- Does not provide centralized routing or egress control.
- Makes DNS, firewall, hybrid connectivity, and inspection inconsistent.
- Does not scale well for enterprise landing zones.
- Pushes connectivity decisions into workload teams.

## Option 4 - Full Mesh Spoke-to-Spoke Networking

Peer workload spokes directly to each other by default.

Advantages:

- Can reduce latency between specific workloads.
- May be useful for narrowly scoped, justified application dependencies.

Disadvantages:

- Creates hard-to-govern lateral connectivity.
- Increases route, peering, and ownership complexity.
- Undermines centralized routing and inspection.
- Does not provide a safe default for enterprise landing zones.

## Decision

Use Hub-and-Spoke as the reference enterprise networking architecture.

Virtual WAN is evaluated but not selected as the default. Virtual WAN remains
an acceptable future implementation for organizations whose scale, branch,
hybrid, routing, operational, or service requirements justify it.

The architecture is enterprise first. The Ordicor Platform Lab may initially
deploy only one region and a reduced set of components, but the architecture
does not change simply because the implementation is smaller.

The implementation direction is Azure native. The architecture should still
preserve cloud-agnostic principles such as centralized inspection, explicit
routing, private connectivity, configuration-driven IPAM, separation of
platform and workload ownership, and least-privilege operational boundaries.

## Rationale

Hub-and-spoke provides a clear and broadly understood Azure Landing Zone network
model. It separates platform-owned connectivity from workload-owned spokes,
centralizes egress and routing policy, supports private DNS ownership, and
provides a practical path for VPN and ExpressRoute.

Virtual WAN is a valid Azure service, but it should be selected when specific
requirements justify its operating model. Making hub-and-spoke the reference
architecture gives the platform a stable default while preserving Virtual WAN
as a future option.

One hub per Azure region supports regional independence. Each regional hub can
be planned, deployed, changed, and recovered as its own unit. The lab can start
with one regional hub and expand by adding hubs in additional Azure regions
without changing the architecture.

## Regional Strategy

The platform uses one hub per Azure region.

Regional hubs are independent deployment units. Each hub should have its own
configuration, state boundary, routing design, and operational validation
appropriate to the final connectivity deployment design.

The lab may initially deploy only one region. Additional regions add additional
hubs under the same architecture.

Regional hub design must consider:

- Address allocation.
- Firewall and egress placement.
- Gateway requirements.
- DNS resolver and forwarding needs.
- Route tables and UDRs.
- Spoke onboarding.
- Diagnostics and operational monitoring.
- Recovery and rollback boundaries.

## Connectivity Ownership

The connectivity platform owns:

- Hub VNets.
- Hub subnets.
- Azure Firewall.
- VPN Gateway.
- ExpressRoute Gateway.
- NAT Gateway.
- Route Tables.
- User Defined Routes.
- Private DNS Zones.
- DNS Links.
- VNet Peering.
- DDoS protection where applicable.

Landing zones own workload VNets and connect to the platform through approved
spoke onboarding patterns.

The connectivity platform owns the shared network control plane. Workload teams
own resources inside their landing-zone subscriptions, subject to inherited
policy, approved connectivity patterns, and platform-owned routing and DNS
controls.

## Routing Model

Routing is centralized through the regional hub.

The default routing model is:

- Spokes connect to the regional hub.
- Traffic requiring shared services, hybrid connectivity, inspection, or
  centralized egress routes through the hub.
- Routing policies remain centralized with the connectivity platform.
- No spoke-to-spoke peering by default.
- Spoke-to-spoke communication, when required, should be routed through
  approved hub routing or another documented platform pattern.

No spoke-to-spoke peering by default reduces lateral connectivity and keeps
route ownership clear. Exceptions require explicit justification and review.

## Internet Egress

Internet egress is centralized through Azure Firewall.

Direct spoke Internet access is the exception, not the default. Exceptions must
be explicitly justified, documented, and reviewed for security, routing,
monitoring, and operational impact.

Centralized egress supports:

- Consistent inspection.
- Central firewall policy.
- Reduced public exposure.
- Reviewable routing behavior.
- Shared operational logging and troubleshooting.

## DNS Strategy

Private DNS is platform owned and centralized.

The connectivity platform owns centralized Private DNS Zones, DNS links, and
related shared DNS integration. Private DNS is shared across landing zones
through approved platform patterns.

DNS ownership remains with the platform because DNS is a shared control plane
for private endpoints, platform services, hybrid resolution, and workload
connectivity. Workload teams must not create conflicting private DNS ownership
models.

Future DNS implementation work must define resolver placement, forwarding
rules, hybrid DNS integration, zone lifecycle, link strategy, and operational
runbooks.

## Private Connectivity Strategy

Private Endpoint First is the platform direction.

Public endpoints require explicit justification. Public endpoint exceptions
must consider:

- Business or technical requirement.
- Data exposure and security risk.
- Firewall and policy controls.
- Diagnostic and monitoring requirements.
- Lifecycle and rollback.
- Whether a private endpoint or private link pattern can satisfy the
  requirement instead.

Private connectivity should be the default for platform services and landing
zone integrations where Azure service support and operational requirements
allow it.

## IPAM Strategy

IP address management is configuration-driven.

Modules must never hard-code enterprise CIDR ranges. Reusable modules must
accept address configuration through inputs and must remain environment-neutral.

Deployment repositories and platform configuration own environment-specific
address allocations. Address allocation must be reviewable, traceable, and
validated where practical before deployment.

Future enterprise IPAM integration is anticipated. The platform should be able
to evolve from configuration-driven IPAM in Git to integration with an
enterprise IPAM system without changing reusable module contracts.

IPAM design must prevent overlap across:

- Regional hubs.
- Gateway subnets.
- Firewall subnets.
- DNS resolver subnets.
- Private endpoint subnets.
- Shared services.
- Workload spokes.
- Hybrid/on-premises networks.

## Enterprise Scaling

Enterprise scaling adds regional hubs without changing the architecture.

Additional regions follow the same model:

- Add a regional hub.
- Allocate non-overlapping address space.
- Configure hub subnets and shared services.
- Configure regional firewall, routing, DNS, and gateway components as needed.
- Onboard regional spokes through approved patterns.
- Keep the regional hub as an independent deployment and recovery unit.

The platform can start with one lab region and grow into multiple enterprise
regions without restructuring the network model.

## Consequences

- Hub-and-spoke is the default networking reference architecture.
- Virtual WAN is not the default, but remains available for justified future
  implementations.
- Connectivity deployments own hub, routing, firewall, DNS, peering, gateway,
  and applicable DDoS concerns.
- Landing zones own workload VNets and consume platform connectivity.
- Routing and egress policy are centralized.
- Direct spoke Internet access and spoke-to-spoke peering require explicit
  justification.
- Private DNS ownership remains with the platform.
- Private Endpoint First becomes the default connectivity posture.
- IPAM must be configuration-driven and environment-specific.
- Reusable modules must not embed enterprise CIDR ranges.

## Risks

- Hub-and-spoke can become operationally complex if exceptions are not
  controlled.
- Centralized egress can become a bottleneck if capacity and regional design
  are not planned.
- Regional hub independence requires disciplined state, configuration, and
  release boundaries.
- DNS centralization can block workloads if ownership, resolver behavior, or
  zone lifecycle is unclear.
- Private endpoint adoption can increase DNS and routing complexity.
- Avoiding direct spoke-to-spoke peering may require careful application
  dependency design.
- Configuration-driven IPAM may not scale without validation or future IPAM
  integration.
- Virtual WAN may be a better fit for some enterprises and may require a future
  architecture decision if requirements change.

## Validation Criteria

This decision is valid when:

- Hub-and-spoke is documented as the reference architecture.
- Virtual WAN is documented as evaluated but not selected as the default.
- One hub per Azure region is the accepted regional model.
- Regional hubs are treated as independent deployment units.
- The lab can deploy one region without changing the target architecture.
- Connectivity ownership includes hub VNets, hub subnets, Azure Firewall, VPN
  Gateway, ExpressRoute Gateway, NAT Gateway, route tables, UDRs, Private DNS
  Zones, DNS links, VNet peering, and DDoS protection where applicable.
- Landing zones own workload VNets and connect through approved platform
  patterns.
- Routing is centralized through the hub.
- No spoke-to-spoke peering exists by default.
- Internet egress is centralized through Azure Firewall.
- Direct spoke Internet access requires explicit justification.
- Private DNS is platform owned and shared across landing zones.
- Private Endpoint First is the default connectivity posture.
- Public endpoint use requires explicit justification.
- IPAM is configuration-driven.
- Reusable modules do not hard-code enterprise CIDR ranges.

## Revisit Conditions

Revisit this decision if:

- Enterprise requirements justify Virtual WAN as the preferred topology.
- Multi-region routing, branch connectivity, or hybrid scale outgrows the
  default hub-and-spoke operating model.
- Centralized Azure Firewall egress becomes technically or economically
  unsuitable.
- Workload requirements demand a different spoke-to-spoke connectivity model.
- Private DNS ownership cannot be operated centrally at required scale.
- Private Endpoint First creates unacceptable operational constraints for
  required services.
- Enterprise IPAM integration becomes mandatory.
- Regulatory, data residency, merger, acquisition, or organizational boundary
  requirements require a different network segmentation model.
- Production incidents show that the routing, DNS, egress, or regional hub
  model is unsafe or unclear.
