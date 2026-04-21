# Production Spec Checklist: Agent-Buildable Specifications

## Purpose

This checklist audits whether a specification is precise enough for autonomous AI coding agents to build from without clarifying questions. Every ambiguous statement in the spec becomes a build failure waiting to happen.

A spec is "production-grade" for AI when every requirement is testable, every entity is defined consistently, and every integration contract is explicit. Passing this checklist means an AI agent can start building with high confidence.

## BUILDABILITY

**Criterion:** Could an AI coding agent start building without asking clarifying questions?

### Pass
- [ ] Every requirement statement includes a testable acceptance criterion
  - **Example (pass):** "The API endpoint must return results within 500ms for 95th percentile of queries. Measure: run 100 queries of typical size; calculate p95 latency; must be <= 500ms."
  - **Example (fail):** "The API should be fast."
- [ ] All ambiguous words are defined: "recent," "soon," "large," "frequently," "as needed," "typically"
  - **Example (pass):** "Load recent orders" → "Load orders created in the past 7 days, sorted descending by created_at."
  - **Example (fail):** "Load recent orders."
- [ ] Every conditional ("if," "when," "unless") has explicit criteria and outcomes
  - **Example (pass):** "If order total > $1000 AND customer credit score < 600, require manager approval. Manager can approve or reject; approval is logged with manager_id and timestamp."
  - **Example (fail):** "If the order is high-value, it might need approval."
- [ ] No modal verbs without scope: "should," "may," "could," "might" are vague. Replace with "must," "will," "can," or "is optional."
  - **Example (pass):** "The system must validate email format using RFC 5322. Validation fails if format does not match spec."
  - **Example (fail):** "Email validation should be included."
- [ ] All cross-feature interactions are described: what happens when Feature A conflicts with Feature B?
  - **Example (pass):** "If both auto_retry AND manual_retry are requested, manual_retry takes precedence. Auto_retry is cancelled. User receives notification that manual override was applied."
  - **Example (fail):** "The system supports both auto-retry and manual retry."

### Failure Signals
- "Implement [feature] using your best judgment" → agent will guess; you'll be unhappy
- "Handle [scenario] appropriately" → appropriate means different things to different agents
- Vague timeline language: "eventually," "in a timely manner," "soon"
- No mention of failure modes, only happy path

---

## ENTITY PRECISION

**Criterion:** Are all entities defined with attributes, relationships, state machines, and validation rules? Same entity = same definition everywhere.

### Pass
- [ ] Every entity has an explicit data model:
  - [ ] Primary key (immutable identifier)
  - [ ] All attributes with types and constraints (required, optional, min/max, format)
  - [ ] Timestamp fields (created_at, updated_at, deleted_at)
  - [ ] Audit fields (created_by, updated_by)
  - [ ] Relationships to other entities (foreign keys, cardinality)

### Example (Pass)
```
Entity: Order

Attributes:
- id: UUID, primary key, immutable, generated on creation
- customer_id: UUID, foreign key to Customer, required, immutable, on delete: restrict (cannot delete a Customer who has Orders)
- status: enum [DRAFT, PLACED, CONFIRMED, SHIPPED, DELIVERED, CANCELLED], required, default DRAFT
- total_amount: decimal(10,2), required, >= 0, computed from line items
- currency: enum [USD, EUR, GBP], required, default USD
- notes: string, optional, max 1000 characters
- created_at: ISO 8601 timestamp, UTC, set on creation, immutable
- updated_at: ISO 8601 timestamp, UTC, updated on any modification
- deleted_at: ISO 8601 timestamp, UTC, null by default, set on soft-delete, immutable once set
- created_by: reference to User who created it, immutable
- updated_by: reference to User who last modified it

State Machine:
DRAFT → PLACED (user clicks "Place Order"; validates address, payment method, inventory)
PLACED → CONFIRMED (payment processed; stock reserved)
PLACED → CANCELLED (user or admin cancels before confirmation)
CONFIRMED → SHIPPED (fulfillment team ships; triggers email notification)
SHIPPED → DELIVERED (tracking updates; final notification)
CANCELLED is terminal

Constraints:
- total_amount must equal sum of (line_item.quantity * line_item.unit_price) for all line items
- Cannot transition to PLACED if customer_id references a deleted customer
- Cannot transition to CONFIRMED if any line item is out of stock
- Cannot modify customer_id, created_at, or created_by after creation
```

### Example (Fail)
```
Entity: Order

The Order entity contains order information. It has an ID, customer reference, status, and total amount. Status can be new, processing, shipped, or done. The system should track when it was created and who created it.
```

