# Agentic Expense Claims: Complete Pipeline Design

## Objective

Redesign the expense claims workflow as a multi-agent AI system that eliminates manual data entry, prevents policy violations before submission, and auto-approves low-risk claims — reducing claim submission from 15-25 minutes to a single conversation.

## Current System & Pain Points

The current expense claims process at SUTD uses SAP Concur — a traditional form-based system where employees manually enter expense data, select categories, attach receipts, and submit for multi-level approval.

### Current Pain Points (Empirically Observed)

Through analysis of the SUTD SAP Concur User Guide, we identified critical workflow inefficiencies:

1. **High error rates in OCR-based receipt scanning**: The system frequently misreads amounts, dates, vendor names, and GST values, requiring manual correction
2. **Lengthy application times**: Average claim submission takes 15-25 minutes per expense line due to repetitive field entry
3. **Policy confusion**: Students must navigate 5 distinct claim policies with different expense type requirements (20+ GL account codes) and approval workflows
4. **Repeated claim rejections**: Common rejection causes include missing/incorrect project codes, wrong expense type selection, missing receipts, incorrect GST calculations, and improper currency conversions
5. **Multi-modal complexity**: Users must reconcile information across receipts (images), credit card statements (PDFs), flight itineraries (structured data), and policy guidelines (text)
6. **Manual allocation and cross-referencing**: No automated validation of project codes, no intelligent suggestion of expense types based on receipt content

## Success Criteria

The agentic redesign succeeds if it demonstrably addresses each pain point:

| Pain Point | Success Criterion | How the System Achieves It |
|-----------|-------------------|---------------------------|
| OCR errors | VLM extraction accuracy > manual correction rate | VLM reads receipt images with contextual understanding, not just OCR |
| Lengthy submission | Claim submission via single conversation (< 3 minutes) | Conversational interface replaces form-filling; agent asks only what it can't extract |
| Policy confusion | Zero policy-related rejections post-submission | Intake Agent validates against policy (via RAG) before submission; surfaces specific fixes |
| Repeated rejections | Pre-submission validation catches all rejectable issues | Evaluator-Optimizer loop ensures compliance before claim leaves the claimant |
| Multi-modal complexity | User uploads receipts, agent handles all reconciliation | VLM + LLM extract and cross-reference across modalities automatically |
| Manual allocation | Agent suggests project codes and expense types | RAG-powered policy lookup + LLM reasoning suggests correct codes from receipt content |

### Quantitative Targets

- **Submission time**: < 3 minutes per claim (vs. 15-25 minutes current)
- **Pre-submission rejection rate**: < 5% (vs. estimated 30-40% current)
- **Auto-approval rate**: 60-70% of claims require no human reviewer
- **VLM extraction accuracy**: > 95% on structured fields (amount, date, vendor)

---

## Agents (4)

| # | Agent | Pattern | Serves |
|---|-------|---------|--------|
| 1 | **Intake Agent** | ReAct + Evaluator-Optimizer | Claimant |
| 2 | **Compliance Agent** | Evaluator-Optimizer | System |
| 3 | **Fraud Agent** | Prompt Chaining | System |
| 4 | **Advisor Agent** | Reflection + Routing | Reviewer / System |

## Orchestration

**LangGraph** — hybrid approach:
- Deterministic graph structure for high-level flow
- LLM-powered agents at key nodes
- LLM-powered conditional edges for routing
- Shared state for inter-agent communication

## MCP Servers (all public, open-source)

| MCP Server | Purpose | Configured With |
|-----------|---------|-----------------|
| **DBHub** | Postgres access (claims, history, budgets) | DB connection string |
| **rag-mcp-server** | Policy document retrieval | Policy markdown → vectors |
| **Frankfurter MCP** | Currency conversion at transaction-date rates | Nothing — public API |
| **mcp-google-map** | Merchant location verification | API key |
| **Forensics-MCP-Server** | Receipt image forensics | TBD — needs verification |
| **mcp-email-server** | Notifications to claimant/reviewer | SMTP credentials |

## Entry Point

```
User sends message
      |
  [role from login?]  <- Deterministic routing
      |
  +---+--------------+
  v   v              v
CLAIMANT  REVIEWER   CLAIMANT
new claim  view queue  check status
```

| Flow | Route |
|------|-------|
| Claimant: new claim | -> Claim Intake Subgraph |
| Claimant: check status | -> DBHub lookup -> return status |
| Reviewer: view queue | -> DBHub query -> present inbox |

