# Deep Dive: SAP Concur & Expense Claims Process

## Strategic Summary

SAP Concur is the dominant enterprise expense management platform, but it suffers from a dated UX, rigid workflows, and painful implementation. The expense claims lifecycle involves two primary personas — the **employee claimant** (capture, categorize, submit, wait) and the **auditor/approver** (review, verify compliance, approve/reject, process for payment). AI is rapidly transforming every step of this lifecycle, from OCR receipt capture to real-time policy enforcement to AI-vs-AI fraud detection, creating significant opportunity for an agentic system that automates the tedious middle while keeping humans in the loop for judgment calls.

## Key Questions This Research Answers

- What does the end-to-end expense claims process look like from the employee's perspective?
- What does the auditor/verifier/approver workflow look like, and what are the different approval configurations?
- Where are the biggest pain points in today's process?
- Where can AI agents add the most value?

---

## Overview

Corporate expense management follows a universal pattern: employees spend money on behalf of the company, document it, submit it for approval, and get reimbursed. What varies is the complexity of policy enforcement, the number of approval layers, and the degree of automation.

SAP Concur dominates this space in the enterprise segment, processing expense reports for thousands of organizations globally. However, its enterprise-grade power comes with enterprise-grade friction — dated UI, slow performance, complex configuration, and workflows designed for Fortune 500 scale that feel burdensome for smaller organizations.

The market is shifting rapidly. Competitors like Ramp, Brex, and Fyle offer simpler, AI-native alternatives. Meanwhile, specialized AI auditing platforms like AppZen and Oversight are demonstrating that agents can automate 50%+ of manual audit work while catching fraud that humans miss.

---

## The Expense Claims Process: Employee Journey

### Step-by-Step Workflow

```
Incur Expense  -->  Capture Receipt  -->  Create Report  -->  Add Line Items  -->  Submit  -->  Wait  -->  Get Reimbursed
     |                    |                    |                    |               |          |
  (spend money)    (photo/email/       (header info:         (categorize,      (to mgr/    (corrections
                    manual entry)       purpose, dates,       add details,       processor)  if returned)
                                        cost center)          attach receipts)
```

#### 1. Incur Expense
Employee spends money during business travel or for business purposes. Common categories:
- **Transportation**: airfare, train, taxi/rideshare, car rental, mileage
- **Lodging**: hotel stays (often with nightly rate limits)
- **Meals**: per diem or actuals (with limits per meal type)
- **Ground transport**: parking, tolls, fuel
- **Miscellaneous**: conference fees, supplies, tips, internet/phone charges

#### 2. Capture Receipt
Multiple capture methods exist:
- **Mobile photo**: snap receipt with phone camera, OCR extracts data
- **Email forwarding**: forward e-receipts to a dedicated email address
- **Credit card feed**: corporate card transactions auto-populate
- **Manual entry**: type in details when no receipt exists (below threshold)
- **ExpenseIt / AI assistant**: automatic extraction with categorization

**Pain point**: OCR accuracy varies. Employees often must manually correct extracted data. Handwritten receipts, foreign-language receipts, and itemized meal bills are particularly problematic.

#### 3. Create Expense Report
Employee creates a new report (or "claim") with header information:
- **Report name** (e.g., "NYC Client Meeting Feb 2026")
- **Business purpose**
- **Date range**
- **Cost center / project code** (often auto-populated from employee profile)
- **Country/region** (affects per diem rates and tax rules)

#### 4. Add Line Items
For each expense within the report:
- Select **expense type** (meal, transport, lodging, etc.)
- Enter **date, amount, currency**
- Add **vendor/merchant** name
- Provide **business justification** (who attended, why)
- **Attach receipt** image
- **Itemize** if required (e.g., hotel bills must show room, tax, other charges separately)
- Flag if **personal expenses** are mixed in (e.g., personal items on hotel bill)

**Pain points**:
- Too many clicks per line item
- Categorization confusion (is a work dinner "meals" or "entertainment"?)
- Itemization is tedious, especially for hotel folios
- Foreign currency conversion complexity
- Policy rules are often unclear until submission triggers an alert

#### 5. Submit Report
Employee submits the report for approval. At this point:
- Automated **policy checks** run (per diem limits, receipt requirements, category rules)
- **Warnings/errors** flag violations before submission (soft warnings can be overridden with justification; hard blocks require correction)
- Employee may need to add **exception justifications** for flagged items
- Report enters the **approval workflow**

