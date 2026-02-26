# Agentic Workflow Design: Multi-Agent Orchestration for Expense Claims

## Strategic Summary

The expense claims workflow maps naturally onto a combination of agentic design patterns. Five specialized agents — Intake, Compliance, Fraud, Router, and Advisor — each own a distinct phase of the lifecycle, orchestrated by a lightweight state machine. The key architectural insight is that different phases demand different patterns: sequential chaining for data capture, evaluator-optimizer loops for pre-submission validation, parallelization for independent assessments (with the Fraud Agent using ReAct internally for exploratory investigation), routing for risk-based triage, and reflection for human-assisted review.

---

## The 5 Agents

| # | Agent | Persona | Serves |
|---|-------|---------|--------|
| 1 | **Intake Agent** | Receipt reader + data enricher | Employee |
| 2 | **Compliance Agent** | Policy enforcer + correction advisor | Employee + Auditor |
| 3 | **Fraud Agent** | Investigator + authenticity verifier | Auditor |
| 4 | **Router Agent** | Risk scorer + workflow decider | System |
| 5 | **Advisor Agent** | Auditor assistant + report summarizer | Auditor/Approver |

---

## The Redesigned Workflow

```
EMPLOYEE SIDE                           SYSTEM SIDE                         AUDITOR SIDE
============                           ===========                         ============

Employee captures                           |
receipt (photo/email/                       |
card feed)                                  |
     |                                      |
     v                                      |
 ┌──────────────────┐                       |
 │  INTAKE AGENT    │  Prompt Chaining      |
 │                  │                       |
 │  OCR/Extract     │                       |
 │      ↓           │                       |
 │  Categorize      │                       |
 │      ↓           │                       |
 │  Itemize         │                       |
 │      ↓           │                       |
 │  Currency Convert│                       |
 │      ↓           │                       |
 │  Enrich (merge   │                       |
 │   card feed data)│                       |
 └───────┬──────────┘                       |
         ↓                                  |
 ┌──────────────────────────┐               |
 │  PRE-SUBMIT VALIDATION   │               |
 │  Evaluator-Optimizer     │               |
 │                          │               |
 │  Compliance Agent        │               |
 │  evaluates report        │               |
 │      ↓                   │               |
 │  Issues found?           │               |
 │   YES → show employee    │               |
 │         specific fixes   │               |
 │         employee corrects│               |
 │         ↓ (re-evaluate)  │               |
 │   NO  → ready to submit ─┼──┐            |
 └──────────────────────────┘  │            |
                                │            |
         Employee clicks submit │            |
                                ↓            |
                     ┌─────────────────────────────────────┐
                     │     PARALLEL ASSESSMENT              │
                     │     Parallelization (Sectioning)     │
                     │                                      │
                     │  ┌─────────────┐  ┌───────────────────────┐  │
                     │  │ COMPLIANCE  │  │ FRAUD AGENT           │  │
                     │  │ AGENT       │  │ (ReAct)               │  │
                     │  │ (Eval-Opt)  │  │                       │  │
                     │  │             │  │ Observe → Think → Act │  │
                     │  │ Full policy │  │   ↓                   │  │
                     │  │ audit       │  │ Observe → Think → Act │  │
                     │  │ Per diem    │  │   ↓                   │  │
                     │  │ Limits      │  │ ... (autonomous loop  │  │
                     │  │ Categories  │  │  until confident)     │  │
                     │  │             │  │   ↓                   │  │
                     │  │             │  │ Fraud assessment      │  │
                     │  │             │  │ + evidence chain      │  │
                     │  └──────┬──────┘  └──────────┬────────────┘  │
                     │         └─────────┬──────────┘               │
                     └──────────────────-┼──────────────────────────┘
                                        ↓
                     ┌──────────────────────────────────────┐
                     │     ROUTER AGENT                     │
                     │     Routing                          │
                     │                                      │
                     │  Aggregate compliance + fraud signals │
                     │  Compute risk score                  │
                     │  Apply routing rules                 │
                     │         ↓                            │
                     │  ┌──────┼──────────┐                │
                     │  ↓      ↓          ↓                │
                     │ LOW    MEDIUM     HIGH               │
                     │ RISK   RISK       RISK              │
                     └──┬──────┬──────────┬────────────────┘
                        ↓      ↓          ↓
                    Auto-    Manager    Escalated
                    Approve  Review     Audit
                        |      |          |
                        |      ↓          ↓
                        |   ┌─────────────────────────┐
                        |   │  ADVISOR AGENT           │
                        |   │  Reflection              │       AUDITOR/APPROVER
                        |   │                          │       SEES:
                        |   │  Generate summary        │  →  Risk score + rationale
                        |   │      ↓                   │  →  Flagged line items
                        |   │  Self-critique against   │  →  Recommended action
                        |   │  evidence                │  →  Supporting evidence
                        |   │      ↓                   │  →  Comparison to norms
                        |   │  Produce recommendation  │
                        |   └─────────────┬────────────┘
                        |                 ↓
                        |         Human decides:
                        |         Approve / Return / Reject
                        |                 |
                        ↓                 ↓
                     ┌──────────────────────┐
                     │   PAYMENT PROCESSING  │
                     └──────────────────────┘
```

