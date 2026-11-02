# Product decisions

The main decisions behind Gatekeeper, with the reasoning and the trade-off accepted.

## 1. Deterministic checks first, LLM only for judgment

**Decision.** Anything a lookup can resolve (ethics documentation present, fields on the approved list, use category approved, sensitivity tier, governance flag) is checked by plain code. The LLM sees only what rules cannot resolve: whether requested fields fit the stated purpose, whether the purpose is ambiguous, and whether the use is novel.

**Why.** Rule-checkable criteria should behave the same every time and be testable without a model. It also keeps the model's job small and its output easy to verify.

**Trade-off.** More up-front work to define rules precisely. Some gray cases move to the model layer by default.

## 2. Two hard escalation rules, decided in code

**Decision.** A request with identifiable-tier fields, or for a dataset that requires Indigenous governance authorization, always recommends escalation, whatever else passes. The packet names which rule fired.

**Why.** These are policy decisions, not judgments. A model should not weigh them, and a clean pass elsewhere should not soften them.

**Trade-off.** Some escalations will be routine for the committee. That cost is accepted for the safety guarantee.

## 3. Governance requests never return for clarification

**Decision.** Ethics approval is a document-presence check: a researcher can supply a missing letter and a rule can verify it is there. Indigenous governance authorization is different. Judging whether it is genuine, current, issued by the right body, and applicable to this purpose and these fields takes authority that Gatekeeper and the front-line reviewer do not have. Gatekeeper never assesses attached authorization letters, and the case always escalates.

**Why.** Returning it would imply the front-line process can resolve it.

**Trade-off.** Less automation for this category, by design.

## 4. Stop and escalate if context cannot be verified

**Decision.** If Gatekeeper cannot retrieve or verify the policy or catalog version, it stops, makes no assessment, and recommends escalation, flagged as a system-level failure that is distinct from uncertainty about the request.

**Why.** An assessment on stale or unverified rules is worse than none.

**Trade-off.** A retrieval outage produces escalations, not forwards.

## 5. Draft only, with no confidence threshold

**Decision.** Every output is a draft. The reviewer approves, edits, rejects, or escalates each one. There is no confidence score above which Gatekeeper acts alone, including for requests that look clearly compliant.

**Why.** The reviewer's checkpoint is the safety mechanism. A threshold would turn it into an optional step.

**Trade-off.** Time saved comes from a better starting point for review, not from removing review.

## 6. Uncertainty escalates, and missing information returns

**Decision.** Missing, mechanically checkable information (for example an ethics letter without an approval number) returns for clarification with a drafted message. Ambiguity, conflicting evidence, or a criterion the model cannot judge reliably escalates with the uncertainty stated. Gatekeeper never resolves uncertainty by forwarding on an assumption.

## 7. Memory scoped to one request

**Decision.** Gatekeeper remembers only a request's own checklist state. It does not use a researcher's or institution's prior requests, a reviewer's past overrides, or patterns from earlier cases.

**Why.** Familiarity should not bias a new request, and researcher profiling should not happen. Policy changes and learning from overrides stay under human control.

**Trade-off.** No personalization. Each request is judged fresh.

## 8. One packet shape for every request

**Decision.** Every request gets the same standalone packet, with the recommendation first. Escalations add framing for the committee, and returns add a draft message. The committee can open the retained packet and audit record later, without rerunning Gatekeeper against newer information.

**Why.** A fixed layout lets a reviewer working in short sittings scan in seconds, and gives the committee a record of what supported each decision.

## 9. Five statuses, and no "rejected"

**Decision.** Awaiting Review, Returned for Clarification, Escalated, Forwarded, and Re-evaluation Pending. "Reject" is an interface action that rejects Gatekeeper's recommendation and then requires a different route. If the final action differs from the recommendation, the reviewer gives an override reason, which is stored in the audit log.

**Why.** Only the committee can approve or deny access, so a "rejected" status would claim an authority the reviewer lacks.

## 10. Reviewer-triggered re-run

**Decision.** Re-run Gatekeeper is something a reviewer chooses, for example after a researcher replies or a missing version becomes retrievable. It is not an intake trigger. The rerun re-checks only unresolved criteria and shows what changed. It never changes an already actioned outcome, and it writes a separate audit entry with the reason and old and new versions.

## 11. A periodic quality pass, not a real-time second agent

**Decision.** I considered a second agent that reviews Gatekeeper's output in real time and rejected it for the first version. It would add another model's errors and cost in the critical path, and it could dilute the reviewer's ownership of the decision. The roadmap item instead is a periodic offline pass that studies override patterns across the audit log to check calibration and reports to humans.