#### 6. Wait for Approval
The employee's report enters a queue. During this phase:
- They can track status (pending, approved, returned, processing)
- Reports may be **returned for correction** (missing receipt, exceeded limit, wrong category)
- Returned reports require re-submission, restarting parts of the approval chain
- Multiple round-trips are common for complex reports

**Pain point**: Slow approval cycles. Some organizations take weeks. Employees front the money on personal cards and wait for reimbursement, creating financial pressure.

#### 7. Reimbursement
After final approval and processing:
- Finance processes payment (typically monthly batch)
- Direct deposit to employee bank account
- Or credit card statement credit for corporate card expenses

---

## The Expense Claims Process: Auditor / Verifier / Approver Journey

### Roles in the Approval Chain

| Role | Responsibility | Typical Position |
|------|---------------|-----------------|
| **Manager/Approver** | Validates business purpose, reasonableness, budget alignment | Direct manager or cost center owner |
| **Second Approver** | Additional sign-off for high-value or cross-department expenses | Senior manager, department head |
| **Authorized Approver** | Approval authority based on dollar limits | Finance director, VP |
| **Processor/Auditor** | Verifies compliance, receipts, policy adherence, fraud checks | AP team member, finance analyst |
| **Finance Controller** | Final oversight, batch payment authorization | Controller, CFO |

### SAP Concur Approval Workflow Configurations

SAP Concur offers five default workflow patterns:

#### 1. Direct to Processor
```
Employee --> Submit --> Processor --> Payment
```
- No manager approval needed
- Suited for small businesses or low-value expenses
- Processor handles all verification

#### 2. Manager then Processor (most common)
```
Employee --> Submit --> Manager Approval --> Processor --> Payment
```
- Manager validates business purpose and reasonableness
- Processor verifies compliance and receipts
- Standard for most organizations

#### 3. Manager then Authorized Approver then Processor (limit-based)
```
Employee --> Submit --> Manager Approval --> [If over limit] --> Authorized Approver --> Processor --> Payment
                                          --> [If under limit] --> Processor --> Payment
```
- Manager has an approval limit (e.g., $5,000)
- Expenses exceeding the limit escalate to an authorized approver
- Authorized approvers can be tiered (different limits at different levels)

#### 4. Manager then Second Manager then Processor
```
Employee --> Submit --> Manager 1 --> Manager 2 --> Processor --> Payment
```
- Dual manager sign-off
- Common for cross-departmental charges
- Both managers must approve before processing

#### 5. Processor Audit then Manager then Payment
```
Employee --> Submit --> Processor Audit --> Manager Approval --> Payment
```
- Processor reviews FIRST, before manager sees it
- Ensures managers only see "clean" reports
- Reduces back-and-forth (processor catches errors early)
- Used when AP team has better compliance knowledge than managers

### What Each Role Does

#### Manager/Approver Workflow
1. **Receive notification** that reports are awaiting approval
2. **Review summary**: total amount, number of expenses, date range
3. **Scan line items**: check business purpose, attendees, amounts
4. **Review flagged items**: policy warnings, missing receipts, unusual amounts
5. **Decision**: Approve, Send Back (with comments), or Reject
6. **Optional**: Add additional approver to chain (if permitted by org config)

**What managers check**:
- Was this expense legitimate business activity?
- Is the amount reasonable for what was purchased?
- Does it align with the project/budget?
- Are there any obviously personal expenses mixed in?

**Pain point**: Manager approval is often a rubber stamp. Managers lack context to catch policy violations and just want reports out of their queue.

#### Processor/Auditor Workflow
1. **Queue management**: Work through submitted reports in order (or by priority/amount)
2. **Receipt verification**: Every line item has an attached receipt matching the claimed amount
3. **Policy compliance**: Check against configured rules (per diem limits, approved vendors, category rules, receipt thresholds)
4. **Itemization review**: Hotel folios properly broken out, meal receipts show attendees
5. **Duplicate detection**: Check for same expense submitted across multiple reports or by multiple employees
6. **Flag exceptions**: Send back items that need correction or additional documentation
7. **Approve for payment**: Mark report as ready for payment batch
8. **Segregation of duties**: Processors cannot process their own reports (fraud safeguard)

**What processors/auditors check**:
- Does the receipt match the claimed amount, date, and vendor?
- Is the expense within policy limits?
- Are all required fields completed?
- Any duplicate submissions?
- Any signs of manipulation or fraud?
- Proper categorization and coding?

**Pain point**: High-volume manual review is tedious. Most items are routine and compliant, but every item gets the same level of scrutiny. No risk-based prioritization.

