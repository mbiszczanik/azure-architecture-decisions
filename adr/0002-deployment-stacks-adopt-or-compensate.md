# ADR-0002: Azure Deployment Stacks: Adopt Now or Compensate

| Field | Value |
| --- | --- |
| Status | Accepted (as a decision framework) |
| Date | 2026-06-12 |
| Decision type | Decision framework (pattern) |
| Tags | deployment-stacks, bicep, drift, governance, lifecycle |

## Context and Problem Statement

Standard ARM/Bicep deployments are create-and-forget: the deployment is a
point-in-time operation, and nothing afterwards prevents out-of-band changes
or removes resources that were deleted from the template. For a landing zone,
this leaves two structural gaps: protection against drift introduced outside
IaC, and lifecycle management of resources removed from code (orphans).

Azure Deployment Stacks address both: deny settings block out-of-band
modification of stack-managed resources, and actionOnUnmanage handles
resources that disappear from the template. The question is not whether
Stacks are useful, but whether to adopt them as the deployment primitive for
a new platform now, or to ship classic deployments and compensate.

The calculus shifted in early 2026: current GA releases of the AVM platform
landing zone pattern adopt Deployment Stacks natively. A framework that
standardizes on classic deployments while its upstream moves to Stacks is
swimming against its own toolchain. At the same time, Stacks bring their own
operational sharp edges, and a team new to IaC inherits whichever choice is
made.

## Decision Drivers

* Drift prevention strength: hard guarantees vs detect-and-respond.
* Orphan lifecycle: automatic cleanup vs manual reconciliation.
* Alignment with the upstream toolchain (AVM PLZ uses Stacks natively).
* Operational maturity required from the inheriting team (unmanage
  semantics, deny-assignment troubleshooting, break-glass procedures).
* Reversibility of the choice in each direction.

## Considered Options

* Option A: Adopt Deployment Stacks as the deployment primitive now
* Option B: Classic deployments plus a compensating control set
* Option C: Hybrid: Stacks for the platform scope, classic for workloads

## Decision Outcome

**For a new greenfield platform built on toolchain components that already
use Stacks natively: Option A**, with deny settings introduced gradually
(start with denyDelete, widen to denyWriteAndDelete per scope as the team's
break-glass procedure matures). Adopting the toolchain's native primitive
costs less than maintaining a divergence from it.

**For an existing framework built on classic deployments, or where no
upstream component forces the issue: Option B is a legitimate v1**, provided
the compensating set below is actually deployed, not just intended. Migrating
a live platform to Stacks is a planned, scope-by-scope exercise; deferring it
is reasonable, abandoning it should be an explicit decision.

**Option C is the pragmatic middle** when the platform scope is stable and
high-blast-radius (where deny settings earn their keep) while workload teams
need iteration speed and would generate constant deny-assignment friction.

The compensating control set for Option B and for any scope not yet under a
stack, in order of importance:

1. Azure Policy deny effects at management group level for the changes that
   must never happen out-of-band (e.g. manual peering creation, public IP on
   restricted subnets).
2. RBAC posture: no standing Contributor/Owner on production outside
   pipelines; humans get Reader, escalation via PIM with time-boxed
   activation.
3. Resource locks (CanNotDelete) deployed by modules for production
   resources.
4. Scheduled what-if drift detection (read-only) with an actionable response
   runbook: investigate via Activity Log, decide roll-back vs reconcile,
   document, and ask whether the drift signals a missing guardrail.

Note what compensation cannot give you: Policy deny covers enumerated cases,
not "everything this deployment manages", and nothing in the compensating set
handles orphaned resources automatically. Drift detection tells you the barn
door was open; deny settings keep it shut.

### Consequences

* We gain: an explicit rule for when Stacks are the default, replacing an
  open-ended "later" that tends to mean never.
* We accept: operating Stacks requires the team to learn unmanage semantics
  and deny-assignment troubleshooting before the first production incident,
  not during it. This is a training-plan item, not a footnote.
* We accept: under Option B, the compensating set is standing engineering
  work (policy catalog upkeep, drift triage) for the platform's lifetime.
* We defer (under B/C): automatic orphan cleanup, which remains manual
  reconciliation through PRs.

## Pros and Cons of the Options

### Option A: Adopt Stacks now

* Good, because deny settings are a hard guarantee, not a detection loop:
  out-of-band writes fail instead of being reported the next morning.
* Good, because actionOnUnmanage closes the orphan gap that classic
  deployments simply do not address.
* Good, because it matches where the toolchain is going; AVM PLZ consuming
  Stacks natively means staying on classic deployments becomes the custom,
  divergent path over time.
* Bad, because the failure modes are new to most teams: resources stuck
  behind deny assignments during legitimate emergencies, unmanage semantics
  surprising operators, and break-glass needing a designed procedure.
* Bad, because granular exclusions (this resource yes, that property no)
  are coarser than what mature Policy authoring can express.

### Option B: Classic deployments plus compensating controls

* Good, because every ingredient (Policy, locks, RBAC/PIM, what-if) is
  mature, well documented and already familiar to Azure-experienced teams.
* Good, because no new failure modes are introduced at deployment time.
* Bad, because the guarantee is softer: detect-and-respond instead of
  prevent, with response latency measured in hours.
* Bad, because the policy catalog only denies what someone thought to
  enumerate; coverage is permanently incomplete by construction.
* Bad, because orphan lifecycle stays manual forever.

### Option C: Hybrid (Stacks for platform, classic for workloads)

* Good, because it spends the deny-assignment friction where blast radius
  justifies it and spares the scopes that iterate daily.
* Good, because it lets the team learn Stacks on the most stable scope first.
* Bad, because two deployment models coexist: pipeline templates, runbooks
  and training all fork.
* Bad, because the workload scopes still carry the full Option B
  compensation burden.

## More Information

Date-sensitive facts: Stacks API maturity, the exact deny-settings
capabilities, and which AVM PLZ components consume Stacks natively all
evolve; verify against
https://learn.microsoft.com/azure/azure-resource-manager/bicep/deployment-stacks
and the AVM PLZ release notes before reuse.

Related: ADR-0001 (the landing zone implementation choice can force this
decision: choosing current AVM PLZ largely chooses Stacks), ADR-0003
(stack-managed scopes change how module updates roll out).