**Why it fails:**
- No data types
- No validation rules
- Status values vague (what is "done" vs "shipped"?)
- No state machine (can you go backwards? SHIPPED → DRAFT?)
- No immutability rules
- "Should track" is vague

### Pass / Fail Checklist
- [ ] All enum values are SCREAMING_SNAKE_CASE and exhaustive (no "other" category)
- [ ] Every date/timestamp is ISO 8601, timezone specified
- [ ] Every numeric field has units and range (currency decimals, percentages, etc.)
- [ ] Every string field has max length and format constraints
- [ ] Foreign keys specify cascade behavior (on delete: cascade, restrict, set null)
- [ ] Computed fields are explicitly marked read-only and their formula is defined
- [ ] State machine is complete: for each state, list all valid transitions and prerequisites
- [ ] No contradictory rules (e.g., "status is immutable" AND "status can change")

---

## DELEGATION BOUNDARIES

**Criterion:** Is it clear what the agent decides alone vs what requires human oversight?

### Pass
- [ ] Every decision point is categorized:
  - **Agent Decides**: Agent has sufficient information and the consequence of being wrong is reversible or low-impact. Examples: format validation, data transformation, retry logic.
  - **Agent Decides + Logs**: Agent decides but always logs the decision (audit trail). Examples: auto-approval of orders under $100, auto-retry of failed API calls.
  - **Agent + Human Review**: Agent acts, human can review and veto. Examples: auto-generated report, populated form, curated suggestions.
  - **Human Decides**: Agent gathers information and presents options; human makes the call. Examples: customer account closure, high-value decisions, non-reversible actions.

- [ ] Escalation triggers are specific:
  - **Example (pass):** "If payment verification fails, order stays in PLACED. Agent logs failure reason with order_id and timestamp. FLS user is notified (email + dashboard flag) for manual review. User must manually approve or reject within 24 hours. If no action in 24h, order auto-cancels."
  - **Example (fail):** "Handle payment failures appropriately."

- [ ] Boundary conditions are explicit:
  - **Example (pass):** "Orders under $500 can be auto-approved by agent if customer credit score >= 700 AND no fraud flags. Orders $500-$5000 require human review. Orders > $5000 require manager approval. Fraud flags always trigger human review regardless of amount."
  - **Example (fail):** "The agent should use judgment to approve orders."

### Pass / Fail Checklist
- [ ] Every action is labeled: [Agent Alone] [Agent + Log] [Agent + Review] [Human]
- [ ] All decision thresholds are numeric or boolean (no fuzzy "might" conditions)
- [ ] Escalation paths are complete: if escalated, what happens next? Who is notified? What is the timeout?
- [ ] Audit trail requirements are explicit: what must be logged? To what system? Retention period?
- [ ] Override mechanisms are documented: can a human override an agent decision? How is it logged?
- [ ] Ambiguity in scope is flagged during drafting: if something is ambiguous, note it as [TODO] and track separately. All [TODO] items must be resolved (clarified, descoped, or moved to Assumptions) before the spec passes the §Final Pass/Fail Decision — a spec with open [TODO] markers fails.

---

## INTEGRATION CONTRACTS

**Criterion:** Are API contracts, data formats, error handling, and retry logic explicit?

### Pass
- [ ] For every external system integration:
  - [ ] System name and purpose
  - [ ] Endpoint URL (or endpoint pattern for multiple endpoints)
  - [ ] Authentication method (API key, OAuth, mTLS) and where credentials are stored
  - [ ] Request format: exact JSON/XML/form structure with all required/optional fields
  - [ ] Response format: exact structure, including error responses
  - [ ] Timeout value (in seconds or ms)
  - [ ] Retry logic: on what conditions? How many retries? Backoff strategy?
  - [ ] Rate limits: requests per minute, per day? Burst allowance?
  - [ ] Data mapping: how do internal entities map to the external system's entities?
  - [ ] Fallback behavior: if the integration is unavailable, what does the agent do?