### Audit Rules Engine

SAP Concur supports configurable audit rules that automatically flag expenses:
- **Amount thresholds**: Flag expenses over $X
- **Category rules**: Require receipts for specific expense types
- **Duplicate detection**: Same amount, same date, same vendor
- **Per diem enforcement**: Auto-calculate allowed amounts based on location
- **Missing data alerts**: Required fields not completed
- **Custom rules**: Organization-specific policies

These rules generate warnings (soft) or errors (hard blocks) at submission time and during processor review.

---

## Pain Points Summary

### Employee Pain Points
| Pain Point | Impact | Agentic Opportunity |
|-----------|--------|-------------------|
| Too many clicks per line item | Time waste, frustration | Auto-populate from receipt data |
| Confusing expense categories | Miscategorization, returns | AI categorization from receipt content |
| Manual itemization (hotel folios) | Tedious, error-prone | Intelligent receipt parsing |
| Foreign currency handling | Errors, manual lookup | Auto-conversion with date-accurate rates |
| Unclear policy rules until violation | Surprise rejections | Real-time policy guidance as expenses are entered |
| Slow reimbursement cycles | Financial pressure | Automated approval fast-tracking for compliant claims |
| Reports returned for corrections | Re-work, re-approval | Pre-submission validation agent catches issues early |
| Lost/damaged receipts | Delays, rejected claims | Digital-first capture, backup from card feeds |

### Auditor/Approver Pain Points
| Pain Point | Impact | Agentic Opportunity |
|-----------|--------|-------------------|
| High-volume manual review | Burnout, missed fraud | Risk-based triage (auto-approve low-risk, flag high-risk) |
| Rubber-stamp approvals | Compliance gaps | AI summary + risk score for each report |
| Duplicate detection across reports | Manual cross-referencing | Automated cross-report, cross-employee duplicate scanning |
| Receipt-to-claim matching | Tedious verification | AI receipt matching with confidence scores |
| Policy rule interpretation | Inconsistent enforcement | Codified policy engine with clear pass/fail |
| Fraud detection (esp. AI-generated receipts) | Financial loss | Multi-layer fraud detection (image forensics, metadata, pattern analysis) |
| Batch payment errors | Financial risk | Validated payment staging with anomaly detection |
| Context switching between reports | Cognitive load | Intelligent queue prioritization and report summarization |

---

## Agentic Automation Opportunities

### For the Employee (Claimant) Side

| Agent Capability | What It Does | Value |
|-----------------|-------------|-------|
| **Receipt Intelligence Agent** | OCR + NLP to extract all data from any receipt (42+ languages, handwritten, itemized) | Eliminates manual data entry |
| **Auto-Categorization Agent** | Classifies expenses into correct categories based on merchant, amount, and context | Reduces miscategorization returns |
| **Policy Guardian Agent** | Real-time guidance as expenses are entered — warns before submission, not after | Prevents returns, speeds approval |
| **Itemization Agent** | Automatically breaks down hotel folios, restaurant bills into required line items | Eliminates tedious manual itemization |
| **Currency Agent** | Auto-converts foreign currency with date-accurate exchange rates | Eliminates manual lookup errors |
| **Report Assembly Agent** | Groups expenses into logical reports, suggests business purpose, fills header fields | Reduces report creation time |
| **Pre-Submission Validator** | Runs full policy check before employee hits submit, suggests fixes | Catches 90%+ of issues that cause returns |

### For the Auditor/Approver Side

| Agent Capability | What It Does | Value |
|-----------------|-------------|-------|
| **Risk Scoring Agent** | Assigns risk score to each report based on amount, patterns, policy compliance | Enables risk-based triage |
| **Auto-Approve Agent** | Automatically approves low-risk, fully-compliant reports without human review | Frees auditors for high-value work |
| **Fraud Detection Agent** | Multi-layer analysis: image forensics, metadata, duplicate detection, pattern matching, AI-generated receipt detection | Catches fraud humans miss |
| **Receipt Verification Agent** | Matches receipt data to claimed amounts, flags mismatches | Eliminates manual receipt checking |
| **Report Summary Agent** | Generates concise summary with flagged items highlighted for approver review | Reduces review time per report |
| **Policy Compliance Agent** | Automated pass/fail against all policy rules with clear explanations | Consistent enforcement |
| **Duplicate Detection Agent** | Cross-report, cross-employee, cross-period duplicate scanning | Catches split/duplicate submissions |
| **Spending Analytics Agent** | Identifies patterns, outliers, vendor concentration, budget impact | Proactive cost management |

