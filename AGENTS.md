# AGENTS.md

## Purpose

Public log of Azure platform decision records (ADRs) in MADR format, licensed CC BY 4.0. It is the topic source for the author's blog, so a record and its article must say the same thing. `README.md` has the index and the backlog; `adr/template.md` is the template.

## Setup

Node and `markdownlint-cli2` are pinned in `mise.toml`; `mise install` provides them.

## Commands

- `mise run check:fast`: Markdown lint of every `*.md` with `.markdownlint.yaml`. Seconds, offline.
- `mise run check`: the offline lane (same as `check:fast` today). The link check (`lychee`) needs the network and runs only in CI (`.github/workflows/quality.yml`).

## Conventions

- Work goes through a branch and a pull request; never commit to `main`.
- No record describes a specific client, engagement or environment. Contexts are generic patterns; numbers, regions and timelines that could match a real project are removed.
- Every Azure fact (GA status, version, support date, module status) carries an "as of" date and is re-verified against a primary source before it is reused.
- English, no hype; one or two real numbers instead of adjectives. Headings are the question a reader would ask; the first sentence answers it.
- A new record updates the README index and removes its line from the backlog in the same change.
- Never present work the author did not author as the author's own.

## Before publishing a record as a blog article

- Re-verify every date-sensitive fact on the publishing day.
- If the article changes a conclusion, update the record in the same week, so the two do not drift.
