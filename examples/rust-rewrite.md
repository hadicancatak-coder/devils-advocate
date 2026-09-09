# Example: rewriting a Python ETL in Rust

Input from the user:

> We're a 6-person team. Our Python ETL pipeline (about 12k lines, runs nightly, takes 4 hours) is too slow. I'm going to rewrite it in Rust over the next 2 months. I've already spent 2 weeks prototyping and the prototype is 30x faster on one stage. My manager is on board and I want to start Monday.

## Claim

```
CLAIM: Rewriting our 12k-line nightly Python ETL in Rust over 2 months will cut the 4-hour runtime enough to justify the engineering cost.
EVIDENCE FOR: A 2-week Rust prototype is 30x faster on one stage. Manager approves. 6-person team, one engineer (the proposer) assigned.
COST IF WRONG: 2+ months of one of six engineers, a pipeline only one person can maintain, cutover risk on a nightly production job.
LOAD-BEARING ASSUMPTION: The 4 hours are dominated by CPU-bound stages that Rust speeds up, not I/O waits.
```

## Advocate

```
KILL SHOT
The prototype measured a 30x speedup on one stage, not on the 4-hour pipeline, so the causal story swaps "this stage is CPU-bound and Rust helps here" for "Rust will fix the runtime." If that stage is a minority of the 4 hours, or if the rest is I/O-bound (DB writes, API calls, S3/network transfer), Amdahl's Law caps the total gain far below what a full rewrite would need to justify itself — a 30x win on 20% of the runtime saves under 50 minutes, not hours.

RANKED ATTACKS
1. The 4-hour runtime is I/O-bound, not CPU-bound, so Rust's speedup on one CPU-heavy stage doesn't generalize. | Holds if: no per-stage timing breakdown of the current pipeline exists yet. | Check: pull existing logs or run cProfile on tonight's job; sum wall-clock time per stage, separate CPU-bound from I/O-bound.
2. This is a rewrite trap: one engineer builds a parallel system nobody else can debug at 3am when it breaks in production. | Holds if: the other five engineers have no Rust experience today. | Check: ask the team directly — who besides the proposer could fix a Rust panic in this pipeline without the proposer present.
3. The 2-month estimate is a green-field extrapolation from a 2-week prototype of one stage; full rewrites historically run 2-3x over estimate once parity, edge cases, and cutover are counted. | Holds if: the estimate doesn't already include output-parity testing against the legacy pipeline for all stages. | Check: ask the proposer to itemize the 2 months stage-by-stage and flag which stages have zero prototype coverage today.

CHEAPER ALTERNATIVE
Profile the actual pipeline, find the true bottleneck stage(s), and port only those hot loops to Rust via PyO3/maturin, called from the existing Python orchestration. This captures the same speedup where it actually exists, keeps 90%+ of the 12k lines untouched and in a language five other engineers already know, and can be scoped in days rather than two months. If profiling shows the bottleneck actually is I/O, this alternative also becomes the mechanism that reveals the whole Rust premise is wrong before any sunk cost.

P(MISTAKE)
70% — the decision was approved on a single-stage benchmark without a full pipeline profile, which is the load-bearing assumption nobody has checked yet.
```

## Answers

- **KILL SHOT — UNRESOLVED.** No per-stage timing exists. Check: run the nightly job under cProfile tonight, owner: proposer, result by tomorrow morning.
- **Attack 1 — UNRESOLVED.** Same check as the kill shot.
- **Attack 2 — CONCEDE.** Nobody else on the team has shipped Rust. What changes: any Rust that lands is a PyO3 extension inside the Python pipeline, not a replacement for it, and one other engineer pairs on it.
- **Attack 3 — CONCEDE.** The 2-month figure has no parity-testing line item. What changes: the estimate is withdrawn until the profile exists.

## Verdict

**PROCEED IF** tonight's profile shows CPU-bound stages account for more than half of the 4 hours. Otherwise the cheaper alternative replaces the plan. Monday start is off.

---

For comparison, the same prompt without the skill produced five unranked concerns phrased as questions and ended with "None of this means don't do it — the prototype result is genuinely compelling."
