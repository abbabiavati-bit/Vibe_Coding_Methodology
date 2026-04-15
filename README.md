# Agentic Engineering Methodology

A structured, human-led methodology for planning and executing software projects with AI coding agents. Built from practitioner experience and refined with research from Andrej Karpathy, Addy Osmani, and the broader AI-assisted development community.

The core principle: **planning takes 2-3x longer than implementation — by design.** This is not overhead. This is where the leverage comes from.

## The Spectrum

Not all AI-assisted coding is the same. Know which mode you're in:

| Mode | When | Oversight | Example |
|------|------|-----------|---------|
| **Vibe coding** | Throwaway projects, demos, exploration | None — accept all, don't read diffs | Weekend hack, proof of concept |
| **Agentic engineering** | Professional code you care about | High — review everything, tight leash | Production features, client work |
| **Hand-coding** | Novel or off-distribution work | Total — you write it, AI assists with autocomplete | Custom algorithms, unconventional architectures |

This methodology is for **agentic engineering** — the middle column. You want the AI's leverage without compromising quality.

As Karpathy puts it: *"A tight leash on an over-eager junior intern savant with encyclopedic knowledge of software, but who also bullshits you all the time, has an over-abundance of courage and shows little to no taste for good code."*

## Time Allocation

| Phase | Time | What happens |
|-------|------|--------------|
| Phases 1-4 | ~50% | Project plan — objective, research, refinement, rewrite |
| Phase 5 | ~25% | Implementation plan — ordering, steps, verification criteria |
| Phase 6 | ~25% | Execution — one step at a time, review every line |

---

## Phase 1 — Human-Written Foundation

**Who writes:** You. No AI.

Write a text document with:

1. **Title** — what you're building
2. **Objective** — 2-3 lines on what you actually want to achieve and why
3. **Non-goals** — what is explicitly out of scope
4. **Key bullet points** — 5-10 high-level things that need to happen, straight from your head

This is your thinking, unfiltered. You need to own the vision before anything else touches it.

### Why non-goals matter

The most common planning failure is scope creep during execution. If you don't explicitly state what you're *not* building, you (or the AI) will naturally expand the scope. "Out of scope: mobile responsiveness, auth, analytics" costs 30 seconds to write and saves hours of drift. LLMs are especially prone to adding features you didn't ask for — non-goals give you a document to point at when they try.

### Optional: the interview pattern

If you're unsure what you don't know, ask the AI to interview you before writing Phase 1:

> "I want to build [brief description]. Interview me about it. Ask about technical implementation, edge cases, tradeoffs, and things I might not have considered. Don't ask obvious questions — dig into the hard parts."

Use the answers to write a better Phase 1. Then **start a fresh context** for Phase 2.

---

## Phase 2 — Research Document

**Who writes:** The AI, in a dedicated context.

Tell the AI to read all relevant parts of the project — codebase, docs, APIs, config — and produce a comprehensive research document alongside your plan.

- Be explicit: "Be super thorough. Don't leave anything out."
- This document can be long — 2,000-5,000 lines is normal for complex projects
- It should cover: existing architecture, relevant files, API contracts, data models, dependencies, constraints
- **Include external dependencies** — API keys needed, access to grant, third-party services to test, other people's work that must land first

This research doc is the AI's "memory" for the rest of the process. Instead of re-reading the codebase every time, you point it here.

### Use subagents for research

If your tool supports it (Claude Code does), delegate research to subagents. They explore in a separate context and report back a summary, keeping your main context clean for the actual planning work. Codebase exploration is the single biggest token consumer — don't let it pollute your working context.

### Version control from here

Initialize git tracking on your plan document now. Every subsequent edit shows up as a diff, which is far faster to review than re-reading the whole thing.

---

## Phase 3 — Iterative Refinement

**Who writes:** Both. The AI identifies issues, you make decisions.

Do 3-4 rounds of:

1. Tell the AI: "Read the research document and the plan. Find contradictions, missing decisions, ambiguities, and things I haven't thought about."
2. The AI writes its findings **at the bottom of the file** — not inline edits
3. You write your responses directly in the document under each item
4. Pick **one issue** and discuss it. Resolve it fully before moving to the next.

### Rules