---

## Current State & Trends

### SAP Concur (2025-2026)
- Rolling out a **new expense report UI** to address long-standing UX complaints
- AI-powered receipt capture agent with improved categorization and itemization
- **Concur Complete**: unified travel + expense + payments platform (partnership with Amex GBT)
- Direct integrations with finance platforms (QuickBooks, NetSuite, Xero) for professional edition

### Market Movement
- **AppZen**: AI audits 100% of expenses, auto-approves 50%+ of low-risk spend, detects AI-generated receipts
- **Oversight**: Global risk model trained on millions of transactions, receipt image forensics
- **Ramp/Brex**: Corporate card + expense in one, real-time policy enforcement, AI categorization
- **Rydoo**: Smart Audit uses AI to fight AI-generated receipt fraud
- **Fyle**: Real-time expense tracking from credit card feeds, no manual report creation needed

### AI-Generated Receipt Fraud
This is the emerging arms race in expense management:
- 14% of detected fake receipts were AI-generated (AppZen, Sep 2025)
- 32% of finance professionals cannot recognize AI fakes (Medius survey)
- 70% of CFOs suspect employees may attempt receipt fabrication (SAP)
- Detection requires multi-layer analysis: image forensics, C2PA metadata, cross-reference with merchant databases, pattern analysis

---

## Key Takeaways

1. **The expense claims process has two distinct user journeys with different needs**: employees want speed and simplicity (capture, submit, get paid); auditors want accuracy and compliance (verify, flag, approve). An agentic system must serve both.

2. **The biggest automation opportunity is the "boring middle"**: the tedious work of data entry, categorization, itemization, receipt matching, and routine policy checking. This is where agents can eliminate 70-80% of manual effort for both sides.

3. **Risk-based triage is the key architectural insight**: instead of treating every expense report equally, an agentic system should auto-approve compliant low-risk claims instantly while surfacing high-risk claims with AI-generated risk summaries for human review. This is what AppZen and Oversight have proven works.

4. **AI-generated receipt fraud is a real and growing threat** that any modern expense system must address. This is a differentiating capability — multi-layer detection combining image forensics, metadata analysis, and pattern matching.

5. **The approval workflow must be configurable** — organizations need different approval chains (direct-to-processor, manager-first, limit-based escalation, dual-approval). SAP Concur offers 5 default patterns; our system should support similar flexibility.

---

## Remaining Unknowns

- [ ] What specific per diem databases / rate sources should the system use? (GSA rates for US, varies by country)
- [ ] What corporate card feed formats and integrations are standard? (OFX, CSV, direct API)
- [ ] What are the specific regulatory requirements by region? (tax reclaim, VAT, FCPA)
- [ ] What ERP/accounting system integrations are most critical? (SAP, NetSuite, QuickBooks, Xero)
- [ ] What is the typical report volume per organization size? (impacts architecture decisions)
- [ ] How do organizations currently handle mileage claims specifically? (GPS tracking, manual, fixed rates)

---

## Implementation Context

### Application
- **When to use**: Building expense management systems, automating financial workflows, designing approval pipelines
- **When not to use**: One-off personal expense tracking, invoice-only processing (different domain)
- **Prerequisites**: Understanding of corporate accounting codes, familiarity with policy rule engines, receipt OCR capabilities

### Technical
- **Key capabilities needed**: OCR/IDP for receipts, configurable workflow engine, rule-based policy enforcement, ML-based risk scoring, image forensics for fraud detection
- **Patterns**: Event-driven architecture (expense submitted -> policy check -> route to approver), CQRS for read/write separation on reports, saga pattern for multi-step approval workflows
- **Gotchas**: Currency conversion must use date-of-transaction rates not current rates; receipt images must be stored for audit trail (7+ years); segregation of duties is a hard requirement (users cannot approve their own expenses)

### Integration
- **Works with**: Corporate card feeds, ERP/accounting systems, HR systems (org hierarchy for approval routing), tax/per diem rate databases
- **Conflicts with**: Manual paper-based processes (requires digital transformation)
- **Alternatives to study**: Ramp, Brex, Fyle (card-first models), AppZen (audit-first model), Oversight (risk-first model)

**Next Action:** Use this research to inform `/sagerstack:code-planning` for the agentic expense claims system.

---

## Sources

