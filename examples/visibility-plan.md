# Example: a real decision, with the checks actually run

Unlike the other examples, this one is real. It was run on 2026-09-09 through the installed plugin, on the question of how to get this plugin discovered. Every check the Advocate proposed was executed, and the result changed the plan.

## Claim

```
CLAIM: Submitting the devil-advocate Claude Code plugin to community "awesome" lists and skill directories, plus one before/after post, will produce meaningful installs (say 100+ in the first month).
EVIDENCE FOR: The plugin is public with a calibration table, worked examples, and a one-command install. The author's other public plugin exists but its install count is unknown. Claude Code plugins and skills are a fast-growing category with several curated lists. The README leads with a before/after quote.
COST IF WRONG: A few hours preparing submissions, PRs under the author's name in third-party repos that may sit unmerged, and a post that gets no traction.
LOAD-BEARING ASSUMPTION: People discover Claude Code plugins through curated lists and directories rather than through word of mouth or the official marketplace.
```

## Advocate

```
KILL SHOT
The "100+ installs" claim is unfalsifiable, so the decision cannot succeed on its own terms. `claude plugin install` from a GitHub repo reports nothing back to the author; GitHub exposes only stars and a rolling 14-day clone/visitor window. Every install-bearing channel in the plan (awesome-list entries, directory listings) delivers an alphabetical line among hundreds of entries with no click-through data, which means the author will never know whether the plan worked and will default to believing it did. A decision whose success metric cannot be observed is a decision made for the feeling of shipping, not for installs.

RANKED ATTACKS
1. Curated lists are not a discovery channel; they are a graveyard with a README. A new entry with ~0 stars sits below dozens of established items, and several lists gate merges on star counts or maintainer taste, so PRs stall for weeks. | Holds if: the target lists have >50 entries and any "minimum stars" or "widely used" criterion in CONTRIBUTING.md. | Check: open the three lists you intend to hit, count entries in the relevant section, read CONTRIBUTING.md, and look at the age of the oldest open PR.
2. The real install driver is the one post, and a post from an account with no existing audience has a base rate near zero. The before/after quote is a product feature, not distribution; distribution is who reshares it. | Holds if: the author's account has under ~500 relevant followers and the post targets no specific community. | Check: count followers and list the three communities the post will go into; if the list is "my feed," this attack lands.
3. A devil's-advocate prompt is trivially copy-pasteable, so install intent is structurally low. Readers who like the README will paste the prompt into their own CLAUDE.md rather than run a plugin install, and stars will overstate usage. | Holds if: the plugin's core value lives in a single system prompt rather than in tooling that a paste cannot replicate. | Check: read your own repo and ask whether a user could reproduce 90% of it by copying one file; if yes, plan for stars, not installs.

CHEAPER ALTERNATIVE
Drop the awesome-list PRs entirely. Turn on GitHub traffic tracking today to get a baseline, then submit once to the official Anthropic plugin marketplace (the one channel with real intent-to-install) and post the before/after in two places where Claude Code users already congregate, with a clear "run this one command" line. That captures the only real install source in the plan, avoids orphaned PRs under your name, and gives you a measurable signal within a week instead of never.

P(MISTAKE)
70% — the channel chosen has no mechanism for delivering installs, the metric cannot be observed, and the single lever that could work (the post) has no stated audience.
```

## Checks run

**Kill shot — are installs observable?**

```
$ python3 -c "... ~/.claude/plugins/install-counts-cache.json ..."
total tracked plugins: 157
non-official tracked: 0
```

Claude Code does publish per-plugin `unique_installs`, but only for plugins in Anthropic's marketplaces. Zero of the 157 tracked entries come from a self-hosted marketplace. The author's other public plugin does not appear.

```
$ gh api repos/<repo>/traffic/views
2026-09-09 views=0 uniq=0
```

The traffic API is available to the repo owner, so clones and unique visitors over 14 days are observable. Baseline is zero, recorded today.

**Attack 1 — are the curated lists alive?**

| List | Stars | Last push | Open PRs | Oldest open PR |
|---|---|---|---|---|
| composio-community | 1946 | 2026-07-26 | 100+ | 2026-07-14 |
| quemsah | 1275 | 2026-09-07 | 2 | 2026-09-01 |
| ananddtyagi/cc-marketplace | 688 | 2026-01-18 | 28 | 2025-10-15 |
| GiladShoham | 51 | 2025-10-12 | 11 | 2026-02-08 |

Three of four are stalled or abandoned; two have had no merge in six to eleven months, with PRs open for up to eleven months. The one that is current is automated, so it needs no PR at all.

**Attack 3 — is it copy-pasteable?**

```
$ wc -l skills/devil-advocate/SKILL.md skills/devil-advocate/advocate-prompt.md agents/advocate.md
 122  65  44   (231 total)
```

Yes. A reader can reproduce most of the behaviour by copying one 65-line file.

## Answers

- **KILL SHOT — CONCEDE, partly.** Installs are genuinely invisible for a self-hosted marketplace. What changes: the plan now targets the one catalog that publishes install counts, and GitHub traffic is the interim metric with a zero baseline recorded.
- **Attack 1 — CONCEDE.** Confirmed by the table above. What changes: no PRs to any of the stalled lists.
- **Attack 2 — UNRESOLVED.** Audience size and target communities are the author's to state. The decision does not wait on it, because the channel changed.
- **Attack 3 — CONCEDE.** Accurate. What changes: stars and traffic become the expected signal, not installs, and the README stops implying otherwise.

## Verdict

**STOP**, then replace. The original plan was to submit to curated lists. Every part of that channel failed its check. The replacement is Anthropic's community marketplace, which offers in-app discovery through `/plugin` and is the only place install counts are published, plus the automated list that requires no PR.

## What it got wrong

The Advocate asserted "submit to the official Anthropic plugin marketplace" as the alternative. Checking that turned up something better and more specific: the official marketplace is curated at Anthropic's discretion with no application process, and third-party submissions go to a separate community marketplace through a Console form. The Advocate was directionally right and factually loose. That is the pattern to expect: it finds the flaw, you find the fact.
