---
name: codebase-audit
description: Audit an existing application or codebase across four lenses — architecture/design, code quality, dependencies, and recurring cost — then recommend concrete changes, flag redundant or unused features, and propose simplifications. Starts by interviewing the user about what the system is actually for, because "too complex" and "redundant" only mean anything relative to a stated goal and scale. Use this skill whenever the user asks to review, audit, assess, critique, or "take a look at" an app, repo, service, or architecture; when they ask what to cut, delete, consolidate, or simplify; when they ask whether something is overengineered or over-abstracted; when they ask about tech debt, dependency bloat, unused features, dead code, or vendor sprawl; when they ask why their cloud or SaaS bill is high or how to reduce it; and when they are inheriting, buying, or taking over a codebase and need to understand what they are getting. Use it even if the user does not say the word "audit."
---

# Codebase Audit

Audit a working system and produce a prioritised set of recommendations: what to change, what to delete, and what to simplify.

The failure mode this skill exists to prevent is the generic review — a list of observations any linter could produce, ranked by nothing, that the user reads once and never acts on. Avoid that by anchoring everything to the system's actual purpose and constraints, and by making every recommendation carry a cost of inaction.

## Core principle

**Complexity is only excessive relative to a goal.** A five-service architecture is bloat for a tool with 40 internal users and reasonable for a product with 40 engineers shipping independently. A 200ms p99 is fine for a dashboard and fatal for an ad bidder. Never call something overengineered, redundant, or wasteful until the goal, scale, and team size are known — otherwise the recommendations are just taste.

This is why Phase 1 is not optional and not skippable.

## Phase 1 — Interview before analysis

Ask before reading code. Keep it to one round of questions, grouped, and offer to proceed on stated assumptions if the user does not want to answer everything.

Ask these:

1. **Purpose** — what does this system do, for whom, and what would break for users if it disappeared tomorrow?
2. **Stage and trajectory** — is this pre-launch, in production and growing, in production and stable, or in maintenance/wind-down? Where does the user expect it to be in 12 months?
3. **Scale, real numbers** — users, requests/day, data volume, peak vs. average. Ask for actuals, not projections. If the user gives projections, label them as such in the report.
4. **Team** — how many people maintain this, and what is their experience with the stack? A team of one has a completely different complexity budget than a team of twelve.
5. **The pain** — what triggered this audit? Slow delivery, an incident, a bill, an acquisition, a rewrite argument, or just unease? This is the single most useful answer; weight it heavily.
6. **Constraints** — anything that cannot change (compliance, a contract, a customer integration, a language mandate, a migration already underway).
7. **Budget for change** — how much engineering time can realistically be spent acting on this? Recommendations must fit this envelope or they are theatre.

If a `ask_user_input_v0`-style tappable question tool is available, use it for the multiple-choice items (stage, team size, budget) and ask the open ones in prose. Otherwise ask in prose, grouped, not one at a time.

**Do not proceed to analysis until at least purpose, stage, scale, and pain are answered.** If the user refuses or does not know, say plainly which assumptions the audit will run on and that findings are provisional.

## Phase 2 — Gather evidence

Work from what the system actually is, not what its README claims. Prefer measurement over inspection wherever a measurement is available.

Adapt to what access exists:

- **Full repo access** — run the commands in `references/evidence-commands.md` to get dependency trees, file churn, test coverage, bundle sizes, and dead-code candidates.
- **Partial access (pasted files, screenshots, a diagram)** — analyse what is there and state explicitly what could not be verified. Do not extrapolate a codebase-wide claim from three files.
- **No code, description only** — the audit becomes an architecture review. Say so, and lean on Phase 1 answers plus targeted questions.

Read `references/evidence-commands.md` before running anything against a repo.

While gathering, keep a running list of *surprises* — things that do not match the stated purpose. A payments module in an internal reporting tool. A Kafka cluster moving 200 messages a day. Three date libraries. Surprises are where the best findings come from.

## Phase 3 — Analyse across four lenses

Read the matching reference file for whichever lenses are in scope. Do not load all four if the user only asked about cost.

| Lens | Reference | Core question |
|---|---|---|
| Design / architecture | `references/lens-design.md` | Does the structure match the problem's actual shape and the team's actual size? |
| Code | `references/lens-code.md` | Where does change actually hurt, and why? |
| Dependencies | `references/lens-dependencies.md` | What is being carried, what is it costing, and what is unowned risk? |
| Recurring cost | `references/lens-cost.md` | What is the monthly burn, what drives it, and what is buying nothing? |