- [SAP Concur Training Portal](https://www.concurtraining.com/toolkit/en/expense/end-user) - Employee end-user toolkit
- [SAP Learning Journey: Creating Expense Reports](https://learning.sap.com/learning-journeys/getting-started-with-concur-expense-learning-journey-for-customer-users/) - Step-by-step claim creation
- [SAP Concur: Expense Report Approval Overview](https://help.sap.com/docs/CONCUR_EXPENSE/93ceb32335c6486d902426c6727b80f2/c4030de651c31015a2ffca6cced57d86.html) - Approval workflow documentation
- [SAP Concur: Approval Routing](https://help.sap.com/docs/CONCUR_EXPENSE/1f13d54352684d6dba6e65c8c5d75ead/c45c02b751c31015a7e0dfe293ca0e68.html) - Routing configuration options
- [SAP Concur: Configuring Expense Approvals](https://learning.sap.com/learning-journeys/getting-started-with-concur-expense-standard-for-administrators/configuring-expense-approvals) - Admin approval setup
- [SAP Concur Community: Approval Workflows](https://community.concur.com/t5/Concur-Expense-Forum/Concur-Approval-Workflows/m-p/4321) - Community workflow discussion
- [SAP Concur Community: Processing Reports](https://learning.sap.com/learning-journeys/getting-started-with-concur-expense-standard-for-administrators/exploring-the-basics-of-processing-reports) - Processor role basics
- [SAP Concur 2025 Innovation Recap](https://www.concur.com/blog/article/2025-innovation-recap-optimizing-sap-concur-solutions-for-travel-and-expense) - AI and platform updates
- [SAP Concur Community: "Most horrible travel expense application"](https://community.concur.com/t5/Concur-Expense-Forum/Most-horrible-travel-expense-application-ever/m-p/43187) - User complaints
- [Capterra: SAP Concur Reviews 2026](https://www.capterra.com/p/380/Concur-Expense/reviews/) - Verified user reviews
- [Gartner: Concur Expense Reviews 2026](https://www.gartner.com/reviews/product/concur-expense) - Peer insights
- [Ramp vs SAP Concur Comparison](https://www.fylehq.com/blog/ramp-vs-concur) - Competitive analysis
- [Qentelli: 5 Common SAP Concur Mistakes](https://qentelli.com/thought-leadership/insights/5-common-sap-concur-mistakes-and-how-to-avoid-them) - Implementation pitfalls
- [SAP Concur Alternatives: Expense Tracker 365](https://www.apps365.com/blog/sap-concur-alternatives/) - Alternative solutions
- [New Concur Expense Report UI](https://community.concur.com/t5/What-s-New-in-Product/Introducing-the-New-Concur-Expense-Report-UI/ba-p/103511) - UI modernization
- [AppZen: AI for Expense Audit](https://www.appzen.com/ai-for-expense-audit) - AI audit platform
- [Oversight: AI Travel & Expense Monitoring](https://www.oversight.com/ai-travel-expense-monitoring) - Risk-based monitoring
- [Ramp: How AI Transforms Expense Management](https://ramp.com/blog/ai-expense-management) - AI capabilities
- [Veryfi: AI Expense Management Automation](https://www.veryfi.com/ocr-api-platform/ai-expense-management-automation/) - OCR/IDP platform
- [Oreate AI: Expense Compliance Trends 2025](https://www.oreateai.com/blog/navigating-the-future-expense-management-compliance-automation-trends-for-2025/91c53897661c5ef31d1f83b9e3b6787b) - Compliance automation
- [Rydoo: Expense Fraud Prevention 2026](https://www.rydoo.com/cfo-corner/expense-fraud-companies/) - AI fraud detection
- [AI CERTs: Agent Fraud Risk with AI Fabricated Expenses](https://www.aicerts.ai/news/agent-fraud-risk-surges-with-ai-fabricated-expenses/) - AI-generated receipt fraud
- [ITILITE: Expense Approval Workflow](https://www.itilite.com/blog/expense-approval-workflow/) - General workflow patterns
- [Rippling: Travel Expense Reimbursement Guide](https://www.rippling.com/blog/travel-expense-reimbursement) - Employer guide
- [Brex: Travel Expense Reimbursement](https://www.brex.com/spend-trends/corporate-travel-management/travel-expense-reimbursement-for-businesses) - Business guide
- [Ramp: Business Travel Expense Reimbursement](https://ramp.com/blog/what-is-travel-and-expense-reimbursement) - Complete guide
- [Fyle: Travel Expense Reimbursement](https://www.fylehq.com/blog/travel-expense-reimbursement) - IRS rules and tools