### Example (Pass)
```
Integration: Payment Gateway (Stripe)

Endpoint: POST https://api.stripe.com/v1/payment_intents
Authentication: Bearer token in Authorization header, stored in secrets manager (key: STRIPE_API_KEY)

Request format (JSON):
{
  "amount": integer (in cents, required),
  "currency": string enum [usd, eur, gbp] (required),
  "customer_id": string Stripe customer ID (required),
  "description": string (optional, max 1000 chars),
  "metadata": object with keys [order_id, customer_email] (optional)
}

Success response (HTTP 200):
{
  "id": string (payment intent ID),
  "status": enum [succeeded, requires_action, requires_capture, canceled],
  "amount": integer,
  "client_secret": string,
  "charges": { data: [...] }
}

Error response (HTTP 4xx/5xx):
{
  "error": {
    "type": string enum [card_error, invalid_request_error, api_error],
    "message": string,
    "param": string (which field caused the error),
    "code": string (specific error code)
  }
}

Timeout: 10 seconds
Retry logic:
- HTTP 5xx (server errors): retry up to 3 times with exponential backoff (2s, 4s, 8s)
- HTTP 429 (rate limit): retry with Retry-After header value
- HTTP 4xx (client errors): do not retry; log and escalate to human review
- Timeout: do not retry; escalate to human review

Rate limits: 100 requests per second per API key
Fallback: if Stripe is unavailable for > 5 minutes, suspend order processing; notify ops; queue orders for later retry

Data mapping:
Internal Order.total_amount (USD cents) → Stripe.amount
Internal Order.currency → Stripe.currency
Internal Order.id → Stripe.metadata.order_id
Internal Customer.email → Stripe.metadata.customer_email
Stripe.customer_id ← lookup in internal customer_stripe_mapping table
```

### Example (Fail)
```
Integration: Payment Gateway

Connect to the payment processor to process payments. Handle errors gracefully. Retry if needed.
```

**Why it fails:**
- No endpoint URL
- No authentication details
- No request/response format
- "Handle errors gracefully" is vague
- "Retry if needed" has no criteria

### Pass / Fail Checklist
- [ ] Every integration has a full endpoint contract (request + response structure)
- [ ] All required fields are marked; all optional fields are marked
- [ ] All enums are exhaustive (no "other")
- [ ] Timeout is numeric (seconds or ms)
- [ ] Retry strategy covers all HTTP status codes (2xx, 3xx, 4xx, 5xx)
- [ ] Rate limits are numeric
- [ ] Data mapping from internal to external is documented in both directions
- [ ] Fallback behaviour is explicit (queue, skip, escalate, fail-fast, graceful degrade)
- [ ] Authentication credentials are sourced from a specific location (env var, secrets manager, config file)
- [ ] Error codes are listed; for each code, the handling is specified

---

## VALIDATION DESIGN

**Criterion:** Are there testable scenarios for happy path, edge cases, failure modes, and cross-capability interactions?

### Pass
- [ ] Happy path is documented with a concrete end-to-end example
  - **Example (pass):** "User creates Order with valid address, in-stock items, and valid payment. System places order, reserves inventory, processes payment, and transitions to CONFIRMED. User receives confirmation email within 5 seconds."

- [ ] Edge cases are listed with expected outcomes:
  - [ ] Empty or null inputs (what is valid? what causes error?)
  - [ ] Boundary values (min/max, exact limits)
  - [ ] Overlapping conditions (what if both A and B are true?)
  - [ ] Timing edge cases (what if two users perform the same action simultaneously?)
  - [ ] State edge cases (what if something is in an unexpected state?)

### Example
```
Validation scenarios for Order.place():

Happy Path:
- Input: Order in DRAFT status, customer ACTIVE, address valid, items in stock, payment valid
- Expected: Order transitions to PLACED, payment processed, inventory reserved, order transitions to CONFIRMED, confirmation email sent, returns {success: true, order_id, status: CONFIRMED}

Edge Cases:
1. Empty notes field
   - Input: notes = "", notes = null, notes = "   " (whitespace only)
   - Expected: treated as null; notes field set to null; no validation error

2. Duplicate line items
   - Input: Order with same product twice (different line items, same product_id)
   - Expected: allowed; items are separate line items; total_amount is sum of all

3. Customer address changed between DRAFT and PLACED
   - Input: Order created with address A; customer updates to address B; user clicks "Place Order"
   - Expected: order placed with current customer address (B); no error; address is not immutable

4. Inventory goes to zero between DRAFT and PLACED
   - Input: Item had 5 units in stock when DRAFT created; now 0 units
   - Expected: transition to PLACED fails; order stays DRAFT; error: "Item X now out of stock"

5. Payment processing timeout
   - Input: Payment gateway times out after 10 seconds
   - Expected: order stays PLACED (payment not yet confirmed; no state change until payment succeeds or is manually cancelled); user can retry payment or contact support

6. Two users try to place the same order simultaneously
   - Input: Two concurrent POST /orders/{id}/place requests
   - Expected: first succeeds; second fails with 409 Conflict; error: "Order already placed"

7. Concurrent inventory updates
   - Input: Order A reserves 2 units of Item X; Order B reserves 3 units; only 4 units in stock total
   - Expected: inventory lock prevents race condition; only one order succeeds; other gets "insufficient inventory" error
```

