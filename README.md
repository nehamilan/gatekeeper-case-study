# Gatekeeper

**An AI agent that drafts triage recommendations for health data access requests. Rules where a rule can decide, an LLM where judgment is needed, and a human reviewer on every decision.**

> Capstone prototype built for the AI Product Management cohort at Product Faculty (Sep–Oct 2026). The organization, policy, datasets, researchers, and requests are all synthetic. This repository is a public case study. The prototype's source code is private.

**[Watch the 4-minute walkthrough](https://www.loom.com/share/c43a1792bbf34f77b3c6bdb28ef4309e)**

<!-- Optional: replace the line above with a Loom thumbnail that links to the video:
[![Gatekeeper walkthrough](images/loom-thumbnail.png)](LOOM_URL) -->

---

## Status

- Working prototype: a review queue, a case page with a decision card, and an append-only audit log, run against 12 synthetic evaluation cases.
- The prototype currently runs with a real model.
- Not a production system. It sends, forwards, grants, and denies nothing.

## The problem

At a data trust, researchers request access to restricted health datasets. A front-line committee reviewer reads each request against a written eligibility policy and decides whether to **forward** it to the committee, **return** it to the researcher for clarification, or **escalate** it for closer review.

Today every request gets the same full manual read, whether it is a routine renewal or a high-risk case. The criteria live in the reviewer's head, so two reviewers can reach different outcomes. When a request is returned and comes back, nothing records what was already checked, so the reviewer starts over.

## Who it is for

A front-line data access committee reviewer who works through requests in short sittings between other duties. They know the policy but are not a specialist in every dataset. Final approval or denial of access belongs to the committee, not the reviewer, and not Gatekeeper.

## What Gatekeeper does

For every request, before the reviewer opens it, Gatekeeper produces one standalone **review packet**:

- a draft recommendation (forward, return for clarification, or escalate), stated first;
- a criterion-by-criterion evidence trail (pass, fail, or uncertain), each row citing the request, the policy rule, and the catalog entry;
- uncertainty and safety callouts, including which fixed rule fired, if any;
- the exact policy and catalog versions used, and the assessment time;
- for escalations, framing for the committee; for returns, an editable draft message to the researcher.

The reviewer then approves, edits, rejects, or escalates that recommendation. Nothing happens until they act.

## How it works

```
Submitted request
   -> retrieve and verify policy + dataset catalog versions
   -> deterministic checks 
   -> fixed safety overrides (always escalate)
   -> LLM judgment, only for what no rule can resolve
   -> review packet
   -> human reviewer decides
   -> append-only audit log
```

See [docs/architecture.md](docs/architecture.md) for diagrams.

## Product decisions worth reading

| Decision | Why |
|---|---|
| Deterministic checks first, LLM only for judgment | Checkable rules should not depend on a model. It makes behavior predictable and testable. |
| Two fixed escalation rules, decided in code | Identifiable fields and Indigenous-governance datasets always escalate, whatever else passes. A model should not weigh these. |
| Governance requests never return for clarification | Judging whether an authorization is genuine, current, and applicable needs authority Gatekeeper and the front-line reviewer do not have. |
| Stop and escalate if policy or catalog cannot be verified | Gatekeeper never assesses a request on unverified context. |
| Draft only, with no confidence threshold | The human checkpoint cannot be bypassed, even for requests that look clearly compliant. |
| Memory scoped to a single request | No researcher or institution history feeds a later recommendation, which avoids profiling and bias from familiarity. |

The full list, with trade-offs, is in [docs/product-decisions.md](docs/product-decisions.md).

## How I tested it

Twelve synthetic cases, each with an expected route and expected policy and catalog version identifiers, fixed before testing. Each case is scored in four separate groups (route, evidence, prohibited action, reasoning) so one failure cannot hide another. A correct route with wrong evidence still fails. See [docs/evaluation-design.md](docs/evaluation-design.md).

## My role

I defined the product: the user, the workflow, the architecture, the escalation rules, the human-control design, the status model, and the evaluation plan. I built the prototype with an AI coding assistant. 

## What I learned

See [docs/learnings.md](docs/learnings.md).

## What is next

A periodic quality-review pass that analyzes override patterns across the audit log to check Gatekeeper's calibration, run offline and reported to humans. I rejected a real-time second reviewer agent for the first version. The reasoning is in the decision log.

---

*All data, names, institutions, and policies in this project are fictional. Gatekeeper is a prototype and is not connected to any real system.*

© 2026 Neha Milan. All rights reserved. 

