# ADR-0001: Choosing a Landing Zone Reference Implementation in Bicep (2026)

| Field | Value |
| --- | --- |
| Status | Accepted (as a decision framework) |
| Date | 2026-06-12 |
| Decision type | Decision framework (pattern) |
| Tags | landing-zone, bicep, avm, alz, iac-strategy |

## Context and Problem Statement

An organization is standing up a greenfield Azure landing zone to receive
Windows Server workloads migrated from on-premises. The delivery timeline is
hard (a go-live measured in a few months), the operations team that will
inherit the platform has limited Azure, IaC and Git experience, and the
delivery partner intends to reuse the resulting framework for subsequent
engagements, so the choice outlives this single project.

The landing zone will be implemented in Bicep. In 2026 there are several
viable reference implementations, and the ecosystem is mid-transition:
Microsoft's classic ALZ-Bicep accelerator is in maintenance mode with a
declared end-of-support horizon, while Azure Verified Modules (AVM) is the
declared strategic direction, with platform landing zone (PLZ) pattern
modules maturing component by component rather than as a monolith.

Choosing wrong has asymmetric costs: a deprecated foundation means a planned
rewrite within 12 to 24 months, while an immature foundation risks the
go-live date.

## Decision Drivers

* Time-to-first-deployment under a hard deadline.
* Strategic alignment with Microsoft's long-term IaC direction (AVM).
* Reusability of the framework for subsequent projects.
* Operability and skill fit for a team new to IaC, post-handover.
* Acceptance by conservative stakeholders ("is this Microsoft best practice?").
* Per-module versioning and upgrade ergonomics over the framework's lifetime.

## Considered Options

* Option A: Classic ALZ-Bicep accelerator
* Option B: AVM platform landing zone (PLZ) pattern modules
* Option C: Custom composition built directly on AVM resource modules
* Option D: Phased: Option A for v1, planned migration to Option B post-go-live

## Decision Outcome

This is a context-dependent decision; the framework below states which option
wins under which conditions, and what we would weigh most.

**Default for new greenfield engagements starting in 2026: Option B**, with
per-component GA verification as an explicit pre-sprint-1 action item. The
strategic question is no longer "is AVM the direction" (it is) but "is each
PLZ component you need GA today". Where a needed component is still preview,
fall back per component, not per framework.

**When a hard deadline meets a conservative stakeholder and key PLZ
components are not yet GA: Option D.** The wave-2 rewrite cost is real and
must be presented as a roadmap item, not hidden. Option D is only honest if
the post-go-live migration is planned, budgeted and communicated.

**Option A standalone is no longer defensible for new work**: it means
knowingly building a multi-year asset on a maintenance-mode foundation.

**Option C is the niche play**: it wins only when requirements genuinely
deviate from the ALZ pattern (unusual management group topology,
non-standard policy baseline) or when the delivery organization explicitly
wants to own an opinionated accelerator as its product. The hidden cost is
the policy catalog: recreating the ALZ initiative's policy set and its
parameter mapping is weeks of work that the pattern options include for free.

### Consequences

* We gain: a repeatable way to make this call per engagement instead of
  re-litigating it; a default (AVM-first) aligned with where the ecosystem
  is going.
* We accept: per-component due diligence is now a standing cost of any
  AVM-based choice; "AVM PLZ is GA" must be verified per module, per date.
* We close: classic ALZ-Bicep as a foundation for anything intended to live
  beyond its support horizon.

## Pros and Cons of the Options

### Option A: Classic ALZ-Bicep accelerator

* Good, because fastest time-to-first-deployment: clone, parameterize, deploy.
* Good, because battle-tested across a large number of enterprise rollouts,
  with the richest community documentation and troubleshooting base.
* Good, because the full ALZ policy initiative ships out of the box.
* Good, because "official Microsoft" is the easiest answer for a conservative
  stakeholder.
* Bad, because it is in maintenance mode: building a long-lived standard on
  it schedules a rewrite.
* Bad, because versioning is repo-level tags, not per-module, so
  cherry-picking updates is awkward.
* Bad, because customization beyond the supported parameters tends toward
  forking, and forks of a maintenance-mode upstream age badly.

### Option B: AVM platform landing zone pattern modules

* Good, because it is the strategic direction: a framework built on AVM is
  the one Microsoft converges toward, not away from.
* Good, because per-module semantic versioning via the public Bicep registry
  enables deliberate, per-component upgrades.
* Good, because AVM enforces standards (testing, docs, telemetry, breaking
  change policy) that exceed typical community module quality.
* Good, because the same resource modules serve both the platform and the
  workload layers, giving the framework end-to-end consistency.
* Bad, because maturity is per-component: some PLZ pattern modules may still
  be preview at any given date, which a conservative stakeholder will read
  as risk.
* Bad, because the community knowledge base (blog posts, answered questions)
  is still thinner than for Option A, raising first-mover cost.
* Bad, because the AVM conventions (module classes, mandatory parameters,
  interface specs) are a real learning curve. AI coding assistants reduce
  the "how do I write this" friction but do not reduce the maturity risk;
  they are not a substitute for per-module GA verification.

### Option C: Custom composition on AVM resource modules

* Good, because maximum control over topology, naming, and policy choices.
* Good, because the result can become the delivery organization's own
  accelerator, a genuine differentiator if that is a strategic goal.
* Good, because AVM resource modules are a mature base, so this is far from
  "custom from scratch".
* Bad, because highest upfront effort: orchestration, parameter design and
  the entire policy baseline are yours to build.
* Bad, because you rediscover edge cases the pattern modules already solved.
* Bad, because "why are you not using the Microsoft pattern?" becomes a
  question you must keep answering.

### Option D: Phased (A for v1, migrate to B post-go-live)

* Good, because it decouples the deadline from the strategic destination:
  proven pattern now, future-proof target later.
* Good, because the delivery organization ends up with two generations of
  framework knowledge, which compounds.
* Bad, because the wave-2 migration is a real, unavoidable cost that must be
  sold twice: once internally, once to whoever pays for it.
* Bad, because if v1 "works", the migration may never be funded, silently
  converting Option D into Option A with its full long-term downside.
* Bad, because two standards coexist during the transition, which taxes a
  small team.

## More Information

Date-sensitive facts in this ADR: the support status of classic ALZ-Bicep
and the per-component GA status of AVM PLZ pattern modules change over time.
Re-verify both against the official repositories and
https://azure.github.io/Azure-Verified-Modules/ before reusing this analysis.

Related: ADR-0002 (Deployment Stacks adoption interacts with this choice,
because current AVM PLZ implementations adopt Stacks natively), ADR-0003
(module versioning strategy for the custom layer that exists regardless of
the option chosen here).
