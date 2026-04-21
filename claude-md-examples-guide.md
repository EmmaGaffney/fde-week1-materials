# CLAUDE.md Examples Guide: Context Engineering for AI Agents

## What is CLAUDE.md and Why It Matters

CLAUDE.md is a project constitution file that configures Claude Code's behaviour for a specific project. Unlike traditional documentation that describes what a system does, CLAUDE.md describes how Claude should think about and work within that system.

When you run `claude code` on a project, Claude reads CLAUDE.md first. This file becomes the cognitive frame—it answers: What is this project? What entities exist? What are the rules? What should I never do? How should I handle uncertainty?

For AI-native development, CLAUDE.md is essential because Claude operates without the ambient knowledge a human developer absorbs through months of working in a codebase. A strong CLAUDE.md compresses that knowledge into actionable instructions, preventing wasted tokens on clarifying questions and misaligned implementations.

## The Three Quality Tiers

### Tier 1: Poor — Generic Boilerplate

This CLAUDE.md wastes tokens and provides no real guidance:

```markdown
# CLAUDE.md

You are a helpful AI assistant. Follow best practices for code quality.
Write clean, maintainable code. Use meaningful variable names.
Always test your work. Ask questions if you're unsure.
```

**Why this fails:**
- "Helpful assistant" and "best practices" are meaningless at context-load time
- No project-specific entities, constraints, or domains
- No escalation rules—when should Claude ask rather than decide?
- No forbidden patterns—what mistakes does this project tend to make?
- Every ambiguity forces a clarifying question

### Tier 2: Acceptable — Project Context With Missing Precision

This provides direction but lacks the specificity needed to prevent build failures:

```markdown
# CLAUDE.md

This is a healthcare staffing platform. We match nurses to shifts.

Core entities: Nurse (name, credentials, availability), Shift (date, role, location), Assignment (nurse + shift + status).

When you write code, ensure data consistency. Validate credentials before assignment. Handle edge cases.

API integrations: We connect to a credentialing system. Details are in docs/api.md.

Don't modify the database schema without approval. Don't write code that locks out the production database.
```

**Why this is acceptable but incomplete:**
- Establishes domain and key entities
- Mentions critical constraints (credentials, approval gates)
- References integration points

**Why it still fails in practice:**
- Entity definitions lack state machines—what are valid Nurse statuses? Shift statuses? When can Assignment move between states?
- No validation rules with acceptance criteria—what makes a credential valid? What's an invalid shift date?
- "Validate credentials" is too vague—connect to the credentialing system? Check local cache? Both?
- No naming conventions—is it `shift_date` or `shiftDate`? `nurse_id` or `nurseId`?
- Integration constraints buried in referenced docs instead of inline
- Escalation patterns undefined—if credential validation fails, should the agent handle it, log it, or raise it?

### Tier 3: Strong — Comprehensive and Agent-Buildable

