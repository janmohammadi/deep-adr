# deep-adr

A family of four Claude skills that co-think Architecture Decision Records with the architect — challenging, pushing back, and refusing to write an ADR until the reasoning holds up. Designed to fix the failure mode of AI-generated ADRs stuffed with filler, hedging, and implementation detail.

These are **not** template fillers. They behave like a senior architect in a review.

![deep-adr — four skills for Architecture Decision Records](media/images/deep-adr.png)

[![skills.sh](https://skills.sh/b/janmohammadi/deep-adr)](https://skills.sh/janmohammadi/deep-adr)

## Quick start

```bash
npx skills add janmohammadi/deep-adr --all
```

Then invoke any skill from your agent: `/adr-discovery`, `/draft-adr`, `/adr-critique`, `/c4-model`. For GitHub Copilot or per-skill installs, see [Install](#install) below.

> **Pairs with [ADR Explorer](https://github.com/janmohammadi/adr-explorer)** — once `deep-adr` has helped you produce a corpus of ADRs on disk, ADR Explorer visualizes the graph (`supersedes` / `amends` / `relates-to`), scores corpus health, surfaces stale or orphan decisions, and can distill bloat. Available as `npx adr-explorer` or as a [VS Code extension](https://marketplace.visualstudio.com/items?itemName=reza-janm.adr-explorer).

![deep-adr to ADR Explorer pipeline](media/images/adr-explorer-flow.png)

## The four skills

| Skill | When to use |
|---|---|
| [`adr-discovery`](skills/adr-discovery/SKILL.md) | Before drafting, especially on unfamiliar systems. Gathers project context through back-and-forth Q&A with **zero hallucination** — every fact confirmed by the human before it counts. Captures the measurable facts needed for an ADR's Architecture Contract and logs unknowns to `docs/architecture/open-questions.md` with where-to-look / who-to-ask guidance. |
| [`draft-adr`](skills/draft-adr/SKILL.md) | When the architect has a decision to make. Co-thinks through Understand → Context → Options → Decide → Draft → **Self-Critique** → Save. Pushes back, drafts an ADL-aware Architecture Contract when the decision creates enforceable assertions, and refuses to write if context is thin or reasoning is weak. 7-phase flow. |
| [`adr-critique`](skills/adr-critique/SKILL.md) | Auditing an existing or legacy ADR that wasn't drafted via `draft-adr`. Line-level flags against the ADR-is-NOT checklist with per-line rewrite approval. Catches missing-why, weak Architecture Contracts, inconsistency, and LikeC4 drift. |
| [`c4-model`](skills/c4-model/SKILL.md) | Generating a **canonical-C4** LikeC4 model reflected from confirmed architectural elements and embedded ADL assertions. Context + Container views only, optionally Deployment. Refuses Component views, custom element kinds, dynamic views, and other LikeC4 features that deviate from Simon Brown C4 conventions. |

## Typical workflow

![deep-adr typical workflow](media/images/flow-chart.png)

## What makes these different

- **Zero hallucination** — `adr-discovery` never states a fact about the project the human hasn't confirmed. Every finding from code is presented as *"I found X in [file]. Is this accurate?"* and waits for a yes/no. Business domain, component purpose, and relationships are always asked, never inferred.
- **Push-back, not capitulation** — `draft-adr` is scripted to challenge weak reasoning, name the strongest counter-argument, and require the architect to articulate 2–3 failure modes before accepting a decision. Forbidden affirmations: "Great question", "Solid approach", "Good thinking".
- **Architecture Contract, not vague compliance** — ADRs can embed a compact `adl` block where each assertion carries its own check metadata. The ADR keeps the human why; the Architecture Contract records the objective facts that must stay true.
- **Self-critique before save** — `draft-adr` runs the ADR-IS-NOT checklist on its own draft output, quotes violating lines, rewrites them, and presents the revised draft for approval. You never see its first draft.
- **Canonical-C4 by construction** — `c4-model` locks LikeC4's flexible DSL to the Simon Brown C4 Model style and reflects embedded ADL assertions into the model. No Component views, no custom kinds, no dynamic views. In-skill lint runs before `likec4 validate`.
- **Open-questions mechanism** — when the architect doesn't know something, the skills don't fabricate. They help scope the answer (where to look, who to ask) and log to `docs/architecture/open-questions.md` — a path deliberately outside the four directories typically scanned by ADR parsers.

## Architecture Contract

`deep-adr` uses an **Architecture Contract** section instead of a vague `Compliance` section. The contract keeps the enforceable assertion and its check metadata in one place:

- **ADL assertions** — the objective architecture facts that must hold.
- **Inline checks** — how each assertion is checked, including trigger, owner, and severity.

The ADL block is optional. Use it when the decision creates enforceable architectural assertions; otherwise write `No enforceable architecture contract for this ADR.`

````markdown
## Architecture Contract

```adl
system "Checkout"
  service "Order API"
    assert order-api-latency: p95_latency <= 600ms under 5000 concurrent_users
      check runtime_monitor trigger continuous owner "Platform" severity feedback

    assert order-db-writes: writes_only database "orders"
      check static_dependency trigger CI owner "Checkout Team" severity stop
```
````

The LikeC4 model is reflected from the ADL and confirmed relationships. The diagram links back to the ADR, not the other way around. `c4-model` adds LikeC4 `link` metadata to affected elements or views:

```likec4
extend checkout.orderApi {
  link ../docs/adr/0007-checkout-architecture-contract.md "ADR-0007"
}
```

Full model DSL belongs in `likec4/`, not inside ADR files.

## Install

### Claude Code plugin (one command, all four skills)

`deep-adr` ships as a Claude Code plugin, so you can install all four skills in one step — no `--all` flag, no per-skill picking:

```bash
/plugin marketplace add janmohammadi/deep-adr
/plugin install deep-adr@deep-adr
```

This bundles `adr-discovery`, `draft-adr`, `adr-critique`, and `c4-model` together. Plugin skills are namespaced, so invoke them as `/deep-adr:adr-discovery`, `/deep-adr:draft-adr`, `/deep-adr:adr-critique`, `/deep-adr:c4-model`. Updates arrive automatically on each new commit; pull them with `/plugin marketplace update`.

### Claude Code, Cursor, OpenCode, etc. (via `skills` CLI)

Install all four skills to any supported agent via the [`skills`](https://github.com/vercel-labs/skills) CLI:

```bash
npx skills add janmohammadi/deep-adr --all
```

### GitHub Copilot

The `skills` CLI's `--agent github-copilot` option copies `SKILL.md` files to `.agents/skills/`, but Copilot Chat doesn't read that directory — it reads `.github/prompts/*.prompt.md` for slash-invokable prompts. To work around this, we ship **pre-built Copilot prompt files** in [.github/prompts/](.github/prompts/) alongside the canonical `skills/*/SKILL.md`.

**To use in your project:**

```bash
# clone or download this repo, then copy the prompt files into your project
mkdir -p .github/prompts
cp -r /path/to/deep-adr/.github/prompts/*.prompt.md .github/prompts/
```

Or, using `curl`:

```bash
mkdir -p .github/prompts
for name in adr-discovery draft-adr adr-critique c4-model; do
  curl -fsSL "https://raw.githubusercontent.com/janmohammadi/deep-adr/main/.github/prompts/$name.prompt.md" \
    -o ".github/prompts/$name.prompt.md"
done
```

Open the VS Code Copilot Chat and invoke: `/adr-discovery`, `/draft-adr`, `/adr-critique`, `/c4-model` — the slash-command picker lists them automatically.

**Keeping `SKILL.md` and `.prompt.md` in sync (maintainers only):**

The `.prompt.md` files are generated from the canonical `skills/*/SKILL.md` by a plain Node script. If you edit a skill, regenerate:

```bash
node scripts/build-copilot-prompts.mjs
```

No dependencies — uses only Node's built-ins.

## Requirements

- **Claude Code** or any agent supported by the `skills` CLI; OR **GitHub Copilot** in VS Code (see Copilot install section above).
- **LikeC4 CLI** (required only for `c4-model`): `npx likec4 validate` and `npx likec4 start`.
- **Node.js** (optional, only for maintainers who want to regenerate the Copilot prompt files after editing a `SKILL.md`).

## ADR is NOT

Every skill enforces the same checklist on ADR content:

```
An ADR is NOT:
- A tutorial. Don't explain what REST is, what a queue is, what Kafka does.
- An implementation guide. No code snippets except fitness functions.
  Compact ADL assertions with inline check metadata are allowed.
  Full LikeC4 DSL, config samples, API signatures, and deployment commands are not.
- A marketing doc. No "leverage", "robust", "scalable", "enterprise-grade",
  "best-in-class", "industry-leading", "seamless", "cutting-edge".
- A hedge. No "it might be good to consider potentially evaluating...".
- A generic best-practice citation. "Industry standard" is not a reason.
- A probability-weighted LLM summary. If the only justification is "this is
  how most teams do it", that is an abdication, not a justification.
- A future-proofing essay. Decisions are made for known forces today.
- Corporate passive voice. "It was decided" is wrong. "We use X" is right.
- A design doc. Implementation detail belongs in the design doc or the code.
- Long. Context ≤ 3 sentences. Decision ≤ 3 sentences. Consequences as bullets.
```

## Inspired by

<a href="https://learning.oreilly.com/library/view/fundamentals-of-software/9781492043447/"><img src="media/images/book.jpg" alt="Fundamentals of Software Architecture" align="right" width="160"></a>

The ADR philosophy baked into these skills draws heavily from **[Fundamentals of Software Architecture](https://learning.oreilly.com/library/view/fundamentals-of-software/9781492043447/)** by Mark Richards and Neal Ford (O'Reilly, 2020) — in particular **Chapter 19, "Architecture Decisions"**. The chapter's framing of ADRs as records of *why* (not *what*), its emphasis on stating consequences honestly, and its warnings against trivial or implementation-detail decisions all show up in the "ADR is NOT" checklist and the push-back behavior of `draft-adr` and `adr-critique`.

<a href="https://learning.oreilly.com/library/view/architecture-as-code/9798341640368/"><img src="media/images/architecture-as-code.jpg" alt="Architecture as Code" align="right" width="160"></a>

The **Architecture Contract** section — embedding compact ADL assertions with inline `check` metadata directly in the ADR — adapts the architecture-as-code practice from **[Architecture as Code](https://learning.oreilly.com/library/view/architecture-as-code/9798341640368/)** by Neal Ford and Mark Richards (O'Reilly, 2026). That book's case for expressing architectural intent as checkable, version-controlled assertions is what replaces the vague `Compliance` section with an enforceable contract.

If you find these skills useful, read the chapters — they explain the *reasoning* behind the rules the skills enforce.

## License

MIT — see [LICENSE](LICENSE).