---

## Phase 1: Claim Intake Subgraph

**Agent**: Intake Agent
**User**: Claimant

```
User message arrives
       |
  [has image?]  <- Deterministic routing (no LLM)
   /        \
  YES        NO
   |          |
  VLM         |
  extraction  |
   |          |
  Merge into  |
  state       |
   \         /
    v       v
  [first message?]  <- Deterministic check
   /           \
  YES           NO
   |             |
  RAG policy     |
  loading        |
   |             |
  Merge into     |
  state          |
   \            /
    v          v
  Intake Agent (ReAct conversation loop)
       |
  [claim complete + policy compliant?]  <- LLM conditional edge
   /           \
  NO            YES
   |             |
  Continue       Present summary to user
  conversation   "Ready to submit?"
  (loop back)         |
                 User confirms
                      |
                 DBHub MCP -> persist to Postgres -> claim_id
                      |
                 Return claim_id to user
                      |
                 State: SUBMITTED -> next phase
```

| Step | Pattern | Model/Tool | What happens |
|------|---------|-----------|--------------|
| Image detection | **Routing** (deterministic) | Content-type check | Branch on image presence |
| Receipt extraction | **Prompt Chaining** | VLM | Image -> structured fields |
| Policy loading | **Prompt Chaining** | rag-mcp-server | One-time, first message only |
| Conversation | **ReAct** | LLM | Gather info, clarify, ask questions |
| Validation | **Evaluator-Optimizer** | LLM + rag-mcp-server | Check against policy, loop until clean |
| Submission | — | DBHub MCP | Persist claim, return claim_id |

---

## Phase 2: Post-Submission Assessment

**Pattern**: Parallelization (Sectioning) — two independent tracks

```
Submitted claim
       |
  +----+----+
  v         v
TRACK 1   TRACK 2
  v         v
Compliance  Fraud
Agent       Agent
  v         v
  +----+----+
       v
  Both assessments in LangGraph state
       v
  Advisor Agent
```

### Track 1: Compliance Agent

**Pattern**: Evaluator-Optimizer
**Model**: LLM

| Action | Tool | What happens |
|--------|------|--------------|
| Read claim data | **DBHub** | Fetch submitted claim |
| Read policy rules | **rag-mcp-server** | Query policy KB |
| Check past claims | **DBHub** | Cross-report patterns |
| Check department budget | **DBHub** | Cumulative spend |
| Currency conversion | **Frankfurter MCP** | Foreign currency at transaction-date rate |
| Evaluate compliance | LLM reasoning | Apply rules, iterate if ambiguous |

**Output**: Structured compliance assessment (pass/fail per rule, flags, severity)

### Track 2: Fraud Agent

**Pattern**: Prompt Chaining
**Model**: LLM

| Action | Tool | What happens |
|--------|------|--------------|
| Image forensics | **Forensics-MCP-Server** | Receipt authenticity analysis |
| Interpret results | LLM reasoning | Assess forensics output |

**Output**: Structured fraud assessment (risk level, confidence, findings)

---

## Phase 3: Advisor Agent

**Pattern**: Reflection + Routing
**Model**: LLM

```
Read both assessments from state
       |
  Generate briefing + risk score
       |
  Self-critique: "Does evidence support this?"
       |
  Revise if needed
       |
  [risk level?]  <- LLM routing
       |
  +----+---------+
  v    v         v
 LOW  MEDIUM    HIGH
  |    |         |
Auto-  Present   Present to
approve to       reviewer with
  |    reviewer  escalation flag
  |    |         |
  v    v         v
```

| Action | Tool | What happens |
|--------|------|--------------|
| Fetch claim + assessments | **DBHub** | Read from Postgres |
| Fetch department norms | **DBHub** | Historical averages for comparison |
| Policy citations | **rag-mcp-server** | Cite specific clauses in briefing |
| Generate + self-critique | LLM reasoning | Reflection pattern |
| Persist decision (auto-approve) | **DBHub** | Update claim status |
| Notify claimant (auto-approve) | **mcp-email-server** | "Claim approved" |
| Notify reviewer (escalated) | **mcp-email-server** | "Claim requires review" |

**Reviewer sees (medium/high only):**
- Claim summary (line items, totals, category)
- Risk score with plain-language rationale
- Flagged items with evidence
- Recommended action with confidence
- Comparison to department norms

