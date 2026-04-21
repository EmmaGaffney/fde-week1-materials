# Spec Ambiguity vs Builder Mistakes: A Taxonomy for Build Failure Diagnosis

## Why This Matters

When an AI coding agent produces output that doesn't match expectations, the instinct is usually to blame the agent: "the builder missed the requirement" or "the agent didn't read the spec carefully."

But this is often wrong. The failure may not be the agent's fault.

Build failures fall into four categories. Learning to diagnose which one you're in is critical to FDE growth. Each category has a different fix, and if you apply the wrong fix, you'll waste time re-prompting the wrong thing.

## The Four Categories

### Category 1: Spec Ambiguity (Not the Builder's Fault)

**Definition:** The spec was unclear or could be interpreted multiple ways. The builder (AI agent) chose a valid interpretation that wasn't the intended one. This is an FDE failure, not a builder failure.

**The key signal:** The agent's implementation matches the spec as written, but not as you intended.

#### Example 1: Validation Rule Interpretation

**Spec (as written):**
> "Validate that the customer's address is valid before processing the order."

**What the agent built:**
> Checks that address has non-empty street, city, state, ZIP. Returns true if all fields present.

**What you intended:**
> Check that the address exists in the US Postal Service database (geocode + verify).

**Why this is spec ambiguity (not builder error):**
- The spec says "validate address is valid"
- The agent interpreted "valid" = "has all required fields"
- You interpreted "valid" = "geocodable in USPS"
- Both are defensible interpretations of "valid"
- The spec didn't specify which validation method to use

**The fix:**
Rewrite the spec. Do not re-prompt the agent.

```markdown
Before (ambiguous):
"Validate that the customer's address is valid before processing the order."

After (clear):
"Before placing an order, call the USPS Address Validation API to geocode the address.
- If geocoding succeeds, address is valid; proceed.
- If geocoding fails (address not found), reject with error: 'Address not found in USPS database. Please verify and try again.'
- If API call times out, escalate to human review."
```

#### Example 2: Rounding Behavior

**Spec (as written):**
> "Calculate tax as 8.5% of the subtotal."

**What the agent built:**
> `tax = round(subtotal * 0.085, 2)`

**What you intended:**
> `tax = floor(subtotal * 0.085 * 100) / 100` (truncation toward zero — always rounds down)

