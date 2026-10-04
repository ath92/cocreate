# Cocreate — Concept & Design Document

**An agentic platform for building and running web apps around a closed human-feedback loop.**

Status: v1 concept draft · Working title: *Cocreate* · Date: draft

---

## Contents

1. [TL;DR — the thesis in one page](#1-tldr--the-thesis-in-one-page)
2. [The problem: building got cheap, knowing got expensive](#2-the-problem-building-got-cheap-knowing-got-expensive)
3. [The core insight and the three bets](#3-the-core-insight-and-the-three-bets)
4. [What the platform is](#4-what-the-platform-is)
5. [The two primitives that make everything else cheap](#5-the-two-primitives-that-make-everything-else-cheap)
6. [The feedback loop as a control system](#6-the-feedback-loop-as-a-control-system)
7. [The five signal channels](#7-the-five-signal-channels)
8. [Consensus: the hardest and most differentiated part](#8-consensus-the-hardest-and-most-differentiated-part)
9. [The agent organisation](#9-the-agent-organisation)
10. [Architecture](#10-architecture)
11. [Worked example: the café rota app](#11-worked-example-the-café-rota-app)
12. [Governance, safety, and ethics](#12-governance-safety-and-ethics)
13. [Business: wedge, ICP, pricing, moat](#13-business-wedge-icp-pricing-moat)
14. [Competitive landscape and positioning](#14-competitive-landscape-and-positioning)
15. [Risks and failure modes](#15-risks-and-failure-modes)
16. [Riskiest assumptions and how to falsify them](#16-riskiest-assumptions-and-how-to-falsify-them)
17. [Roadmap and MVP scope](#17-roadmap-and-mvp-scope)
18. [Metrics](#18-metrics)
19. [Open decisions, with recommendations](#19-open-decisions-with-recommendations)
20. [Naming](#20-naming)
21. [Glossary](#21-glossary)

---

## 1. TL;DR — the thesis in one page

Generating a plausible web app is now a commodity. Almost every agentic builder can turn a prompt into a working frontend and backend in minutes (Lovable, v0, Bolt, Replit Agent, Base44, Firebase Studio, and a long tail — a category that is crowded and moving fast). What is *not* commoditised is knowing whether the resulting app is any good for the humans it was built for, and keeping it good as it changes.

**Cocreate is a platform where agents build and run web apps, and where every app is born instrumented so that human evidence flows back into the build automatically.**

The bet is that the scarce resource has moved. It used to be code. Now it is *validated evidence about what to build and whether it works*. Whoever owns the fastest, cheapest, most trustworthy loop from "idea" to "evidence" to "shipped change" wins — and that loop only works if the app itself is machine-legible.

Three claims make the platform work:

1. **The spec is the app.** One declarative Product Spec IR is the source of truth. The running app, its end-to-end tests, its telemetry taxonomy, its guided-user-test scripts, its feedback affordances, and its accessibility assertions are all *derived* from it. Instrumentation cost collapses to near zero because it is not a separate project — it is a build output.
2. **The Action Surface is the shared interface between agents and humans.** Every generated app publishes a machine-readable manifest of its user-perceivable actions (stable IDs, preconditions, effects, the job each serves, the ARIA node that realises it). Agents drive tests through it. Humans are guided through tests using the same labels. Telemetry is emitted at it. Feedback is attached to it. Agents and humans operate *the same description of the app* — which makes their evidence comparable and their tests stable.
3. **Every signal carries provenance, and synthetic evidence can never close a loop.** Signals are typed `human_observed`, `human_reported`, `machine_inferred`, or `synthetic_simulated`. The platform refuses to let an agent claim success on synthetic evidence. This is what stops the whole system from becoming a very expensive machine for confirming its own assumptions.

North star metric: **Time-to-Evidence** — hours from "we think this should change" to "we have human evidence that it did (or didn't)". Incumbent reality is weeks. Target is under 48 hours, eventually under 4.

Primary wedge: **land as the feedback and evidence layer for apps built by any tool, then expand into building.** Competing head-on with funded app builders on day one is a bad idea. Owning the loop is a better idea — and once you own the loop, you own the roadmap, and building is the obvious next step.

---

## 2. The problem: building got cheap, knowing got expensive

### 2.1 The build cost collapse left a vacuum downstream

When code generation was expensive, most product failure was attributable to not shipping. Now generation is nearly free and the failure mode has changed shape. It is trivially easy to produce a large, polished, plausible application that nobody wants, and equally easy to produce five of them. Vibe-coding tools optimised for the first 20 minutes of a project; almost nobody has optimised the *second* 20 hours, when you find out whether the thing works.

Three specific gaps follow from this:

**Gap 1: agents can write the app but cannot operate it.** After generation, the app is an opaque pile of markup. An agent asked to verify a change has to re-derive how to drive the UI from scratch, every time — brittle selectors, guessed flows, no notion of what the app is *for*. This is why agent-run browser testing is still flaky in practice despite good tooling. The app wasn't built to be operated.

**Gap 2: human evidence lives in a different tool, and a different head.** Telemetry is in an analytics tool, feedback boards are in another, user research is a calendar-blocked human activity, and the code is in a third. A human PM is the integration layer — reading dashboards, writing up findings, translating them into tickets. That human is the bottleneck, and their translation is lossy.

**Gap 3: the loop is slow, so it runs rarely, so it produces weak priors.** When a full learn-cycle takes three weeks, you get maybe fifteen cycles a year. Fifteen data points is not enough to distinguish a good product decision from a lucky one, so teams fall back on taste and seniority. Speed isn't a convenience here; it's what makes the loop statistically meaningful at all.

### 2.2 The structural asymmetry

| | Cost today | Cost driver | Who pays |
|---|---|---|---|
| Generate an app | ~zero | inference tokens | anyone |
| Operate it as a user would | high | agent re-derives the UI every time | the builder |
| Get a human's real reaction | high | recruiting, scheduling, incentives, analysis | the builder |
| Aggregate many human reactions into a decision | very high | a human PM's judgement | the builder |
| Keep it all true as the app changes | unmeasured | nobody owns it | future users |

The bottom four rows are the business. The top row is a feature.

### 2.3 What "without reading or writing code" actually requires

The user-facing promise — build and run without reading or writing code — has a trap in it. Non-technical builders do not need to read *code*, but they absolutely need to inspect *behaviour and intent*. If the platform hides everything behind a chat box, the user can't audit what they're agreeing to, and trust collapses the first time something surprising happens.

So the interface principle is: **no code, but full legibility.** The user reviews:

- the **spec** (jobs, personas, flows, rules, policies) — in structured prose, not syntax
- the **preview app** itself
- the **evidence ledger** (what we believe, why, and how strongly)
- the **decision records** (what changed, what was rejected, who disagreed)
- the **guardrails** (what the agents are not allowed to do)

Four review surfaces instead of four hundred files. That is a genuine simplification, not a dodge.

---

## 3. The core insight and the three bets

**Core insight:** if an app is generated from a machine-readable spec, then everything needed to test, measure, research, and critique it can be *derived* from that same spec. Feedback infrastructure stops being a project and becomes a build output.

This is not a wholly novel observation in the abstract — intermediate representations for scaffolded app generation exist in the literature ([Athena: Intermediate Representations for Iterative Scaffolded App Generation](https://acm-stag.literatumonline.com/doi/pdf/10.1145/3742413.3789133?download=true)). What is novel is using the IR as the *shared substrate for human evidence*, not just as a scaffolding convenience for codegen.

### Bet 1 — Derivable instrumentation

From one spec, derive:

| Derived artifact | Derived from |
|---|---|
| Frontend + backend | flows, data model, rules, integrations |
| E2E "job tests" | user jobs + flows |
| Telemetry taxonomy | jobs + actions + success criteria |
| Guided user test scripts | jobs + hypothesised friction + success criteria |
| In-app feedback affordances | surfaces + actions (attach to what the user is looking at) |
| Accessibility assertions | action semantics (a11y and testability are the same tree) |
| Authorisation test matrix | roles × entities × permissions |
| Performance budgets | critical flows |
| Message board structure | problem domains derived from jobs |
| Guardrail definitions | metrics + policies |

Nine artifacts, one source. Change the spec, all nine update. **This is the compounding advantage of the architecture**, and it is why the platform can offer per-iteration research at a price a human agency cannot approach.

### Bet 2 — A shared Action Surface

The single most important technical decision in this document. Generated apps must expose a **semantic action layer**, not just DOM.

```jsonc
// GET /__spec/actions  (dev + preview only, not public prod)
{
  "surfaceVersion": "1.4.0",
  "app": "rota",
  "actions": [
    {
      "id": "shift.request_swap",
      "label": "Ask someone to take my shift",
      "jobs": ["J1"],
      "surface": "schedule.shift_detail",
      "preconditions": ["viewer.owns(shift)", "shift.start > now + 24h", "shift.status == 'assigned'"],
      "effects": ["swap_request.created", "notifications.sent(candidates)"],
      "inputs": [
        { "name": "reason", "type": "string?", "maxLength": 280 },
        { "name": "candidateIds", "type": "PersonId[]", "minItems": 1 }
      ],
      "aria": { "role": "button", "name": "Ask someone to take my shift" },
      "telemetry": ["action.shift.request_swap.start", "action.shift.request_swap.success"]
    }
  ],
  "jobs": [
    { "id": "J1", "statement": "Give away a shift I can't work without phoning anyone",
      "success": "swap.accepted_within(4h) && no manager_intervention",
      "personas": ["barista"] }
  ]
}
```

Consequences:

- E2E tests address actions by stable ID, never by CSS selector or pixel position. Refactors don't break tests.
- A11y and testability share one tree. An accessible app is a testable app, and vice versa. This is a real, under-exploited symmetry.
- Guided user tests can instruct a human using the same label the agent uses, and instrumentation can mark *where* the human hesitated against the same action IDs.
- Feedback widgets attach to actions and surfaces, so a complaint arrives pre-classified ("on `shift.request_swap`") instead of as free text needing triage from zero.
- If the agent removes an action, the spec diff shows it and dependent tests fail loudly and legibly. No silent capability regressions.

### Bet 3 — Provenance-typed evidence

Every signal in the system carries a provenance type and an evidence strength. Agents are required to reason over evidence quality, and the platform enforces a hard rule:

> **A change may not be marked `validated` without at least one human-sourced signal.**

Synthetic and simulated evidence is admissible only for (a) generating hypotheses, (b) generating test cases, (c) pre-screening before spending human attention. Never for closure. Section 6.5 covers the failure mode this prevents.

---

## 4. What the platform is

### 4.1 Three layers

```
┌──────────────────────────────────────────────────────────────────────┐
│  LAYER 1 — BUILD                                                     │
│  Spec authoring (conversational + structured review)                 │
│  Agents: PM, Designer, Architect, Builder                            │
│  Output: spec → app (frontend + backend) → preview env               │
├──────────────────────────────────────────────────────────────────────┤
│  LAYER 2 — OBSERVE & ASK          ← the wedge, and the differentiator │
│  Action Surface · telemetry · feedback CTAs · message board ·        │
│  guided user tests · recruiting · consent · evidence ledger          │
├──────────────────────────────────────────────────────────────────────┤
│  LAYER 3 — DECIDE & ITERATE                                          │
│  Triage · synthesis · consensus · hypotheses · change sets ·         │
│  guardrails · canary · rollback · decision records · the ratchet     │
└──────────────────────────────────────────────────────────────────────┘
```

Most competitors are building Layer 1. Layer 2 is where the moat is. Layer 3 is where the money is, because Layer 3 is what customers actually want to buy ("tell me what to do next, and do it").

### 4.2 What a user actually does

1. Describes a product idea, or a problem, in conversation.
2. Reviews a **spec** — jobs, personas, flows, rules — in structured prose. Corrects it in words.
3. Gets a running app at a preview URL, with a seeded demo dataset, within minutes.
4. Watches agents run job tests and report what works.
5. Clicks "run a study" — agents write the script, recruit from a panel or from existing users, run it, and return findings mapped to spec elements and actions.
6. Reads the **evidence ledger** and the board's consensus snapshot.
7. Approves (or auto-approves, depending on risk class) a **change set** — a bundle of spec edits + regenerated code + new tests, with a stated hypothesis and expected metric movement.
8. Watches a canary roll out, with automatic rollback on guardrail breach.
9. Repeats. Every cycle is faster and better-informed than the last, because the evidence graph is accumulating.

The user's actual job title is "person with a problem," not "developer." The interaction is review-and-decide, not author-and-debug.

### 4.3 What this is not

- **Not a general-purpose IDE.** It is opinionated and narrow by design; narrowness is what makes instrumentation derivable.
- **Not a Figma replacement.** Design artefacts are a by-product of the spec, not the primary object.
- **Not an analytics tool.** Analytics infers; this asks. Both matter, but the platform's distinctive capacity is *asking humans directly, cheaply, in context*.
- **Not an autonomous product manager.** Agents propose; humans hold accountability. The platform is designed to make the human's judgement better-informed and faster, not to remove it.
- **Not (initially) a marketplace.** Template and participant marketplaces are Phase 4, not Phase 1.

---

## 5. The two primitives that make everything else cheap

### 5.1 The Product Spec IR

The single artefact agents are permitted to edit. Code is a build output. Sketch:

```yaml
spec: 1
product:
  name: Rota
  oneLiner: Shift swapping for small hospitality teams.
  problem: Managers rebuild the rota by phone every week; swaps happen in WhatsApp and get lost.

personas:
  - id: barista
    goals: [know my schedule, get rid of shifts I can't work, pick up extra hours]
    context: on phone, standing, often 1–2 minutes of attention
  - id: manager
    goals: [know coverage 48h out, avoid overtime, keep it fair]
    fears: [understaffed Friday, staff gaming the system]

jobs:                       # everything links back to these IDs
  - id: J1
    persona: barista
    statement: Give away a shift I can't work without phoning anyone
    success: swap.accepted_within(4h) && !manager_intervention
    priority: critical
  - id: J2
    persona: manager
    statement: See coverage gaps 48h out and fill them without overtime
    success: coverage.gap_hours == 0 && overtime.hours <= 4
    priority: critical

surfaces:
  - id: schedule.shift_detail
    jobs: [J1]
    shows: [shift.role, shift.time, shift.coworkers]

flows:
  - id: F1
    job: J1
    steps:
      - action: shift.request_swap
      - await: swap_request accepted OR 4h elapsed
      - action: swap.confirm
    happyPath: [shift.request_swap, candidates.select, swap_request.send, swap.accept]
    failureModes:
      - { when: candidates.length == 0, then: offer shift to open pool }

rules:
  - id: R1
    statement: Swaps under 8h never require manager approval; manager is notified and may veto within 2h.
    origin: decision_record/DR-014
  - id: R2
    statement: A shift may not be swapped within 24h of start without manager approval.
    origin: decision_record/DR-009

data:
  entities:
    - id: Shift
      fields: [id, role, start, end, assigneeId, status]
      rls: "assignee or manager_of(location)"
    - id: SwapRequest
      fields: [id, shiftId, requesterId, candidateIds, status, createdAt]

policies:
  - { id: P1, statement: Two users accepting the same shift resolves first-write-wins with a clear error to the loser. }
  - { id: P2, statement: No PII in telemetry events. }

metrics:
  - { id: M1, job: J1, definition: swap.accepted_within(4h) / swap.requested, target: 0.75, guardrail: false }
  - { id: M2, job: J2, definition: coverage.gap_hours, target: 0, guardrail: true }

guardrails:                 # auto-rollback triggers
  - { metric: M2, breach: "> 2h for 30m", action: rollback }
  - { metric: error.rate, breach: "> 1% for 5m", action: rollback }

integrations:
  - { id: I1, kind: email, purpose: notify candidates, gate: owner_approval }
```

Two things to notice. First, `rules` carry an `origin` pointing at a decision record — the spec has *provenance for its own contents*. Second, guardrails are declared next to the metrics they protect, so the safety envelope is inspectable by a non-technical owner in the same document as the product intent.

**Generated versus authored code.** A pure "spec compiles to everything" approach breaks down on real complexity. The pragmatic architecture is hybrid:

- **Generated, never hand-edited:** routing, data access + migrations, auth wiring, the Action Surface bindings, telemetry emission, test scaffolding, infra.
- **Declarative, spec-level:** forms, lists, detail views, permissions, state machines, notification rules, simple branching logic. This covers the large majority of CRUD-and-workflow apps.
- **Fenced custom modules:** escape hatch for genuinely bespoke logic. Custom code is still *wrapped* in the Action Surface, so it stays testable and measurable. Custom modules are the exception, and their existence is visible in the spec as a flagged risk (they get less automatic verification).

### 5.2 The Action Surface

Covered in §3 Bet 2. Operationally:

- Served at `/_cocreate/actions.json` in dev and preview; a signed, permission-filtered subset is exposed in production for the SDK.
- The testkit resolves actions by ID; the guided-test runner highlights the DOM node bound to an action; telemetry emits `action.<id>.{start,success,failure,abandon}`.
- Actions carry `preconditions`, which lets the platform *generate* negative tests (what should happen when the precondition fails?) — a category of bug that manual testing almost always misses.
- Actions carry `jobs`, which is what makes the evidence graph possible: a signal → action → job → spec element → change → metric, all traversable.

**Determinism support (a product feature, not an implementation detail):** seeded fixtures, injectable fake clock, third-party network virtualisation with recorded replays, and per-test data sandboxing. If tests are flaky, the loop is worthless, because nobody trusts a gate that cries wolf. Target flake rate < 1%, hard cap 2%, with quarantine + agent auto-repair.

---

## 6. The feedback loop as a control system

### 6.1 The canonical cycle

```
   ┌──────────────────────────────────────────────────────────────┐
   │  1. SIGNAL      something happened, or someone said something│
   │  2. INTERPRET   cluster, classify, link to job, weigh evidence│
   │  3. HYPOTHESISE state a falsifiable belief + expected effect  │
   │  4. CHANGE      spec edit → code → tests → preview → canary   │
   │  5. VERIFY      did the metric move? did the human confirm?    │
   │  6. DECIDE      keep / iterate / revert, and write it down    │
   └──────────────────────────────────────────────────────────────┘
             ▲                                            │
             └──────────── evidence ledger grows ─────────┘
```

Every loop instance is a first-class object with an ID, an owner, a hypothesis, a cost, and an outcome. Loops that produced nothing are still recorded — negative results are evidence, and a platform that hides them trains its agents to generate optimistic loops.

### 6.2 Three loop speeds

| Loop | Latency | Channels | Who runs it | Example |
|---|---|---|---|---|
| **Fast** (validation) | minutes–hours | machine tests, telemetry, guardrails | build + test agents | "the swap flow breaks when two people accept at once" |
| **Medium** (usability) | hours–days | in-app feedback CTAs, guided tests, session friction | research agent + humans | "users don't realise the candidate list scrolls" |
| **Slow** (discovery) | days–weeks | message board, consensus, interviews, cohort analysis | research + community agents + humans | "staff want autonomy; managers want control — we need a policy that serves both" |

The platform's job is to make the fast loop automatic, the medium loop cheap, and the slow loop *legible* (most teams never do the slow loop at all, because its outputs feel like opinions rather than data).

### 6.3 The evidence ledger

The central data structure and the durable asset. Every belief the platform holds about an app is a row:

| Field | Purpose |
|---|---|
| `claim` | the belief, stated falsifiably |
| `links` | job / action / spec element / metric it concerns |
| `provenance` | `human_observed` \| `human_reported` \| `machine_inferred` \| `synthetic_simulated` |
| `method` | telemetry, in-app CTA, board thread, unmoderated study, AI interview, E2E test, chaos test |
| `n` | sample size, with an explicit CI or "qualitative, n<10" label |
| `strength` | 0–5 composite of provenance × method rigour × n × recency |
| `status` | proposed / supported / contested / refuted / superseded |
| `cost` | tokens + incentives + human minutes to obtain |
| `decidedBy` | which loop, which decision record, which human approved |

Rules enforced by the platform:

- A `claim` may not move the roadmap unless `strength ≥ 3` **and** at least one linked signal is human-sourced.
- Claims never get deleted, only `superseded`, with a pointer to what replaced them. This makes the platform's reasoning auditable and makes "why is it like this?" answerable a year later.
- Conflicting claims are a first-class state (`contested`), not an error. The UI shows the shape of disagreement.

### 6.4 What makes the loop *work* rather than just *run*

Four disciplines, each of which most teams skip:

1. **Falsifiability required.** A hypothesis must state the observation that would prove it wrong. Agents are structurally biased toward confirming their own prior work; requiring a disconfirming observation is the cheapest possible correction.
2. **Pre-registration.** The hypothesis, metric, and expected effect size are recorded *before* the change ships. Post-hoc metric selection is the most common way analytics lies.
3. **Independent verification.** Where feasible, the agent that interprets the evidence is not the agent that proposed the change. Same rubric, no shared conversation context. Cheap, and it catches a large class of motivated reasoning.
4. **Negative results retained.** A change that didn't move the metric is recorded with its cost. Without this, the same failed idea gets re-proposed forever, and the platform's priors stay garbage.

### 6.5 The existential failure mode: simulation collapse

The single largest risk to this business, and the one that deserves a hard architectural answer.

When agents can (a) generate synthetic user reactions, (b) author the tests, (c) interpret the results, and (d) write the summary, there is a closed loop containing no reality. Every turn of the crank produces more confident, more internally-consistent, more detailed output that corresponds to nothing. This is not hypothetical — LLM-based synthetic user research is already marketed and already criticised on exactly these grounds ([The Synthetic Persona Fallacy](https://interactions.acm.org/blog/view/the-synthetic-persona-fallacy-how-ai-generated-research-undermines-ux-research); [Whose Personae? Synthetic Persona Experiments in LLM Research](https://ojs.aaai.org/index.php/AIES/article/view/36553)).

Structural countermeasures, all mandatory:

| Measure | Mechanism |
|---|---|
| Provenance typing | synthetic signals are a distinct type; the ledger will not accept them as validation |
| Human-evidence requirement | `validated` status requires ≥1 human-sourced signal, enforced in the schema, not by policy |
| Synthetic use limits | permitted only for hypothesis generation, test generation, and pre-screening |
| Adversarial red team agent | explicit objective: falsify the current belief; must propose the *cheapest human test that would break it* |
| Blinded verification | interpreter agent has no access to the proposer's reasoning |
| Ratio monitoring | `synthetic:human` signal ratio tracked per app; the UI warns the owner when it climbs |
| Hold-out human evidence | a standing panel of real users never used for iteration, only for periodic independent checks |
| Visible honesty in the UI | synthetic-derived claims are visually marked everywhere, never blended into a single confidence score |

The commercial framing of this is important: **honesty about evidence quality is the product's core value proposition, not a compliance overhead.** A platform that inflates its evidence is worth nothing to a founder about to bet a year on it.

### 6.6 Anti-Goodhart

Agents optimising a metric they can partially influence is a structural hazard (they control instrumentation *and* the thing being instrumented). Mitigations:

- **Metric definitions live in the spec**, are versioned, and changes to a metric definition require human approval and invalidate prior comparisons (flagged in the ledger, not silently rebased).
- **Guardrail metrics are declared by humans** and cannot be edited by agents. Agent edits to guardrail config are a hard block (§12).
- **Synthetic traffic is excluded** from product metrics by construction, tagged at ingest.
- **Counter-metrics**: every primary metric has a paired counter-metric that would be expected to move the *wrong* way if the primary were gamed (e.g., swap completion rate paired with manager override rate and support contacts). A win on the primary with a bad counter-metric is flagged, not celebrated.
- **North-star is not a product metric.** The platform optimises *validated learnings*, not any single app's numbers. This aligns agents with finding truth rather than with making a dashboard green.

---

## 7. The five signal channels

### 7.1 Telemetry / usage data

Auto-derived from the spec: events at the action layer, funnels at the job layer, definitions from `success` expressions.

```yaml
# derived, not authored
events:
  - action.shift.request_swap.start      # {shiftId, candidateCount, autofilled}
  - action.shift.request_swap.abandon    # {step, elapsedMs, reason?}
  - action.shift.request_swap.success    # {candidateCount, elapsedMs}
  - job.J1.outcome                       # {result: accepted|expired|withdrawn, hoursToAccept}
funnels:
  - job: J1
    steps: [surface.shift_detail, action.shift.request_swap.start, candidates.select, swap_request.send, swap.accept]
```

Design commitments:

- **Action-layer, not click-layer.** Clicks are noise; actions are meaning. This is only possible because of the Action Surface.
- **No PII by construction.** Event schemas are validated at ingest; a payload that doesn't match its schema is rejected, not coerced.
- **Small-n suppression.** Funnels and segments with n < 5 are suppressed by default (privacy, and honesty — five users is not a conversion rate).
- **Explicit "why" prompts, used sparingly.** A tiny, budgeted fraction of sessions get a single contextual question ("you left at this step — what were you trying to do?"). High signal, but only if rationed (§15, research fatigue).
- **Consent state is a first-class field** on every event; non-consented events are usable only in aggregate and never joined to identity.

Telemetry's role in the loop is *detection*: it finds where attention is warranted. It is very good at "what" and structurally incapable of "why." The other four channels exist to supply "why."

### 7.2 In-app feedback CTAs

Not a generic "🐞 Feedback" button. Context-bound, spec-aware, budgeted.

| Trigger type | Example | Why it works |
|---|---|---|
| **Friction-triggered** | rage clicks, dead clicks, 3× backtrack, error recovery on `shift.request_swap` | arrives with the moment of failure still in working memory |
| **Outcome-triggered** | after `job.J1` succeeds: "did that work out?" | validates the *job*, not the UI; catches silent failures |
| **Threshold/exit** | abandonment with a reason chip set | cheap, closes the "why did you leave" gap |
| **Elicited, budgeted** | 1-in-200 sessions: one open question | keeps an open-text firehose for discovery |

Mechanics: the widget is a web component bound to the Action Surface, so it always knows `{surface, action, job, sessionContext, consentState}`. Submission creates a typed signal, pre-linked to spec elements, deduplicated by embedding, and routed into triage. Free text is optional; structured chips plus an optional sentence outperform a blank textarea by a wide margin in practice.

Guardrails: hard cap on prompts per user per month (start at 2), suppression for users who have dismissed twice, never during a critical flow, never a modal that blocks a job, and a global off-switch the owner controls. **Feedback prompts are a tax on the user; spend the budget deliberately.**

### 7.3 The message board

Where users discuss, complain, propose, and — the ambitious part — reach consensus. Deliberately not a feature-request upvote board, for reasons in §8.

Entities:

- **Post** — typed as `problem` | `idea` | `question` | `praise` | `bug`. The type determines routing (bugs → ratchet, ideas → deliberation, problems → research candidates).
- **Cluster** — agent-grouped posts describing the same underlying problem, with a canonical statement and back-references to every constituent post. Users can contest a cluster ("that's not what I meant"), and contestation is a first-class signal.
- **Rationale** — every vote/support action requires a one-line reason. This is the difference between "50 people clicked upvote" (uninformative, brigadeable) and "50 people want X because of Y" (analysable, clusterable). Rationales are the raw material for synthesis.
- **Deliberation thread** — structured argument: claims, supporting evidence, counter-arguments, trade-offs. Agents summarise but must cite; synthesis is always traceable to source posts.
- **Proposal** — a candidate change to the spec, with stated costs and trade-offs, generated from clustered deliberation. Proposals can originate from users or agents, but only agents can produce a proposal that is *implementable* (spec-level), which keeps the loop tight.
- **Decision record** — the output. Immutable. Contains: what was decided, on what evidence, what was rejected, **who dissented and why**, and what would cause a revisit.
- **Consensus snapshot** — a living view over the whole board: agreed problems ranked, contested points with the shape of the disagreement, decided-and-shipped, decided-and-rejected, open questions with the evidence that would resolve them.

Board-feel commitments: it must feel like a place where being specific is rewarded. Votes without reasoning are de-emphasised visually. Agents participate openly and are labelled as agents. Slow, considered dissent is protected — including a deliberate "minority report" affordance so that a lone well-argued objection isn't buried by a popular one.

### 7.4 Guided user tests

The channel that most differentiates this from an analytics product, and the most operationally demanding.

**Study object:**

```yaml
study:
  id: S-041
  researchQuestion: Why do baristas abandon the swap flow at candidate selection?
  job: J1
  hypothesis: Candidates are ordered by seniority, so users don't recognise anyone and assume the list is complete.
  method: unmoderated_task_thinkaloud
  participants: { segment: barista, n: 5, source: existing_users, incentive: "£15 gift card" }
  tasks:
    - instruction: You can't work this Friday. Try to get someone to cover your shift.
      success: swap.accept reached OR explicit, reasoned abandonment
      capture: [screen, audio, transcript]
  questions:
    - After picking someone, what did you expect to happen next?
  consent: { version: c3, recording: true, jurisdiction_checked: true }
  preRegistered: { primaryObservation: "hesitation > 10s at candidates.select", decidedBefore: true }
```

**Method ladder, cheapest first.** The research agent must justify escalating:

| Rung | Method | Cost | Answers |
|---|---|---|---|
| 1 | Five-second / first-click test | ~£1/participant | is the intent discoverable at all? |
| 2 | Unmoderated task, text think-aloud | low | where do people get stuck? |
| 3 | Unmoderated task + screen & audio | medium | why do they get stuck? |
| 4 | AI-moderated interview (adaptive follow-ups) | medium | what's the underlying model? |
| 5 | Human-moderated session (platform-assisted) | high | ambiguous, high-stakes, or emotionally loaded |
| 6 | Longitudinal diary / field study | very high | behaviour over time in context |

**Recruitment modes:**

- **Existing users via in-app intercept** — highest signal, lowest cost, fastest. Bias: skewed to engaged users. Requires an intercept budget and consent handling.
- **Panel providers** (integrated, pass-through with margin) — fast, demographic targeting, but participants are professional testers; good for usability, weak for motivation.
- **The app's own board community** — high motivation, subject to self-selection and power-user bias; excellent for discovery, poor for representative measurement.
- **Own participant network** (Phase 4) — the long-term asset; also the highest abuse risk (see §15).

**Agent analysis pipeline:** transcribe → segment by task phase against action IDs → detect friction markers (long pauses, backtracking, dead clicks, error loops, self-correction, abandonment) → extract quotes → cluster into findings → map each finding to a spec element and an action → score severity (`frequency × impact × confidence`) → propose change sets.

**Honesty constraints, enforced in the UI:**

- Any percentage from n < 10 is displayed as counts, never percentages.
- Every finding displays n, method, and a confidence band.
- "One user said X" is labelled a hypothesis, not a finding. Requiring two independent signals to promote a hypothesis to a finding is a crude but effective guard against the most common research error in small-N work.
- Disconfirming clips are surfaced alongside confirming ones. The analysis view shows counter-evidence by default, not on request.

### 7.5 Automated tests and synthetic agents

**Generated test layers:**

| Layer | Source | Purpose |
|---|---|---|
| Spec/API contract | `data` + `rules` | backend correctness without UI |
| Authz matrix | `personas × entities × rules` | the category of bug manual QA never finds |
| Job E2E | `jobs` + `flows` | "can a barista actually swap a shift?" |
| Negative / precondition | `actions.preconditions` | what happens when it shouldn't work |
| Accessibility | Action Surface ↔ ARIA | conformance + semantic integrity |
| Performance budget | critical flows | p95 per job, bundle budget |
| Visual regression | key surfaces | only where visuals encode meaning |
| Chaos / concurrency | `policies` | double-accept, partial failure, offline |

**Synthetic user agents** are used for: exploratory fuzzing of flows, generating candidate friction hypotheses, and pre-screening a study design before spending human attention. They are *labelled* `synthetic_simulated` and can never satisfy the human-evidence requirement. Their honest use is as a *test-case generator and hypothesis mill*, not as a source of truth.

**The Ratchet — the single most valuable mechanism in the product:**

> Every defect reported by a human, and every production incident, becomes a permanent automated test before the fix is considered done.

Consequences: quality is monotonically non-decreasing with respect to discovered defects; the test suite becomes a precise encoding of everything the product has ever got wrong; and the marginal cost of reliability falls as the app ages, which inverts the usual trajectory of software. Make it non-negotiable — a fix without a new test is rejected by the platform, mechanically.

---

## 8. Consensus: the hardest and most differentiated part

"Reaching consensus between users about what the software should do" is where most community platforms fail, and where the interesting design work is. Agents make it newly *possible* at scale, and introduce a genuinely dangerous failure mode (manufactured agreement). Some strong opinions:

### 8.1 Seek consensus on problems, constraints, and priorities — never on solutions

Users are reliable, motivated reporters of their own problems. They are unreliable prescribers of solutions: they cannot see implementation cost, downstream effects, or other users' constraints, and they anchor on the first workaround they found.

So the board's deliberation object is the **problem statement**, not the feature request. Feature requests are treated as *evidence about problems*. The agent's job is to infer the problem, state it canonically, and get users to agree that the *statement* is accurate. Agreement on a well-stated problem is durable and generalisable; agreement on a feature is brittle and often regretted.

Practical consequence: a post titled "add a calendar export button" is triaged into the problem cluster "I can't see my shifts where I already plan my week." The consensus question becomes "is that the real problem?" — which is answerable — rather than "should we build calendar export?" — which is not.

### 8.2 Segment consensus, not global consensus

In any multi-sided product, a global majority can systematically harm a minority segment whose cooperation the product depends on. In Rota, baristas outnumber managers 12:1. A global vote on approval requirements produces a policy managers will quietly route around — and an app that fails at its job (J2) because the people responsible for coverage stopped trusting it.

**Rule: consensus is computed per persona segment, with per-segment quorums, and cross-segment conflict is surfaced explicitly rather than averaged away.** The agent's first obligation when segments disagree is to search for an **integrative solution** that satisfies both (expand the pie) before proposing a trade-off. In Rota: auto-approve small swaps (barista autonomy) *plus* notify the manager with a 2-hour veto (manager control). Both jobs improve. Most apparent zero-sum product conflicts have an integrative option that nobody proposed because nobody had both sides' rationale in one view — which is exactly what agents can now provide.

### 8.3 The mechanism ladder

| Scale | Mechanisms | Notes |
|---|---|---|
| N < 20 | facilitated deliberation, sortition panels, agent synthesis of the argument map | deliberative quality beats statistical representativeness at this size |
| 20–500 | structured proposals, rationale-required support, agent clustering + dedup, periodic open deliberation | the sweet spot; most early customers live here |
| 500–10k | representative panels, stratified sampling, referenda on direction only, standing user council | direct participation stops scaling; move to representation |
| > 10k | statistical sampling + council + cohort-level preference modelling | treat preference measurement as survey science, not voting |

Deliberately avoided: quadratic voting as a default (favours the wealthy or the intensely motivated in ways that are hard to justify to a small-business owner), and any mechanism whose output is an unexplained number.

### 8.4 Anti-pathology rules

- **Rationale required to count.** A support action without a reason is recorded but carries less weight. This alone defuses most brigading.
- **Brigade detection.** Coordinated-vote heuristics, account-age/tenure weighting, rate limits, and a visible "this thread shows signs of coordinated activity" flag.
- **Dissent preservation.** Every decision record lists rejected alternatives and named dissent. Agents must not synthesise away disagreement to produce a tidier summary.
- **Representativeness check.** Before a decision, the platform reports who is *absent* from the conversation (segments below quorum, users who churned, users who never post). Silence is data.
- **Misrepresentation flag.** Any participant can flag "the agent misrepresented my point," with the source post inline. These flags are a first-class quality metric for the synthesis agents, and a training signal.
- **Reversibility weighting.** Decisions that are hard to reverse need higher consensus and human sign-off. Cheap-to-reverse decisions should be made fast and tested, not debated.
- **Time-boxing.** Deliberation has a deadline and a decision rule stated up front. Open-ended consensus never converges and burns community goodwill.

### 8.5 What agents are actually good for here

- Reading 400 posts and producing 6 problem statements with citations (nobody does this manually; it is the reason boards rot).
- Noticing that the same complaint appears in feedback CTAs, board posts, *and* a friction marker in session data — triangulation that no human has time to perform.
- Writing the neutral framing of a contested question (a genuinely hard, valuable writing task).
- Producing the integrative option by combining both segments' rationales.
- Detecting that a decision's stated rationale doesn't match the evidence that actually drove it.
- Maintaining the decision archive so the same debate doesn't recur every 6 months.

What agents must *not* do: declare consensus that isn't there; rank problems by vote count alone; average away segment conflict; or act as the final decision authority.

---

## 9. The agent organisation

Agents are organised as a small product team with explicit, separable mandates — separation is what makes independent verification possible.

| Agent | Mandate | Key outputs | Must not |
|---|---|---|---|
| **PM agent** | own the spec's product intent | jobs, personas, priorities, problem statements | invent user evidence |
| **Research agent** | design and run studies | study scripts, recruitment briefs, findings | run a study without consent + owner approval |
| **Analyst agent** | interpret signals | clusters, severity scores, evidence ledger rows | interpret evidence for a change it proposed |
| **Red team agent** | falsify current beliefs | counter-evidence, cheapest disconfirming test | propose features; it only attacks |
| **Architect agent** | translate spec → technical design | data model, authz, contracts, migrations | change product intent |
| **Builder agent** | generate app + tests | preview envs, change sets, migrations | deploy to prod directly; touch guardrails |
| **Test agent** | generate + repair tests | job E2E, authz matrix, a11y, ratchet tests | delete a ratchet test |
| **Triage agent** | route inbound signals | typed, deduped, job-linked signals | close a signal without a resolution |
| **Community agent** | facilitate the board | syntheses, argument maps, decision records | vote, or claim consensus without quorum |
| **Governance agent** | enforce policy | gate decisions, audit log, budget checks | be modifiable by other agents |
| **Scribe agent** | maintain memory | decision records, changelog, glossary | rewrite history (append-only) |

**Interaction protocol.** Agents communicate through the spec, the evidence ledger, and the board — never through unlogged conversation. Every agent action, prompt, cited evidence, and decision lands in an append-only audit log. This is essential for three reasons: debugging (why did it do that?), trust (show the owner the reasoning), and safety (the log is the thing you inspect after an incident).

**Escalation.** Agents escalate to the human owner when: evidence is contested or weak; the change is in a gated risk class; cost budget would be exceeded; segments conflict with no integrative option; or a guardrail is at risk. Escalation is a *product surface* — it should be a well-written brief with a recommendation, not a stack trace.

**Cost model.** Each agent invocation is budgeted. A loop has a token budget and a stop rule (stop when marginal expected evidence gain < marginal cost). Without this, an enthusiastic agent can burn four figures chasing a two-user anomaly.

---

## 10. Architecture

### 10.1 The compile pipeline

```
spec (IR)  ──▶ plan ──▶ generate ──▶ provision ──▶ instrument ──▶ test ──▶ preview
   ▲                                                                        │
   │                                                                        ▼
   └──────────── change set ◀── decision ◀── evidence ◀── canary ◀── promote ─┘
```

| Stage | Artifact | Gate |
|---|---|---|
| Validate | typed spec, diff vs previous | schema + policy lint (e.g., every job has a success criterion) |
| Plan | change plan, risk class, migration plan | risk classification determines human approval |
| Generate | code + Action Surface + tests + telemetry schema | compile, typecheck, contract tests |
| Provision | preview env, seeded DB, secrets, integrations stubbed | preview URL live |
| Test | job E2E, authz matrix, a11y, perf, visual | all green, flake rate under cap |
| Research (opt) | study results | owner approves recruitment + spend |
| Canary | % rollout behind flag | guardrail metrics hold for soak window |
| Promote | immutable prod artifact | same artifact as preview — never rebuild |
| Rollback | re-point to prior artifact | automatic on guardrail breach |

Non-negotiables: **immutable artifacts** (preview and prod are byte-identical, so "works in preview, breaks in prod" is eliminated as a class), **flag-gated rollout**, **automatic rollback**, **append-only audit**.

### 10.2 Runtime and hosting topologies

Three topologies, one adapter contract.

**Topology A — Platform-hosted (default).**
Per-app namespace: edge CDN, app server (container or edge function), managed Postgres with row-level tenancy, object storage, job queue, cron, secrets vault, email/SMS adapters behind approval gates. Optimised for the 90% case: CRUD + workflow + notifications. This is what makes "works in 5 minutes" possible.

**Topology B — BYO PaaS (adapters).**
```ts
interface DeployTarget {
  provision(appId: string): Promise<EnvRef>
  deploy(env: EnvRef, artifact: Artifact): Promise<DeploymentRef>
  previews(): Promise<PreviewCapability>   // must support ephemeral per-iteration envs
  env(env: EnvRef): Promise<EnvVars>
  logs(ref: DeploymentRef): AsyncIterable<LogLine>
  rollback(ref: DeploymentRef): Promise<void>
  secrets(): SecretStore
  db(): DbControl | null
}
```
Candidate targets: Vercel, Netlify, Cloudflare, Fly.io, Render, Railway, AWS, plus managed data (Supabase, Neon, Turso). **Preview-environment support is a hard requirement of the contract** — a target that can't do ephemeral previews can't participate in the loop, because the loop depends on testing changes before they reach users.

**Topology C — Customer cloud (enterprise).**
Emit Terraform/Pulumi from the spec; the control plane holds metadata only. This is the answer to data residency, procurement, and "our security team says no."

**Consequence for the SDKs:** because apps must run anywhere, the SDKs are deliberately thin and framework-agnostic:

- `@cocreate/client` — a web component (feedback CTA, intercept runner, guided-test runner) + a small JS SDK (event batching, offline queue, consent state, flag evaluation, exposure reporting).
- `@cocreate/server` — middleware for common runtimes (Node, Bun, Deno, Workers) handling ingest auth, schema validation, PII rejection, and flag evaluation server-side.
- `@cocreate/testkit` — resolve actions by ID, seed fixtures, control the clock, virtualise third parties, assert on jobs.
- `@cocreate/cli` — `spec validate`, `spec diff`, `test`, `preview`, `promote`, `rollback`, `export`.

Deliberately *not* React-specific, and deliberately usable by a hand-written app that was never generated by the platform. That last point is the wedge (§13).

### 10.3 Signal store

One table, many shapes.

```
Signal {
  id, appId, createdAt,
  provenance,            // human_observed | human_reported | machine_inferred | synthetic_simulated
  channel,               // telemetry | cta | board | study | test | incident
  payload,               // schema-validated per channel
  links: { jobId?, actionId?, specPath?, metricId?, studyId?, postId? },
  subject,               // pseudonymous participant ref (or null)
  consentState, jurisdiction,
  clusterId?, severity?, status
}
```

Plus: vector index for clustering and dedup; a `Claim` table (the ledger); a `Loop` table (each cycle with hypothesis, cost, outcome); a `DecisionRecord` table (append-only); an `Action` audit log.

The **evidence graph** — signals → actions → jobs → spec elements → changes → metrics — is the queryable asset. "Which changes to the swap flow had human evidence, and what happened to M1?" must be a one-query question. That query being cheap is the whole point of the architecture.

### 10.4 Preview environments and data

Every iteration gets its own URL and its own data sandbox. Data strategy:

- **Synthetic fixtures** by default, generated from the spec's data model with realistic distributions and edge cases (empty states, long strings, timezone boundaries, unicode names, 400-row lists).
- **Recorded real data** only with consent, anonymised, in a separate namespace.
- **Fake clock and time travel** for anything time-dependent (rotas, bookings, trials, expiry) — this alone removes a huge class of untestable behaviour.
- **Third-party virtualisation**: recorded replays of Stripe/Twilio/SendGrid etc., with a fail-injection mode. Tests never hit real third parties.

---

## 11. Worked example: the café rota app

Following one app through the loop, to make the abstraction concrete.

**Context.** A 4-site café group, 60 staff. The owner has no engineering team and describes the problem in chat: *"Managers rebuild the rota by phone every week. Swaps happen in WhatsApp and get lost. I want an app."*

**Day 0 — spec, not code.** The PM agent asks questions and drafts a spec. Jobs surface: J1 (barista: give away a shift without phoning anyone), J2 (manager: see coverage gaps 48h out, avoid overtime). Personas conflict on control. The owner reviews the spec — 2 pages of structured prose — and corrects one thing: "agency staff can't use the app, only permanent staff." That correction propagates into authz, tests, telemetry, and the board's audience. **Cost of the correction: one sentence.**

**Day 0, +20 min — running app.** Preview URL, seeded with a fake rota, 4 sites, 60 fake staff. The Action Surface is published; the job tests for J1 and J2 are generated and green.

**Day 0, +1h — fast loop already firing.** The test agent's precondition tests catch that `shift.request_swap` fails ungracefully when the shift starts in 23h (rule R2 should require approval). The generated negative test found a rule the owner had stated but the builder had implemented only in the UI, not the backend. **Nobody wrote this test. It exists because the spec declared the rule and the action declared its preconditions.**

**Day 1 — first real users.** 8 staff on a preview link. Telemetry (action-layer) shows: 62% of swap attempts reach `candidates.select`; 24% abandon there. Friction markers cluster on `candidates.select`.

**Day 1 — research agent proposes a study, not a fix.** Research question: *why* do they abandon at candidate selection? Hypothesis (from a synthetic pre-screen, labelled as such): the list is ordered by seniority, so users don't recognise anyone and assume it's the whole list. Method: unmoderated, screen + think-aloud, n=5 baristas from existing users, £15 incentive, one task. The owner approves the spend (£75 + tokens) with one click.

**Day 3 — findings.** 4 of 5 participants never scrolled. Two said out loud "there's nobody on here I know." One explicitly looked for a search box. A fifth participant scrolled immediately and completed in 40 seconds — the disconfirming case is shown *first* in the finding, because it reframes the problem from "the list is too short" to "the list gives no signal about who's on it."

**Day 3 — in-app CTAs triangulate.** An outcome-triggered CTA after a *successful* swap returns 6 responses, 3 of which say a variant of "I got lucky, I asked in WhatsApp anyway." This reframes the metric: M1 measures in-app completion, but the real job is being solved *outside* the app. **A telemetry-only view would have reported success.**

**Day 4 — the board does the slow loop.** 12 posts cluster into 2 problem statements:
- *P-1 (broad agreement, both segments):* "I can't tell who is eligible or available to take my shift."
- *P-2 (contested):* "Approval takes too long" (baristas) vs "I need to approve swaps, otherwise I find out Friday" (managers). Per-segment quorum reached; segments disagree.

**Day 4 — the integrative option.** The community agent, holding both segments' rationales in one view, proposes: auto-approve swaps more than 24h out with no overtime cost; notify the manager with a 2-hour veto; require approval below 24h or above the overtime threshold. Barista autonomy preserved, manager control preserved, both J1 and J2 expected to improve. Neither segment had proposed this, because neither could see the other's rationale.

**Day 5 — change set CS-118.** Spec edits (R1 amended, `candidates.select` gains availability + shared-shift signals, search added), with pre-registered hypothesis and expected effect: M1 +15pp, manager override rate < 10%, M2 (coverage gap hours) non-increasing.

**Day 6 — canary.** 25% of sites, 48-hour soak, guardrails armed (`error.rate > 1%` → rollback; `M2 > 2h` → rollback). Test suite: 3 new job tests, 2 new precondition tests, 1 authz test (agency staff must *not* see swap), a11y assertions on the new list.

**Day 6 — first incident, and the ratchet.** A concurrency bug: two staff accept the same shift within the same second; the loser sees a generic 500. Human-reported via a friction CTA. Fix ships *with* a new permanent concurrency test (`swap.double_accept`). That test now exists forever. It cost one incident; it will never cost another.

**Day 8 — decision record DR-014.** Shipped, M1 61% → 84%, manager override 7%, M2 non-increasing (i.e., the guardrail held). Recorded: what was rejected (mandatory approval for all swaps), who dissented (2 managers, with their reasoning), and what would cause a revisit (override rate > 20%, or any overtime increase for 2 consecutive weeks). The dissent is preserved, not smoothed over.

**Week 3 — the slow loop pays off.** The ledger now holds 41 claims, 12 with human evidence. The owner asks "what should I do next?" and gets a ranked list, each row citing its evidence and its strength — plus one row in `contested` state, clearly flagged as an open question with the specific study that would resolve it.

**Month 2 — compounding.** 14 iterations, 9 validated, 3 reverted, 2 inconclusive (retained as negative results). Every human-reported defect has become a permanent test. The owner's judgement has improved because the ledger remembers what was tried and what happened — which is precisely the thing a team of one cannot otherwise do.

---

## 12. Governance, safety, and ethics

Agents that change live software used by real people need a real safety envelope. This should be a *selling point* to small businesses, who are otherwise reasonably terrified of autonomous changes.

### 12.1 Risk classes and gates

| Class | Examples | Gate |
|---|---|---|
| **R0 — internal** | tests, telemetry, instrumentation, refactors | auto |
| **R1 — reversible UI** | copy, layout, ordering, empty states | auto behind flag, canary, auto-rollback |
| **R2 — behavioural** | new rules, flow changes, notification logic | review + explicit owner approval |
| **R3 — sensitive** | schema migrations, authz/permission changes, data deletion, payments, outbound messaging to users | owner approval + dual control + staged migration + rollback rehearsed |
| **R4 — irreversible / external** | legal terms, deleting user data, contacting users without consent record, changing prices retroactively | human only; agents are blocked from executing |

### 12.2 Hard blocks (agents may never)

- Modify guardrail definitions, metric definitions, consent config, or audit logging.
- Author their own evaluation criteria or review their own gate.
- Remove a ratchet test.
- Contact real users without a verified consent record and an approved study.
- Exceed a declared budget (tokens, incentives, spend).
- Mark a change `validated` on synthetic evidence.
- Delete or rewrite evidence ledger rows, or edit a decision record.
- Deploy to prod outside the flag/canary path.

### 12.3 Blast radius and rollback

Canary percentage, soak window, and guardrail thresholds all declared per-app in the spec. Automatic rollback on breach. Rollback rehearsed for R3 changes. Kill switch that halts all agent activity on an app, reachable from the app's home screen by the owner.

### 12.4 Research ethics

- **Informed consent** per study, versioned, with recording consent handled per jurisdiction (including two-party-consent regimes).
- **Fair incentives** — paid per completed task, not per signup, and never exploitative ("test this for exposure" is not acceptable).
- **Right to withdraw** and delete a participant's contribution, including derived data where feasible.
- **Vulnerable participants** excluded by default; explicit review required to include them.
- **AI disclosure** — participants are told when an agent moderates or analyses. Non-negotiable.
- **Anti-spam** — hard caps per user per period, opt-out honoured globally and instantly, sender reputation protected. A platform that spams its customers' users is finished.
- **IRB-adjacent posture** for anything resembling human-subjects research at scale; a standing ethics reviewer once volume justifies it.

### 12.5 Legal and compliance surface

GDPR/UK GDPR (lawful basis, DPIA for telemetry + research, data residency via Topology C), CCPA/CPRA, PECR/ePrivacy for cookies and tracking, the **European Accessibility Act** (in application since 28 June 2025 for many digital services — a real compliance driver for anyone selling into the EU), WCAG 2.2 AA as the default target, plus DPA/sub-processor hygiene for the research panel. Accessibility being a *default build output* rather than a retrofit is a genuine, quotable business benefit.

### 12.6 Liability posture

Clear published limits: who owns the app and its data (the customer), who owns the platform metadata (us), what the platform guarantees (the loop, the audit trail, the rollback), and what it does not (business outcomes). Errors and omissions insurance once agents are touching revenue-affecting flows. The audit log is the primary defence and must be immutable and exportable.

---

## 13. Business: wedge, ICP, pricing, moat

### 13.1 The wedge — sell the loop before the builder

Competing with well-funded app builders on generation quality is a poor first move: it is a commoditising capability, the buyer is fickle, and CAC is brutal. The loop is a better wedge because:

1. It works on apps built by *any* tool — including Lovable, Bolt, v0, or hand-written code. That multiplies the addressable market by ~100x on day one and avoids a head-on fight.
2. It has a measurable ROI the buyer already believes in ("we know we should talk to users; we don't have time").
3. It installs the SDK and the Action Surface — the durable moat — inside apps we did not build.
4. It creates *pull* for the builder: once the platform has the spec, the evidence ledger, and the test suite, "let us make the change" is the obvious next sentence. Expansion is a consequence of the wedge, not a separate sale.

So: **Phase 1 sells "instrument anything, learn fast." Phase 3 sells "and we'll build the changes."**

### 13.2 ICP ladder

| Priority | Segment | Why they buy | Why they're a good first customer |
|---|---|---|---|
| 1 | **Product studios & digital agencies** (5–50 people) | they bill for discovery + build; the loop lets them do more with fewer people and *prove* value to clients | already run feedback loops manually; have many apps; care about margin; will co-design; great distribution to their clients |
| 2 | **Solo / non-technical founders pre-PMF** | they can build but not validate; terrified of building the wrong thing | high volume, low ACV, viral, but churny — good for learning, weaker for revenue |
| 3 | **Internal tools / ops teams in SMB–midmarket** | captive users, fast feedback, real cost of failure, budget exists | excellent fit for segment-consensus features; low competitive noise |
| 4 | **PM / research teams at scale-ups** | research throughput is the bottleneck; they already pay for Maze/UserTesting/Dovetail | biggest budgets, longest sales cycles, most demanding on compliance |

Start with 1 and 3. Agencies fund the company; internal-tools teams prove the enterprise path.

### 13.3 Pricing model

| Component | Structure | Rationale |
|---|---|---|
| Platform fee | per workspace / month, by app count | predictable, aligns with "many apps" agency shape |
| Iteration credits | metered agent compute (loops, tests, studies) | token cost is the real COGS; must be metered or margins die |
| Participant pass-through | panel cost + 20–30% margin | standard; also funds the future owned panel |
| Research seats | per-seat for human-moderated sessions and review surfaces | captures value from teams, not usage |

Avoid pure outcome pricing early (unmeasurable, gameable, hard to forecast). Consider later: "per validated learning" as a premium tier, once attribution is trustworthy.

Explicitly guard against token blowup with per-loop budgets, stop rules, model tiering (cheap models for triage and clustering; strong models for synthesis and spec editing), caching of spec→artifact derivations, and a visible spend meter with hard caps.

### 13.4 The moat, ranked by durability

1. **The cross-app causal dataset.** Which interventions actually moved which metrics, across N apps, with provenance. This is the compounding asset — it turns into better default hypotheses, benchmarking products, and eventually priors that no competitor can buy. Nothing else here is defensible long-term.
2. **The evidence graph per app.** Switching costs rise with every iteration because the ledger *is* the product's memory.
3. **The Action Surface as a de facto standard.** If third-party tools target `actions.json`, the platform becomes infrastructure. Open-source the SDK to accelerate this — the standard is worth more than the SDK revenue.
4. **The participant network.** Real, verified, consented, well-treated humans who will test things. Hard to build, hard to fake, valuable.
5. **Workflow embedding.** Decision records, board history, and the ratchet become the team's operating memory.

Note what is *not* on this list: code generation quality. Assume it commoditises completely, and architect so that it doesn't matter. Be model-agnostic (BYO model key, route to the cheapest capable model) so a foundation-model vendor's price move isn't fatal.

### 13.5 GTM motions

- **Agency partner programme** — co-selling, white-labelled research reports, revenue share. Agencies bring apps; we bring the loop.
- **Open-source the SDK + Action Surface spec** — distribution and standard-setting; monetise the control plane, the agents, and the panel.
- **Vertical templates** — rota/scheduling, clinic bookings, field service, member/community management, internal approvals. Each template ships with jobs, metrics, studies, and board structure pre-built. Templates are the answer to "blank canvas" cold-start.
- **Publish the benchmarks.** An annual report on "what actually moved the needle across N thousand iterations" is genuinely novel content, gets cited, and demonstrates the moat. This is the marketing flywheel for a business whose value *is* accumulated evidence.
- **Free research-ethics and consent tooling** — good citizenship, and it makes the platform the safe default.

---

## 14. Competitive landscape and positioning

The landscape as of early 2026 is crowded on both sides but thin in the middle. (This category moves fast; re-verify before betting.)

| Category | Examples | What they do well | What they don't do |
|---|---|---|---|
| Agentic app builders | Lovable, v0, Bolt, Replit Agent, Base44, Firebase Studio | fast, polished generation; great first 20 minutes | no loop; verification is "you look at it"; no instrumented evidence |
| AI research platforms | Listen Labs and the emerging AI-led interview category, Maze, UserTesting, Strella, Outset | automate interviews and unmoderated tests | don't build or change the product; separate tool, separate data |
| Feedback boards | Canny, UserVoice, Frill, Featurebase | collect and rank requests | upvote-count roadmaps; no evidence weighting; no link to code |
| Product analytics | PostHog, Amplitude, Mixpanel | superb "what happened" | structurally cannot answer "why"; no action |
| Product ops / roadmap | Productboard, Savio | aggregate feedback into roadmaps | human glue required; no verification of whether the change worked |
| Agent testing tools | Playwright + AI layers, Momentic, QA Wolf, Octomind | test automation with agent repair | test the app as an opaque artifact; brittle; no human evidence; no product decisions |

**The gap:** nobody closes the loop. Builders stop at generation. Research tools stop at findings. Boards stop at votes. Analytics stops at dashboards. Every one of them leaves the integration work to a human PM, and that integration work is now the expensive, slow part.

**Positioning statement.** *Cocreate is the platform that tells you what to build next — and builds it. Every app it makes is born instrumented, so real user evidence flows straight back into the next version.*

**The two-sentence differentiation.** Others help you ship faster. We help you ship the right thing and prove it — because the spec, the tests, the telemetry, the research, and the community all run off one source of truth.

**Where incumbents could attack:** a builder adding a feedback widget (easy — but without the spec-derived Action Surface it's a bolt-on, not a loop); a research platform adding an app builder (harder — they have no runtime); PostHog adding agents that act (plausible, and the most credible threat — their distribution is enormous). The defence is depth of the loop and the cross-app dataset, not feature breadth.

Prior art to be aware of and to differentiate from: IR-based scaffolded app generation ([Athena](https://acm-stag.literatumonline.com/doi/pdf/10.1145/3742413.3789133?download=true)) and the active debate over synthetic users ([ACM Interactions](https://interactions.acm.org/blog/view/the-synthetic-persona-fallacy-how-ai-generated-research-undermines-ux-research), [AIES](https://ojs.aaai.org/index.php/AIES/article/view/36553)) — which is a *tailwind* for a platform whose pitch is evidence honesty.

---

## 15. Risks and failure modes

Ordered by expected damage.

| # | Risk | Why it's dangerous | Mitigation |
|---|---|---|---|
| 1 | **Simulation collapse** | the loop closes on itself; confident, coherent, worthless output | provenance typing, human-evidence requirement, red team, blinded verification, ratio monitoring (§6.5) |
| 2 | **Goodhart / metric gaming** | agents optimise the measure, not the goal; dashboards go green while users suffer | counter-metrics, human-owned definitions, synthetic traffic excluded, learnings (not metrics) as the platform objective |
| 3 | **Small-N overfitting** | 5 users' opinions reified as product truth; this is the most common real-world research failure | n<10 shows counts not percentages; two-signal promotion rule; triangulation requirement; confidence display everywhere |
| 4 | **Research fatigue / user harassment** | over-instrumented apps annoy users; feedback prompts suppress the usage they measure; recruiting becomes spam | prompt budgets, dismissal memory, consent, incentives, global opt-out, sender reputation monitoring |
| 5 | **Cost blowup** | loops burn tokens chasing noise; margin evaporates | per-loop budgets, diminishing-return stop rules, model tiering, derivation caching, hard spend caps |
| 6 | **Trust / liability** | an agent breaks a live business process; the customer was never really in control | risk classes, canary + auto-rollback, kill switch, immutable audit, rehearsal for R3, insurance |
| 7 | **Consensus pathologies** | tyranny of the majority, brigading, manufactured agreement, alienated minority segments | segment-level consensus with quorums, rationale-required voting, dissent preservation, brigade detection, representativeness checks (§8.4) |
| 8 | **Category confusion** | "is it a builder or a research tool?" kills marketing efficiency | one promise, one north-star metric, wedge-first GTM, ruthlessly consistent language |
| 9 | **Platform risk** | a foundation-model vendor ships this natively | model-agnostic; own the runtime, ledger, standard, and panel; assume generation commoditises |
| 10 | **Lock-in anxiety** | buyers fear hostage-taking; blocks enterprise deals | full code export, open SDK, BYO cloud, portable data, published exit path. Make export a feature, sold openly |
| 11 | **Flaky tests destroy the gate** | a gate nobody trusts is worse than no gate, because it launders bad changes as verified | <1% flake target, quarantine, agent repair, hard cap that halts the loop |
| 12 | **False confidence in non-technical users** | "the dashboard said users wanted it" → bad bets with a veneer of rigour | evidence strength shown in the UI, dissent shown, contested states visible, humility in the copy |
| 13 | **Legal / ethical exposure** | consent, recording, GDPR, accessibility, participant treatment | §12.4–12.6, standing ethics review, jurisdiction checks, DPA hygiene |
| 14 | **Horizontal over-reach** | general-purpose builder is a swamp; instrumentation stops being derivable | stay vertical until the loop is undeniably excellent; templates over generality |
| 15 | **Cold start** | empty board, no users, no telemetry — the loop has nothing to eat | vertical templates with pre-built jobs/studies; synthetic pre-screening to seed hypotheses; concierge recruiting in Phase 0 |

### The one to watch above all others

Risk 1 is not like the others. Risks 2–15 are business problems. Risk 1 is an *epistemic* problem: if the platform's evidence becomes untrustworthy, the product has no reason to exist, and the failure is invisible from the inside — the metrics look great. Every architectural choice in §5 and §6 should be read as an answer to it. The one-line test to apply to any proposed feature: **does this make it easier or harder for a synthetic signal to be mistaken for a human one?**

---

## 16. Riskiest assumptions and how to falsify them

| # | Assumption | Why it might be false | Falsification test | Kill / pivot signal |
|---|---|---|---|---|
| A1 | Spec-derived Action Surfaces yield E2E tests stable enough to gate on | real apps have third-party widgets, iframes, animation, race conditions | build 3 apps of differing complexity; measure flake over 30 days and 500 runs | flake > 3% after agent repair → the gate premise fails |
| A2 | Non-technical owners can review a *spec* and make good decisions | specs are still abstractions; review fatigue is real | 10 non-technical owners; compare their spec-review decisions to an expert panel's | agreement near chance → the "no code, full legibility" principle is broken |
| A3 | In-app intercepts + board + guided tests beat agency research on cost *and* usefulness | recruited users ≠ real users; panel quality is poor | same research question, both methods, blinded comparison of findings | agency findings materially better → the loop is a supplement, not a replacement |
| A4 | Agents can triage feedback to the right job and severity without a PM | tone, sarcasm, implicit context, multi-problem posts | held-out human-labelled feedback set; measure agreement + calibration | agreement < 80% on job linkage → keep a human in triage |
| A5 | Segment-consensus mechanisms produce decisions that losing users still accept | people dislike losing regardless of process fairness | pilot community; measure acceptance, dissent, churn after 10 decisions | losers churn → consensus features harm retention; drop or radically rethink |
| A6 | Users tolerate in-app research prompts without retention damage | interruption is a real cost, and the cost is invisible for weeks | prompt-budget A/B on live apps; measure D7/D30 retention and task completion | measurable retention drop → prompt budgets near zero, shift to panel-only |
| A7 | Agencies and SMBs will pay for the loop | "we know we should talk to users" ≠ budget line | 10 design partners; ask for prepay | < 3 prepay → wedge is wrong or the buyer is wrong |
| A8 | The cross-app causal dataset is real and usable | confounds everywhere; every app is different | after 200 iterations, do intervention priors predict outcomes on held-out apps? | no predictive lift → the moat thesis needs rethinking |
| A9 | One spec can derive nine artifacts without collapsing into special cases | real requirements are messy; escape hatches multiply | count spec-level special cases per app; track how often the escape hatch is used | escape hatch > 40% of logic → the IR is too weak; narrow the vertical |

Run A1, A2, A4, and A7 first. They are cheap, and they test the load-bearing claims.

---

## 17. Roadmap and MVP scope

### Phase 0 — Concierge (0–6 weeks): prove the loop, write nothing

Run the full loop manually for 3 design-partner agencies using off-the-shelf agents. Deliverable is not software; it is the **loop playbook** plus reusable artifact templates: spec format, evidence ledger, decision record, study script, ratchet convention.

*Goal:* learn what the artifacts must contain. *Kill criteria:* if partners won't engage with the loop even when done for them, the problem isn't tooling.

### Phase 1 — Instrumentation layer (2–4 months): the wedge product

Ship `@cocreate/client` + `server` + CLI, the Action Surface spec, telemetry, the feedback CTA, the board, and the guided-test runner — **for apps built by any tool.** Prove it on 5–10 real apps, at least 3 of which were not built by us.

*Goal:* a working app, built elsewhere, gets a feedback loop in under a day. *Success:* 5 paying apps, ≥1 human signal per app per week, evidenced learning cycle < 5 days.

### Phase 2 — The loop (4–8 months)

Triage, analyst, research, test, and red-team agents. Evidence ledger UI with strength and contested states. Generated job E2E tests + the ratchet. Preview environments. Change sets with hypotheses, canary, and auto-rollback — initially with a human doing the spec edit and the agent doing the rest.

*Goal:* a validated iteration in under 48 hours. *Success:* ≥60% of changes have linked evidence; >50% of iterations produce a non-obvious learning.

### Phase 3 — The builder (8–14 months)

Full spec→app generation for one narrow vertical. PM/builder/architect agents. Vertical template with jobs, metrics, studies, and board structure pre-built. Consensus mechanisms v1 (segment quorums, rationale-required support, decision records).

*Goal:* idea → running app → first human evidence in under 24 hours. *Success:* 20 apps shipped, agency repeat usage, 1 template with >60% adoption among new apps.

### Phase 4 — Platform (14–24 months)

PaaS adapters + BYO cloud. Template marketplace. Owned participant network. Open-source SDK and Action Surface standard. Benchmarks product built on the cross-app dataset. Enterprise tier with Topology C and data residency.

### MVP — the smallest thing that proves the loop

**In scope:** one vertical template (recommend scheduling/rota or member management) · the Action Surface + SDKs · telemetry auto-derived from spec · one feedback CTA type · a minimal board with rationale-required support · one study method (unmoderated task, text + screen) · the evidence ledger with provenance and strength · generated job E2E tests · the ratchet rule enforced by the platform.

**Explicitly out of scope:** visual editor, general-purpose codegen, multi-tenant enterprise controls, marketplace, human-moderated sessions, mobile native, third-party PaaS adapters, consensus voting beyond rationale-required support.

**The MVP's single acceptance test:** take a real app with real users, and produce one decision backed by human evidence that the owner says they would not otherwise have made. Everything else is instrumentation in service of that.

### If you only had six weeks

1. Write the Action Surface spec (2 days) and instrument one hand-written app by hand (3 days).
2. Generate job tests from it; measure stability for a week (2 days).
3. Ship the feedback CTA + ledger to that app's real users (1 week).
4. Run one guided study manually, n=5, with agent-assisted analysis (1 week).
5. Write one decision record, make one change, verify the metric moved (1 week).
6. Write down what was painful. That list is the product roadmap, and it will be more accurate than this document.

---

## 18. Metrics

**Platform north star: Time-to-Evidence (TTE).** Median hours from a stated belief to human-sourced evidence for or against it. Sub-48h by end of Phase 2; sub-4h by end of Phase 3. This is the number the whole company optimises, and it is honest about what the product is for.

**Loop health**

| Metric | Target | Why |
|---|---|---|
| Evidence coverage | >60% of shipped changes have ≥1 human signal | the core discipline; below this the platform is just a builder |
| Change-to-evidence latency | <48h median | the wedge's promise |
| Ratchet count | monotonic ↑ | quality is compounding, mechanically |
| Inconclusive/negative results retained | >90% recorded | anti-self-delusion |
| Test flake rate | <1% | a gate nobody trusts is worse than none |
| Revert rate | 5–20% | <5% suggests evidence is too conservative; >20% suggests it's not working |
| Cost per validated learning | ↓ trend | unit economics of the loop |
| **Synthetic:human signal ratio** | watch it; warn above 5:1 | the canary for simulation collapse |
| Guardrail breaches caught pre-prod | ↑ | the safety envelope is working |

**Product health:** activation (first instrumented app < 1 day) · weekly active boards (owners engaging with evidence, not vanity posts) · participant NPS and completion rate · owner actions taken per evidence item (is the ledger actually read?) · D30/D90 workspace retention.

**Business:** net revenue retention (>120% target, driven by iteration credits) · gross margin after model + panel cost (>65%) · iterations per account per month · % of apps built by third-party tools (the wedge's health check) · agency-sourced apps per partner.

**Anti-metrics (deliberately not optimised):** number of agents run, tokens consumed, tests generated, posts created, signal volume. All are easy to inflate and none of them means anything.

---

## 19. Open decisions, with recommendations

| # | Decision | Options | Recommendation |
|---|---|---|---|
| 1 | Wedge: builder-first or feedback-first? | (a) compete on generation (b) sell the loop, expand into building | **(b).** Avoid the commoditising fight; the loop installs the moat and creates pull |
| 2 | First vertical for Phase 3 | rota/scheduling · clinic bookings · field service · member/community · internal approvals | **scheduling/rota or member management.** Two-sided (consensus is real, not decorative), time-dependent (fake-clock testing pays off), feedback-rich, low compliance risk |
| 3 | Generated vs hybrid code | full generation · hybrid · code-first with generated wiring | **hybrid** (generated wiring + declarative logic + fenced custom modules wrapped in the Action Surface) |
| 4 | Consensus scope | global · per-segment with quorum | **per-segment.** Global consensus systematically harms minority segments the product depends on |
| 5 | Consensus object | features · problems and priorities | **problems, constraints, priorities.** Users report problems well and prescribe solutions badly |
| 6 | Default hosting | platform · BYO PaaS | **platform by default, adapters shipped in Phase 4.** Preview support is a hard requirement of any adapter |
| 7 | Model strategy | own fine-tunes · multi-model routing · single vendor | **multi-model routing, BYO key.** Assume generation commoditises; keep cost lever and negotiating power |
| 8 | Open-source the SDK? | closed · open SDK + open Action Surface spec | **open both.** The standard is worth more than SDK revenue, and it accelerates the wedge |
| 9 | Own participant panel, and when? | never · Phase 1 · Phase 4 | **Phase 4**, via panel providers until then. Owning it early means owning a trust-and-safety problem before product-market fit |
| 10 | Human-moderated research in scope? | yes · no | **Yes, but Phase 2+, platform-assisted.** Some questions need a human; refusing them caps the ceiling. Don't build it first |
| 11 | Location/data posture | single region · multi-region from day one | **single region + Topology C escape hatch** until enterprise demands otherwise |
| 12 | Sell to builders or to the person with the problem? | developer-led · owner-led | **owner-led.** The buyer is the person with the problem. Developers arrive via the SDK and the open standard |
| 13 | How opinionated should the platform be? | toolkit · opinionated platform | **opinionated.** Derivable instrumentation requires a narrow target. Generality is the enemy of the entire architecture |
| 14 | Name | see §20 | **Cocreate** works; verify trademark and domain before committing |

---

## 20. Naming

The name should say *loop* or *evidence*, not *build* (the build is not the differentiator). Candidates:

- **Cocreate** — matches the ambition (humans + agents + users co-creating), warm, non-technical, already the project's folder name. Risk: generic, crowded namespace.
- **Loopwright** — a maker of loops. Distinctive, pronounceable, hints at craft and iteration.
- **Groundtruth** — names the epistemic core directly (the thing that must be human). Strong, but heavy and slightly militantly nerdy.
- **Ratchet** — names the compounding-quality mechanism. Memorable, but obscure without explanation, and mildly menacing.
- **Substrate** — what the spec is. Elegant, but abstract and heavily used in dev tooling.
- **Kettle** — warm, small-business friendly, memorable; says nothing about the product. Fine if the brand carries the explanation.

Recommendation: **Cocreate** for warmth and accuracy, **Loopwright** if the trademark search on Cocreate is bad. Register the Action Surface spec under a neutral name regardless, so the standard can outlive the brand.

---

## 21. Glossary

| Term | Meaning |
|---|---|
| **Action Surface** | The machine-readable manifest of a web app's user-perceivable actions, with stable IDs, preconditions, effects, job linkage, and ARIA binding. The shared interface between agents and humans. |
| **Change set (CS)** | A bundle of spec edits + regenerated artifacts + new tests, with a pre-registered hypothesis and expected metric effect. The unit of iteration. |
| **Claim** | A falsifiable belief about the app, stored in the evidence ledger with provenance, n, strength, and status. |
| **Consensus snapshot** | Living board view: agreed problems ranked, contested points with the shape of disagreement, decided-and-shipped, decided-and-rejected, open questions. |
| **Counter-metric** | A metric expected to move the wrong way if a primary metric is being gamed. |
| **Decision record (DR)** | Immutable output of a loop: what was decided, on what evidence, what was rejected, who dissented, and what would cause a revisit. |
| **Evidence graph** | signal → action → job → spec element → change → metric. The queryable asset. |
| **Evidence ledger** | The set of all claims with provenance and strength. The platform's memory. |
| **Integrative solution** | A change that satisfies conflicting segments rather than trading between them. The first thing an agent should look for in a cross-segment conflict. |
| **Job (J)** | A user's goal, stated as an outcome, with a success criterion. Everything links back to these. |
| **Pre-registration** | Recording hypothesis, metric, and expected effect *before* shipping, to prevent post-hoc metric selection. |
| **Product Spec IR** | The single declarative source of truth from which app, tests, telemetry, studies, and guardrails are derived. The only artifact agents edit. |
| **Provenance** | `human_observed` \| `human_reported` \| `machine_inferred` \| `synthetic_simulated`. Determines evidential weight. |
| **Ratchet** | The rule that every human-reported defect becomes a permanent automated test before the fix is done. |
| **Red team agent** | Agent whose only mandate is to falsify the current belief and propose the cheapest human test that would break it. |
| **Signal** | Any observed datum entering the loop: telemetry event, CTA response, board post, study observation, test result, incident. |
| **Simulation collapse** | The failure mode in which synthetic evidence, agent interpretation, and agent action form a closed loop containing no reality. |
| **Spec diff** | The reviewable change to the spec — the human's primary approval surface, in place of a code diff. |
| **Strength (0–5)** | Composite evidential weight: provenance × method rigour × sample size × recency. |
| **Time-to-Evidence (TTE)** | Median hours from a stated belief to human-sourced evidence for or against it. The north star. |
| **Validated** | A change status requiring ≥1 human-sourced signal. Cannot be granted on synthetic evidence. |

---

*This document is a concept draft. The three load-bearing claims — derivable instrumentation, the shared Action Surface, and provenance-typed evidence — are the ones to attack first; §16 lists the cheapest experiments that would break them.*