```markdown
# CLAUDE.md

## Project Purpose
Healthcare staffing platform: match available nurses to open shifts based on credentials, availability, and compliance constraints. System of record for assignments; credentialing system is authoritative for license status.

## Core Entities

### Nurse
Attributes:
- `id`: UUID, immutable
- `name`: string, required
- `email`: string, required, unique
- `credentials`: array of Credential objects (see below)
- `availability`: array of DateRange objects (start_date, end_date, both ISO 8601)
- `status`: enum [ACTIVE, ON_LEAVE, INACTIVE], default ACTIVE
- `created_at`: ISO 8601 timestamp, set on creation, immutable
- `updated_at`: ISO 8601 timestamp, updated on any modification

State machine:
- ACTIVE → ON_LEAVE (nurse requests leave)
- ACTIVE → INACTIVE (admin deactivates; also auto-triggered when all credentials expire — see §Nurse Validation)
- ON_LEAVE → ACTIVE (leave ends)
- INACTIVE → ACTIVE (admin reactivates; requires at least one non-EXPIRED credential)
- No assignment can be created for INACTIVE nurses

### Shift
Attributes:
- `id`: UUID, immutable
- `date`: ISO 8601 date (not timestamp), required
- `role`: enum [RN, LPN, CNA, RT], required
- `location_id`: foreign key to Location, required
- `start_time`: HH:MM in 24-hour format, required
- `end_time`: HH:MM in 24-hour format, required, must be > start_time
- `required_count`: positive integer, minimum 1
- `filled_count`: non-negative integer, calculated field (count of assignments with status CONFIRMED), read-only
- `status`: enum [OPEN, CONFIRMED, CANCELLED], default OPEN
- `created_at`: ISO 8601 timestamp, immutable
- `updated_at`: ISO 8601 timestamp
- `created_by`: foreign key to User (admin or system), immutable
- `notes`: optional string, max 500 characters

State machine:
- OPEN → CONFIRMED (when filled_count == required_count)
- OPEN → CANCELLED (admin cancels)
- CONFIRMED → OPEN (admin revokes an assignment, reduces filled_count)
- CANCELLED → OPEN (admin re-opens)
- Shifts cannot be modified 6 hours before start_time (immutable_threshold = start_time - 6h)

### Assignment
Attributes:
- `id`: UUID, immutable
- `nurse_id`: foreign key to Nurse, required, immutable
- `shift_id`: foreign key to Shift, required, immutable
- `status`: enum [PENDING, CONFIRMED, REJECTED, WITHDRAWN], default PENDING
- `credentials_verified_at`: ISO 8601 timestamp, nullable
- `verified_by`: reference to credentialing system check ID, immutable
- `created_at`: ISO 8601 timestamp, immutable
- `updated_at`: ISO 8601 timestamp
- `created_by`: foreign key to User (system or admin), immutable

State machine:
- PENDING → CONFIRMED (admin approves after credentials verified)
- PENDING → REJECTED (credentials fail validation or admin rejects)
- CONFIRMED → WITHDRAWN (nurse withdraws with 24h notice)
- REJECTED is terminal
- Transition to CONFIRMED requires credentials_verified_at to be non-null

Credential object (nested in Nurse):
- `license_type`: enum [RN, LPN, CNA, RT], required
- `license_number`: string, required
- `issued_date`: ISO 8601 date, required
- `expiration_date`: ISO 8601 date, required
- `status`: enum [VALID, EXPIRED, REVOKED, PENDING_RENEWAL], computed from credentialing system
- `last_verified_at`: ISO 8601 timestamp, updated after each credentialing check
- `verified_by_system`: string identifier of credentialing source

## Naming Conventions
- Database table names: snake_case, plural (nurses, shifts, assignments, locations)
- Column names: snake_case, no type prefixes (id not nurse_id for primary keys; foreign keys: nurse_id, shift_id)
- Enum values: SCREAMING_SNAKE_CASE (ACTIVE, CONFIRMED, PENDING)
- Timestamps: always ISO 8601 with timezone (UTC internally; display tz determined by location_id)
- Foreign key constraints: cascading soft-delete on Nurse; restricting delete on Shift (assignments prevent deletion unless explicitly orphaned)
- API field names: match database column names exactly (no transformation to camelCase)

## Validation Rules and Acceptance Criteria

### Nurse Validation
- `email`: must match regex `^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`
- `credentials`: array length >= 1, at least one non-EXPIRED credential for nurse to be ACTIVE
- `availability`: date ranges must not overlap; start_date <= end_date
- ACTIVE nurse with all EXPIRED credentials → auto-status INACTIVE (trigger in credential sync)

### Shift Validation
- `date`: must be >= today (no historical shifts)
- `date`: must be <= today + 365 days (no shifts beyond 1 year in future)
- `start_time` and `end_time`: must be within location operating hours (location object defines hours)
- `filled_count` <= `required_count` unless status == CANCELLED (no overfilling; equality is the CONFIRMED trigger per the state machine above)
- Cannot create shifts < 48 hours from current time (blocking rule, exception requires admin override flag)

### Assignment Validation
- Nurse must be ACTIVE at time of assignment creation
- Nurse credentials must include license type matching Shift.role, with status != EXPIRED and != REVOKED
- Nurse availability must include Shift.date
- Nurse cannot have two CONFIRMED or PENDING assignments on same date
- Assignment creation triggers immediate credential verification call to external system (sync, not async)
- If credential verification fails, Assignment.status = REJECTED, Assignment.credentials_verified_at = null
- If verification succeeds, Assignment.credentials_verified_at = timestamp, status remains PENDING (awaits admin confirmation)

## Integration Constraints

### Credentialing System Integration
- **Endpoint**: `POST /api/verify-license`
- **Request**: `{ "license_number": string, "license_type": enum, "expiration_date": ISO 8601 date }`
- **Response**: `{ "valid": boolean, "status": enum [VALID, EXPIRED, REVOKED, UNKNOWN], "verified_at": ISO 8601 timestamp }`
- **Timeout**: 5 seconds; if timeout, return `{ "valid": false, "status": "UNKNOWN" }` and log error; assignment moves to REJECTED with note "Credential verification timeout"
- **Retry logic**: on timeout or 5xx error, retry once after 2 seconds; if still fails, reject assignment and alert ops
- **Rate limit**: 100 requests/minute; if exceeded, hold the assignment request until a verification slot opens (max wait: 30 seconds). If the slot does not open within 30 seconds, reject the assignment with reason "credential verification rate limit exceeded" and alert ops. Do not queue for async verification — all credential checks must complete synchronously before any assignment is created (see "sync, not async" above and §What the Agent Should NOT Do)
- **Authentication**: Bearer token in `Authorization` header, rotated quarterly, never logged

### Location System Integration
- **Endpoint**: `GET /api/locations/{location_id}/hours`
- **Request**: path parameter `location_id` (UUID); query parameter `date` (ISO 8601 date, optional — defaults to today)
- **Response**: `{ "location_id": UUID, "date": ISO 8601 date, "open_time": "HH:MM", "close_time": "HH:MM", "is_open": boolean }`
- **Timeout**: 3 seconds; if timeout, treat as "location hours unavailable" (see Fallback below)
- **Retry logic**: on timeout or 5xx error, retry once after 1 second; if still fails, treat as unavailable
- **Rate limit**: 200 requests/minute; read-only endpoint, no write-side rate concerns
- **Authentication**: same Bearer token as credentialing system (shared service auth)
- **Caching**: location hours cached locally with 24-hour TTL; cache invalidation is manual trigger via admin API only (no auto-sync)
- **Fallback**: if location hours unavailable (timeout, error, or cache miss with API failure), do not default to assumed hours — escalate per §Handling Ambiguity and Escalation item 3 (ask the user to load location data first)

## What the Agent Should NOT Do

- Never modify Credential status directly; it is computed from the credentialing system
- Never create assignments for INACTIVE or ON_LEAVE nurses, even if explicitly instructed (guard rail)
- Never soft-delete shifts; only use CANCELLED status (data integrity for audit)
- Never modify created_at, created_by, nurse_id, or shift_id on existing assignments
- Never bypass credential verification for assignment approval
- Never automatically transition PENDING → CONFIRMED; always require explicit admin action
- Never create bulk assignments without individual credential verification
- Never modify credentialing system data in this service; queries only

## Handling Ambiguity and Escalation

If a request is ambiguous, ask before acting. Specific escalation patterns:

1. **Credential Verification Failure**: Log with nurse_id, shift_id, failure reason. Flag assignment as REJECTED. Do NOT escalate unless verification timeout (alert ops). Nurse can retry with corrected credential.

2. **State Machine Violation** (e.g., attempt to CONFIRM assignment for INACTIVE nurse): Reject request with clear error message citing the entity status and the rule preventing the transition. Do not override.

3. **Missing Location Data**: Do not use fallback hours; ask the user to load location data first.

4. **Shift Date Edge Case** (e.g., request for shift < 48 hours away): Reject by default. If user explicitly provides `override_blocking_rule: true` in request, allow with admin audit log entry.

5. **Conflicting Nursing Assignments**: Reject the second assignment with clear conflict message. Do not attempt to auto-resolve.

6. **Ambiguous Availability Ranges**: If user specifies availability but dates are unclear (e.g., "around March"), ask for exact start_date and end_date before creating.

## When to Ask vs When to Decide

**Decide alone** (do not ask):
- Validating input against documented rules
- Computing filled_count from confirmed assignments
- Rejecting state machine violations
- Setting timestamps (created_at, updated_at, last_verified_at)
- Normalizing enums to SCREAMING_SNAKE_CASE

**Ask the user** before proceeding:
- Requests that require override flags (blocking_rule overrides, admin escalations)
- Requests involving data outside the model (e.g., "sync credentialing system" or "load external data")
- Ambiguity in entity relationships or state transitions not covered in this spec
- Changes to the validation rules themselves
```

