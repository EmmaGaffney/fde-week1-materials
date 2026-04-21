# Agentic Returns Processing Design

## Problem Statement & Success Metrics

### Problem Statement

**Operational problem:**  
Twelve manual processors handle 4,500 returns/week with 78% reason-code accuracy and 40% fraud-detection rate. Manual classification drives misrouting to wrong disposition lanes (restock/outlet/refurbish/donate/destroy), eroding margin. 60% of fraud cases surface only in chargeback reports weeks later. All work relies on processor memory with no standardised decision framework.

**Business problem:**  
Low accuracy and late fraud detection directly impact margin. VP Operations requires agentic automation for straightforward cases while preserving human judgment for suspected fraud, high-value items, and customer escalations. CFO requires credible payback with measurable margin improvement.

**Solution requirement:**  
Deploy agentic system to automate classification and disposition routing for low-risk returns while escalating fraud-flagged, high-value, and customer-escalated cases to human review. Must improve reason-code accuracy and fraud-detection rate while freeing processor capacity for complex cases.

**Constraints (non-negotiable):**
- Must not auto-approve returns flagged as potential fraud
- Must not auto-process high-value items without human validation
- Must not auto-process customer-escalated cases
- Must maintain full audit trail for compliance and chargeback defense

---

### Success Metrics

| **Metric** | **Current State** | **Target State (12 months)** | **Measurement Method** |
|------------|-------------------|------------------------------|------------------------|
| **Reason-code accuracy** | 78% (from scenario) | ≥92% | Weekly sample audit: 200 random returns re-classified by quality team; accuracy = % match between agent classification and audit ground truth |
| **Fraud detection rate (pre-chargeback)** | 40% (from scenario) | ≥75% | Monthly: (fraud cases flagged by system ÷ total confirmed fraud cases) × 100, reconciled against chargeback reports |
| **Agent-handled returns (no human touch)** | 0% | ≥60% of total volume | Weekly: count of returns processed end-to-end by agent ÷ total returns × 100 |
| **Processing time per return** | ~18 min/return (calculation: 4,500 returns ÷ 12 processors ÷ 40 hrs/week × 60 min) | ≤8 min for agent-handled; ≤22 min for human-reviewed | Time-stamped logging: timestamp(disposition_complete) - timestamp(return_received), measured at p50 and p95 |
| **Escalation precision** | N/A | ≥80% | Monthly audit: (correctly escalated cases ÷ total escalations) × 100. "Correct" = human agrees escalation was necessary. |

**Volume baseline (from scenario):**
- 4,500 returns/week = 234,000 returns/year
- 12 processors = 375 returns/processor/week

---

## Delegation Analysis

