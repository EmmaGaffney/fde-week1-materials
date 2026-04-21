# FDE Accelerated Development Program — Welcome & Week 1

Welcome to the program. This document is your starting point. It covers:

- What the program is and what to expect
- The tooling you need installed on the Friday 17 April prep day
- How the calendar works (virtual weeks vs physical dates)
- Your Week 1 goals, calendar, and what you'll hand in
- What to do if something goes wrong (credit, tooling, blockers)

Keep this document pinned. Read the Intro section once and come back to the Week 1 section whenever you need it. Separate week-specific files (`README-Participants-Week2.md` through `README-Participants-Week5.md`) will guide you through weeks 2–5 — each one becomes your reference for that week, to be shared prior the beginning of the relevant week.

---

## Part 1 — Intro: What the Program Is

### What is an FDE?

A **Forward Deployed Engineer** works at the intersection of two things: designing and specifying AI-native systems, and transforming how businesses actually run by delegating cognitive work to AI agents. You don't write most of the code yourself — you write specifications precise enough for AI coding agents to build from, you validate what those agents produce, and you iterate until the solution works.

The FDE's durable asset is not what you know — it is how you orient when you don't know. That sentence is the single most important idea in this program. You will see it again.

### The next 5 weeks

Five weeks, five themes, one gate at the end of each week:

| Week | Theme | What you'll do |
|---|---|---|
| **Week 1** | AI-Native Specification | Given a business problem, design an agentic solution and write a spec an AI coding agent can build from |
| **Week 2** | Cognitive Work Assessment & Agent Design | Assess a real business process, decompose the cognitive work, design the agent |
| **Week 3** | End-to-End AI-Native Engagement | Run the full FDE engagement arc — discovery, specification, build-loop diagnosis, stakeholder management |
| **Week 4** | Economics, Governance & Platform Strategy | Build the business case, govern quality across engagements, defend your capstone proposal |
| **Week 5** | Capstone + Final Exam | Design and build a working prototype for a sealed scenario, then take a solo 8-hour hybrid final exam |

At the start of each week you'll get a weekly intro document (like this one) so you know what's being asked of you and how the week is shaped.

### The weekly rhythm

Every week of the program follows the same rhythm:

- **Start of week (Virtual Monday):** Coach-led orientation — your coach walks you through the week's objective and resources. About one hour.
- **Virtual Monday through Thursday:** Self-directed practice work. You pick your path through the resources, run your own build loops, and meet with your squad daily for short standups.
- **Virtual Wednesday:** Live mid-week checkpoint with your coach. Not a grade — a chance to get diagnosis on your in-progress work.
- **Virtual Friday morning (Weeks 1–2):** Peer cross-review. You read two peers' work against the week's rubric and they read yours. 90 minutes.
- **End of week:** Timed gate exercise — a previously unseen scenario, sealed until the moment you start. This is how your week is scored. For most weeks the gate sits on Virtual Friday afternoon; **Week 1 is the exception** — Gate 1 runs on the Monday after Virtual Friday so new participants get a weekend buffer before their first gate. See your Week 1 calendar in Part 5 below for the specific dates.
- **Following the gate:** Gate results delivered early the next working day.

The rhythm is stable. What changes week to week is the content — and the specific physical day each gate lands on, which is always listed in the relevant week's calendar.

### How to read the calendar — virtual weeks vs physical dates

This program runs on **virtual week days** (Monday, Tuesday, Wednesday, Thursday, Friday) for each of the 5 weeks. These are the positions within a week — not necessarily the physical calendar days those positions fall on.

Throughout the program, you will see entries in the form:

> **Mon 20.04.2026 ("Monday, Week 1")**

The **physical date** on the left is when the event actually happens in real life. The **virtual day in brackets** tells you where you are within the 5-day week structure (Monday through Friday).

Two specific things you need to know about physical dates vs virtual days:

1. **Week 1 has a dedicated prep day on Friday 17 April 2026**, a weekend, and then a 5-day virtual week that maps 1:1 to physical Monday–Friday (Mon 20.04 = Virtual Monday, Tue 21.04 = Virtual Tuesday, and so on). **Gate 1 is held on Monday 27 April** — the Week 1 gate is deliberately pushed a day past Virtual Friday so new participants get a weekend buffer between peer review and the gate. The Week 1 table below shows the full mapping.
2. **Weeks 2–5 normally start on a physical Tuesday** rather than a physical Monday — there is a weekend gap after each gate. Specific physical dates for Weeks 2–5 shift further due to public holidays on **Friday 1 May** (Labour Day) and **Thursday 14 May** (Ascension Day). Your coach team confirms the exact physical date for each virtual day in the **Teams General channel** at the start of every week.

When in doubt, go by the **virtual day** (it tells you what to do) and check the Teams General channel for the **physical date** (it tells you when to do it).

### How hard is this?

Honest answer: 20–30% of participants graduate the program. The other 70–80% do not. This is the program's design target — a filter that identifies people who can operate in ambiguity, learn what they need to learn, and produce work that meets a professional standard under pressure. If that sounds like you, you're in the right place. If it doesn't, an early honourable exit is a better outcome than a late failure — tell your squad lead or the program lead and we will not argue you into staying or out of staying.

You will hear this framing again. It is not meant to intimidate you. It is meant to be honest so that nothing surprises you later.

### A note on rubrics

The specific rubric (criteria, weights, and pass thresholds) for **Gates 1 through 4 and the Week 5 Final Exam** is **sealed until the start of that gate exercise**. This is deliberate — it keeps you focused on doing the work well rather than gaming a numeric weight table. When each of those gates begins, your coach will share the rubric along with the scenario. **The Week 5 Capstone is the one exception** — its rubric is released at the start of Virtual Monday of Week 5, because the 4-day capstone format makes it valuable to have the criteria visible while you design and build. Details for Week 5 are in `README-Participants-Week5.md`. The deliverable lists in each week's file tell you exactly what to submit; the weighting tells you where to prioritise effort once the exercise is running.

---

## Part 2 — Tooling

You will use **Claude Code** as your primary working tool across all five weeks. Cursor, GitHub Copilot, and Dial are supporting tools you can combine with it.

### Claude Code (primary) — via CodeMie CLI

Install Claude Code through the CodeMie CLI wrapper:

- **Repository:** https://github.com/codemie-ai/codemie-code
- **Install:** follow the repo README. Typical flow is `codemie setup` → `codemie install claude --supported` → verify with `codemie-claude`.

**Model selection matters.**
- **Use standard Claude models as your default.** They are sufficient for most of the work — drafting specs, running build loops, diagnosing build signals, writing ADRs, producing cognitive-work assessments.
- **Limit premium/opus-class models to cases that actually need them.** Complex multi-step reasoning, tangled delegation boundaries, cross-capability consistency on large specs, one-shot "I need this right" moments.
- **Check your usage daily** at https://codemie.lab.epam.com/#/settings/profile. Check before every Friday gate. If your burn rate is trending toward running out before Week 5, adjust immediately.

### Cursor (supporting, optional)

Cursor can be requested via the EPAM support portal. Cursor has agentic capabilities and can be combined with Claude Code where your workflow benefits from using both.

### GitHub Copilot (supporting, optional)

Access via the EPAM Leap portal: https://leap.epam.com/assistants/github-copilot?page=1

GitHub Copilot has agentic capabilities and can be combined with Claude Code where your workflow benefits from using both.

### Dial (encouraged)

We also encourage you to use **Dial**, EPAM's chat interface: https://chat.lab.epam.com/

Dial is useful for quick model queries, comparing responses across models, and conversational exploration where a chat interface fits better than a full coding workflow.

### Minimum setup checklist (complete on the prep day, Fri 17.04)

Your dedicated setup day is **Friday 17 April 2026**. After the short kickoff plenary in the morning, the rest of the day is for working through this checklist. If anything slips, use the weekend (Sat–Sun 18–19.04) as a buffer. Week 1 begins in earnest on **Monday 20 April**, and you should arrive at the Virtual Monday orientation with a working environment.