- [ ] Failure modes are documented with recovery steps:
  - **Example (pass):** "If payment fails, order stays in PLACED state. User can update payment method and retry. System retries payment automatically every 24 hours for 3 days; if all fail, order is auto-cancelled and user is notified."

- [ ] Cross-capability interactions are tested:
  - **Example:** "User updates order total while fulfillment is in progress. What happens? Expected: if already in SHIPPED state, update is rejected. If in PLACED, update is allowed but triggers re-validation of payment."

### Pass / Fail Checklist
- [ ] At least one end-to-end happy-path scenario with concrete inputs and outputs
- [ ] At least 5 edge cases covering: empty/null, boundary, concurrent, timing, state
- [ ] For each edge case, the expected outcome is explicit (pass, fail, escalate, log)
- [ ] At least 3 failure mode scenarios with recovery steps
- [ ] Concurrency/race conditions are explicitly addressed (locks, transactions, idempotency)
- [ ] Field interactions are documented (if A changes, what must happen to B?)
- [ ] All validation rules have test criteria (input that passes, input that fails)

---

## ASSUMPTIONS REGISTER

**Criterion:** What is assumed vs what is known? Are assumptions flagged for client validation?

### Pass
- [ ] Every assumption is documented:
  - [ ] Assumption statement
  - [ ] Why it matters
  - [ ] What breaks if it's wrong
  - [ ] Status: [Known] [Assumed] [Flagged for Validation]

### Example
```
Assumptions Register:

A1: Payment gateway will respond within 10 seconds 99% of the time
   Why: Sets timeout value in integration contract
   If wrong: Agent will reject valid orders due to timeout; customer satisfaction impact
   Status: [Flagged for Validation] - need SLA from Stripe

A2: Customer addresses are geocodable (lat/long can be derived)
   Why: Address validation relies on geocoding service
   If wrong: Orders with ambiguous addresses will fail validation
   Status: [Flagged for Validation] - test with international addresses

A3: Inventory is updated in real-time when stock is sold elsewhere
   Why: Agent assumes inventory counts are current
   If wrong: Agent may reserve stock that's already sold; leads to fulfillment failures
   Status: [Known] - client confirmed real-time inventory sync exists

A4: Email service is always available
   Why: Agent sends confirmation emails synchronously
   If wrong: Order transitions to CONFIRMED but email fails; customer doesn't know
   Status: [Assumed] - needs async queue or fallback notification method
```

### Pass / Fail Checklist
- [ ] All assumptions about external systems (uptime, latency, data quality, API contracts) are listed
- [ ] All assumptions about business rules (e.g., "customers never request more than X items") are listed
- [ ] All assumptions about user behavior are listed
- [ ] Each assumption is marked [Known/Assumed/Flagged]
- [ ] For flagged assumptions, a validation question is written and assigned to a stakeholder
- [ ] If assumptions are wrong, the failure mode is clear

---

## ECONOMICS ALIGNMENT

**Criterion:** Does the spec support the cost model? Are token-expensive operations identified?

### Pass
- [ ] Agent operations are classified by cost:
  - [ ] Check: read-only, no generation (cheap)
  - [ ] Validate: compare against rules (cheap)
  - [ ] Generate: create content, make decisions (moderate)
  - [ ] Coordinate: call external systems, retrieve data (expensive)
  - [ ] Transform: process data at scale (very expensive)

- [ ] For each expensive operation:
  - [ ] Is it necessary? Could it be cached?
  - [ ] Could it be batched (amortize cost)?
  - [ ] Is there a circuit breaker (stop if cost > threshold)?
  - [ ] Is there an async alternative (defer cost)?

### Example
```
Operation: Validate Order line items against inventory system

Cost: EXPENSIVE (each item requires external inventory API call)

Current design: validate each item synchronously; agent calls inventory API once per line item

Problems:
- If order has 20 items, 20 API calls
- Each call costs tokens + latency
- If inventory API is slow, order placement is slow

Alternatives:
1. Batch validation: send all items in single request to inventory API (if API supports)
   Cost: 1 API call instead of N
   Tradeoff: requires inventory system change

2. Cache inventory counts locally with TTL
   Cost: occasional cache miss, background refresh
   Tradeoff: inventory data may be stale for a few seconds

3. Deferred validation: place order, validate asynchronously, send notification if fails
   Cost: moved from user-blocking path to background job
   Tradeoff: user sees "confirmed" before validation complete

Chosen approach: Batch validation (Alternative 1); requires inventory API enhancement
```