| **Workflow Step** | **Ownership** | **Volume Target** | **Decision Criteria** | **Rationale** |
|-------------------|---------------|-------------------|----------------------|---------------|
| **Return receipt & data capture** | **AI-led** | 100% (4,500/week) | Always automated unless barcode unreadable → human assist | Structured data extraction (barcode scan, OCR, order matching). No judgment required. |
| **Reason-code classification** | **AI-led with confidence threshold** | 85% AI-classified; 15% human-reviewed | IF confidence ≥0.85 → agent classifies. IF confidence <0.85 → human review queue. | Addresses current 78% accuracy problem. Agent trained on historical data + condition photos. Low confidence = ambiguity that needs human judgment. |
| **Fraud risk scoring** | **AI-led (mandatory human review for flags)** | 95% scored; ~15% flagged for human review (assumption: ~675/week) | IF fraud_score >0.30 OR return_value >$150 OR customer_escalation_flag = TRUE → human review queue (non-negotiable). Else → agent continues. | Addresses current 40% detection rate. Agent analyzes: return frequency, high-value items, condition mismatches, serial returner patterns. Human makes all final fraud decisions. |
| **Physical condition inspection** | **Human-overseen** | 70% agent-graded with spot-checks; 30% human-inspected | IF item_value >$150 OR fraud_flag = TRUE OR CV_confidence <0.80 OR random_sample(10%) → human validates. Else → agent grade accepted. | Computer vision provides preliminary grade (new/like-new/good/fair/poor/damaged) from uploaded photos. Human validates high-risk cases and quality-control sample. |
| **Disposition routing** | **AI-led with guardrails** | 65% agent-routed; 35% human-decided | IF fraud_flag = FALSE AND condition_confidence ≥0.80 AND item_value <$150 → agent routes. Else → human decides. | Rules-based routing (reason code + condition + item category → restock/outlet/refurbish/donate/destroy). Fraud-flagged items blocked from auto-routing. |
| **Refund/credit approval** | **Human-overseen** | 60% agent-approved; 40% human-approved | IF fraud_flag = FALSE AND item_value <$150 AND return_age ≤60 days AND no_customer_escalation → agent approves. Else → human approves. | Agent calculates refund per policy. High-value, fraud-flagged, out-of-policy, or exception requests require human approval. |
| **Fraud investigation** | **Human-led** | 0% agent-decided; 100% human (~675/week assumption) | Always human. Agent provides evidence only. | Processors review all fraud flags. Agent surfaces supporting data (history, patterns, similar cases) but humans make approve/deny/escalate decisions. |
| **Customer escalations** | **Human-led** | 0% agent-decided; 100% human (volume unknown) | Always human. Agent provides context only. | Complaints, social media mentions, VIP customers, manager escalations = human end-to-end. Agent surfaces order history and sentiment analysis. |
| **Exception/edge cases** | **Human-led** | 0% agent-decided; 100% human (volume unknown) | Always human. Agent flags as "requires judgment." | Wrong item shipped, damaged in transit, missing tags, partial returns = ambiguous cases requiring human judgment. |
| **Quality audit & retraining** | **Human-led** | 200 samples/week for audit | Ongoing human oversight. | Weekly QA audits agent-processed returns, labels ground truth, feeds corrections to retrain models. Monthly review of escalation accuracy. |

**Expected volume distribution:**
- **Fully agent-handled (no human touch):** ~60% of volume (~2,700/week)
- **Agent-assisted (human validates/approves):** ~25% of volume (~1,125/week)
- **Fully human-handled:** ~15% of volume (~675/week)

**Capacity impact (calculation):**  
Current: 12 processors × 375 returns/week = 4,500 returns  
With agents handling 2,700/week at 8 min each = 360 hours/week freed  
Equivalent to ~9 FTE-hours/week redeployed to fraud investigation, escalations, and quality audits

---

## Agent Specification

### Core Capabilities

**1. Return Intake & Data Extraction**
- Input: Barcode scan, return form (paper/digital), customer-uploaded photos
- Output: Structured return record (order_id, sku, customer_id, stated_reason, return_date, photos)
- Method: OCR + database lookup
- Failure handling: IF barcode unreadable OR order not found → flag for human data entry within 5 minutes

