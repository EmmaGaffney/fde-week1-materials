# Week 1 — Practice Scenario Pool

This document contains the 7 practice scenarios for Week 1. You will pick **one** of these scenarios at your Virtual Monday orientation (physical Mon 20.04.2026) and work against it for the rest of Week 1 — closed build loop, critique session submission, and Friday peer cross-review all use the same committed scenario.

For the Week 1 rhythm, deliverables, and gate format, see **`README-Participants-Intro-Week1.md`** § Part 5.

---

## Why the pool exists

Each problem has been shaped so that an agentic solution is the right answer **and** the human/agent boundary is genuinely non-obvious. That non-obviousness is the Week 1 skill this pool exists to train: participants who can draw a defensible delegation boundary on Monday–Thursday practice work will be ready for Gate 1's previously unseen scenario.

The pool also solves a comparability problem for peer review. With 7 shared scenarios, every reviewer encounters at most 7 universes of context rather than 75 different self-invented domains — and can absorb a submission in 15 minutes and score it fairly against the Gate 1 rubric. The Wednesday critique session benefits from the same bound.

---

## Selection rules

- **Pick one scenario by end of Virtual Monday** (physical Mon 20.04). This is your Week 1 practice scenario.
- **You may switch scenarios once, by end of Virtual Tuesday** (physical Tue 21.04, 18:00 CET hard cutoff) if you realise your pick isn't letting you practice a specific skill. After Tuesday 18:00, your scenario is locked for the rest of the week.
- **The critique session submission (Wednesday 12:00 CET) and the Friday peer review submission (Friday 09:15 CET) must use the same scenario** — this is what makes cohort comparison meaningful.
- **You may not invent your own scenario** for Week 1 practice. If you have a compelling reason to use a domain outside the pool, raise it with a coach during Monday office hours — approval is at coach discretion and must be recorded in the critique pool naming convention.
- **Squad leads encourage distribution across the 7 scenarios within a squad** (aim for all 7 represented in squads of 7–8 people), but this is a soft guideline, not an assignment. Distribution makes squad standups and informal peer exchange more productive because participants are not all staring at the same problem.
- **Scenario interpretation may be discussed with peers and with AI; your artefacts remain individual work.**

---

## What every scenario in the pool shares

So you know what to expect, regardless of which one you pick:

- **An incumbent human team currently under operational pressure** — there is always a named stakeholder role asking for help
- **Concrete volume, headcount, and accuracy metrics** — the numbers are precise enough to force real success-metric reasoning
- **A non-obvious delegation boundary with at least one hard constraint** that prevents pure automation — drawing that boundary defensibly is the skill being trained
- **At least one real-world system or compliance constraint** — not a greenfield build
- **No "right answer"** — two strong FDEs could draw the boundary in two defensible places for the same scenario

---

## Scenario 1 — HR Onboarding Coordination

> A regional professional-services firm (1,200 employees, 220+ hires per year) runs new-hire onboarding through a 3-person HR Ops team. Each onboarding spans ~40 tasks across 2 weeks: IT provisioning, benefits enrolment, compliance training assignment, buddy matching, welcome materials, 30-day checkpoint scheduling, and manager handoff. Tasks originate from 6 different systems. Roughly 15% require judgment calls — which compliance track applies to a contractor versus a full employee, whether a buddy assignment crosses seniority norms, whether a late I-9 triggers a hold.
>
> The HR Ops lead says: "Most of this is paperwork my team should not be touching, but every time we try to automate, something falls through the cracks because the edge cases never look the same twice." Their stack is Workday for core HR, ServiceNow for IT requests, a separate LMS for compliance training, and email for everything the other three don't cover. They have no AI infrastructure today.
>
> Design the agentic solution.

---

## Scenario 2 — Vendor Contract Clause Review

> A mid-sized B2B software company's legal team (4 lawyers, 1 paralegal) reviews ~300 inbound vendor contracts per quarter. Each contract runs 15–40 pages and must be checked against the company's negotiation playbook: liability caps, data processing addenda, termination clauses, IP ownership, SLA commitments, governing law, indemnity scope. Roughly 70% of incoming contracts have standard terms matching the playbook; 20% have negotiable deviations the paralegal can redline; 10% contain clauses that must be escalated for senior-lawyer review before any counteroffer goes out.
>
> Average review time is 90 minutes per contract. Turnaround is 4–6 business days, which the procurement team considers unworkable. The general counsel is comfortable with AI-assisted review but has one hard rule: no counteroffer may leave legal's queue without a named lawyer's sign-off on the specific clauses being negotiated.
>
> Design the agentic solution.

---

## Scenario 3 — Retail Returns Disposition