---

## Phase 4: Reviewer Decision

**User**: Reviewer (via chat inbox)

```
Reviewer reads briefing
       |
  Decides: approve / return / reject
       |
  +----+---------+
  v    v         v
APPROVE RETURN  REJECT
  |     |         |
  |   Message to  |
  |   claimant's  |
  |   thread with |
  |   comments -> |
  |   resubmit -> |
  |   back to     |
  |   Phase 2     |
  |               |
  v               v
Claim approved  Claim closed
DBHub + email   DBHub + email
```

| Action | Tool |
|--------|------|
| Persist decision | **DBHub** |
| Notify claimant | **mcp-email-server** |

---

## Summary: Patterns x Agents x Phases

| Phase | Agent | Pattern | MCP Tools |
|-------|-------|---------|-----------|
| Entry routing | — | **Routing** (deterministic) | — |
| Claim intake: image processing | Intake | **Routing** (deterministic) + **Prompt Chaining** | VLM |
| Claim intake: policy loading | Intake | **Prompt Chaining** | rag-mcp-server |
| Claim intake: conversation | Intake | **ReAct** | DBHub |
| Claim intake: validation | Intake | **Evaluator-Optimizer** | rag-mcp-server |
| Claim intake: submission | Intake | — | DBHub |
| Post-submit: parallel assessment | Compliance + Fraud | **Parallelization** | — |
| Post-submit: compliance | Compliance | **Evaluator-Optimizer** | DBHub, rag-mcp-server, Frankfurter |
| Post-submit: fraud | Fraud | **Prompt Chaining** | Forensics-MCP-Server |
| Risk scoring + routing | Advisor | **Reflection + Routing** | DBHub, rag-mcp-server, mcp-email-server |
| Reviewer decision | — (human) | — | DBHub, mcp-email-server |

## Tech Stack

| Component | Technology |
|-----------|-----------|
| Agent orchestration | **LangGraph** |
| Chat UI | **Chainlit** |
| Database | **Postgres** (via DBHub MCP) |
| Policy KB | **rag-mcp-server** (markdown -> vectors) |
| VLM | TBD |
| LLM | TBD |

---

## Sample Interactions

### Scenario 1: Claimant Submits a Dinner Expense

```
CLAIMANT: Hi, I need to submit an expense from last night's client dinner
          [attaches receipt_dinner.jpg]

-- behind the scenes -----------------------------------------------
   Router: role=claimant, intent=new_claim -> Claim Intake Subgraph
   Image detected -> VLM extracts:
     vendor: "Osteria Mozza"
     date: 2026-02-25
     amount: $312.40
     currency: USD
     items: [food $248, wine $52, tax $12.40]
   RAG loads: entertainment expense policy, per diem rules (LA, Feb 2026)
-----------------------------------------------------------------

INTAKE AGENT: I've extracted the details from your receipt:

   Osteria Mozza - Feb 25, 2026
   Food:  $248.00
   Wine:   $52.00
   Tax:    $12.40
   Total: $312.40

   A couple of questions:
   1. What was the business purpose of this dinner?
   2. How many attendees, and who were they?

CLAIMANT: Q1 pipeline review with Acme Corp. 4 people - me, Sarah Chen,
          and two clients: Tom Brooks and Lisa Park

-- behind the scenes -----------------------------------------------
   Agent updates claim in state:
     purpose: "Q1 pipeline review with Acme Corp"
     attendees: 4 (2 internal, 2 external)
     per_person: $78.10
-----------------------------------------------------------------

INTAKE AGENT: Got it. One flag before we submit:

   ! Policy note: Alcohol ($52.00) on client entertainment
   requires VP pre-approval for amounts over $50. Do you
   have approval, or would you like to remove the wine?

CLAIMANT: Yes, my VP Diana approved it verbally before the dinner

-- behind the scenes -----------------------------------------------
   Agent records override:
     rule: alcohol_limit_client_entertainment
     justification: "VP Diana pre-approved verbally"
   Re-evaluates -> no remaining issues
-----------------------------------------------------------------

INTAKE AGENT: Thanks. Here's your claim summary:

   Client Entertainment - Osteria Mozza
   Date: Feb 25, 2026
   Total: $312.40 ($78.10/person, 4 attendees)
   Purpose: Q1 pipeline review with Acme Corp
   Attendees: You, Sarah Chen, Tom Brooks (Acme), Lisa Park (Acme)
   Note: Alcohol approved by VP Diana

   Ready to submit?

CLAIMANT: yes

-- behind the scenes -----------------------------------------------
   Agent -> DBHub MCP: INSERT claim -> returns claim_id EC-20260226-0087
   State updated: status=SUBMITTED
-----------------------------------------------------------------

INTAKE AGENT: Submitted! Your claim ID is EC-20260226-0087.
              You'll be notified once it's processed.

-- behind the scenes -----------------------------------------------
   State transitions to Post-Submission Assessment
   Parallelization:
     Track 1 - Compliance Agent: deep audit, cross-report check,
               budget check -> PASS (minor flag: alcohol override)
     Track 2 - Fraud Agent: Forensics MCP on receipt image -> PASS
   Advisor Agent:
     Risk score: MEDIUM (alcohol override + amount above $300)
     Self-critique: evidence supports medium - override is documented
                    but verbal approvals are weaker evidence
     Route: escalate to reviewer
     -> DBHub: update status=PENDING_REVIEW
     -> mcp-email-server: notify reviewer
-----------------------------------------------------------------
```