**2. Reason-Code Classification**
- Input: Stated reason, item category, return timing (days since purchase), condition photos
- Output: Reason code (defective / didn't fit / changed mind / wrong item shipped / suspected fraud) + confidence score (0.00–1.00)
- Method: Multi-class classifier trained on historical labeled returns
- Decision rule: IF confidence ≥0.85 → assign reason code. IF confidence <0.85 → escalate to human review queue with top-3 candidate reasons displayed
- Target accuracy: ≥92% on held-out test set before deployment

**3. Fraud Risk Scoring**
- Input: Customer return history, item value, stated reason vs. photo condition, return frequency (90-day window), cross-channel patterns
- Output: Fraud risk score (0.00–1.00) + flag (TRUE/FALSE)
- Scoring factors:
  - Return frequency: >5 returns in 90 days → +0.25
  - High-value: item retail >$150 → +0.20
  - Condition mismatch: stated "defective" but photos show wear/use → +0.30
  - Serial returner: >10 returns in 12 months → +0.25
- Decision rule: IF fraud_score >0.30 → flag = TRUE → mandatory human review queue (agent CANNOT approve)
- Target detection rate: ≥75% of confirmed fraud cases flagged pre-chargeback

**4. Condition Grading (Computer Vision)**
- Input: 3–5 photos per item (front, back, tags, close-ups of damage if any)
- Output: Condition grade (new / like-new / good / fair / poor / damaged) + confidence score (0.00–1.00)
- Method: CNN-based image classifier trained on labeled condition photos
- Decision rule: IF item_value >$150 OR fraud_flag = TRUE OR confidence <0.80 → human validates. Else → accept agent grade
- Validation: 10% random sample reviewed by human weekly for quality control

**5. Disposition Routing**
- Input: Reason code + condition grade + item category + inventory signal (assumption: stock levels updated weekly)
- Output: Disposition lane (restock as new / discount to outlet / refurbish / donate / destroy)
- Routing rules (example framework):
  - "Defective" + damaged → destroy
  - "Didn't fit" + like-new + in-demand SKU → restock as new
  - "Changed mind" + good + slow-moving SKU → outlet
  - "Wrong item shipped" + new → restock as new
- Constraint: IF fraud_flag = TRUE → block routing until human clears
- Failure handling: IF no matching rule → escalate to human with routing suggestion

**6. Refund Calculation & Approval**
- Input: Disposition lane, condition grade, return window (days since purchase), item retail price
- Output: Refund amount + approval decision (auto-approved / escalate to human)
- Calculation: Base refund = item price. Apply condition deduction per policy (assumption: like-new = 0%, good = 10%, fair = 25%)
- Decision rule: IF fraud_flag = FALSE AND item_value <$150 AND return_age ≤60 days AND calculated_refund = standard → auto-approve. Else → human approves
- Audit trail: Log all refund calculations with reasoning for compliance

---

### Guardrails & Risk Controls

**Hard constraints (agent CANNOT violate):**
1. Agent cannot approve any return with fraud_flag = TRUE
2. Agent cannot process returns with item_value >$150 without human validation
3. Agent cannot process returns with customer_escalation_flag = TRUE
4. Agent cannot assign reason codes with confidence <0.85
5. Agent cannot route items to disposition lanes if fraud_flag = TRUE

**Escalation triggers (mandatory human review):**
- Fraud score >0.30
- Item value >$150
- Reason-code confidence <0.85
- Condition-grade confidence <0.80
- Customer escalation flag = TRUE
- Return age >60 days
- Refund amount deviates from policy by >$10
- No matching disposition rule found

**Failure handling:**
- IF agent system down → all returns route to human processing (fallback mode)
- IF confidence score unavailable (model error) → escalate to human
- IF photo upload fails → block processing until photos received
- IF fraud score calculation fails → default to human review (fail-safe bias)

**Audit & compliance:**
- Log all agent decisions with reasoning (reason code, confidence, fraud score, disposition, refund calculation)
- Retain all input data (photos, forms, scores) for 24 months minimum
- Weekly audit: 200 random agent-processed returns re-graded by QA team
- Monthly audit: Review all escalations for precision (were they necessary?)

---

## Validation Design

### Pre-Deployment Testing

**1. Model accuracy validation (offline)**
- Reason-code classifier: ≥92% accuracy on 10,000-return held-out test set
- Fraud risk scorer: ≥75% recall at 15% escalation rate (on historical fraud-confirmed cases)
- Condition grader: ≥88% accuracy on 5,000-image labeled test set
- Test set must include edge cases: damaged items, ambiguous reasons, borderline fraud patterns

**2. End-to-end workflow testing (shadow mode)**
- Run agent in parallel with human processors for 4 weeks (2,000 returns minimum)
- Measure: agreement rate between agent decisions and human decisions
- Target: ≥90% agreement on reason code, ≥85% agreement on disposition routing
- Review all disagreements: classify as agent error, human error, or legitimate judgment difference

**3. Escalation precision testing**
- Manually review 200 agent escalations from shadow mode
- Measure: precision = (escalations humans agree were necessary ÷ total escalations) × 100
- Target: ≥80% precision (avoid over-escalation that wastes human capacity)

---

### Post-Deployment Monitoring

**Daily metrics (automated dashboards):**
- Volume processed: agent-handled vs. human-handled vs. escalated
- Processing time: p50 and p95 latency per workflow step
- Escalation rate: % of returns escalated to human review
- System uptime: % of time agent available (target: ≥99.5%)

**Weekly metrics (QA team):**
- Reason-code accuracy: 200-return random sample re-classified by humans
- Condition-grade accuracy: 200-return random sample re-graded by humans
- Escalation precision: review 50 random escalations (were they necessary?)
- Model drift detection: compare current-week accuracy to baseline (alert if drops >5%)

**Monthly metrics (operations review):**
- Fraud detection rate: reconcile agent flags vs. chargeback reports
- Financial impact: calculate margin saved from improved accuracy (tracked via disposition outcomes)
- Capacity freed: hours/week redeployed from agent-handled volume
- Customer satisfaction: survey sample of customers whose returns were agent-processed (target: ≥85% satisfied)

**Retraining cadence:**
- Weekly: incorporate 200 QA-audited returns into training set
- Monthly: retrain reason-code and condition-grade models if accuracy drops below target
- Quarterly: retrain fraud scorer with updated chargeback data

**Failure response:**
- IF reason-code accuracy drops below 90% for 2 consecutive weeks → pause agent classification, investigate model drift
- IF fraud detection rate drops below 70% → escalate to fraud team, review scoring logic
- IF escalation rate exceeds 25% → review thresholds (may be over-escalating)

---

## Assumptions & Unknowns

### Assumptions Introduced (Not in Scenario)

**Volume & fraud assumptions:**
- Fraud prevalence estimated at ~5% of total returns (~225/week flagged for review)
- Customer escalations estimated at ~4% of returns (~180/week)
- Exception/edge cases estimated at ~6% of returns (~270/week)
- High-value threshold set at $150 retail (not specified in scenario)

**Processing assumptions:**
- Processors work 40-hour weeks, 5 days/week
- Current 18 min/return calculated from volume ÷ processors ÷ hours
- Agent processing time estimated at 8 min/return (50% improvement assumption)
- Photos are uploaded by customers or captured at facility (not specified)

**Technical assumptions:**
- Agent system achieves ≥95% uptime
- Photos are sufficient quality for computer vision (3–5 images per item)
- Inventory stock levels updated weekly for disposition routing
- Condition deduction policy exists and is codified (not specified in scenario)

**Financial assumptions:**
- Margin impact of misclassification not quantified (removed unsupported estimates)
- Payback period not calculated (requires cost data not in scenario)

---

### Unknowns Requiring Clarification

**Policy & risk tolerance:**
1. What is the exact high-value threshold for mandatory human review? ($150 assumed, needs CFO confirmation)
2. What is the acceptable false-positive rate for fraud flags? (Currently targeting 80% precision = 20% false positives)
3. What defines "customer escalation"? (Social media mention? Complaint email? VIP status? Manager override?)
4. What is the condition-deduction policy for calculating refunds? (Assumed like-new = 0%, good = 10%, fair = 25%)
5. Is there a return-age policy limit? (60 days assumed)

**Operational constraints:**
1. How are photos captured? (Customer upload, facility scan booth, manual processor photos?)
2. What is current fraud prevalence (actual %)? (5% assumed for volume estimates)
3. What is peak return volume during post-holiday season? (Affects system capacity planning)
4. Are there SKU-level exceptions (e.g., final-sale items, intimate apparel, damaged packaging = auto-destroy)?

**Data availability:**
1. Do we have 12 months of labeled historical returns for model training? (Need minimum 50,000 labeled examples)
2. Are chargeback reports reconcilable back to specific return records? (Required for fraud-detection validation)
3. Is customer return history accessible in real-time? (Required for serial-returner scoring)
4. Are inventory stock levels available via API for disposition routing?

**Success criteria:**
1. What margin improvement ($ or %) does CFO require for payback approval?
2. What is acceptable processor headcount reduction or redeployment plan?
3. What is risk tolerance for agent errors (e.g., 1 wrong high-value approval per month acceptable)?