### Pass / Fail Checklist
- [ ] All external API calls are identified and classified by cost
- [ ] Batch/caching opportunities are documented
- [ ] Circuit breakers are specified for expensive operations
- [ ] Cost assumptions are validated (not assumed)
- [ ] Async/deferred alternatives are considered for blocking operations
- [ ] Token budgets are defined (if applicable): max tokens per request, per day, per operation

---

## GOVERNANCE

**Criterion:** Audit trail requirements, HITL checkpoints, compliance constraints.

### Pass
- [ ] Every action that affects data or compliance is loggable:
  - **Example (pass):** "Every order state transition is logged with: timestamp, from_status, to_status, agent_id, reason, user_id (if human-triggered). Logs are immutable and retained for 7 years."
  - **Example (fail):** "Log important actions."

- [ ] Compliance constraints are documented:
  - [ ] Regulations that apply (GDPR, HIPAA, PCI-DSS, SOX, etc.)
  - [ ] What actions require audit trail?
  - [ ] What actions require human approval?
  - [ ] What data can be deleted vs. what must be retained?
  - [ ] What are the consequences of non-compliance?

- [ ] Human-in-the-loop checkpoints are specified:
  - **Example (pass):** "Orders > $10,000 require manager approval before CONFIRMED. Manager has 24 hours to approve/reject. If no action, order auto-approves. Approval is logged with manager_id and decision_reason."
  - **Example (fail):** "High-value orders should be reviewed."

### Example
```
Governance: Order Processing

Audit requirements:
- Order creation: log customer_id, user_id, timestamp, items, total_amount
- State transitions: log from_status, to_status, reason, timestamp, agent_id
- Payment processing: log payment_id, amount, result (success/fail), timestamp
- Customer data access: log who accessed what customer data, timestamp, reason
- HITL approval: log approver_id, decision (approve/reject), reason, timestamp

Retention:
- Transaction logs: 7 years (financial compliance)
- Access logs: 1 year (security audit)
- Customer PII: deleted upon account deletion + 90-day grace period (GDPR right to be forgotten). PII in retained records (orders, transaction logs) is anonymized, not deleted — the record stays for financial compliance but is no longer attributable to the individual.

Compliance:
- GDPR: customer data must be deletable. For records under financial retention (orders, transaction logs), anonymize all PII fields (customer name, address, contact details) rather than deleting the record — this satisfies the right to be forgotten while preserving the audit trail. Cascade full deletion only to records with no retention requirement (e.g., saved addresses, wishlists, preferences).
- PCI-DSS: payment card data must not be stored; only tokenized references allowed
- SOX: all financial transactions must be reconcilable to source

HITL checkpoints:
- Orders > $10,000: require manager approval
- Orders with fraud flags: require review before processing
- Account closures: require customer confirmation (email) + manager approval
- Bulk refunds: require approvals from Finance + Customer Service leads

Escalation:
- If HITL approval is not given within SLA, escalate to next level
- SLA: $10K orders 24 hours; fraud flags 6 hours; closures 48 hours
```

### Pass / Fail Checklist
- [ ] All regulated data is identified; applicable regulations are listed
- [ ] Audit trail schema is documented (what fields, how long retained?)
- [ ] HITL approval gates are specified with SLAs
- [ ] Data deletion/retention policies are explicit
- [ ] Non-repudiation is addressed (can someone deny they made a decision? How do we prove they did?)
- [ ] All compliance requirements map to specific requirements in the spec

---

## FINAL PASS/FAIL DECISION

**The spec passes production review if:**
- [ ] All criteria sections have no open items
- [ ] Every ambiguous statement has been clarified or removed
- [ ] Every integration contract is complete
- [ ] Every entity has a full data model and state machine
- [ ] Delegation boundaries are clear (no guesswork)
- [ ] Validation design meets the minimum bar per §Validation Design: at least 1 happy-path scenario, at least 5 edge cases, and at least 3 failure mode scenarios per feature, each with explicit expected outcomes
- [ ] Governance and audit trails are explicit
- [ ] Assumptions are flagged; critical ones have validation questions assigned

**The spec fails if:**
- [ ] Any required section is incomplete or vague
- [ ] Delegation boundaries are ambiguous
- [ ] An integration contract is missing any of: request format, response format, timeout, retry logic, rate limit, fallback
- [ ] Validation scenarios are missing edge cases or failure modes
- [ ] Governance doesn't address applicable compliance requirements

**Use this checklist before handing a spec to an AI coding agent.** Every unchecked box is technical debt that will manifest as clarifying questions, failed builds, or misaligned implementations.