> A mid-sized apparel retailer (60 stores, online channel, ~$280M annual revenue) processes ~4,500 returns per week through a central returns facility. Each item is classified by reason code (defective, didn't fit, changed mind, wrong item shipped, suspected fraud), inspected for condition, and routed to a disposition lane: restock as new, discount to outlet, refurbish, donate, or destroy. Twelve returns processors currently do all of this visually and by memory.
>
> Current reason-code accuracy is ~78%. Fraud detection catches ~40% of suspected-fraud cases — the other 60% only surface in chargeback reports weeks later. The VP of Operations wants agentic handling for the easy cases and human judgment preserved for suspected fraud, high-value items, and anything a customer has escalated. She is also under a margin-improvement mandate from the CFO and needs a credible payback story, not a "maybe someday" pitch.
>
> Design the agentic solution.

---

## Scenario 4 — Community Content Moderation

> An online community platform for tabletop-miniature hobbyists (~180K active users, ~12K posts per day across forums and galleries) has grown past its moderation team (8 volunteer moderators, 2 paid staff). The community policy is 14 pages long and covers harassment, off-topic content, commercial spam, self-promotion limits, doxxing, and IP disputes over custom sculpts.
>
> Most moderation decisions are routine — obvious spam, clear rule violations, miscategorised posts. A persistent ~3% of posts sit in genuine grey zones: hobbyist critique that reads harsh, commercial posts from long-standing community members, IP claims from sculptors who may or may not hold rights, cultural references that read differently across regions. The community manager says: "We need something that handles the obvious 97% instantly, flags the grey 3% for human review with real context, and never makes a decision that will damage community trust. We will absorb a lot of false positives to avoid one viral false negative."
>
> Design the agentic solution.

---

## Scenario 5 — Small-Clinic Patient Intake

> A 6-physician family medicine practice (2 locations, ~180 patients per day) runs its patient intake through a 4-person front-desk team. The intake process for each visit spans: insurance verification, prior-authorisation check for scheduled procedures, pre-visit questionnaire, reason-for-visit triage (routine / urgent / same-day), medication reconciliation, and allergy-flag review. Physicians regularly discover at the visit that something was missed in intake — most commonly an expired prior auth or an unreviewed medication change.
>
> The practice manager wants to offload the administrative load to an agentic workflow but has three hard constraints: (1) no clinical judgment by the agent, (2) any contact with the stated visit reason must preserve a clear human escalation path, (3) HIPAA and state medical-records compliance is non-negotiable. They use athenahealth for EHR and a separate tool for insurance eligibility. They have no AI infrastructure today.
>
> Design the agentic solution.

---

## Scenario 6 — Academic Journal Desk-Review Triage

> A mid-tier academic journal in applied statistics receives ~1,400 manuscript submissions per year. The editor-in-chief (1 person, 20% of their time on this role) and 3 associate editors currently run a desk-review step before sending any paper to peer reviewers: check scope fit, methodology adequacy, novelty threshold, obvious plagiarism, baseline writing quality. Roughly 55% of submissions are desk-rejected at this step. Average time-to-first-decision is currently 11 weeks, and the peer-reviewer pool is burning out because too many marginal papers reach them.
>
> The editor-in-chief wants agentic help with the desk-review step but is acutely sensitive to two failure modes: rejecting a paper that would have been a breakthrough, and letting through a fundamentally flawed paper that wastes a peer reviewer's time. Reputation matters. The journal runs on a thin editorial budget and cannot absorb a scandal.
>
> Design the agentic solution.

---

## Scenario 7 — Municipal Building Permit Screening

> A mid-sized city's building department (18 plan reviewers, 3 senior plans examiners) processes ~5,800 residential permit applications per year. Each application must be checked against the International Building Code, state amendments, and ~40 pages of local amendments covering zoning, fire, energy, and accessibility. Roughly 65% of applications have clean documentation and routine compliance questions; 25% have correctable deficiencies that get kicked back with comments; 10% require senior plan-examiner review for unusual structural, historic-district, or variance issues.
>
> Current average time-to-first-review is 19 business days, and the local development community considers this a crisis. The department director wants agentic pre-screening to compress the easy cases and surface real issues faster. The city council has mandated one non-negotiable: **no permit decision — approval, rejection, or kickback — may be made without a named human plan reviewer attached to the record.**
>
> Design the agentic solution.

---

## How to work with your chosen scenario

Once you've committed to a scenario, you will produce (over Virtual Monday–Thursday) the artefacts you will need for the Wednesday critique session submission and the Friday peer review submission:

1. **A problem statement and success metrics** tied to the numbers given in your scenario (not generic business-speak)
2. **A delegation analysis** that names which parts of the work become fully agentic, agent-led with human oversight, or stay human-led — and **why**. The "why" is the skill being tested.
3. **A first-draft capability specification** for the agentic part: purpose, scope, inputs/outputs, decision logic, escalation triggers, integration points. Target 6–10 requirements minimum. Precise enough that an AI coding agent could start building from it.
4. **A first-draft validation design** with at least 3 scenarios spanning happy path, edge case, and failure mode — including at least one failure scenario that tests the delegation boundary itself.
5. **An honest assumptions & unknowns section** — at least 5 genuine unknowns, not filler. What are you assuming about the client's data, systems, and organisation? What must be validated before building?

Between Virtual Monday and end-of-day Virtual Thursday you must also complete at least one full **closed build loop** against your scenario: draft a spec → hand it to Claude Code → review what got built → diagnose the gap → fix the root cause → re-run → verify. This is the move that turns "I wrote something that looks buildable" into "I saw what my spec actually produces."

---

## A note on what Week 1 is NOT testing

Week 1 Gate 1 uses a **previously unseen scenario** (released at the start of the gate exercise on Monday 27.04). It is **not** one of the 7 scenarios in this pool. The pool is for practice, not for the gate. Your week's work on a pool scenario does not transfer directly into a gate submission — you cannot reuse your peer-reviewed artefact as your Gate 1 answer. The pool is how you **rehearse the move**; the gate is where you **prove you can do it on a fresh problem**.

---

## Where to find more

- **How to read the Week 1 calendar and where these scenarios fit into the week:** `README-Participants-Intro-Week1.md` § Part 5
- **Specification quality reference:** `production-spec-checklist.md`
- **Build-loop diagnostic taxonomy (used in your closed build loop and the Wednesday critique session):** `spec-ambiguity-vs-builder-mistakes.md`
- **CLAUDE.md examples:** `claude-md-examples-guide.md`
