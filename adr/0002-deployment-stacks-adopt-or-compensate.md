# ADR-0002: Azure Deployment Stacks: Adopt Now or Compensate

| Field | Value |
| --- | --- |
| Status | Accepted (as a decision framework) |
| Date | 2026-06-12 |
| Revised | 2026-11-16 (see [Revision history](#revision-history)) |
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

A caveat that strengthens the gradual approach rather than weakening it:
adopting Stacks is not the same as getting the drift guarantee for free.
The accelerator bootstrap (`Azure/accelerator-bootstrap-modules` v7.3.2,
released 2026-10-10, checked 2026-11-16) passes `DenySettingsMode = "None"`
to its stacks in the local script, the Azure DevOps helper and the GitHub
Actions action, so the deny enforcement that actually prevents out-of-band
change is opt-in, set per scope as a deliberate act. It also passes
`ActionOnUnmanage = "DeleteAll"`: a resource removed from a template is
deleted, not detached. Plan for "Stacks deployed, deny widened
deliberately" as two separate steps, not one.

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
   document, and ask whether the drift signals a missing guardrail. Note that
   a template what-if (`az deployment ... what-if`) runs against a template,
   not against a stack: it is the compensating control for scopes outside
   Stacks. What-if for stacks is generally available since 2026-08-18 and
   previews what a stack update would create, detach or delete; whether it
   reports out-of-band drift on stack-managed resources is not verified
   (last checked 2026-11-16).

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

Date-sensitive facts, all as of 2026-11-16 unless stated:

* Deployment Stacks are generally available, announced 2024-05-23 ([ARM
  Deployment Stacks now GA](https://techcommunity.microsoft.com/blog/azuregovernanceandmanagementblog/arm-deployment-stacks-now-ga/4145469)).
* Deny modes are `None`, `DenyDelete` and `DenyWriteAndDelete`;
  `actionOnUnmanage` options are `detachAll`, `deleteResources` and
  `deleteAll` ([Deployment stacks in Bicep](https://learn.microsoft.com/azure/azure-resource-manager/bicep/deployment-stacks),
  page dated 2026-11-09).
* What-if for stacks is generally available in all regions, announced
  2026-08-18 ([announcement](https://techcommunity.microsoft.com/blog/azuregovernanceandmanagementblog/now-generally-available-what-if-for-azure-deployment-stacks/4547614),
  [Preview deployment stack changes with what-if](https://learn.microsoft.com/azure/azure-resource-manager/bicep/deployment-stacks-what-if),
  page dated 2026-11-04). The result is a
  `Microsoft.Resources/deploymentStacksWhatIfResults` resource, not a
  transient output.
* The accelerator's own "What-If" mode does not use stack what-if: its
  pipeline helper runs `New-Az<scope>Deployment -WhatIf` with
  `ValidationLevel = "providerNoRbac"` against the template, and the
  "Deployment Stack" mode runs `New-Az<scope>DeploymentStack`
  (`Azure/accelerator-bootstrap-modules` v7.3.2, the Azure DevOps helper
  `alz/azuredevops/pipelines/bicep/templates/helpers/bicep-deploy.yaml` and
  the GitHub action `alz/github/actions/bicep/templates/actions/bicep-deploy/action.yaml`).
* The same bootstrap creates management group, subscription and resource
  group stacks with `DenySettingsMode = "None"` and
  `ActionOnUnmanage = "DeleteAll"`, so current AVM PLZ deployments adopt
  Stacks natively with deny enforcement off by default.

These evolve; verify against the sources above and the bootstrap releases
before reuse.

Related: ADR-0001 (the landing zone implementation choice can force this
decision: choosing current AVM PLZ largely chooses Stacks), ADR-0003
(stack-managed scopes change how module updates roll out).

## Revision history

### 2026-11-16: facts re-checked, stack what-if now supported

Re-checked every date-sensitive fact against its primary source on this day.
Changes: each fact now carries an "as of" date and a link; the accelerator
bootstrap v7.3.2 is cited for the deny mode and for the Stacks deployments;
`ActionOnUnmanage = "DeleteAll"` in the bootstrap was added; the claim that
what-if is not supported for stacks was replaced, because what-if for stacks
is generally available (announced 2026-08-18) and the accelerator's "What-If"
mode uses a template what-if, not stack what-if. The default recommendation
(Option A for a greenfield platform on a toolchain that already uses Stacks)
is unchanged, so the record is revised in place.

Not verified: whether stack what-if reports out-of-band drift on
stack-managed resources (the pages describe previewing the changes a stack
update makes).

The June wording, superseded by this revision:

> Stacks have been GA since mid-2024. [...] As of mid-2026, what-if is not
> supported for stacks, and current AVM PLZ releases default the deny mode to
> None.
>
> Note that what-if runs against a template, not against a stack: it is a
> compensating control for scopes outside Stacks, and does not (yet) report
> drift on stack-managed resources directly.

### 2026-06-12: accepted