**Why this is spec ambiguity:**
- Rounding has multiple valid approaches (round, ceil, floor, banker's rounding, etc.)
- The spec didn't specify which one
- The agent chose a reasonable default

**The fix:**
```markdown
Before:
"Calculate tax as 8.5% of the subtotal."

After:
"Calculate tax as follows:
1. Multiply subtotal by 0.085
2. Round down to nearest cent (floor)
3. Example: subtotal = $100.00 → tax = floor($8.50) = $8.50
           subtotal = $100.01 → tax = floor($8.50085) = $8.50"
```

#### Example 3: Default Behavior When Input is Missing

**Spec (as written):**
> "If the user provides a phone number, validate it. Otherwise, skip validation."

**What the agent built:**
> If phone number field is empty, set phone = null and proceed.

**What you intended:**
> If phone number field is empty, use a default phone number from company config.

**Why this is spec ambiguity:**
- "Skip validation" is satisfied by both: "set to null" and "use default"
- The spec didn't say what to do with the field if it's missing

**The fix:**
```markdown
Before:
"If the user provides a phone number, validate it. Otherwise, skip validation."

After:
"Phone number field is optional. If user provides a phone number:
1. Validate format using regex [pattern]
2. If invalid, return error
If user does NOT provide a phone number:
1. Use default phone number from config.COMPANY_PHONE
2. Do not ask user; silently apply default
3. Log: 'Phone field empty; used default: {phone}'"
```

---

### Category 2: Builder Misread (The Builder's Fault)

**Definition:** The spec was clear. The agent ignored, misinterpreted, or didn't find a relevant spec section. The builder made an error. Fix: re-prompt with the relevant spec section highlighted.

**The key signal:** The agent's implementation contradicts the spec as written.

#### Example 1: Ignored a Constraint

**Spec (clear):**
> "Orders over $5,000 require manager approval before payment is processed. Do NOT process payment until approval is received."

**What the agent built:**
> Processes payment immediately; approvals are post-hoc.

**Why this is a builder error:**
- The spec explicitly says "require approval BEFORE payment"
- The spec explicitly says "DO NOT process until approval"
- The agent violated both statements
- This is not ambiguous; it's a misread

**The fix:**
Re-prompt the agent with the constraint highlighted.

```
The spec says:
"Orders over $5,000 require manager approval BEFORE payment is processed. Do NOT process payment until approval is received."

Your implementation processes payment immediately. This violates the spec. Please revise:
1. When order total > $5,000, transition to PENDING_APPROVAL state
2. Do not call payment API until PENDING_APPROVAL → APPROVED transition
3. Only APPROVED orders can be processed for payment
```

#### Example 2: Didn't Read the Data Model

**Spec (clear):**
```
Entity: Order

Attributes:
- id: UUID
- customer_id: UUID, foreign key to Customer, immutable
- status: enum [DRAFT, PLACED, CONFIRMED, SHIPPED, CANCELLED]
- ...

Constraint: "customer_id is immutable. Once an order is created, the customer cannot be changed."
```

**What the agent built:**
> Allows changing customer_id on existing orders.

**Why this is a builder error:**
- The spec explicitly defines customer_id as "immutable"
- The spec explicitly has a constraint stating this
- The agent ignored both

**The fix:**
```
The spec defines:
- customer_id: "immutable"
- Constraint: "customer_id is immutable. Once an order is created, the customer cannot be changed."

Your implementation allows modifying customer_id. This violates the immutability constraint.
Please add a guard: if order.id is not null (already created), reject any attempt to modify customer_id with error: "Customer ID is immutable after order creation."
```

#### Example 3: Chose Wrong Default

**Spec (clear):**
```
Entity: Shift

Attributes:
- status: enum [OPEN, CONFIRMED, CANCELLED], default OPEN
```

**What the agent built:**
> Sets status to CONFIRMED by default.

**Why this is a builder error:**
- The spec explicitly says default is OPEN
- The agent used CONFIRMED instead
- No ambiguity

**The fix:**
```
The spec says:
status: enum [OPEN, CONFIRMED, CANCELLED], default OPEN

Your code sets status = CONFIRMED by default. This is wrong. Change to status = OPEN.
```

---

### Category 3: Test Problem (The Validation's Fault)

**Definition:** The build matches the spec. The test expectation is wrong. The spec and the build are aligned; it's the validation that's wrong.

**The key signal:** When you carefully read both the spec and the agent's code, they match. But the test fails.

#### Example 1: Test Expects Wrong Value

**Spec:**
> "Round tax to nearest cent using standard rounding (round half up)."

**What the agent built:**
> `Math.round(tax * 100) / 100` (correct for round half up)

**Test expects:**
```python
assert tax_on_100_001 == 8.50  # expects floor behavior
```

**What actually happens:**
```python
assert tax_on_100_001 == 8.51  # round half up behavior
```

**Why this is a test problem:**
- The spec says "round half up"
- The agent implemented round half up
- The test is checking for floor behavior
- The test is wrong, not the implementation

**The fix:**
Update the test, not the code.

```python
Before:
assert tax_on_100_001 == 8.50  # wrong; expects floor

After:
assert tax_on_100_001 == 8.51  # correct; round half up
```

#### Example 2: Test Assumes Implementation Detail

**Spec:**
> "Retrieve the user's recent orders. Return them sorted by date, most recent first."

**What the agent built:**
```python
orders = db.query(Order).filter(Order.user_id == user_id).order_by(Order.created_at.desc())
```

**Test expects:**
```python
assert orders[0].id == expected_first_order  # expects specific order ID
```

**Problem:**
If two orders have the exact same created_at timestamp, the sort order is undefined. The test may pass or fail depending on database implementation or row insertion order.

**Why this is a test problem:**
- The spec says "sort by date, most recent first"
- The agent implemented that correctly
- The test is assuming a specific behavior when timestamps are identical
- The test is too brittle

**The fix:**
Make the test more robust by either:
1. Ensuring test data has distinct timestamps
2. Using a secondary sort key (e.g., id) to break ties
3. Testing the ordering relationship, not specific order IDs

---

### Category 4: Design Gap (The Hardest to Detect and Most Important for Growth)

**Definition:** The spec was precise about what was stated, but failed to address something that should have been obvious. The agent built what was asked for, but missed something important that wasn't explicitly mentioned.

This is different from spec ambiguity—it's not ambiguous, it's missing.

**The key signal:** The implementation is "correct" according to the spec, but it's obviously incomplete.

#### Example 1: No Error Handling

**Spec (as written):**
> "When user clicks 'save order,' validate all fields. If validation passes, save the order to the database."

**What the agent built:**
> Validates; if valid, calls db.save(); returns success.

**What's missing:**
> Database call can fail (network error, permission denied, duplicate key). The spec didn't say what to do if db.save() fails.

**Why this is a design gap (not ambiguity):**
- The spec was clear about the happy path
- The spec was silent on failure modes
- A production system obviously needs error handling
- But the spec didn't explicitly require it

#### Example 2: No Concurrency Control

**Spec (as written):**
> "Create an assignment between a nurse and a shift."

**What the agent built:**
> Creates a record in the assignments table.

**What's missing:**
> Two simultaneous requests create two assignments for the same nurse/shift. Spec didn't say this is forbidden, but in practice it should be.

**Why this is a design gap:**
- The spec didn't mention race conditions
- The spec didn't say "prevent duplicate assignments"
- But a real system needs this constraint

#### Example 3: No Observability

**Spec (as written):**
> "Process payment through the payment gateway."

**What the agent built:**
> Calls payment gateway API; returns success/failure.

**What's missing:**
> No logging. If payment fails, there's no audit trail of what went wrong.

**Why this is a design gap:**
- The spec said "process payment"
- It didn't say "log this"
- But compliance and debugging obviously require it

---

## Diagnostic Signals: How to Tell Which Category You're In

### Diagnosis Decision Tree

```
Question 1: Does the agent's implementation match the spec as written?

YES → Question 2: Does the implementation match your intent?
      YES → PASS (no problem)
      NO → Category 1 (Spec Ambiguity)
            Fix: Rewrite the spec

NO → Question 2: Is the agent's implementation valid under any reasonable interpretation?
     YES → Question 3: Is the difference meaningful (does it affect correctness)?
           YES → Category 1 (Spec Ambiguity)
                 Fix: Clarify the spec
           NO → ACCEPTABLE VARIATION (not a defect)
                 The builder's choice differs from the spec's literal wording but is
                 valid and does not affect correctness.
                 Action: Note the variation. Optionally update the spec to match
                 the builder's choice if it is clearer or more conventional.
                 Do not re-prompt — this is not an error.
     NO → Question 3: Does the spec address this scenario?
          YES → Category 2 (Builder Misread)
                Fix: Re-prompt
          NO → Category 4 (Design Gap)
               Fix: Add to spec explicitly — do not leave it as an implied "best practice"
```

### Signals for Each Category

#### Spec Ambiguity Signals
- The agent's code is well-written and logically consistent
- The agent's choice is defensible under the spec
- You have to trace through the code to see why it doesn't match your intent
- Two different people reading the spec might interpret it differently
- The spec uses words like "appropriate," "valid," "reasonable," "handle"

#### Builder Misread Signals
- The agent's code contradicts an explicit statement in the spec
- A sentence in the spec directly refutes the agent's implementation
- The agent missed a required constraint or field
- The agent used a wrong default value
- The error is something you'd catch in a code review: "wait, did you read the part about...?"

#### Test Problem Signals
- The code is correct per the spec
- The test checks for a specific value that's implementation-dependent
- The test makes assumptions not stated in the spec (e.g., sort order on ties)
- The agent's logic is sound, but the test data or assertions are wrong

#### Design Gap Signals
- The spec is clear, but it's missing an entire category (error handling, concurrency, observability)
- The agent built exactly what was asked for, but it's obviously incomplete
- A production system obviously needs X, but the spec doesn't mention X
- The gap is something that's true across most systems (error handling, logging, retry logic)

#### Acceptable Variation Signals
- The builder's code works correctly
- The difference is cosmetic, stylistic, or involves an equivalent technical choice (e.g. HTTP 422 vs 400 for validation errors, a different but equivalent date format, a renamed variable that preserves semantics)
- A reasonable person reading the spec could justify the builder's choice
- Changing the implementation back to match the spec's literal wording would not improve correctness, performance, or maintainability
- The difference would not confuse a future reader of the code

---

## Response Templates: What to Do for Each Category

### If It's Spec Ambiguity

**What to change:** The spec

**Process:**
1. Rewrite the ambiguous statement to be precise
2. Add examples if the statement is complex
3. List all possible interpretations; clarify which one is correct
4. If needed, add acceptance criteria ("must pass this test")
5. Re-run the agent with the updated spec

**Example fix script:**
```markdown
I've identified a spec ambiguity. The original statement:
"Validate that the customer's address is valid"

Could mean:
A) Check that all address fields are present and non-empty
B) Verify the address against a database (USPS, OpenAddress, etc.)
C) Geocode the address to lat/long

I intended (B). Please revise the spec to clarify this. The new statement should include:
- The specific validation method (USPS API)
- What counts as success/failure
- How to handle failures (reject, escalate, log)
```

### If It's Builder Misread

**What to change:** Re-prompt the agent (don't change the spec)

**Process:**
1. Extract the relevant spec section
2. Highlight the statement that the agent missed
3. Show the agent what it built vs. what the spec requires
4. Give the agent the specific code change needed

**Example re-prompt:**
```
The spec says:
"Orders over $5,000 require manager approval before payment is processed. 
Do NOT process payment until approval is received."

Your code processes payment immediately for all orders. This violates the spec.

Please revise:
1. When order total > $5,000:
   - Set order.status = PENDING_APPROVAL
   - Do not call the payment API
   - Send an email to the manager with approval link

2. When manager approves:
   - Set order.status = APPROVED
   - Call the payment API
   - Log the approval with timestamp and manager ID

3. Reject any attempt to process payment if status != APPROVED
```

### If It's a Test Problem

**What to change:** The test (not the code, not the spec)

**Process:**
1. Verify that the implementation matches the spec
2. Verify that the implementation is logically correct
3. Fix the test to match the spec and implementation
4. Re-run tests

**Example:**
```
The spec says: "Sort orders by creation date, most recent first."
The code does: ORDER BY created_at DESC
The test expects: orders[0].id == '12345' (brittle; depends on data)

The test is wrong because it assumes a specific ID when multiple orders
could have the same created_at. Fix the test:

Before:
assert orders[0].id == expected_id

After:
assert orders[0].created_at >= orders[1].created_at
assert all(orders[i].created_at >= orders[i+1].created_at for i in range(len(orders)-1))
```

### If It's a Design Gap

**What to change:** Update the spec with the missing requirement

**Process:**
1. Identify the missing category (error handling, concurrency, observability, etc.)
2. Add it to the spec explicitly
3. Re-prompt the agent with the updated spec
4. Do not accept the gap as an "implied best practice" — AI coding agents do not reliably infer best practices from context. If it matters enough to notice, it matters enough to specify.

**Example (update spec):**
```
Original spec section:
"When user saves an order, validate all fields and save to database."

Updated spec section:
"When user saves an order:
1. Validate all fields
2. If validation fails, return error with details
3. Save to database
4. If save fails (network error, constraint violation, etc.):
   - Log the error with order_id, timestamp, error_message
   - Return error to user: 'Order could not be saved. Please try again.'
   - Alert ops if database is consistently failing
5. On success, return {success: true, order_id: ..., created_at: ...}"
```

---

## Reading Build/Test Signals Critically

### Don't Assume the Builder is Wrong

When you see output that doesn't match expectations:
- Don't immediately blame the agent
- Read the spec; does it actually require what you expected?
- Read the agent's code; is it logically wrong, or just interpreted differently?
- If the code is defensible under the spec, the spec is incomplete, not the code

### Don't Assume the Spec is Right

- Specs are written by humans and are often incomplete or ambiguous
- Just because something is in the spec doesn't mean it was thought through
- Design gaps are common (missing error handling, missing concurrency control)
- Be willing to update the spec when the agent surfaces a missing requirement

### Diagnose Before Acting

- Take 5 minutes to classify the failure
- Is it spec ambiguity? Don't re-prompt; rewrite the spec
- Is it builder misread? Re-prompt with the relevant section highlighted
- Is it test problem? Fix the test
- Is it design gap? Update the spec — every gap you specify now prevents a build failure later

### Use Spec Updates as Learning

Every time you find spec ambiguity or a design gap, update the spec. Over time, your specs become more complete and production-ready. Future builds from the updated spec will be better.

---

## Example Workflow: Full Diagnosis

**Scenario:** Agent built a feature. Tests fail. You're frustrated.

**Step 1: Read the Failure**
```
Test: test_order_invalid_address
Expected: Order rejected with error "Address not found"
Actual: Order accepted; address field set to empty string
```

**Step 2: Check the Spec**
Spec says: "Validate address is valid before processing."

**Step 3: Read the Code**
Agent checks: if address not None and address != "", then valid.

**Step 4: Diagnose**
- Spec says "address is valid"
- Agent interpreted valid = "not empty"
- You intended valid = "geocoded in USPS"
- This is spec ambiguity (not builder error)
- The agent's interpretation is defensible

**Step 5: Fix Appropriately**
Update the spec (don't re-prompt):

```markdown
Before:
"Validate address is valid before processing."

After:
"Before processing, geocode the address using the USPS Address Validation API:
- If geocoding succeeds (returns a match), address is valid; proceed
- If geocoding fails (no match found), address is invalid; reject with error
- If API timeout (> 5s), escalate to human review (do not auto-reject)

Example valid address: '123 Main St, Springfield, IL 62701' → geocodes to (39.7817, -89.6501)
Example invalid address: '999 Fake Lane, Nowhere' → geocoding returns no match"
```

**Step 6: Re-run**
Give the agent the updated spec; ask it to revise.

Result: Fewer clarifying questions, better builds, faster delivery.
