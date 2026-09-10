# Community marketplace submission

Values to paste into the submission form at
[platform.claude.com/plugins/submit](https://platform.claude.com/plugins/submit)
(Console form, for individual authors outside a Team or Enterprise org). The
claude.ai form at claude.ai/admin-settings/directory/submissions/plugins/new is
the alternative, but it needs a Team or Enterprise org with directory access.

Most fields are read from `.claude-plugin/plugin.json` at the commit you submit.
Anything the form asks for on top of the repo URL is below.

## Field values

**Repository URL**
```
https://github.com/hadicancatak-coder/devils-advocate
```

**Plugin name** (must be unique in the catalog; the plural is taken by an unrelated plugin)
```
devil-advocate
```

**Version**
```
1.0.1
```

**Short description** (one line, for the catalog card)
```
Play devil's advocate on a decision before you commit. Kill shot, three ranked attacks with runnable checks, cheaper alternative, P(mistake), forced verdict.
```

**Long description** (what the plugin does and when to reach for it)
```
Play devil's advocate on a decision before you commit. A cold-context adversary returns one kill shot, three ranked attacks each with a check you can run in under an hour, the cheaper alternative, and a probability you're wrong. You answer with evidence or concede, then a forced verdict. For rewrites, budget moves, launches, and any call that is expensive to reverse.

Unlike asking an assistant to "play devil's advocate" in the same conversation, the adversary runs as a subagent that sees only a four-line claim block, never the transcript, so it cannot inherit the anchoring of the plan it is attacking. The in-context agent then executes every check that is a command, labels each attack CONCEDE / REBUT-with-evidence / UNRESOLVED, and issues PROCEED, PROCEED IF, or STOP.

Calibrated on 8 scenarios run twice each: decisions built on an unchecked causal story scored 65-88% probability of mistake, sound decisions scored 20-35%, with no overlap. Scenarios, results, and pass criteria for prompt changes are in the repo under evals/.
```

**Author**
```
Hadican Catak — https://github.com/hadicancatak-coder
```

**License**
```
MIT
```

**Category** (pick the nearest; the catalog leans toward workflow/productivity tooling)
```
Development workflows  /  Productivity
```

**Keywords**
```
devil-advocate, devils-advocate, red-team, decision-making, critique, adversarial, pre-mortem, second-opinion, stress-test
```

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