- [ ] CodeMie CLI installed
- [ ] Claude Code installed via `codemie install claude --supported` and verified with `codemie-claude`
- [ ] CodeMie profile accessible at https://codemie.lab.epam.com/#/settings/profile — current credit visible
- [ ] GitHub account active, able to create and push to repositories
- [ ] Basic command-line terminal familiarity (clone, cd, git add/commit/push)
- [ ] Cursor installed (if requested via support portal)
- [ ] GitHub Copilot access enabled (if using)
- [ ] Dial accessible at https://chat.lab.epam.com/
- [ ] Pre-reading complete: `Sources/the-fde.md` (the FDE role definition) and Addy Osmani's "The 70% Problem" - https://addyo.substack.com/p/the-70-problem-hard-truths-about (plus there are some nice videos on the topic, google them)

If any item is incomplete, flag it in the Teams Support channel. Do not arrive at your Week 1 orientation with a broken environment.

---

## Part 3 — If You Run Out of Credit

You start the program with a CodeMie credit allocation. If you are careful with model selection you should finish Week 5 with credit to spare. If you run out before then:

### Step 1 — Contact Klimentiy Misyuchenko

Write to **Klimentiy Misyuchenko** (Resource Librarian / credit contact for the program) with your CodeMie profile ID and a brief note on what triggered the high burn. An additional **$200 USD budget** will be enabled on your account.

### Step 2 — After the $200 top-up: personal licenses

If you exhaust the $200 top-up, the next step is to use your own personal licenses. No further program top-ups are provided. The program contains **no EPAM or client-sensitive data**, so personal licenses are acceptable — and you can use your own license from Day 1 if you prefer, especially if you already have a Claude Code Max subscription.

---

## Part 4 — Key Contacts & Where Things Live

| Role | Who | Contact for |
|---|---|---|
| **Program Lead** | **Aliaksandr Kaliadka** | Escalations, exit decisions, anything you cannot resolve with your coach |
| **Your coach** | Assigned at Week 1 orientation | Office hours, gate review questions, live feedback |
| **Your squad lead** | Assigned at Week 1 orientation | Daily standups, logistics, peer review assignments, first-line support |
| **Resource Librarian / credit & tooling contact** | **Klimentiy Misyuchenko** | Credit top-ups, CodeMie profile issues, tooling access, where to find templates, navigation of resource materials |

The **cohort MS Teams space** is where all of this lives. Program announcements go to the **General** channel. Tooling and credit questions go to the **Support** channel. Squad standups and squad-internal discussion go to your **Squad channel**.

---

# Part 5 — Week 1: AI-Native Specification

## Your goal this week

Given an ambiguous business problem, decompose it into a solution where the core mechanism is an AI agent. Produce a specification precise enough for an AI coding agent (Claude Code) to build from. Use AI iteratively to test, refine, and verify that spec.

## By Friday, you must demonstrate that you can:

- Frame a business problem from both user and business perspectives, with measurable success criteria
- Determine that the right solution is an AI agent — not traditional software, not RPA, not a human process change
- Produce a specification in a format an AI coding agent can execute against
- Use AI iteratively: prompt → assess output → refine → re-prompt until the spec converges on quality
- Identify what you don't know, what you're assuming, and what questions remain for the client

## Week 1 calendar

Week 1 gives you a dedicated **prep day on Friday 17 April** (tooling setup, pre-reading, reading through the supporting materials), a weekend, and then a standard 5-day virtual week that maps 1:1 to physical Monday 20.04 – Friday 24.04. **Gate 1 is held on Monday 27 April** — a weekend after Virtual Friday so new participants get a short buffer between peer review and the gate.

**All times are CET.** Specific start times for coach-led sessions are confirmed in the Teams General channel during the week.

