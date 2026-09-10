# Devil's Advocate

**A Claude Code plugin that attacks your decision before you act on it.**

One cold adversary. No consensus, no balance, no "on the other hand". You answer every attack with evidence or you concede. Then a one-word verdict.

By [Growth Fabric](https://github.com/hadicancatak-coder/GF-Creative-Team). MIT.

## Before and after

Same decision, same model. The user asked for a devil's advocate on rewriting a Python ETL in Rust.

**Without the skill**, five unranked concerns phrased as questions, ending with:

> None of this means don't do it — the prototype result is genuinely compelling.

**With the skill:**

> **KILL SHOT** — The prototype measured a 30x speedup on one stage, not on the 4-hour pipeline, so the causal story swaps "this stage is CPU-bound and Rust helps here" for "Rust will fix the runtime." If that stage is a minority of the 4 hours, or if the rest is I/O-bound, Amdahl's Law caps the total gain far below what a full rewrite would need to justify itself — a 30x win on 20% of the runtime saves under 50 minutes, not hours.
>
> **P(MISTAKE)** 70%

Then three ranked attacks, each with a check that takes under an hour, a cheaper alternative, and a verdict the in-context agent has to defend line by line. Full run in [examples/rust-rewrite.md](examples/rust-rewrite.md).

## Why not just ask "play devil's advocate"?

1. **It is anchored.** The assistant helped write the plan. It cannot attack the plan from inside the same context.
2. **It has the wrong shape.** No ranking, no kill shot, no number, no verdict. You still don't know whether to proceed.

This fixes both. The Advocate runs as a subagent that sees only a four-line claim block, never the conversation. It must return a single kill shot, exactly three ranked attacks each with a sub-hour check, the cheapest alternative, and a probability that you are making a mistake. The in-context agent then runs every check it can, labels every attack CONCEDE / REBUT / UNRESOLVED, and issues PROCEED / PROCEED IF / STOP.

## Does the number mean anything?

Calibration on 8 scenarios, 2 runs each, Sonnet as the Advocate:

| Decision type | P(MISTAKE) range |
|---|---|
| Built on an unchecked causal story (4 scenarios) | 65–88% |
| Sound, claim worded precisely (2 scenarios) | 20–35% |
| Sound action, claim overclaimed or bundled (2 scenarios) | 55–60% |

No overlap between sound and bad at a threshold of 60. The kill-shot theme agreed across both runs in all 8 scenarios. On the overclaimed ones the Advocate said, both times, that the action was fine and the sentence was too big. Full table, scenarios, and the one contract violation found during testing are in [evals/](evals/).

Not yet measured: whether kill shots are usually *right* on real decisions with known outcomes. If you run it on a real decision and later learn what happened, file an [outcome report](https://github.com/hadicancatak-coder/devils-advocate/issues/new?template=outcome-report.yml). That is the only way the accuracy claim gets built.

## It killed its own launch plan

[examples/visibility-plan.md](examples/visibility-plan.md) is a real run, not a scenario. The plan was to submit this plugin to the community "awesome" lists. The Advocate said curated lists are "a graveyard with a README" and told us to go count open PRs. We did:

| List | Stars | Last push | Oldest open PR |
|---|---|---|---|
| composio-community | 1946 | 2026-07-26 | 2026-07-14 |
| ananddtyagi/cc-marketplace | 688 | 2026-01-18 | 2025-10-15 |
| GiladShoham | 51 | 2025-10-12 | 2026-02-08 |

Three of four stalled, PRs open up to eleven months. Verdict: **STOP**. The plan changed. It also conceded, with the file line counts as evidence, that most of this repo's value is one 65-line prompt anyone can paste, so stars will overstate real usage. That concession is in the README because the skill's own rules say a CONCEDE has to change something.

## Install

As a plugin (skill + `/devil-advocate` command + `advocate` agent):

```
/plugin marketplace add hadicancatak-coder/devils-advocate
/plugin install devil-advocate@devil-advocate
```

Skill only, any agent that reads `SKILL.md` (Claude Code, Codex, Cursor, others):

```bash
npx skills add hadicancatak-coder/devils-advocate
```

Manual:

```bash
git clone https://github.com/hadicancatak-coder/devils-advocate ~/.claude/plugins/devil-advocate
ln -s ~/.claude/plugins/devil-advocate/skills/devil-advocate ~/.claude/skills/devil-advocate
```

## Use

```
/devil-advocate I'm going to rewrite our Python ETL in Rust over the next 2 months
```

Or describe a decision and say "stress test this", "poke holes", "red team this", "what am I missing". The skill triggers on its own.

## What you get back

```
## Claim
CLAIM / EVIDENCE FOR / COST IF WRONG / LOAD-BEARING ASSUMPTION

## Advocate
KILL SHOT
RANKED ATTACKS (exactly 3, each with a <1h check)
CHEAPER ALTERNATIVE
P(MISTAKE) NN%

## Checks run
<every check that was a command, with its result>

## Answers
KILL SHOT — REBUT / CONCEDE / UNRESOLVED
...

## Verdict
PROCEED | PROCEED IF <checks> | STOP
```

## Compared to Karpathy's LLM Council

| | LLM Council | Devil's Advocate |
|---|---|---|
| Agents | N models + chairman | 1 adversary + you as judge |
| Goal | Best-rounded answer | Find the one reason you're wrong |
| Peer review | Anonymous cross-ranking | None. The adversary isn't graded on fairness |
| Output | Synthesis | Verdict you defended line by line |
| Rounds | 3 stages | 1, optionally 2. Never 3 |

Council is for "which of these is best". This is for "am I about to do something stupid". One idea kept from council: if you have more than one model, run the Advocate on a different one than the judge.

## Layout

```
.claude-plugin/        plugin + marketplace manifests
skills/devil-advocate/SKILL.md          the workflow the in-context agent follows
skills/devil-advocate/advocate-prompt.md   exact prompts for the Advocate and the rebuttal round
agents/advocate.md     the Advocate as a first-class subagent
commands/devil-advocate.md    the slash command
evals/                 calibration scenarios, results, pass criteria for prompt changes
examples/              full runs
```

## Contributing

Prompt changes must pass [evals/README.md](evals/README.md) before they ship. Outcome reports are the most valuable contribution.

## License

MIT