### Scenario 2: Reviewer Approves a Flagged Claim

```
-- behind the scenes -----------------------------------------------
   Reviewer logs in -> Router: role=reviewer -> view queue
   DBHub query: claims WHERE status=PENDING_REVIEW
   Returns: 3 claims in queue
-----------------------------------------------------------------

REVIEWER: Show me my pending claims

SYSTEM: You have 3 claims pending review:

   1. EC-20260226-0087 - Sarah's team, $312.40 dinner
      Risk: MEDIUM - Alcohol override, above $300 threshold
   2. EC-20260225-0063 - Dev team, $1,240 team offsite
      Risk: MEDIUM - Above department threshold
   3. EC-20260224-0051 - James Liu, $89.50 taxi
      Risk: MEDIUM - Unusual route flagged

REVIEWER: Open the first one

-- behind the scenes -----------------------------------------------
   Advisor Agent generates briefing for EC-20260226-0087
   Fetches: claim data, assessments, dept norms (DBHub)
   Policy citations (rag-mcp-server)
   Reflection: generate -> self-critique -> finalize
-----------------------------------------------------------------

ADVISOR AGENT:

   Claim EC-20260226-0087
   ----------------------
   Claimant: Sagar Singh
   Type: Client Entertainment
   Amount: $312.40 | Date: Feb 25, 2026
   Vendor: Osteria Mozza, Los Angeles

   Line Items:
     Food    $248.00
     Wine     $52.00
     Tax      $12.40

   Risk Score: MEDIUM (0.45)

   Flags:
   1. Alcohol $52.00 exceeds $50 client entertainment limit
      - Claimant states VP Diana approved verbally
      - No written pre-approval on file
   2. Total $312.40 above auto-approval threshold ($300)

   Compliance: PASS with noted override
   Fraud: PASS - receipt authentic (forensics verified)

   Comparison to norms:
     - Avg client dinner (Sales, LA): $275/event
     - This claim: $312.40 - 14% above average
     - Per-person $78.10 - within normal range

   Recommendation: APPROVE (confidence: 0.75)
   Rationale: Amount is slightly above average but per-person
   cost is normal for 4 attendees in LA. Alcohol override
   is the only concern - suggest confirming with VP Diana.

   What would you like to do?
   -> Approve | Return for changes | Reject

REVIEWER: approve, but remind the team that alcohol pre-approvals
          need to be in writing

-- behind the scenes -----------------------------------------------
   -> DBHub: UPDATE claim status=APPROVED,
            reviewer_note="Approved. Reminder: alcohol
            pre-approvals must be in writing."
   -> mcp-email-server: notify claimant
     "Claim EC-20260226-0087 approved. Note from reviewer:
      alcohol pre-approvals must be in writing for future claims."
-----------------------------------------------------------------

ADVISOR AGENT: Done. Claim EC-20260226-0087 approved.
               Claimant has been notified with your note.

               2 claims remaining in your queue. Next?
```

---

## Sources

- Anthropic, "Building effective agents" (2024) — workflow and agentic pattern taxonomy
- Andrew Ng, "Agentic Design Patterns" talks (2024) — reflection, tool use, planning, multi-agent patterns
- SUTD SAP Concur User Guide — informed pain points and current workflow analysis
- SAP Concur approval workflow configurations — informed the routing design
