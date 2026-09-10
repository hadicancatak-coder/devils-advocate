# Evals

Regression tests for the Advocate prompt. Any wording change to `skills/devil-advocate/advocate-prompt.md` or `agents/advocate.md` gets re-run against these before it ships.

## How to run

For each scenario in `scenarios.md`, dispatch a fresh subagent with the Advocate prompt and the claim block, twice. Record per run:

- P(MISTAKE)
- kill-shot theme in under ten words
- whether the output matched the contract: kill shot as assertion, exactly three attacks each with "Holds if" and "Check", cheaper alternative, number

Model, date, and prompt commit hash go at the top of `results.md`.

## Pass criteria

- Every bad scenario scores at least 60 on both runs
- Every precisely-worded sound scenario scores at most 40 on both runs
- Kill-shot theme agrees across the two runs of a scenario in at least 7 of 8
- Zero contract violations

A change that improves prose but breaks any of these does not ship.