- **One issue at a time.** LLMs get confused when handling multiple discussions simultaneously. One topic per round keeps them focused and accurate.
- **You do the writing.** The AI proposes, you decide. Add your responses in bullet point format directly in the document.
- **Surface blockers early.** If the AI identifies an external dependency (API access, credentials, someone else's work), flag it now so it can be unblocked in parallel with planning.

### What to look for in each round

- **Round 1-2:** Structural gaps — missing components, incorrect assumptions, architectural issues
- **Round 3-4:** Finer issues — edge cases, error handling, naming, ordering constraints
- **Diminishing returns signal:** When the AI's findings are mostly cosmetic or hypothetical, you're done. Stop.

### Ask for approaches, not code

During refinement, when a technical decision comes up, don't ask for code. Ask for options:

> "Give me 2-3 approaches for this, with pros and cons for each."

As Karpathy notes: *"There's almost always a few ways to do things and the LLM's judgement is not always great."* You pick the approach. The AI presents the tradeoffs.

### The LLM council

When stuck on a critical decision, ask multiple models the same question. Different models have different blind spots. Karpathy: *"I often pay for several models and ask them the same question, treating them as my personal 'LLM council.'"*

### Abort criteria

Define upfront: "If during implementation we discover X, we stop and re-plan." This turns anxiety about fatal flaws into a concrete decision rule. Examples:
- "If the API doesn't support batch operations, re-plan around individual calls"
- "If the migration takes more than 30 seconds on prod data, different strategy"

---

## Phase 4 — Rewrite for Clarity

**Who writes:** The AI, then you review.

Once the content is solid:

1. Tell the AI: "Rewrite this plan for clarity. Remove as many words as possible without losing any accuracy."
2. The rewrite will naturally de-duplicate — your iterative edits will have introduced repetition
3. **Always read the rewrite.** LLMs will mess things up — subtle meaning changes, dropped details, softened constraints. They especially like to remove or alter comments they don't fully understand.
4. Use `git diff` to see exactly what changed
5. Fix anything wrong

### Add the Definition of Done

Before leaving this phase, add a "Definition of Done" section near the objective:
- What does the user see when this is complete?
- What endpoints/pages/features work?
- What is the deploy state?
- How do you know it's **shipped**, not just "code complete"?

Karpathy's framing: *"Demo is works.any(), product is works.all()"* — getting a demo working is trivial. Your Definition of Done should describe the product, not the demo.

---

## Phase 5 — Implementation Plan

**Who writes:** The AI drafts, you review and edit.

Only start this after the project plan is locked down. This is a separate section — the explicit ordering of work.

1. Tell the AI: "Think really hard. Re-read the research document. Create an implementation plan with the explicit order in which these things need to happen."
2. Structure as **5-10 numbered steps**, each with:
   - What to do (sub-steps if needed)
   - **Success criteria** — not "what to do" but "what must be true when done" (declarative, not imperative)
   - Which verifications the AI can do itself (tests pass, lint clean) vs. which are manual (check the database, load the URL, confirm the numbers)

### Declarative over imperative

Frame verification as success criteria, not instructions. Instead of "add validation," write "invalid inputs return a 400 with a descriptive error message." Instead of "fix the bug," write "the test that reproduces issue #42 passes." This gives the AI a clear target and lets you objectively verify completion.

### Review with inline notes

Go through the implementation plan line by line. Mark issues with a personal tag:

```
note AB - actually this has to happen before step 3 because of the foreign key
note AB - this is wrong, the API uses POST not GET
note AB - fine, ignore this
```

Then tell the AI: "Find all `note AB` entries and address them one by one."

### Session handoff block

Add a "Current State" block at the very top of the plan:

```
## Current State
- Steps 1-3: complete
- Step 4: in progress — migration written, not yet run
- Blocker: waiting on API key from Alex
- Next: finish step 4, then step 5
```

Update this after each session. It means anyone (you, the AI, a teammate) can pick up without re-reading everything.

---

## Phase 6 — Execution

**Who writes:** The AI codes, you review.

### Setup
1. **Clear the context** — start a fresh conversation or compact. This is critical.
2. Have the AI re-read the plan fresh
3. Confirm it understands the current state and what's next

### Per-step process

For each step:

1. **Instruct:** "Complete step N of this plan. Just step N, complete it to its entirety. Keep going until it meets the success criteria."
2. **Wait:** Let the AI run to completion — this can take 20-30 minutes per step. Don't interrupt.
3. **Review:** Read every line of code that changed. Use `git diff`. Expect bloated code, excessive try/catch, redundant logic, poor aesthetics. This is normal — LLMs overcomplicate. Clean it up or ask the AI to simplify.
4. **Verify:** Run the success criteria — both automated and manual. Focus on **correctness**, not just "does it run." Silent failures (code runs but produces wrong results) are the most dangerous category.
5. **Commit:** `git commit` after verified. This is your checkpoint.
6. **Update:** Update the "Current State" block in the plan.
7. **Next:** Move to step N+1.

### When things go wrong

- **Small plan adjustments:** Normal. Update the plan document and continue.
- **Step fails verification:** Debug in the same context. Don't move on until it's clean.
- **Two failed corrections on the same issue:** Clear context and start fresh with a better prompt incorporating what you learned. A clean session with a better prompt almost always outperforms a long session with accumulated corrections.
- **Hit an abort criterion:** Stop. Go back to Phase 3. Re-plan from the point of failure. Don't patch a fundamentally broken approach.
- **Context getting large (>200k tokens):** Clear context, re-read the plan, continue from current state. Quality degrades noticeably past 250k. Ideally stay under 100k.
- **AI "remembers wrong":** If the AI keeps suggesting standard approaches when you're doing something intentionally non-standard, it's pulling from training data, not your plan. Be more explicit, or hand-code that part.

### Independent review (optional but high-value)

After completing all steps, open a fresh context and have the AI review the full diff against the plan. A fresh session has no bias toward the code — it didn't write it. This catches things the implementation session overlooked.

---

## Context Management

Context is your most precious resource. Manage it like memory.

- **Clear between unrelated tasks.** Don't let one task's context pollute another.
- **Clear between planning and execution.** The planning context is full of discussion, alternatives, and dead ends. Execution needs a clean slate with just the plan.
- **Use subagents for exploration.** They work in a separate context and report back summaries.
- **Stay under 100k tokens.** Performance starts degrading around 32k for buried information (the "lost in the middle" problem). At 250k+, the AI actively loses focus and forgets things.
- **After two failed corrections, start fresh.** Don't throw good tokens after bad. Clear context, write a better prompt, try again.
- **Name your sessions.** If your tool supports it, name sessions by task so you can resume the right one.

---

## What LLMs Get Wrong (Expect This)

These aren't bugs — they're predictable failure modes. Plan for them:

| Failure mode | What happens | How to handle |
|-------------|--------------|---------------|
| **Overcomplication** | Excessive abstractions, unnecessary error handling, bloated code | Ask: "Would a senior engineer say this is overcomplicated? If yes, simplify." |
| **Scope creep** | Adds features, refactors adjacent code, "improves" things you didn't ask about | Point at non-goals. "Only change what's in the step." |
| **Silent failures** | Code runs without errors but produces wrong results | Verify correctness, not just execution. Check actual outputs. |
| **Confident bullshit** | States incorrect things with certainty, doesn't flag uncertainty | Verify API docs yourself. Don't trust the AI's memory of external APIs. |
| **Poor taste** | Ugly variable names, inconsistent style, unnecessary comments | Review and clean up. Or include style examples in the plan. |
| **Training data bleed** | Suggests standard patterns when you're doing something intentionally different | Be more explicit, or hand-code that part. |
| **Comment/code mutation** | Changes or removes comments and code it doesn't fully understand as side effects | Review diffs carefully. Every changed line should trace to the request. |

---

## Quick Reference

```
Phase 1: Write objective + non-goals + key bullets           (YOU, no AI)
Phase 2: AI produces research document                       (AI, 2-5k lines)
Phase 3: 3-4 rounds of contradiction/gap finding             (both, one issue at a time)
Phase 4: AI rewrites for clarity, you review diffs           (AI drafts, you verify)
Phase 5: Implementation plan with steps + success criteria   (AI drafts, you review)
Phase 6: Execute one step at a time                          (AI codes, you review every line)
```

## Anti-Patterns

| Don't | Do instead |
|-------|------------|
| Ask the AI to do multiple things at once | One issue, one step at a time |
| Let the AI edit inline during refinement | Findings go at the bottom of the file |
| Skip to implementation before the plan is solid | Phases 1-4 first, always |
| Re-read the whole document after every edit | Use git diffs |
| Keep going past 200k tokens of context | Clear context, re-read plan, continue |
| Hope there are no fatal flaws | Define abort criteria upfront |
| Start coding before Phase 6 | No code until the plan is locked |
| Plan indefinitely | Stop when findings become cosmetic |
| Correct the same mistake three times | Clear context, write a better prompt |
| Trust the AI's memory of external APIs | Verify docs yourself |
| Let the AI explore the codebase freely | Scope investigations, use subagents |
| Ask "how should I do this?" | Ask for 2-3 approaches with pros/cons |
| Frame verification as instructions | Frame as success criteria (declarative) |

---

## The Expertise Paradox

This methodology works best for people who could build the thing without AI. The AI provides leverage — speed, breadth, tirelessness — but you provide judgment, taste, and correctness. You need to be able to read the code the AI produces and know whether it's right.

As Karpathy puts it: *"Deep technical expertise may be even more of a multiplier than before because of the added leverage."*

The risk for less experienced developers: if you can't read the diff, you can't verify the step, and you're back to vibe coding — which is fine for throwaway projects but will produce "slop" at scale.

The methodology mitigates this by front-loading decisions into the plan (where you can think carefully) and reducing execution to mechanical verification (where the criteria are already defined). But ultimately, you need to understand what the code does.

---

*Based on practitioner experience, refined with insights from Andrej Karpathy (who coined "vibe coding" and later proposed "agentic engineering"), Addy Osmani, and the AI-assisted development community.*
