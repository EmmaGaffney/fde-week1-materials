# FDE Week 1 — A Thinking-Discipline Primer

**Frame:** Treat Week 1 as a mini-discovery. You don't have access to the real business or real customers yet — your "customer" is your coach, who can role-play stakeholders, and the rest you fill in with explicit assumptions and hypotheses. Borrowing Cagan: your job is to answer *"what's worth building, for whom, and will it actually work?"* — and to be brutally honest about which answers are tested versus assumed.

**The stakes:** The problem you choose this week is the one you will develop in the weeks that follow. Everything you build, test, and demo from Week 2 onwards rests on the framing you set now. Week 1 is where the vision of the solution takes shape — not the features, but the shape of what "good" looks like for this customer.

**The final outcome:** By the end of Week 1 you walk out with a spec for one feature or capability — precise enough that you can hand it to Claude Code and have it built. The spec is the deliverable; the closed build loop against Claude Code is how you pressure-test it. Everything else in the week — problem statement, delegation analysis, validation design, assumptions log — exists to make that spec sharp enough to survive contact with an AI builder.

---

## The one thing to internalise

Week 1 is your first week producing AI-native artefacts against a real scenario — but without a real customer. You cannot validate your framing against end users; the closest thing you have to a stakeholder is your coach in role-play.

That shifts the bar. You're judged on two things: the **quality of the artefacts** you produce (the deliverables list) **and the honesty of the reasoning** behind them. Every non-trivial claim in your problem statement, delegation analysis, agent spec, and validation design is either (a) tested through a coach role-play, (b) derived from something you can reasonably cite, or (c) assumed — and in that case, you say so, loudly, in writing.

**Hidden assumptions are the failure mode. Stated assumptions are discovery.** An assumption log at the top of your doc is not overhead; it is what turns a plausible-sounding spec into a reviewable one.

---

## How to show your thinking

For every non-trivial claim, use this shape:

> **Assumption:** [what you're taking as given]
> **Hypothesis:** If [X is true], then [Y will happen], because [reasoning].
> **How I'd test it:** [the coach session, prototype, or data probe that would confirm/refute]
> **Confidence:** low / medium / high — and why.

A reviewer should be able to scan your assumption log and immediately see where your reasoning is load-bearing.

---

## Cagan's four risks as a pressure-test lens

Cagan's four risks aren't the scoring spine of Gate 1 — the five deliverables are. But they're a useful lens for pressure-testing what you've produced before you hand it in.

- **Value risk** — does the customer actually care about this problem, and does the agent solve it in a way they'd pay for? Pressure-tests your *problem statement* and *success metrics*.
- **Usability risk** — can the people interacting with the agent (inputs, handoffs, escalations) figure out how to work with it? Pressure-tests your *delegation analysis* and *validation design*.
- **Feasibility risk** — is the spec precise enough that an AI coding agent can actually build from it? Pressure-tests your *agent specification*, and it's the risk your **closed build loop** is designed to attack directly.
- **Viability risk** — does it work for the business (compliance, procurement, handoffs with humans still in the loop)? Pressure-tests your *assumptions & unknowns* and your *delegation boundaries*.

You can't fully resolve any of these in Week 1 without real users. That's fine. The goal is to name which risks each artefact is attacking, and be explicit about what's still assumed.

---

## Where thinking-discipline shows up in each Gate 1 deliverable

- **Problem statement & success metrics** — every hedged claim is lifted into the assumption log with a hypothesis and a test. *Good:* a reviewer sees exactly where your confidence is load-bearing. *Bad:* prose that sounds confident but quietly rests on three untested premises.
- **Delegation analysis** — each boundary (fully agentic / agent-led / human-led) is justified with *why*, not just *what*. *Good:* you've named what makes a task safe to delegate fully, and what tacit knowledge or accountability keeps a task human-led. *Bad:* arbitrary splits you couldn't defend against a coach's "why there?"
- **Agent specification** — precise enough that an AI coding agent wouldn't need to ask a clarifying question. Your closed build loop against Claude Code is how you test this directly. Every time the builder produces something unintended, you classify it: spec ambiguity (you own the fix), builder misread (the builder owns the fix), or unjustified addition (ask for rollback).
- **Validation design** — you've named what "working" looks like in testable terms and what the most likely failure modes are. *Good:* failure modes tied to specific decisions in the spec. *Bad:* "we'll write tests" without specifying what behaviour they'd defend.
- **Assumptions & unknowns** — at least 5 genuine unknowns, not filler. "I don't know" beats a plausible-sounding guess.

---

## The coach is a scarce interview slot

Treat every coach session as one interview with a stakeholder you won't get again. Arrive with:

- A prioritised list of hypotheses you want to test.
- Specific questions (not open-ended chat) — the kind that move a claim from "assumed" to "tested."
- Your current assumption log, so you can update it live based on what the coach says, doesn't say, and deflects.

Unstructured chat burns the slot. Structured pressure-testing earns it.

---

## Anti-patterns to catch yourself on

- Writing confident prose that hides assumptions.
- Treating your first instinct as the answer.
- Asking the coach open-ended questions that don't test anything.
- Building before you can name what you're de-risking.
- Confusing "I decided" with "I validated."
- Producing a polished spec that dodges the riskiest unknown instead of naming it.
- Forgetting that "we don't know yet" is a legitimate, valuable answer when paired with a plan to find out.

---

## Self-check before Friday

- Is my assumption log visible, numbered, and honest about confidence?
- For each major claim across my five Gate 1 deliverables, can I point to either a test, a source, or an explicit assumption?
- Did I use my coach sessions to move specific hypotheses, not just to chat?
- Did I complete the closed build loop against Claude Code — and can I diagnose the gap between what I asked for and what got built?
- Would a reviewer be able to challenge my thinking because I've exposed it, rather than in spite of hiding it?

Four yeses is a strong Week 1. Five is rare and earned.