### Workflow Summary: Agents x Patterns x Phases

| Phase | Pattern | Agent(s) | Why |
|-------|---------|----------|-----|
| Data capture | Prompt Chaining | Intake | Sequential with quality gates |
| Pre-submit validation | Evaluator-Optimizer | Compliance | Loop until clean, prevent returns |
| Post-submit assessment | Parallelization | Compliance + Fraud | Independent concerns, run concurrently |
| — Compliance (within above) | Evaluator-Optimizer | Compliance | Deterministic rule checking |
| — Fraud (within above) | **ReAct** | **Fraud** | **Exploratory investigation, tool-driven** |
| Risk triage | Routing | Router | Classify and direct to correct path |
| Human-assisted review | Reflection | Advisor | Self-checked recommendation for human |
| Overall system | Orchestrator-Workers | All 5 | Deterministic flow, intelligent workers |

---

## Pattern-to-Phase Mapping

### Phase 1: Data Capture — Prompt Chaining

**Agent**: Intake Agent
**Why this pattern**: Receipt processing is inherently sequential with quality gates between steps. Each step depends on the previous output, and a failed gate (e.g., unreadable receipt) should halt the chain early rather than propagate garbage downstream.

```
OCR extract → [is readable?] → Categorize → [category valid?] → Itemize → [amounts balance?] → Enrich
```

Each link in the chain is a focused LLM call with a clear input/output contract. The gates between steps catch errors before they compound.

**Intake Agent responsibilities**:
- OCR / intelligent document processing on receipt images
- Extract structured data: date, amount, currency, vendor, line items
- Categorize expense type from merchant and receipt content
- Itemize complex receipts (hotel folios, restaurant bills with attendees)
- Convert foreign currency using date-of-transaction exchange rates
- Merge with corporate card feed data when available

### Phase 2: Pre-Submission Validation — Evaluator-Optimizer

**Agent**: Compliance Agent (employee-facing mode)
**Why this pattern**: This is the loop that eliminates the most painful part of the current process — submitting a report only to have it returned days later. The Compliance Agent evaluates the report against policy, surfaces specific issues with specific fix suggestions, the employee corrects, and it re-evaluates. This loops until the report is clean or the employee explicitly overrides with justification.

```
Compliance Agent evaluates → issues? → YES → show fixes → employee corrects → re-evaluate
                                      → NO  → ready to submit
```

This pattern directly addresses the top employee pain point (surprise rejections after submission).

**Compliance Agent responsibilities (pre-submit mode)**:
- Validate each line item against company expense policy
- Check receipt requirements (threshold-based)
- Verify per diem limits for location and date
- Flag missing fields or documentation
- Provide specific, actionable fix suggestions (not just "policy violation")
- Allow override with justification for soft policy warnings

### Phase 3: Post-Submission Assessment — Parallelization (Sectioning)

