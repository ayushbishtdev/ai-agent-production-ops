# AI Agent Production Ops — Cheatsheet

> One-page quick reference for on-call incidents, sprint planning, and architecture decisions.
> Full context and sources: [README.md](./README.md)

---

## Production Gap at a Glance

| Stat | Figure | Source |
|---|---|---|
| Orgs that deploy agents to *some* environment | ~71% | 2026 enterprise surveys |
| Orgs that reach *genuine* production | ~11% | 2026 enterprise surveys |
| Avg agents per enterprise in pilot | 12 | Belitsoft 2026 |
| Of those 12, % that can't communicate with each other | ~50% | Belitsoft 2026 |
| Orgs planning autonomous agents by end of 2026 | 52% | Korn Ferry |
| Median pilot-to-production timeline (successful 11%) | 4–6 months | agileleadershipdayindia.org |
| Scrum.org AI integration uncertainty gap | 54.3% | Scrum.org |
| Standup time reduction when agent updates removed | up to 47% | agileleadershipdayindia.org |

---

## The 5-Tier Orchestration Stack

```
Tier 5  │ Governance & Observability  │ Policy, telemetry, spend caps, kill-switch
Tier 4  │ Sequencing & Planning       │ Which agent acts when; delegation; conflict resolution
Tier 3  │ State & Memory              │ Shared durable context — prevents agent amnesia
Tier 2  │ Communication               │ A2A protocol, schema enforcement, versioned payloads
Tier 1  │ Identity & Registry         │ Verified agent identity, ownership, live registry
```

If Tier 1 is missing → you cannot audit or secure your fleet.
If Tier 2 is missing → silent schema drift; data corruption.
If Tier 3 is missing → agents lose context between steps; quality degrades.
If Tier 4 is missing → no deterministic sequencing; infinite loops.
If Tier 5 is missing → CISO won't sign off; EU AI Act exposure.

---

## Platform Quick-Pick

| Scenario | Platform | Reason |
|---|---|---|
| Regulated industry (BFSI, healthcare) | **Camunda** | Audit trails, BPMN compliance, human-in-the-loop routing built in |
| Complex cyclic control flow, technical team | **LangGraph** | Granular developer control; must build governance layer yourself |
| Rapid internal prototype / hackathon | **CrewAI** | Fast to spin up; fails past ~50 concurrent agents |
| Salesforce-native workflow | **Agentforce** | Only if all agents live inside the Salesforce ecosystem |

**TCO reminder:** LangGraph and CrewAI are free to license. The real cost is senior engineers building what the framework left out.

---

## A2A Protocol: Handshake Sequence

```
1. DISCOVER  →  Read remote Agent Card (JSON: name, capabilities, URL)
2. NEGOTIATE →  Agree on modalities (text / forms / streaming)
3. DELEGATE  →  JSON-RPC 2.0 POST with contextId, locale, instructions, history
4. EXECUTE   →  Remote agent cycles through "working" → "completed"
                 Artifact returned (optionally streamed via SSE)
```

**Auth hardening (the gap the docs skip):**
- Do NOT use default A2A unauthenticated endpoints in production
- Enforce mTLS + RBAC at the API gateway layer, not inside agents
- Restrict quotas at gateway to prevent resource-draining task storms

**A2A vs MCP:** MCP = model ↔ tools. A2A = agent ↔ agent. Both can coexist.

---

## 7 Failure Modes — Detection and Fix

| Mode | Signal | Fix |
|---|---|---|
| **Silent Retry Loop** | API spend spike, no monitoring alert | Hard retry cap 3–5 + exponential backoff + escalation |
| **Prompt Drift** | Output quality degrades across multi-step workflows | Shared memory state + strict schema validation |
| **Loop Bomb** | Token spend accelerating overnight between two agents | Timer boundaries + retry limits in orchestration layer |
| **Cascade Failure** | Final output wrong; no obvious single root cause | Trace prompt-response backward through centralized logs |
| **Agent vs Tool Failure** | SREs debugging wrong stack for hours | Separate agent reasoning logs from tool execution logs |
| **Prompt Injection Chain** | Privileged agents taking unexpected actions | Zero-trust A2A boundaries (not just network perimeter) |
| **EU AI Act Article 15** | Failed compliance audit | Immutable logs of all tool calls, state changes, A2A messages |