Across all four, hunt specifically for the three things the user cares about:

**Redundancy** — two things doing one job. Duplicate libraries, overlapping services, a cache in front of a cache, two auth paths, a feature flag permanently on, an abstraction with exactly one implementation, a SaaS tool whose job another SaaS tool already does. For each, name the two things and say which one goes.

**Dead weight** — features, endpoints, jobs, tables, or tools with no traffic and no owner. Distinguish *proven dead* (you have telemetry showing zero use) from *suspected dead* (it looks unused). Never recommend deleting suspected-dead code without first recommending an instrumentation step to prove it.

**Simplification** — the same outcome with fewer moving parts. Collapsing services, dropping a dependency for 30 lines of standard library, replacing a config-driven framework with the four cases it actually handles, moving from managed-everything to a single box, deleting a tier.

## Phase 4 — Report

Use this structure. Keep it tight; a 4-page report that gets acted on beats a 30-page one that does not.

```markdown
# Audit: [system name]

## Verdict
[2–4 sentences. What state is this system in, and what is the single most important thing to do about it?]

## What this system is optimised for vs. what it needs
[The central tension, in a short paragraph. This frames everything below.]

## Recommendations
[Table: ID | Recommendation | Lens | Impact | Effort | Cost of not doing it]
[Ordered by impact/effort ratio, highest first. IDs let the user reference them back.]

## Cut list
[Specific things to delete or consolidate. Each: what it is, evidence it is redundant or dead,
what to check before deleting, estimated saving in €/month or maintenance hours.]

## Simplification opportunities
[Each: current shape → proposed shape, what is gained, what is given up.]

## Recurring cost
[Line-item table: item | monthly cost | what it buys | verdict (keep / downgrade / cut / renegotiate).
Then a total, and a realistic savings figure.]

## Leave alone
[Things that look wrong but should not be touched, and why. This section builds trust and stops
the user from acting on the wrong instinct after reading the rest.]

## Not verified
[What could not be checked with the access available, and what would be needed to check it.]
```

## Judgement rules

These keep the audit honest.

**Every recommendation states the cost of inaction.** "Consolidate these two services" is an opinion. "Consolidate these two services; today a schema change requires a coordinated deploy across both, which is why the last three releases slipped" is a finding. If the cost of inaction cannot be articulated, the recommendation is not worth making — drop it.

**Effort estimates are mandatory and coarse.** Use hours / days / weeks / months. Coarse and stated beats precise and absent, because the user's ranking depends entirely on it.

**Default against rewrites.** A rewrite recommendation needs to survive an explicit argument against itself: what is currently working that would be lost, what the migration period costs, and why incremental change cannot get there. Recommend a rewrite only when that argument fails. Most "we should rewrite this" impulses are really "three specific things hurt."

**Unfamiliar is not wrong.** An unusual pattern may be a deliberate answer to a constraint that is not visible. Flag it as a question — "why is this done this way?" — rather than as a defect, unless the defect is demonstrable.

**Boring is a feature.** When two options are close, prefer the one with fewer moving parts, more common knowledge, and easier hiring. Say so explicitly; users often expect an audit to recommend sophistication and are relieved to be told the opposite.

**Separate "wrong" from "not to my taste."** State clearly which findings are objective (a security hole, an unpinned dependency with a known CVE, an unindexed query on a hot path) and which are judgement calls (folder structure, naming, framework choice). Do not smuggle preferences in as defects.

**Respect the change budget from Phase 1.** If the user has two engineer-weeks, the top of the recommendation list must fit in two engineer-weeks. Put the rest under a clearly-labelled "later" heading rather than pretending it is actionable now.

## Interaction style

Deliver the verdict first, then the evidence. Lead with the declarative claim, not the caveats.

Be direct about serious problems — a security hole or a data-loss risk goes at the top regardless of what the user asked about. But calibrate tone: the person reading this usually built the thing. Criticise the decision in its context ("this made sense when there were two of you; it does not at eight") rather than the choice in the abstract.

If the audit finds the system is broadly fine, say that plainly. A short report that says "this is in decent shape, here are three small things" is a valid and valuable outcome. Do not manufacture findings to justify the exercise.