| Physical date | Virtual day | Main event |
|---|---|---|
| **Fri 17.04.2026** | **Program prep day** *(pre-week)* | Program kickoff (short plenary session, you'll learn about the 7 Week 1 practice scenarios); then **complete tooling setup, pre-reading, and reading through the supporting materials** — this is your dedicated setup day before Week 1 proper begins |
| **Mon 20.04** | **Monday, Week 1** | Coach-led week orientation; **pick your practice scenario** from the 7-scenario pool; self-directed practice begins |
| **Tue 21.04** | **Tuesday, Week 1** | Self-directed practice; **scenario switch cutoff** by 18:00 CET (one switch allowed before this) |
| **Wed 22.04** | **Wednesday, Week 1** | **Critique pool submission** deadline 12:00 CET; **coach critique session** in the afternoon |
| **Thu 23.04** | **Thursday, Week 1** | Self-directed practice; **finish your required closed build loop** by end of day |
| **Fri 24.04** | **Friday, Week 1** | **Peer cross-review** 09:30–11:00 CET (submission due 09:15); feedback incorporation window 11:00–12:00; optional final practice / preparation in the afternoon |
| **Mon 27.04** | **Gate 1 day** | **Gate 1 timed exercise (2.5h) in the morning** (sealed scenario released at the start of the exercise), followed by **live walkthroughs with a coach** in the afternoon. Week 2 begins on Tuesday 28.04. |

The mapping from physical Monday 20.04 through Friday 24.04 is 1:1 to virtual Monday through Friday — **Monday is Monday, Tuesday is Tuesday**, and so on. The only two things that make Week 1 slightly different from Weeks 2–5 are: (a) the dedicated prep day on Friday 17 April, and (b) Gate 1 sitting on the following Monday (27.04) instead of immediately on Virtual Friday afternoon. Gate 1 results are delivered later in Week 2 — your coach team will confirm the exact day in the Teams General channel.

## Picking a scenario

You'll first hear about the **Week 1 Practice Scenario Pool** at the program kickoff on the prep day (Fri 17.04). At the Virtual Monday Week 1 orientation (physical Mon 20.04), your coach walks through the 7 scenarios in detail and you pick one by end of that day. You can switch once before the following evening (Virtual Tuesday / physical Tue 21.04, 18:00 CET); after that your scenario is locked. Everything you do for the rest of Week 1 is anchored to your committed scenario.

The 7 scenarios are pre-framed agentic business problems (HR onboarding, vendor contract review, retail returns, community moderation, clinic patient intake, academic journal triage, municipal permit screening). **The full text of all 7 scenarios, selection rules, and guidance on how to work with your chosen scenario is in a separate document: [`README-Participants-Week1-Scenarios.md`](./README-Participants-Week1-Scenarios.md).** You will receive this document alongside the other Week 1 materials on the prep day.

## The closed build loop (required)

Sometime between Virtual Monday (Mon 20.04) and end-of-day Virtual Thursday (Thu 23.04) you must complete at least one full closed loop against your scenario:

1. Draft a spec for one feature or capability
2. Hand it to Claude Code — let it build
3. Review the output; identify at least one gap between what you asked for and what got built
4. Diagnose the gap: is it a **spec ambiguity** (you own the fix), a **builder misread** (the builder owns the fix), or an **unjustified builder addition** (ask for rollback)?
5. Apply the fix to the root cause
6. Re-run and verify the fix actually closed the gap

This is not graded on Gate 1 — but your squad lead tracks whether you did it. Skipping it is an engagement signal coaches follow up on. The loop is how you learn the difference between "I wrote a spec that looks buildable" and "I saw what my spec actually produces when an AI agent reads it." Do it honestly.

## The Virtual Wednesday coach critique session

On **Wednesday Week 1** (physical Wed 22.04) in the afternoon, your full cohort attends a 60-minute live critique session with a coach. Beforehand, you submit one anonymised draft (problem statement + spec + validation design + short self-diagnosis note) to the critique pool by **12:00 CET that day**. The coach picks 4 weak examples from across the cohort and walks through them live — diagnosing what isn't buildable yet and showing the minimum change that would fix it. You will not know in advance which examples are selected. This session is the fastest way in the program to learn what "buildable by an AI coding agent" actually means.

## Virtual Friday morning peer cross-review

By **09:15 CET on Fri 24.04** (Virtual Friday) you submit your strongest Week 1 practice artefact to the peer review pool. Between 09:30 and 11:00 you read two cross-squad submissions, score them against the Gate 1 rubric (which you will have at that point), and return specific written feedback. You also receive feedback on your own submission from two cross-squad peers. Between 11:00 and 12:00 you incorporate the feedback. The afternoon is yours for optional final practice or preparation. You then have the weekend (Sat 25, Sun 26) as a buffer before **Gate 1 on Monday 27.04 — the timed exercise starts in the morning, with live walkthroughs in the afternoon**. You **cannot** reuse your peer-reviewed artefact as your Gate 1 submission — Gate 1 is a fresh, previously unseen scenario.

## Gate 1 — what you'll hand in

On **Monday 27 April** (Gate 1 day) you receive a sealed scenario (released at the start of the exercise) in the **morning**. You have **2.5 hours** to produce **5 deliverables**:

1. **Problem statement & success metrics** — frame the problem from user *and* business perspectives, with measurable outcomes that justify the investment
2. **Delegation analysis** — which parts of the workflow become fully agentic, which are agent-led with human oversight, which stay human-led? Justify each boundary
3. **Agent specification** — purpose, scope, inputs/outputs, decision logic, escalation triggers, integration points. Precise enough that an AI coding agent could start building from it
4. **Validation design** — how do you know the agent is working? What do you test? What does failure look like?
5. **Assumptions & unknowns** — what are you assuming about the client's data, systems, and organisation? What must be validated before building?

After submission, you give a **10-minute live walkthrough** to a coach in the afternoon — roughly a 3-minute summary plus 7 minutes of coach questions on why you drew the delegation boundaries where you drew them. This is longer than the standard Gate 1 walkthrough format because the extended Week 1 schedule and the morning gate start give coaches more afternoon time for substantive review — use it: the walkthrough is where coaches most directly assess your judgment.

**The full Gate 1 rubric (criteria, weights, and pass threshold) is shared at the start of the gate exercise.** Until that moment, focus on the deliverables list and the "what coaches are looking for" guidance below — that is enough to tell you where to put your effort.

## What coaches are looking for

Three things more than anything else:

1. **Delegation boundaries are defensible, not arbitrary.** If you can't explain why a specific task is fully agentic versus human-overseen, you haven't done the thinking yet.
2. **The spec is precise enough that an AI coding agent wouldn't need to ask a clarifying question.** The critique session and your own closed build loop are both designed to teach you what this feels like. Use them.
3. **Assumptions and unknowns are honest.** "I don't know" beats a plausible-sounding guess. At least 5 genuine unknowns, not filler.

## Week 1 Suggested Resource Library

These are starting points, not assignments. Navigate them based on what you need to achieve the week's objective.

**AI-Native Development:**
- Anthropic: "Building Effective AI Agents" — https://www.anthropic.com/research/building-effective-agents
- Anthropic: "Effective Context Engineering for AI Agents" — https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
- Addy Osmani: "AI Coding Workflow" — https://addyosmani.com/blog/ai-coding-workflow/
- Anthropic Academy: Claude Code in Action (Skilljar) — https://anthropic.skilljar.com/claude-code-in-action
- Anthropic: Claude Code documentation — https://code.claude.com/docs
- Anthropic: How Anthropic teams use Claude Code — https://claude.com/blog/how-anthropic-teams-use-claude-code
- DeepLearning.AI: Prompt Engineering for Developers — https://www.deeplearning.ai/short-courses/chatgpt-prompt-engineering-for-developers/

**Specification as Agent Input:**
- Martin Fowler: Given-When-Then — https://martinfowler.com/bliki/GivenWhenThen.html
- BDD Guide (Inviqa) — https://inviqa.com/blog/bdd-guide
- ForgeCode: AI Agent Best Practices — https://forgecode.dev/blog/ai-agent-best-practices/
- Tweag: Introduction to Agentic Coding — https://www.tweag.io/blog/2025-10-23-agentic-coding-intro/

**Problem Framing & Business Thinking:**
- Jeff Patton: Story Mapping Quick Reference — https://jpattonassociates.com/story-mapping-quick-ref/
- `Sources/the-fde.md` — sections on Validation Design, Root Cause Diagnosis

**Claude Code Workflow:**
- CLAUDE.md examples and failure modes — `claude-md-examples-guide.md`

**Specification craft (in-folder):**
- Specification-quality reference: `production-spec-checklist.md`
- Build-loop diagnostic taxonomy: `spec-ambiguity-vs-builder-mistakes.md`

**FDE role (pre-reading):**
- `Sources/the-fde.md`