**Retry-backoff standard:** 3–5 max retries → exponential delay → log failure reason to persistent memory → escalate to human.

---

## Kill-Switch: 4 Telemetry Triggers

Fire the kill-switch (automated, not human-triggered) on **any** of:

1. **Token velocity spike** — 500%+ increase in 60-second rolling window
2. **Tool call recursion** — same function + same parameters called 5+ times consecutively
3. **IAM boundary violation** — agent attempting to access resource outside zero-trust perimeter
4. **Sentiment/toxicity flag** — erratic text generation on customer data (if applicable)

**Circuit breaker ≠ Kill-switch:**
- Circuit breaker: pauses agent while *external system* recovers
- Kill-switch: revokes IAM token, terminates agent when *agent itself* is failing

**Target halt latency: < 90 seconds**

**Post-halt audit log must capture:**
- Exact telemetry threshold breached
- Agent's internal memory state at halt time
- Final prompt executed
- Timestamp of IAM revocation

---

## 7 CFO-Grade KPIs

| KPI | Formula | Red flag |
|---|---|---|
| Agent ROI per Workflow | (Labor saved − Agent cost) ÷ Agent cost | Negative means kill the project |
| Human Intervention Rate | % tasks requiring human correction | > 40% = you built a chatbot, not an agent |
| Blast Radius | Max $ / data loss if agent fails completely | No cap defined = no CISO sign-off |
| Cost-Per-Agent-Task | Total API + infra spend ÷ tasks completed | Should be ≥ 80% cheaper than human equivalent |
| Agentic Uptime | % tasks reaching "done" without loops | ≠ API uptime (agent can loop at 100% API uptime) |
| Agent Story Point Contribution | Points completed by agents (separate track) | Do NOT mix with human velocity — distorts forecasting |
| OKR Alignment | Agent KR directly maps to corporate objective | No OKR tie = first cut in budget downturn |

---

## Agent Definition of Done (Minimum Gates)

- [ ] Task completed per acceptance criteria
- [ ] API spend within predefined rate limits
- [ ] Reasoning logs saved and auditable (EU AI Act Article 15)
- [ ] Human-in-the-loop validation passed (for write actions)
- [ ] Output passes downstream schema validation
- [ ] No retry-limit breaches during execution

---

## Agile Ceremony Quick Reference

| Ceremony | Old pattern | Agent-augmented pattern |
|---|---|---|
| **Daily standup** | Includes agent status updates | Pre-standup async review of agent output queues; standup = humans only |
| **Story points** | Fibonacci for all tasks | Agents: estimate blast radius + token cost instead |
| **Definition of Done** | Task + tests pass | Add: compliance logs, spend within limits, human validation |
| **Mid-sprint failure** | Reallocate points | Automated kill-switch fires; task removed from active velocity calc |
| **Retrospective** | Team sentiment | Review intervention rate + cost-per-task telemetry |
| **Backlog refinement** | Feature priority | Add: system prompt audit for prompt drift on active agents |

---

## NIST AI RMF Quick Map

| Subcategory | What to check |
|---|---|
| Measure 2.6 | Agent performance validated in real-world conditions before go-live |
| Manage 1.3 | Mitigations implemented for all identified risks (blast radius, loops, drift) |
| Manage 2.3 | Incident response and recovery procedures documented and tested |

All three must be binary deployment gates — not advisory recommendations.

---

*Full source articles and deep-dives: [agileleadershipdayindia.org](https://agileleadershipdayindia.org)*
*Last updated: May 2026 · Update cadence: quarterly*
