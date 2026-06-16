# Azure Architecture Decisions

A public log of architecture decision records (ADRs) for Azure platform
engineering, written as reusable decision frameworks.

## What this is

Most public Azure content explains **how** to deploy something. This vault
documents **why**: the options that were on the table, the trade-offs that
mattered, and the conditions under which each option wins. Every record
follows the [MADR](https://adr.github.io/madr/) format, adapted with a
"decision framework" type for records that capture a context-dependent
pattern rather than a single project's verdict.

## What this is not

No record here describes a specific client, engagement, or environment. All
ADRs are written as generic patterns; contexts are deliberately
de-identified and any resemblance of numbers, regions or timelines to a real
project is removed by design. Facts about the Azure ecosystem (product GA
status, support timelines) are accurate as of each record's date and flagged
as date-sensitive where they can age.

## Index

| ID | Title | Status | Tags |
| --- | --- | --- | --- |
| [ADR-0001](adr/0001-landing-zone-reference-implementation.md) | Choosing a Landing Zone Reference Implementation in Bicep (2026) | Accepted (framework) | landing-zone, bicep, avm, alz |
| [ADR-0002](adr/0002-deployment-stacks-adopt-or-compensate.md) | Azure Deployment Stacks: Adopt Now or Compensate | Accepted (framework) | deployment-stacks, drift, governance |
| [ADR-0003](adr/0003-bicep-module-versioning.md) | Versioning Custom Bicep Modules: Path-Based, Private Registry, or Template Specs | Accepted (framework) | bicep, modules, versioning, acr |

## Backlog

Decisions queued for future records:

* Monorepo vs poly-repo for an IaC platform framework.
* Pipeline secrets: Key Vault references vs CI variable groups, and why
  "secret pipeline variables" is the anti-pattern in between.
* Trunk-based development for a team new to Git and IaC: branching as a
  skill-fit decision, not a religion.
* Workload identity federation (OIDC) as the only pipeline authentication
  mechanism.
* PSRule vs Pester: where static analysis ends and behavioral assertion
  begins in IaC testing.
* Az CLI vs Azure PowerShell as the default toolchain in Bicep pipelines.
* APIM as an AI gateway in front of Azure OpenAI: when a gateway from day
  one prevents a phase-two rearchitecture.

## Conventions

* One decision per record, numbered `NNNN-kebab-case-title.md` under `adr/`.
* New records start from [adr/template.md](adr/template.md).
* Records are immutable once accepted; revisiting a decision produces a new
  record that supersedes the old one.
* Each record names its date-sensitive facts so future readers know what to
  re-verify.

## License

Content is licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).
You are welcome to reuse these frameworks with attribution.

## Author

Marcin Biszczanik, Cloud & Platform Consultant.
[GitHub profile](https://github.com/mbiszczanik) |
[LinkedIn](https://www.linkedin.com/in/marcin-biszczanik/)
