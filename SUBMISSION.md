# Community marketplace submission

Values to paste into the submission form at
[platform.claude.com/plugins/submit](https://platform.claude.com/plugins/submit)
(Console form, for individual authors outside a Team or Enterprise org). The
claude.ai form at claude.ai/admin-settings/directory/submissions/plugins/new is
the alternative, but it needs a Team or Enterprise org with directory access.

Most fields are read from `.claude-plugin/plugin.json` at the commit you submit.
Anything the form asks for on top of the repo URL is below.

## Field values

These are the actual fields on the Console form, in order.

**Link to plugin** (required)
```
https://github.com/hadicancatak-coder/devils-advocate
```

**Path within repository** (optional) — leave blank. The plugin is at the repo root.

**Plugin homepage** (optional)
```
https://github.com/hadicancatak-coder/devils-advocate
```

**Plugin name** (required; the form warns to check the name is not taken — `devils-advocate`
with the s is taken by an unrelated plugin, `devil-advocate` is free)
```
devil-advocate
```

**Plugin description** (required)
```
Play devil's advocate on a decision before you commit. A cold-context adversary returns one kill shot, three ranked attacks each with a check you can run in under an hour, the cheaper alternative, and a probability you're wrong. You answer each attack with evidence or concede, then issue a forced verdict: PROCEED, PROCEED IF, or STOP.

Asking an assistant to "play devil's advocate" in the same conversation gives you unranked concerns phrased as questions, with no verdict, from a model that helped build the plan it is attacking. This fixes both problems. The adversary runs as a subagent that sees only a four-line claim block and never the transcript, so it cannot inherit your anchoring. The in-context agent then executes every proposed check that is a runnable command, labels each attack CONCEDE, REBUT-with-evidence, or UNRESOLVED, and issues the verdict. An unresolved kill shot can never produce PROCEED.

Calibrated on eight scenarios run twice each. Decisions built on an unchecked causal story scored 65 to 88 percent probability of mistake; sound decisions scored 20 to 35 percent, with no overlap at a threshold of 60. The scenarios, per-run numbers, and the pass criteria any prompt change must clear are all in the repo under evals/, so the calibration is reproducible rather than asserted.

Ships a skill, a /devil-advocate command, and a read-only advocate agent. No hooks, no MCP servers, no scripts, no network calls.
```

**Example use cases** (required)
```
Example 1: "I'm going to rewrite our 12k-line Python ETL in Rust over the next two months. The prototype is 30x faster on one stage and my manager is on board." The Advocate returns a kill shot on Amdahl's Law, three attacks each with a profiling check that takes under an hour, PyO3 as the cheaper alternative, and 70% probability of mistake. Verdict: PROCEED IF tonight's profile shows CPU-bound stages are more than half the runtime.

Example 2: "Our clicks are down 52% year over year because AI Overviews cannibalised our content. I'm presenting a 40% budget cut to the CMO tomorrow." The Advocate points out the cited metric is a paid-channel number while the claimed mechanism is an organic one, and gives a segmentation check that can be run before the meeting.

Example 3: "Ship checkout variant B to 100% of traffic, it's up 12% after two days." The Advocate runs the two-proportion test in the kill shot, reports p is about 0.6, and returns 85%. Verdict: STOP.

Example 4: A sound decision, to confirm it does not manufacture a kill shot. "Add a B-tree index on orders.customer_id to fix the nightly full scan." The Advocate returns 20%, flags that the query text was missing from the evidence, and proposes a skew check that takes thirty seconds. Verdict: PROCEED IF no single customer holds more than 5% of rows.
```

**Later steps** may ask for author, license (MIT), and category (productivity). Those also live
in `.claude-plugin/plugin.json` at the submitted commit.

## Pre-admission review

Run before submitting, 2026-09-10. The claim under attack was "this will be approved on
first submission." The Advocate returned 65% and three attacks. Each was checked.

**Is the near-identical name a dedupe or rename risk?** No. Fifty name pairs already in the
approved catalog differ by exactly one character, including `agent-discover`/`agent-discovery`,
`ai-dev-kit`/`ai-devkit`, `chuck`/`chucks`, and `clarify`/`clarity`. One-character-apart names
are routinely approved.

**Is the incumbent `devils-advocate` the same product?** No. It is a Japanese-language
single agent, five files and 367 lines, with no skill, no verdict, no probability, and no
evals. Its stated rule is the opposite of this one: it requires every criticism to arrive
with an improvement suggestion attached. Different language, different mechanism, different
output contract.

**Is approval a fast human quality gate?** No, and this is the finding that matters. The
community repo carries ten public submission-status inquiries, nine of them still open with
no reply. Authors report waits of six days, one month, and about three months with no
confirmation and no rejection. Treat submission as joining a slow queue, not as a decision
that comes back quickly.

Changes made as a result: the marketplace metadata no longer reads as one company's internal
tooling, the README now points at the scenario files so the calibration is reproducible
rather than asserted, and the self-critique section no longer undersells the plugin as just
a pasteable prompt.

## What the review pipeline checks

`claude plugin validate ./devils-advocate` — the same check the pipeline runs. Passes as of v1.0.1.

Automated safety screening. Current surface:

| Surface | This plugin |
|---|---|
| Hooks | none |
| MCP servers | none |
| `bin/` executables | none |
| Shell or Python scripts | none |
| Files with the executable bit | none |
| Agent tool grants | `Read, Glob, Grep` (read-only) |
| Network calls | none |
| Secrets or key-shaped strings | none |

Everything shipped is Markdown and JSON. Context cost is roughly 1,000 tokens
for the command and agent definitions, plus about 1,600 more only when the skill
is actually triggered.

## After submitting

Approved plugins are pinned to a commit SHA in
[`anthropics/claude-plugins-community`](https://github.com/anthropics/claude-plugins-community/blob/main/.claude-plugin/marketplace.json),
and CI bumps the pin as new commits land. The public catalog syncs nightly, so
there is a delay between approval and the plugin being installable. To check:

```bash
curl -sL https://raw.githubusercontent.com/anthropics/claude-plugins-community/main/.claude-plugin/marketplace.json \
  | grep -c '"devil-advocate"'
```

Once listed, the install becomes:

```
/plugin marketplace add anthropics/claude-plugins-community
/plugin install devil-advocate@claude-community
```

The official marketplace (`claude-plugins-official`) is curated separately at
Anthropic's discretion. This form does not submit to it.
