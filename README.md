# GF Devil's Advocate

Part of the Growth Fabric agent tooling, alongside [GF Creative Team](https://github.com/hadicancatak-coder/GF-Creative-Team). A Claude Code skill that attacks your decision before you act on it.

One fresh-context adversary. No consensus, no balance, no "on the other hand". You answer every attack with evidence or you concede. Then a one-word verdict.

## Why not just ask "play devil's advocate"?

Ask an assistant that has been helping you build a plan to poke holes in it and you get a polite list of five concerns of equal weight, phrased as questions, ending with "none of this means don't do it". Two things are wrong with that:

1. **It is anchored.** It helped write the plan. It cannot attack the plan from inside the same context.
2. **It has the wrong shape.** No ranking, no kill shot, no number, no verdict. You still don't know whether to proceed.

This skill fixes both. The Advocate runs as a subagent that sees only the claim, never the conversation. It must return a single kill shot, three ranked attacks each with a sub-one-hour check, the cheapest alternative, and a probability that you are making a mistake. Then you, in context, must label every attack CONCEDE / REBUT / UNRESOLVED and issue PROCEED / PROCEED IF / STOP.

## Compared to Karpathy's LLM Council

| | LLM Council | GF Devil's Advocate |
|---|---|---|
| Agents | N models + chairman | 1 adversary + you as judge |
| Goal | Best-rounded answer | Find the one reason you're wrong |
| Peer review | Anonymous cross-ranking | None. The adversary isn't being graded on fairness |
| Output | Synthesis | Verdict you defended line by line |
| Rounds | 3 stages | 1, optionally 2. Never 3 |

Council is for "which of these is best". This is for "am I about to do something stupid".

## What it does and doesn't prove

Tested so far: the skill reliably changes the *shape* of the critique (ranked, assertive, numeric, verdict) versus an unprompted "play devil's advocate", across five subagent runs on test scenarios, and it returns a low P(MISTAKE) on a decision that is actually sound rather than manufacturing a kill shot. Not yet measured: whether the kill shots are more often *correct* on real decisions with known outcomes. Treat P(MISTAKE) as the adversary's calibrated guess, not a measurement. If you run it on a real decision and later learn the outcome, open an issue with both.

## Install

Claude Code:

```bash
git clone https://github.com/hadicancatak-coder/GF-Devils-Advocate ~/.claude/skills/gf-devils-advocate
```

Or with the skills CLI (works for Claude Code, Codex, Cursor and others):

```bash
npx skills add hadicancatak-coder/GF-Devils-Advocate
```

## Use

```
/gf-devils-advocate I'm going to rewrite our Python ETL in Rust over the next 2 months
```

Or just describe a decision and say "stress test this" / "poke holes" / "red team this". The skill triggers on its own.

## What you get back

```
## Claim
CLAIM / EVIDENCE FOR / COST IF WRONG / LOAD-BEARING ASSUMPTION

## Advocate
KILL SHOT
RANKED ATTACKS (3, each with a <1h check)
CHEAPER ALTERNATIVE
P(MISTAKE) NN%

## Answers
KILL SHOT — REBUT / CONCEDE / UNRESOLVED
...

## Verdict
PROCEED | PROCEED IF <checks> | STOP
```

See [examples/](examples/) for a full run on a test scenario, including what the same prompt produced without the skill.

## Files

- `SKILL.md` — the workflow the in-context agent follows
- `advocate-prompt.md` — the exact prompt the adversary subagent receives
- `examples/` — full runs on test scenarios

## License

MIT
