# ADR-0003: Versioning Custom Bicep Modules: Path-Based, Private Registry, or Template Specs

| Field | Value |
| --- | --- |
| Status | Accepted (as a decision framework) |
| Date | 2026-06-12 |
| Decision type | Decision framework (pattern) |
| Tags | bicep, modules, versioning, acr, registry, monorepo |

## Context and Problem Statement

Even a framework that maximizes consumption of public AVM modules ends up
with a custom layer: a naming module, a tagging module, opinionated wrappers
with organizational defaults, helpers for diagnostics and locks. These
modules need a versioning and distribution story.

The decision matters twice. Within a single project, it determines how
changes propagate and how atomic a cross-cutting refactor can be. Across
projects, it determines whether the custom layer is a copy-paste liability
or a governed, consumable asset. Choosing heavyweight distribution too early
buys infrastructure and process before there is a second consumer; choosing
too late means several projects already drifted apart on incompatible copies.

## Decision Drivers

* Number of consumers: one repository today vs several projects tomorrow.
* Atomicity of cross-cutting changes (e.g. a naming refactor touching every
  layer at once).
* Upgrade ergonomics for consumers: deliberate, per-version adoption vs
  implicit latest.
* Infrastructure and governance cost: what must exist and be operated for
  the option to work.
* Reversibility: how expensive is migrating from this option to another.

## Considered Options

* Option A: Path-based imports in a monorepo (version = git commit)
* Option B: Private Bicep registry in Azure Container Registry, semantic
  versioning
* Option C: ARM Template Specs as the distribution vehicle

## Decision Outcome

**Single project, single repository: Option A.** Path-based imports cost
nothing, keep cross-cutting changes atomic in one PR, and the git history is
the version history. Pay the registry tax only when a second consumer
actually exists.

**The moment a second project consumes the modules: Option B**, and the
migration should be planned rather than discovered. The trigger is concrete:
the first time someone proposes copying `shared/` into another repository,
that copy is the fork you will never merge back. Stand up the registry
instead.

A workable bridge: keep developing in the monorepo (Option A ergonomics) and
publish tagged releases of selected modules to ACR via a pipeline. Consumers
get semver; maintainers keep atomic refactors. This is the standard
inner-source pattern and the recommended target state for a framework
intended as a multi-project standard.

**Option C is not recommended as the primary mechanism** for Bicep-native
teams. It solves distribution but fights the toolchain: no semver semantics
(versions are names), weaker authoring experience, and a second artifact
type to govern alongside the registry you will likely need anyway.

### Consequences

* We gain: a cheap default with a concrete, observable trigger for when to
  upgrade the machinery, instead of either premature infrastructure or
  perpetual copy-paste.
* We accept: under Option A, "version" means a commit hash; consumers inside
  the monorepo always build against head, so a breaking module change must
  fix all in-repo consumers in the same PR (this is a feature, but it is
  also a discipline).
* We accept: under Option B, someone owns the registry: publishing pipeline,
  retention, access for other projects, and a breaking-change policy
  (semver major bumps, deprecation windows).

## Pros and Cons of the Options

### Option A: Path-based imports in a monorepo

* Good, because zero infrastructure: no registry, no publishing pipeline,
  no credentials story.
* Good, because cross-cutting refactors are atomic: one PR updates the
  module and every consumer, and CI proves the whole repo still builds.
* Good, because review happens in one place with full context.
* Bad, because there is no consumption from outside the repository without
  copying, and copies fork silently.
* Bad, because there is no deliberate version adoption: every consumer in
  the repo moves to the new module behavior on merge, ready or not.
* Bad, because "which version of naming does project X use" has no better
  answer than a commit hash.

### Option B: Private Bicep registry in ACR with semantic versioning

* Good, because consumers pin `br:<registry>/bicep/<module>:1.2.0` and
  upgrade deliberately, per module, per project.
* Good, because semver communicates intent: a major bump is a contract
  change, visible in the diff of one line.
* Good, because it scales to many projects and supports a real
  deprecation/upgrade policy for a firm-wide standard.
* Bad, because it is standing infrastructure: an ACR, a publish pipeline,
  RBAC for consumers, and someone accountable for all of it.
* Bad, because cross-cutting changes now span publish-then-consume cycles
  instead of one atomic PR.
* Bad, because version sprawl is real: without a support policy, old majors
  accumulate consumers that never upgrade.

### Option C: ARM Template Specs

* Good, because distribution is native to ARM: RBAC-controlled, no ACR
  needed, consumable by deployment tooling beyond Bicep.
* Good, because specs are first-class Azure resources, so governance tooling
  (Policy, RBAC, activity log) applies directly.
* Bad, because versioning is a flat namespace of names, not semver; nothing
  expresses "breaking" except convention.
* Bad, because the Bicep authoring loop (build, publish, reference) is
  clunkier than registry references, and module metadata support is weaker.
* Bad, because for a Bicep-first team it adds a second distribution
  mechanism without retiring the need for the first.

## More Information

A structural convention that makes any option work better: each custom
module ships as a folder with `main.bicep`, a `README.md` (purpose,
parameters, outputs, an example), a `tests/main.test.bicep` reference
deployment, and a `version.json` carrying the semantic version even before a
registry exists. Versioning metadata that predates the registry makes the
Option A to B migration mechanical instead of archaeological.

Date-sensitive facts: Bicep registry capabilities and template spec
limitations evolve; verify against
https://learn.microsoft.com/azure/azure-resource-manager/bicep/private-module-registry
before reuse.

Related: ADR-0001 (the more the framework leans on public AVM, the smaller
the custom layer this ADR governs), ADR-0002 (stack-managed scopes affect
how module upgrades roll out to consumers).
