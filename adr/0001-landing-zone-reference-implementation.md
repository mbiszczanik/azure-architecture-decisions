# ADR-0001: Choosing a Landing Zone Reference Implementation in Bicep (2026)

| Field | Value |
| --- | --- |
| Status | Accepted (as a decision framework) |
| Date | 2026-06-12 |
| Revised | 2026-10-25 (see [Revision history](#revision-history)) |
| Decision type | Decision framework (pattern) |
| Tags | landing-zone, bicep, avm, alz, iac-strategy |

## Context and Problem Statement

An organization is standing up a greenfield Azure landing zone to receive
Windows Server workloads migrated from on-premises. The delivery timeline is
hard (a go-live measured in a few months), the operations team that will
inherit the platform has limited Azure, IaC and Git experience, and the
delivery partner intends to reuse the resulting framework for subsequent
engagements, so the choice outlives this single project.

The landing zone will be implemented in Bicep. As of 2026-10-25 the
ecosystem's transition is largely complete: the Azure Verified Modules (AVM)
based Bicep platform landing zone is generally available and the default
starter module of the ALZ accelerator. The classic ALZ-Bicep starter module
was removed from the accelerator on 2026-02-16, and the classic repository
receives only bug fixes, security patches and policy refreshes until
2027-02-16, when it is archived. Maturity is now a question of module
versions, not of components: the accelerator's current release (v2.4.0 of
2026-10-06) references 23 AVM modules, all on 0.x versions.

Choosing wrong has asymmetric costs: a foundation that leaves support in
under four months means the rewrite starts before the platform has settled,
while an AVM foundation on 0.x modules needs version discipline from the
first sprint.

Organizations already running classic ALZ-Bicep face a separate, dated
question: how to move off it before 2027-02-16.

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

**Default for new greenfield engagements: Option B**, with per-module version
and status verification as an explicit pre-sprint-1 action item. The
strategic question is no longer "is AVM the direction" or "is the platform
landing zone GA" (both yes, as of 2026-10-25) but "which version of each AVM
module does the accelerator pin, and what does that version promise". AVM
states that 0.x modules are production ready, and that full semantic
versioning, with breaking changes avoided and signalled by a major version,
applies from v1.0. Below v1.0 a minor version bump carries no such promise.
In practice: pin every module version, read each module's changelog before
a bump, and treat a 0.x minor bump as potentially breaking. Where one
module's behaviour at its pinned version does not fit, replace or wrap that
module (Option C for that piece), not the framework.

**A hard deadline with a conservative stakeholder is also Option B.** The
"official Microsoft" answer now points to the AVM accelerator, which removes
the stakeholder argument that once favoured classic.

**Option D is no longer recommended for new work** (revised 2026-10-25). Its
premise was a proven classic foundation now and a modern one later. With the
AVM platform landing zone GA and the default, and classic ALZ-Bicep archived
on 2027-02-16, a v1 on classic leaves support within months of go-live: the
deferred migration lands during hypercare, and the "proven now" benefit no
longer exists. The trade keeps the rewrite cost and buys nothing.

**Option A standalone is not defensible for new work**: the accelerator no
longer offers it, and its repository is archived on 2027-02-16.

**Option C is the niche play**: it wins only when requirements genuinely
deviate from the ALZ pattern (unusual management group topology,
non-standard policy baseline) or when the delivery organization explicitly
wants to own an opinionated accelerator as its product. The hidden cost is
the policy catalog: recreating the ALZ initiative's policy set and its
parameter mapping is weeks of work that the pattern options include for free.

**Existing classic ALZ-Bicep deployments: migrate in place to the AVM
platform landing zone before 2027-02-16.** Microsoft publishes a migration
guide for in-place brownfield convergence: the accelerator is configured with
identifiers that match the existing environment, the management group
hierarchy stays in place, and ARM converges the resources. As of 2026-10-25
the guide does not yet cover networking, so plan the hub network part
separately and leave it for last.

### Consequences

* We gain: a repeatable way to make this call per engagement instead of
  re-litigating it; a default (AVM-first) aligned with where the ecosystem
  is going.
* We accept: per-module version due diligence is a standing cost of any
  AVM-based choice; until a module reaches v1.0, every version bump is
  reviewed against its changelog.
* We close: classic ALZ-Bicep for new work, both standalone (Option A) and
  as a first phase (Option D).

## Pros and Cons of the Options

### Option A: Classic ALZ-Bicep accelerator

* Good, because battle-tested across a large number of enterprise rollouts,
  with the richest community documentation and troubleshooting base.
* Good, because the full ALZ policy initiative ships out of the box.
* Bad, because it was removed from the accelerator on 2026-02-16 and its
  repository is archived on 2027-02-16 (as of 2026-10-25): building on it
  schedules a rewrite within months.
* Bad, because versioning is repo-level tags, not per-module, so
  cherry-picking updates is awkward.
* Bad, because customization beyond the supported parameters tends toward
  forking, and forks of an upstream that is about to be archived age badly.

### Option B: AVM platform landing zone pattern modules

* Good, because it is GA and the default starter module of the ALZ
  accelerator (as of 2026-10-25), which is also the easiest answer for a
  conservative stakeholder.
* Good, because per-module semantic versioning via the public Bicep registry
  enables deliberate, per-component upgrades.
* Good, because AVM enforces standards (testing, docs, telemetry, breaking
  change policy) that exceed typical community module quality.
* Good, because the same resource modules serve both the platform and the
  workload layers, giving the framework end-to-end consistency.
* Good, because it is built for Deployment Stacks (see ADR-0002): its
  cross-management-group role assignments sit in separate templates because
  stacks do not support them, a limitation the accelerator already handles.
* Bad, because every AVM module the accelerator uses is on a 0.x version
  (23 of 23 at v2.4.0, as of 2026-10-25): production ready per AVM, but
  without the v1.0 promise that breaking changes come with a major version.
* Bad, because the community knowledge base (blog posts, answered questions)
  is still thinner than for Option A, raising first-mover cost.
* Bad, because the AVM conventions (module classes, mandatory parameters,
  interface specs) are a real learning curve. AI coding assistants reduce
  the "how do I write this" friction but do not reduce the version risk;
  they are not a substitute for per-module version verification.

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

Not recommended for new work since 2026-10-25; kept for the record.

* Good, because it decoupled the deadline from the strategic destination
  while the AVM platform landing zone was not yet the default.
* Bad, because (as of 2026-10-25) classic ALZ-Bicep is archived on
  2027-02-16, so a v1 on classic leaves support within months of go-live.
* Bad, because the wave-2 migration is a real, unavoidable cost that must be
  sold twice: once internally, once to whoever pays for it.
* Bad, because if v1 "works", the migration may never be funded, silently
  converting Option D into Option A with its full long-term downside.
* Bad, because two standards coexist during the transition, which taxes a
  small team.

## More Information

Date-sensitive facts in this ADR, each verified on 2026-10-25 against the
primary source. Re-verify them before reusing this analysis.

* Classic ALZ-Bicep: AVM is GA and the default starter module; classic
  starter removed from the accelerator 2026-02-16; bug fixes, security
  patches and policy refreshes until 2027-02-16, then archived.
  Source: https://github.com/Azure/ALZ-Bicep
* Accelerator release v2.4.0 of 2026-10-06 references 23 distinct AVM
  modules (templates and examples), all on 0.x versions.
  Source: https://github.com/Azure/alz-bicep-accelerator
* AVM: 0.x modules are production ready; full semantic versioning applies
  from v1.0.
  Source: https://azure.github.io/Azure-Verified-Modules/resources/faq/
* The accelerator's cross-management-group role-assignment templates exist
  because Deployment Stacks do not support those assignments (stated in the
  templates' own metadata).
  Source: https://github.com/Azure/alz-bicep-accelerator
* In-place migration from classic to the AVM accelerator; networking not
  yet covered.
  Source: https://azure.github.io/Azure-Landing-Zones/bicep/migration/

Related: ADR-0002 (Deployment Stacks adoption interacts with this choice,
because the AVM accelerator is built for Stacks), ADR-0003 (module
versioning strategy for the custom layer that exists regardless of the
option chosen here).

## Revision history

### 2026-10-25: classic end of support dated, Option D retired

Re-checked against the primary sources while writing the long-form article.
Changes: dated the classic ALZ-Bicep support timeline, marked Option D as
not recommended for new work, replaced per-component GA verification with
per-module version and status verification, added the path for existing
classic deployments, and confirmed the Deployment Stacks link to ADR-0002.

The June reasoning, superseded by this revision:

> **Default for new greenfield engagements starting in 2026: Option B**, with
> per-component GA verification as an explicit pre-sprint-1 action item. The
> strategic question is no longer "is AVM the direction" (it is) but "is each
> PLZ component you need GA today". Where a needed component is still preview,
> fall back per component, not per framework.
>
> **When a hard deadline meets a conservative stakeholder and key PLZ
> components are not yet GA: Option D.** The wave-2 rewrite cost is real and
> must be presented as a roadmap item, not hidden. Option D is only honest if
> the post-go-live migration is planned, budgeted and communicated.

It rested on this context: classic ALZ-Bicep was "in maintenance mode with a
declared end-of-support horizon", AVM platform landing zone pattern modules
were "maturing component by component rather than as a monolith", and a
deprecated foundation meant "a planned rewrite within 12 to 24 months".
Option A was credited with the fastest time-to-first-deployment ("clone,
parameterize, deploy") and with being the easiest "official Microsoft"
answer; both no longer hold now that the accelerator defaults to AVM.
Option D was also credited with leaving the delivery organization "two
generations of framework knowledge", a benefit that does not justify a
foundation about to be archived.

### 2026-06-12: accepted

First version.