**Why this is strong:**
- Every entity defined with attributes, state machines, and constraints
- Validation rules include acceptance criteria (what makes something valid?)
- Naming conventions consistent across the project
- Integration contracts explicit—request format, response format, timeout, retry, rate limit
- Clear guard rails—what the agent should refuse to do
- Escalation patterns specific (credential timeout → ops alert; missing location → ask user)
- Timestamps and audit fields defined (who created it, when was it modified)

## Common Failure Modes

### Too Vague
Problem: "Make sure data is consistent."
Fix: Specify which data can be modified, which is immutable, which is computed. Give examples.

### Too Long
Problem: CLAUDE.md becomes 10,000 words with full code examples and historical context.
Fix: Keep it focused. Full code examples belong in docstrings. Historical context belongs in a design doc.

### Contradictory Instructions
Problem: "Always ask before modifying the database" + "Proceed with migrations autonomously."
Fix: Make escalation rules explicit. When should the agent ask? When should it decide alone?

### Missing Scope Boundaries
Problem: Integration section mentions "external systems" without naming them.
Fix: List every external dependency. For each: endpoint, authentication, timeout, fallback behaviour.

## How to Iterate: Use Claude Code to Critique Your Own CLAUDE.md

Once you've written an initial CLAUDE.md, test it:

1. **Ask Claude Code to write a build spec** for a feature and flag any ambiguities in CLAUDE.md that forced clarifying questions. Those are gaps.

2. **Ask Claude Code to validate an entity definition** against the rules you wrote. "For a Nurse with status ACTIVE but all credentials EXPIRED, what should happen?" If Claude has to guess, the rule is incomplete.

3. **Ask Claude Code to trace a failure scenario**: "Show me the state of Assignment if credential verification times out." If the trace requires invention, the spec is incomplete.

4. **Ask for escalation path reviews**: "If a user requests an override to the 48-hour shift-creation block, what happens?" If the answer is "Claude would ask the user," then you need an explicit override mechanism in the spec.

5. **Iterate in small changes**: Rewrite one entity at a time. Validate it. Move to the next.

A strong CLAUDE.md is the difference between "Claude asked 20 clarifying questions before writing a line of code" and "Claude built it in one pass and got 90% right."
