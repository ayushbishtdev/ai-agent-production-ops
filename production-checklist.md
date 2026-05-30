# Autonomous Agent Production Checklist

**23 binary deployment gates mapped to NIST AI RMF subcategories**

Use this checklist before any autonomous agent touches live, business-critical data. Every gate is a binary pass/fail. A single unchecked gate is grounds to delay deployment. This is a certification document, not a vibe check.

> Source analysis: [The Autonomous Agent Production Checklist NIST Hides](https://agileleadershipdayindia.org/blogs/autonomous-agent-production-checklist/autonomous-agent-production-checklist.html)

---

## How to Use This Checklist

- Run this audit for **every agent** before production launch, including agents already in staging for months
- Each gate includes the **failure consequence** if skipped
- Gates marked `[NIST]` have a direct NIST AI RMF subcategory mapping
- After completion, have both the engineering lead and enterprise risk management provide formal sign-off

**Sign-off block (copy into your internal doc):**

```
Agent name / ID:
Environment:
Audit date:
Engineering lead (name + signature):
Risk management sign-off (name + signature):
Gates passed: ___ / 23
Gates failed / deferred (list):
```

---

## Section 1: Identity and Registry (Tier 1)

- [ ] **Gate 1 — Agent Registry Entry**
  Agent is registered in a live, queryable registry with: name, version, owner, purpose, data it reads, data it writes, and escalation contact.
  *Failure consequence: Cannot audit, secure, or decommission the agent. Fleet management impossible at scale.*

- [ ] **Gate 2 — Verifiable Agent Identity**
  Agent has a verifiable, cryptographically signed identity token. It cannot impersonate another agent or assume a broader IAM role than its defined scope.
  *Failure consequence: Prompt injection chains can leverage the agent's identity to access privileged resources.*

- [ ] **Gate 3 — Ownership and On-Call Assignment**
  A named human (or team) is responsible for the agent's behaviour in production. On-call rotation is documented.
  *Failure consequence: When the agent fails at 2 AM on a Saturday, no one knows who to wake up.*

---

## Section 2: Communication and Schema (Tier 2)

- [ ] **Gate 4 — A2A Protocol Compliance** `[NIST: Measure 2.6]`
  All agent-to-agent communication uses schema-validated payloads. Agent Cards are current and versioned. No bespoke, undocumented message formats exist between agent pairs.
  *Failure consequence: Silent schema drift corrupts downstream agent context without triggering alerts.*

- [ ] **Gate 5 — API Gateway Authentication**
  All A2A endpoints are protected at the gateway layer with mTLS and RBAC. Authentication is NOT left as optional or defaulting to unauthenticated A2A spec defaults.
  *Failure consequence: Any compromised internal system can assign infinite tasks to the agent, draining API budget.*

- [ ] **Gate 6 — Schema Versioning Strategy**
  A schema versioning strategy is defined and documented. Breaking schema changes require agent card update + downstream consumer notification.
  *Failure consequence: An update to one agent silently breaks the JSON inputs expected by peer agents.*

---

## Section 3: State and Memory (Tier 3)

- [ ] **Gate 7 — Shared Memory State**
  If the agent operates in a multi-agent workflow, shared durable memory is configured. The agent does not start each step with an empty context window.
  *Failure consequence: Agent amnesia — context lost between handoffs causes prompt drift and downstream cascade failures.*

- [ ] **Gate 8 — Context Decay Testing** `[NIST: Measure 2.6]`
  Agent has been tested on workflows longer than its expected typical run. Output quality verified at steps 5, 10, and 20 of a multi-step chain to confirm intent is preserved.
  *Failure consequence: Prompt drift silently degrades output quality in production without triggering errors.*

---

## Section 4: Blast Radius Containment (Tier 4 / Tier 5)

- [ ] **Gate 9 — Blast Radius Defined and Capped** `[NIST: Manage 1.3]`
  The maximum theoretical financial and data damage if the agent's control logic fails completely is mathematically defined. The figure is documented and approved by the risk officer.
  *Failure consequence: No CISO sign-off; no rational budget approval; unlimited liability exposure.*

- [ ] **Gate 10 — Zero-Trust IAM Roles**
  Agent IAM roles follow the principle of least privilege. The agent cannot: mutate critical databases, execute external financial transactions, or access data stores outside its explicit operational scope — without a secondary validation layer.
  *Failure consequence: A rogue or compromised agent can overwrite production databases or trigger unauthorised financial actions.*

- [ ] **Gate 11 — Write-Action Human Approval (Initial Launch)**
  On initial production launch, all write actions require human-in-the-loop approval. This gate may be systematically lowered as the agent builds a reliable logging track record.
  *Failure consequence: First-run hallucinations or prompt drift can cause irreversible data mutations.*

---

## Section 5: Hard Spending Controls

- [ ] **Gate 12 — Hard API Spending Cap Set**
  A hard spending cap is configured at the orchestration layer. This is a physical severance of execution, not a Slack notification. The cap is set below the blast radius maximum.
  *Failure consequence: A silent retry loop can run for a weekend and burn through five- or six-figure API budgets undetected.*

- [ ] **Gate 13 — Kill-Switch Tested and Timed** `[NIST: Manage 2.3]`
  The automated kill-switch has been triggered in staging. Documented halt latency is under 90 seconds. All 4 telemetry triggers are verified to fire correctly: token velocity spike, tool call recursion, IAM boundary violation, and sentiment/toxicity flag (if applicable).
  *Failure consequence: Kill-switch exists on paper but fails in production, leaving the loop bomb running.*

- [ ] **Gate 14 — Retry Cap Configured**
  Maximum retries are hard-coded at the orchestration layer (recommended: 3–5). Exponential backoff is configured between retries. After cap is hit, the failure is logged to persistent memory and escalated to human-in-the-loop.
  *Failure consequence: Uncapped retries cause the silent retry loop failure mode — API provider shuts down your account for abuse.*

---

## Section 6: Observability and Logging

- [ ] **Gate 15 — Real-Time Telemetry Active** `[NIST: Measure 2.6]`
  OpenTelemetry (or equivalent) is streaming agent telemetry in real time. Telemetry is not batch-logged on a delay — delayed logging guarantees you catch incidents hours or days after they start.
  *Failure consequence: Blind to runaway cost loops; cannot trigger sub-90-second halt.*

- [ ] **Gate 16 — Agent Reasoning Logs Separated from Tool Logs**
  Agent reasoning/LLM logs are stored separately from external tool execution logs. An SRE can independently verify whether a failure was caused by LLM hallucination (agent failure) or by an external API going offline (tool failure).
  *Failure consequence: Engineers debug the wrong stack layer for hours; root cause analysis takes days.*

- [ ] **Gate 17 — Immutable Compliance Logs Configured** `[NIST: Manage 2.3]`
  Immutable, time-stamped logs capture every: tool call, state transition, API response, and A2A message exchange. Logs cannot be overwritten. Retention period meets regulatory requirements.
  *Failure consequence: EU AI Act Article 15 violation; cannot prove agent decision trail to regulators or auditors.*

- [ ] **Gate 18 — Audit Trail for Kill-Switch Events**
  When the kill-switch fires, the following are automatically captured: exact telemetry threshold breached, agent memory state at halt, final prompt executed, timestamp of IAM revocation.
  *Failure consequence: Cannot perform compliant post-mortem; potential regulatory fine under EU AI Act.*

---

## Section 7: Rollback and Incident Response

- [ ] **Gate 19 — Rollback Plan Documented and Tested** `[NIST: Manage 2.3]`
  A rollback plan exists and has been tested in staging. The plan documents: how to disable the agent, scripts to revert database mutations to pre-agent state, and the designated human(s) who take over the workflow post-rollback.
  *Failure consequence: When an agent causes a production incident, the team scrambles to undo actions with no plan, extending downtime.*

  **Rollback plan minimum template:**
  ```
  1. State Reversion: [script / procedure to undo database writes]
  2. Message Queue Flush: [procedure to clear pending tasks from orchestration layer]
  3. Human Handoff: [named person / team + contact + SLA]
  4. Communication: [who is notified; internal + external if applicable]
  5. Re-enable criteria: [conditions required before agent is reactivated]
  ```

- [ ] **Gate 20 — Incident Response Procedure Assigned** `[NIST: Manage 2.3]`
  A documented incident response procedure exists for agent failures. Escalation path is defined: who is paged, in what order, with what SLA for response.
  *Failure consequence: Production incident becomes chaos — multiple people try to fix simultaneously, make it worse.*

---

## Section 8: Regulatory Compliance

- [ ] **Gate 21 — NIST AI RMF Measure 2.6 Satisfied**
  Agent performance has been validated in real-world (production-equivalent) conditions. Staging tests included: API outage simulation, conflicting input injection, prompt drift scenarios, and recursive logic loop tests. Agent stayed within blast radius in all scenarios.
  *Failure consequence: Staging passed; production fails. The most expensive version of "works on my machine."*

- [ ] **Gate 22 — EU AI Act Article 15 Compliance Verified** (for high-risk systems)
  If the agent is classified as high-risk under the EU AI Act: record-keeping obligations are met, human oversight mechanisms are functional, and reconstructable logs exist for every decision the agent makes.
  *Failure consequence: Regulatory fine; potential requirement to take system offline during investigation.*

- [ ] **Gate 23 — Human-in-the-Loop Escalation Path Active**
  A tested escalation path exists for edge cases: a human can take over any workflow the agent is executing, at any point, within a defined SLA. This path is not theoretical — it has been tested in staging.
  *Failure consequence: When the agent produces an edge-case failure, no human can intervene cleanly. The system requires a full restart.*

---

## Sign-Off Checklist Summary

| Section | Gates | Status |
|---|---|---|
| 1. Identity & Registry | 1–3 | ☐ Pass ☐ Fail |
| 2. Communication & Schema | 4–6 | ☐ Pass ☐ Fail |
| 3. State & Memory | 7–8 | ☐ Pass ☐ Fail |
| 4. Blast Radius Containment | 9–11 | ☐ Pass ☐ Fail |
| 5. Hard Spending Controls | 12–14 | ☐ Pass ☐ Fail |
| 6. Observability & Logging | 15–18 | ☐ Pass ☐ Fail |
| 7. Rollback & Incident Response | 19–20 | ☐ Pass ☐ Fail |
| 8. Regulatory Compliance | 21–23 | ☐ Pass ☐ Fail |
| **TOTAL** | **23 gates** | **☐ Certified ☐ Not ready** |

**Certification requires: all 23 gates passed with no deferred items.**

---

*For updates to this checklist, open an issue or PR at the repository.*
*NIST AI RMF subcategory references: Measure 2.6, Manage 1.3, Manage 2.3*
*EU AI Act reference: Article 15 (record-keeping for high-risk AI systems)*
*Last updated: May 2026*