**Agents**: Compliance Agent (full audit mode) + Fraud Agent
**Why this pattern**: Policy compliance and fraud detection are independent concerns that don't depend on each other's output. Running them in parallel cuts processing time in half. They operate on the same input (the submitted report) but look for completely different things.

**Compliance Agent responsibilities (full audit mode)** — Evaluator-Optimizer internally:
- Deep policy audit (beyond pre-submission — checks cross-report patterns, budget impact, approval thresholds)
- Verify expense coding against cost centers
- Check cumulative spend against department budgets
- Flag expenses that require additional documentation or justification

**Fraud Agent** — **ReAct internally**:

Unlike the other agents, the Fraud Agent doesn't know in advance how many steps it needs or which tools to call. Each observation changes what it investigates next. Fraud investigation is inherently exploratory — the agent reasons about what to do, acts via a tool, observes the result, and repeats until it reaches a confident assessment.

Example investigation trace:
```
Observe: Receipt for $847 dinner, "Le Petit Bistro", Tuesday night
Think:   High amount for a weekday dinner. Let me check the merchant.
Act:     merchant_lookup("Le Petit Bistro", "Chicago, IL")
Observe: Merchant exists but is a lunch cafe, avg transaction $35
Think:   $847 doesn't match this merchant's profile. Check receipt image.
Act:     image_forensics(receipt_image)
Observe: Image metadata shows creation in Canva, not a POS system
Think:   Strong fabrication signal. Check employee history.
Act:     employee_history(employee_id=4521)
Observe: 3 claims from this same merchant in 4 months, no other employees claim it
Think:   Pattern of repeated fabrication. Sufficient evidence. Flag as high risk.
→ Done: Return structured fraud assessment with evidence chain
```

Tool belt:
- `image_forensics(receipt_image)` — authenticity analysis, manipulation detection
- `merchant_lookup(name, location)` — verify merchant exists, get profile and avg transaction
- `duplicate_search(amount, date, vendor, employee)` — cross-report, cross-employee duplicates
- `employee_history(employee_id)` — past claims, past flags, spending patterns
- `anomaly_check(expense, role_norms)` — compare against role/department baselines
- `metadata_inspect(file)` — C2PA provenance, EXIF data, creation tool detection

Termination conditions:
- Confident assessment reached (high/medium/low risk with evidence)
- Max investigation steps reached (budget cap to prevent runaway cost)
- No further leads to investigate

Output structure:
- `risk_level`: low | medium | high
- `confidence`: 0.0 - 1.0
- `findings`: list of {signal, evidence, tool_used}
- `investigation_trace`: full ReAct chain for audit trail

### Phase 4: Risk Triage — Routing

**Agent**: Router Agent
**Why this pattern**: This is a pure classification problem. The Router consumes the structured signals from Compliance and Fraud agents, computes a composite risk score, and routes to one of three paths. The routing logic is deterministic once the risk score is computed, but the score computation itself benefits from LLM reasoning (weighing ambiguous signals, contextual judgment).

| Risk Level | Routing Decision | Criteria |
|-----------|-----------------|----------|
| Low | Auto-approve, no human review | Fully compliant, no fraud signals, below threshold |
| Medium | Manager approval | Minor flags, within-policy but unusual, above threshold |
| High | Escalated audit | Fraud signals, policy violations, high amount, repeat offender |

**Router Agent responsibilities**:
- Aggregate structured outputs from Compliance and Fraud agents
- Compute composite risk score with weighted signals
- Apply configurable routing rules (org-specific thresholds)
- Select approval path: auto-approve / manager review / escalated audit
- Log routing decision with full rationale (audit trail)

This is where the system delivers the biggest efficiency gain — most expense reports (estimated 60-70%) are low-risk and can be auto-approved.

### Phase 5: Human-Assisted Review — Reflection

**Agent**: Advisor Agent
**Why this pattern**: The Advisor doesn't make the final decision — the human does. Its job is to generate a recommendation, then self-critique that recommendation against the evidence before presenting it. This prevents the agent from making superficial or biased recommendations.

