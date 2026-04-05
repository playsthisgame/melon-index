---
name: agentic-spec-dev
description: >
  Guidelines for Spec-Driven Development (SDD) with AI agents and a human in the loop.
  Use this skill whenever you are starting a new feature, fixing a non-trivial bug, planning
  a refactor, or breaking down any coding task that will involve an AI agent doing the implementation.
  Also trigger when the user asks about how to write a spec, how to give an agent instructions,
  how to review agent output, or how to keep a coding project on track with agents. This skill
  covers the full SDD loop: writing the spec, agent execution, human review, and iteration.
---

# Spec-Driven Development (SDD) with Agents

Spec-Driven Development is a workflow where a human writes a clear specification before an agent
writes any code. The spec is the source of truth throughout the session. The human stays in the
loop at defined checkpoints rather than letting the agent free-run until something breaks.

The goal is predictable, reviewable, correctable output — not just fast output.

---

## The SDD Loop

```
Write Spec → Agent Executes → Human Reviews → Update Spec / Iterate
     ↑                                                     |
     └─────────────────────────────────────────────────────┘
```

Each phase has a job. Don't skip phases or blend them — the separation is where the value comes from.

---

## Phase 1: Writing the Spec

The spec is a markdown document the agent will treat as its contract. Write it before the agent touches any code.

### What a good spec contains

- **Goal** — one sentence: what does this accomplish for the user or system?
- **Context** — what the agent needs to know about the existing codebase, data model, or constraints
- **Tasks** — a numbered or bulleted list of concrete implementation steps
- **Acceptance criteria** — how you'll know it's done; ideally checkable line by line
- **Out of scope** — explicit list of things the agent should NOT do in this pass
- **Open questions** — things you're unsure about that the agent should surface, not silently decide

### Principles for writing specs

**Be specific about what you want, not how to do it.** Describe outcomes, not implementation steps — unless a specific approach is required. Over-specifying the "how" boxes the agent in and produces brittle, hard-to-review code.

**Every ambiguity will be resolved by the agent.** Whatever you leave undefined, the agent will fill in using its own judgment. That's sometimes fine, but you should choose where to give discretion intentionally, not by accident.

**Short specs ship faster than long specs.** Aim for the minimum spec that gives the agent enough signal to avoid bad decisions. If you're spending more than 15 minutes writing a spec for a 30-minute task, it's too long.

**One goal per spec.** If the spec is doing two things, split it. Mixed-goal specs produce mixed-up implementations that are hard to review.

**Write acceptance criteria as assertions, not descriptions.** "The endpoint returns 200" is checkable. "The endpoint works correctly" is not.

---

## Phase 2: Agent Execution

The agent implements against the spec. Your job in this phase is to stay out of the way — but stay available.

### What the agent should do

- Read the full spec before writing any code
- Ask clarifying questions **before starting**, not mid-implementation — one batch of questions is better than ten interruptions
- Stay within the scope defined by the spec; if something out of scope is blocking progress, surface it rather than silently expanding scope
- Follow the acceptance criteria as a checklist
- Note any deviations from the spec explicitly (e.g., "I used X instead of Y because...")

### What you should do as the human

- **Don't interrupt mid-task** unless the agent is clearly going off the rails. Let it finish a meaningful unit of work before redirecting.
- **Watch for scope creep in real time.** If the agent starts doing things not in the spec, it's okay to pause and redirect.
- **Be reachable for blockers.** A good agent will surface genuine ambiguities rather than guess; respond quickly when it does.

---

## Phase 3: Human Review

This is the most important phase and the most commonly skipped. Review against the spec — not just against intuition.

### How to review

1. **Read the diff, not the explanation.** Agents are optimistic about their own output. Read what actually changed.
2. **Check each acceptance criterion.** Go line by line. If a criterion isn't met, it's a bug — not a style preference.
3. **Look for silent scope expansion.** Did the agent change things the spec didn't ask for? Sometimes helpful, often risky. Decide consciously.
4. **Run it.** Acceptance criteria that can be verified by running the code should be. Don't skip this.
5. **Check for deferred problems.** Agents sometimes solve the immediate problem by creating a future one (e.g., hardcoding a value, skipping error handling). Look for shortcuts.

### What to do with what you find

- **Spec was wrong or incomplete** → Update the spec, then ask the agent to re-implement the affected parts. Don't ask the agent to "fix it" without updating the spec first — you'll get a patch on a misunderstood foundation.
- **Agent deviated from spec** → Point to the spec, ask for correction. Be specific about which criterion wasn't met.
- **Spec was right, implementation is right, but something feels off** → That feeling is usually data. Articulate it and add it to the spec before the next iteration.

---

## Phase 4: Iteration

Most tasks take more than one pass. That's expected and fine. The loop is the process.

### Updating the spec

- Treat the spec as a living document during the session. Update it when you learn something.
- When you change acceptance criteria, note why — this prevents you from reverting decisions you already made.
- Don't delete old acceptance criteria that were met — strikethrough them so the agent can see the history.

### Keeping context alive across turns

- At the start of a new session or a long gap, re-read the spec with the agent before continuing.
- If the agent seems confused, re-grounding in the spec is usually faster than explaining the confusion.
- For multi-session work, keep a short "current state" section at the top of the spec: what's done, what's in progress, what's next.

---

## Common Failure Modes

**The agent ignored the spec.** Usually because the spec was ambiguous or the agent wasn't asked to follow it explicitly. Start the session with: "Follow this spec closely. Ask before deviating."

**The spec was too vague.** "Refactor the auth module" is a goal, not a spec. Add tasks, constraints, and acceptance criteria.

**Review was skipped because it "looked right."** This is how bugs and scope creep accumulate silently. Review every meaningful change against the spec.

**The spec kept growing mid-implementation.** New requirements during implementation belong in a new spec for a follow-up task, not bolted onto the current one. Keep the current spec stable while the agent is working on it.

**The human and agent lost track of where they were.** Add a "current state" section to the spec. Update it at the end of every meaningful exchange.

---

## Spec Template

Use this as a starting point. Remove sections that don't apply.

```markdown
# [Feature / Task Name]

## Goal
One sentence describing what this accomplishes.

## Context
- Relevant files: 
- Relevant constraints: 
- Background the agent needs: 

## Tasks
1. 
2. 
3. 

## Acceptance Criteria
- [ ] 
- [ ] 
- [ ] 

## Out of Scope
- 

## Open Questions
- 

## Current State
- Status: not started / in progress / review / done
- Last completed: 
- Next: 
```
