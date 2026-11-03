# Architecture

High-level design of the Gatekeeper prototype. All components work on synthetic data.

## Request flow

```mermaid
flowchart TD
    A[Researcher submits request<br/>status: Submitted] --> B[Gatekeeper retrieves policy<br/>and dataset catalog versions]
    B --> C{Versions retrieved<br/>and verified?}
    C -- No --> X[Stop. No assessment.<br/>Recommend escalate:<br/>context-verification failure]
    C -- Yes --> D[Deterministic checks]
    D --> E{Fixed override?<br/>identifiable-tier fields or<br/>Indigenous-governance dataset}
    E -- Yes --> F[Recommend escalate.<br/>Packet names the rule.]
    E -- No --> G{Any criterion unresolved<br/>by rules?}
    G -- Yes --> H[LLM judgment:<br/>proportionality,<br/>ambiguous purpose, novel use.<br/>Bounded answer or explicit uncertainty]
    G -- No --> I
    H --> I[Build review packet<br/>recommendation, evidence trail,<br/>callouts, versions, timestamp]
    F --> I
    X --> I
    I --> J[Status: Awaiting Review]
    J --> K[Reviewer: approve, edit,<br/>reject, or escalate further]
    K --> L[Forwarded, Returned for Clarification,<br/>or Escalated]
    L --> M[(Append-only audit log)]
    K -. differs from recommendation .-> N[Override reason required]
    N --> M
```

## Request statuses

Five statuses. There is no "rejected" or "denied" status, because only the committee can approve or deny access.

```mermaid
stateDiagram-v2
    [*] --> AwaitingReview: Gatekeeper packet complete
    AwaitingReview --> Forwarded: reviewer forwards
    AwaitingReview --> ReturnedForClarification: reviewer returns
    AwaitingReview --> Escalated: reviewer escalates
    ReturnedForClarification --> ReevaluationPending: reviewer re-runs Gatekeeper
    ReevaluationPending --> AwaitingReview: new draft packet ready
```

"Reject" is a reviewer action in the interface that rejects Gatekeeper's recommendation, not the researcher's request. The reviewer then chooses one of the three routes.

## Memory

memory is limited to one request, and no researcher or institution history is used.

## Boundaries

Gatekeeper cannot grant, deny, or finalise access, send anything to a researcher, forward a request, change the policy or catalog, or judge Indigenous governance authorization. Every output is a draft for the reviewer.