```
Generate recommendation → Critique: "Does the evidence actually support this?" → Revise if needed → Present to human
```

The auditor/approver sees:
- **1-paragraph summary** of the report
- **Risk score** with plain-language rationale
- **Flagged items** with specific concerns and evidence
- **Recommended action** (approve/return/reject) with confidence level
- **Comparison to norms** (how this compares to typical claims from similar role/department)

**Advisor Agent responsibilities**:
- Generate concise report summary for approver
- Highlight flagged items with evidence and context
- Produce recommendation with confidence score
- Self-critique: verify recommendation is supported by evidence
- Provide comparison to organizational norms (avg spend by role, department patterns)
- Support approver's decision with structured information, not pressure

---

## Overall Orchestration: Orchestrator-Workers

The system-level pattern wrapping all 5 agents is **Orchestrator-Workers**. A lightweight orchestrator (state machine, not an LLM) manages the flow:

```
Orchestrator owns the state machine:
  DRAFT → VALIDATING → SUBMITTED → ASSESSING → ROUTING → REVIEWING → DECIDED → PROCESSING
```

The orchestrator doesn't reason — it follows the state transitions and delegates to the right agent at each phase. This keeps the system predictable and debuggable while the agents handle the intelligence.

---

## Summary Table

| Phase | Pattern | Agent(s) | Why |
|-------|---------|----------|-----|
| Data capture | Prompt Chaining | Intake | Sequential with quality gates |
| Pre-submit validation | Evaluator-Optimizer | Compliance | Loop until clean, prevent returns |
| Post-submit assessment | Parallelization | Compliance + Fraud | Independent concerns, run concurrently |
| — Compliance (within above) | Evaluator-Optimizer | Compliance | Deterministic rule checking |
| — Fraud (within above) | **ReAct** | **Fraud** | **Exploratory investigation, tool-driven** |
| Risk triage | Routing | Router | Classify and direct to correct path |
| Human-assisted review | Reflection | Advisor | Self-checked recommendation for human |
| Overall system | Orchestrator-Workers | All 5 | Deterministic flow, intelligent workers |

---

## Design Decisions & Rationale

### Why 5 agents and not fewer?
Each agent has a distinct concern, distinct tools, and a distinct user it serves. Merging agents (e.g., Compliance + Fraud) would create bloated agents that are harder to test, tune, and debug independently. Five is the minimum for clean separation of concerns.

### Why a state machine orchestrator and not an LLM orchestrator?
The workflow is well-defined and doesn't require runtime reasoning about what to do next. An LLM orchestrator would add latency, cost, and unpredictability for no benefit. The intelligence lives in the worker agents, not the flow control.

### Why Compliance Agent serves both pre-submit and post-submit?
Same policy knowledge, different depth. Pre-submit runs a fast check to prevent obvious issues. Post-submit runs a full audit including cross-report patterns. Same agent, two modes — avoids duplicating policy knowledge.

### Why ReAct for the Fraud Agent instead of a fixed checklist?
Fraud investigation is inherently open-ended. A fixed checklist (run forensics, check duplicates, check anomalies) treats every expense the same and misses contextual leads. ReAct lets the agent follow the evidence — if merchant lookup reveals something suspicious, it pivots to employee history; if image forensics shows manipulation, it digs into metadata. The investigation path depends on what it finds, not a predetermined sequence. This mirrors how a human fraud investigator actually works.

### Why Reflection for the Advisor instead of just generation?
Approvers need to trust the recommendation. A self-critiqued recommendation with explicit evidence mapping is more trustworthy than a generated opinion. The reflection step catches cases where the agent's recommendation doesn't actually follow from the evidence.

---

## Sources

- Anthropic, "Building effective agents" (2024) — workflow and agentic pattern taxonomy
- Andrew Ng, "Agentic Design Patterns" talks (2024) — reflection, tool use, planning, multi-agent patterns
- SAP Concur approval workflow configurations — informed the routing design
- AppZen AI expense audit — informed the parallel assessment and risk-based triage design
- Oversight Global Risk Model — informed the fraud detection agent design
