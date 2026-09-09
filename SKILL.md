---
name: gf-devils-advocate
description: Use when about to commit to a decision, plan, conclusion, or recommendation and it needs to be attacked before anyone acts on it — especially when everyone already agrees, money or time is already sunk, a deadline is pushing toward yes, or the user says "poke holes", "stress test", "red team", "what am I missing".
---

# GF Devil's Advocate

One fresh-context adversary whose only job is to prove the decision wrong. You judge. No consensus, no balance, no "on the other hand". A forced verdict at the end.

**Core principle:** an argument you have been helping build cannot be attacked from inside the same context. The Advocate must start cold.

## When to use

- Any decision that is expensive to reverse: rewrites, budget moves, hires, launches, migrations, "I'm presenting this tomorrow"
- A conclusion drawn from data that fits a story everyone already believes
- The user asks for pushback, holes, a stress test, or a red team

Don't use for: factual questions, obvious execution tasks, code review (use a reviewer), or choosing between many open options (that is an LLM Council problem, see the last section).

## Workflow

### 1. Write the claim (you, in context)

Reduce the decision to this block. If you cannot fill every line, the decision is not ready to be attacked.

```
CLAIM: <one sentence, falsifiable: "X will cause Y">
EVIDENCE FOR: <the 2–4 facts the claim actually rests on>
COST IF WRONG: <what is lost — money, months, credibility>
LOAD-BEARING ASSUMPTION: <the one thing that, if false, kills the claim>
```

### 2. Dispatch one Advocate (fresh subagent, no conversation history)

Give it ONLY the claim block plus any raw data or files it needs. The prompt tells the Advocate that you chose the load-bearing assumption and that it may attack a different one; do not remove that line. Never the transcript, never your reasoning, never your preferred answer. Use the prompt in [advocate-prompt.md](advocate-prompt.md) verbatim.

The Advocate returns, in this order:

1. **KILL SHOT** — the single strongest reason the decision is wrong, as an assertion with a mechanism, not a question
2. **RANKED ATTACKS** (exactly 3) — each with: the attack, what must be true for it to land, a check that takes under one hour
3. **CHEAPER ALTERNATIVE** — what gets 80% of the benefit at 20% of the cost
4. **P(MISTAKE)** — a number, 0–100%, with one sentence of reasoning

### 3. Answer every attack (you, in context)

Show the Advocate's output to the user verbatim. Then answer each item with exactly one label:

| Label | Meaning | Requires |
|---|---|---|
| CONCEDE | The attack holds | Say what changes because of it |
| REBUT | The attack fails | Evidence, not opinion. A number, a file, a test result, a source |
| UNRESOLVED | Can't tell yet | Name the check and who runs it |

A REBUT without evidence is an UNRESOLVED. Write it as one.

### 4. Verdict

Exactly one of:

- **PROCEED** — every attack is CONCEDE-with-adjustment or evidenced REBUT
- **PROCEED IF** — list the UNRESOLVED checks; the decision waits on them
- **STOP** — the kill shot stands, or a CONCEDE changes the cost/benefit

An UNRESOLVED kill shot can never produce PROCEED.

### 5. One rebuttal round (optional, max one)

If you REBUT the kill shot, dispatch a fresh Advocate with the original kill shot, your rebuttal, and its evidence, using the rebuttal prompt in [advocate-prompt.md](advocate-prompt.md). It replies ACCEPT or ESCALATE with one paragraph. Then verdict. Never a third round.

## The output the user sees

```
## Claim
<block from step 1>

## Advocate
<verbatim, unsoftened>

## Answers
KILL SHOT — REBUT: <evidence>
Attack 1 — CONCEDE: <what changes>
Attack 2 — UNRESOLVED: <check, owner>
...

## Verdict
PROCEED IF: <checks>
```

## Common mistakes

| Mistake | Fix |
|---|---|
| Passing the conversation to the Advocate | It inherits your anchoring. Claim block only |
| Summarising the Advocate "for tone" | Verbatim. The harshness is the product |
| REBUT with "I think" or "in my experience" | That is UNRESOLVED |
| Five attacks of equal weight | The Advocate ranks. If it didn't, send it back |
| Ending with "but overall it's promising" | The verdict is one word. Nothing after it |
| Skipping the claim block because the decision is "obvious" | Obvious decisions with sunk cost are the ones this exists for |

## Relation to LLM Council

Karpathy's council asks several models the same question, has them rank each other anonymously, and lets a chairman synthesise. That optimises for a well-rounded answer. This optimises for finding the one reason you are wrong. One adversary, no peer review, no synthesis, a verdict you have to defend line by line.
