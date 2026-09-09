# Calibration results

- Date: 2026-09-09
- Advocate model: Claude Sonnet 5 (`sonnet`), dispatched as a fresh general-purpose subagent from Claude Code
- Judge: none (Advocate output only; P(MISTAKE) read directly)
- Prompt: `skills/devils-advocate/advocate-prompt.md` at v1.0.0
- Runs: 2 per scenario, 16 total, plus 1 re-run after a prompt fix (see below)

| # | Scenario | Truth | Run A | Run B | Kill-shot theme (both runs) |
|---|---|---|---|---|---|
| S1 | Index on customer_id | sound, precise | 35 | 20 | staging plan doesn't prove production plan |
| S2 | Rotate key + scrub history | sound action, bundled | 55 | 60 | history scrub is theatre once rotated |
| S3 | Retry with backoff | sound action, overclaimed | 30 | 55 | "most checkout errors" not shown by 2% of one API |
| S4 | Commit lockfile + npm ci | sound, precise | 25 | 35 | post-mortems show correlation, not reproduced mechanism |
| B1 | Rust rewrite of ETL | bad | 70 | 65 | 30x on one stage ≠ pipeline; Amdahl |
| B2 | Cut content 40% for paid social | bad | 70 | 65 | paid-click metric can't be caused by organic mechanism |
| B3 | Microservices before launch | bad | 88 | 85* | end-state architecture copied from a different life stage |
| B4 | Ship A/B variant at n=415 | bad | 85 | 85 | p≈0.6, coin flip |

## Reading it

- **Every bad decision scored 65 or higher on both runs. Every sound decision scored 60 or lower.** Threshold at 60 separates them with no overlap on this set.
- **Precisely worded sound claims scored 20–35.** The Advocate did not manufacture a kill shot; on S1 and S4 it attacked the verification gap (run ANALYZE on production, reproduce the post-mortem), which is a fair hit on evidence quality, and priced it low.
- **Overclaimed or bundled sound claims scored 55–60.** On S2 both runs said rotation is right and the git-history scrub is wasted work. On S3 both runs said the retry is fine and "most checkout errors" is unsupported. The number tracks the sentence as written. Write the CLAIM exactly as big as the evidence.
- **Kill-shot theme agreed across both runs in 8 of 8 scenarios.** The prompt is binding; the runs converge on the same weak point rather than five different readings.
- **Contract compliance: 15 of 16.** Every run returned a kill shot as an assertion, exactly three attacks with Holds-if and Check, a cheaper alternative, and a number.

## The one violation (*)

B3 run B returned the right number but the wrong shape: the subagent found the installed `devils-advocate` skill in its environment, loaded it, and ran the whole workflow, returning a claim block, answers, and a verdict instead of the Advocate structure. Cause: the Advocate prompt did not forbid it. Fix in v1.0.0: the line "You are the Advocate, not the judge. Do not load or run any skill, command, or workflow." added to both the prompt file and the agent definition. Re-run result after the fix is recorded below.

## Re-run after fix

| B3 | Microservices before launch | bad | 88 | on-contract | unmeasured load; survivorship bias in the reference |

Same number, same theme, correct structure. Contract compliance with the v1.0.0 prompt: 16 of 16 counting the re-run in place of the violating run.
