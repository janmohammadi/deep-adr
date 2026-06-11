# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

`deep-adr` is **not an application** — it is a distributable family of four authored agent *skills* for co-thinking Architecture Decision Records (ADRs). The deliverables are Markdown prompt files, not running software. There is no test suite, no lint config, and the only executable is one zero-dependency Node script.

The four skills live in [skills/](skills/), one directory each with a `SKILL.md`:

| Skill | Role |
|---|---|
| [adr-discovery](skills/adr-discovery/SKILL.md) | Gathers project context before drafting, zero-hallucination Q&A, writes `docs/architecture/discovery-brief.md`. Captures measurable facts for Architecture Contracts. Optional live LikeC4 diagram mode. |
| [draft-adr](skills/draft-adr/SKILL.md) | Co-drafts an ADR through 7 phases (Understand → Context → Options → Decide → Draft → Self-Critique → Save), including an Architecture Contract section. |
| [adr-critique](skills/adr-critique/SKILL.md) | Audits a pre-existing/legacy ADR line-by-line against the checklist, including weak or missing Architecture Contracts. |
| [c4-model](skills/c4-model/SKILL.md) | Generates a canonical-C4 LikeC4 model (Context + Container views only), reflected from confirmed discovery facts and embedded ADL assertions. |

## The only build command

Copilot Chat cannot read `skills/*/SKILL.md`; it discovers slash prompts from `.github/prompts/*.prompt.md`. After editing **any** `SKILL.md`, regenerate the parallel Copilot copies:

```bash
node scripts/build-copilot-prompts.mjs
```

The script ([scripts/build-copilot-prompts.mjs](scripts/build-copilot-prompts.mjs)) strips each skill's YAML frontmatter, rewrites it into Copilot's format (`description` / `name` / `agent: 'agent'`), and writes `.github/prompts/<name>.prompt.md`. The body is copied verbatim. It uses only Node built-ins — no `npm install`.

**`skills/*/SKILL.md` is the single source of truth.** Never hand-edit `.github/prompts/*.prompt.md` — those are generated artifacts and will be overwritten. Edit the SKILL.md, then re-run the build.

## Architecture and cross-skill invariants

These rules are duplicated across the four SKILL.md files by design. When you change one, check whether the others must change too — they are meant to stay consistent.

- **The "ADR is NOT" checklist** appears verbatim in all four skills. `adr-critique` is its canonical home (it says so in its own text). Treat any edit to the checklist as a four-file change.
- **Shared term definitions** (Component = C4 Container, System, Architectural characteristic, Tension, RFC, Fitness function, ADL, Architecture Contract) are restated in each skill so the skill is self-contained. Keep the definitions aligned across files.
- **Architecture Contract is the canonical ADR section name.** It replaces `Compliance` and contains compact ADL assertions with inline `check` metadata only. If a decision has no enforceable assertions, the section says `No enforceable architecture contract for this ADR.`
- **LikeC4 is reflected from ADL, not authored as a sibling contract.** ADRs do not carry a Model subsection. LikeC4 elements/views link back to the ADR using LikeC4 `link` metadata; full LikeC4 DSL lives under `likec4/`.
- **One-question-per-message** is the load-bearing behavior of `adr-discovery` and `draft-adr`. Never let edits collapse a phase into a wall of text or a bulk option table — that defeats the skill.
- **Zero hallucination / human-confirmed facts** is the core rule of `adr-discovery` and `c4-model` intake: nothing about the project counts as known until the human confirms it.
- **Parser-avoidance file paths**: `discovery-brief.md` and `open-questions.md` MUST live at `docs/architecture/` (NOT inside `docs/adr/`, `adr/`, `docs/decisions/`, or `docs/architecture/decisions/`) — those four directories are scanned by ADR parsers and the helper files would be mistaken for ADRs. Every skill repeats this constraint.
- **The skill handoff chain**: `adr-discovery` → writes brief → `draft-adr` and `c4-model` read the brief instead of re-asking. `c4-model` reads the same brief `adr-discovery` writes. Keep the brief's section headings (`## Domain`, `## Components`, `## Relationships`, etc.) consistent across the writer and the readers.
- **canonical-C4 lock**: `c4-model` (and `adr-discovery`'s live diagram mode) deliberately restrict LikeC4 to the Simon Brown C4 style — only `actor / externalSystem / system / container` kinds, only `uses/reads/writes/publishes/consumes` relationships, Context + Container (+ optional Deployment) views. Refusals are scripted. Do not relax these toward general-purpose LikeC4.

## ADR output conventions the skills produce

- ADR files: `NNNN-kebab-case-title.md`, written to the first existing of `docs/adr/`, `docs/decisions/`, `docs/architecture/decisions/`, `adr/` (else create `docs/adr/`).
- ADR frontmatter and section template live in [draft-adr](skills/draft-adr/SKILL.md) (search "Template"). Status values: `proposed | rfc | accepted | superseded | deprecated`.
- ADR body sections are Context, Decision, Consequences, Architecture Contract, Alternatives Considered, Notes.

## External dependency

`c4-model` targets **LikeC4 v1.47+** and validates with `npx likec4 validate` / previews with `npx likec4 start`. LikeC4 DSL drifts across versions — the skill itself instructs consulting `/likec4/likec4` docs via `ctx7` rather than hand-writing complex DSL from memory.
