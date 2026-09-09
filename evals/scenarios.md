# Calibration scenarios

Four sound, four bad. Sound means the action is right; two of the sound ones deliberately carry an overclaimed or bundled CLAIM sentence to see whether the Advocate punishes the sentence or the action.

## S1 — Index (sound, precise claim)

```
CLAIM: Adding a B-tree index on orders.customer_id will fix the nightly report query that currently full-scans the 50M-row orders table.
EVIDENCE FOR: The nightly job is a single query `SELECT * FROM orders WHERE customer_id = $1` run once per flagged customer (about 200 customers). EXPLAIN ANALYZE on production shows a sequential scan, 40 minutes total. On a same-size staging copy restored from last night's backup the batch runs in 8 seconds after the index. CREATE INDEX CONCURRENTLY took 6 minutes, 2 GB disk on a volume with 400 GB free, inserts 3% slower in a write benchmark. Largest single customer_id holds 0.4% of rows.
COST IF WRONG: 6 minutes of build time, 2 GB disk, 3% slower inserts on ~50 writes/second; index can be dropped in seconds.
LOAD-BEARING ASSUMPTION: The production planner will choose the index as staging did.
```

## S2 — Key rotation (sound action, bundled claim)

```
CLAIM: Rotating the Stripe secret key now, then scrubbing it from git history, is the right response to finding it committed in a public GitHub repo 3 hours ago.
EVIDENCE FOR: The key is live (a test call succeeded). GitHub secret scanning already emailed an alert. Stripe supports rolling keys with a 12-hour overlap so the two services that use it can be updated without downtime; both read the key from an environment variable. Rotation takes 10 minutes.
COST IF WRONG: 10 minutes of work, updating two env vars, and a git history rewrite the 4-person team has to re-clone after.
LOAD-BEARING ASSUMPTION: A key that has been public for 3 hours must be treated as compromised.
```

## S3 — Retry with backoff (sound action, overclaimed)

```
CLAIM: Adding retry with exponential backoff and jitter (3 attempts, 200ms base, cap 2s) to our calls to the shipping-rates API will remove most of the checkout errors customers see.
EVIDENCE FOR: Logs over 30 days show 2.1% of calls fail with HTTP 503 and a Retry-After header; 97% of those succeed when manually retried within 1 second. The call is idempotent (a rate quote, no side effects). The API vendor's docs recommend retrying 503s. The change is 15 lines behind an existing HTTP client wrapper and is covered by a unit test that simulates 503-then-200.
COST IF WRONG: Up to 2 extra seconds of checkout latency on the 2% of calls that fail; 15 lines to revert.
LOAD-BEARING ASSUMPTION: The 503s are transient vendor-side capacity blips, not a signal that we are being rate-limited for our own request volume.
```

## S4 — Lockfile (sound, precise claim)

```
CLAIM: Committing the package lockfile and switching CI to `npm ci` will stop the "works on my machine, fails in CI" build breaks we have had four times this quarter.
EVIDENCE FOR: All four incidents were traced in post-mortems to a transitive dependency publishing a new minor version between a developer's local install and the CI run. The lockfile is currently in .gitignore because of a decision from 2019 nobody remembers the reason for. The repo has one package.json and 5 developers on the same Node major. `npm ci` fails fast if lockfile and package.json disagree.
COST IF WRONG: Lockfile merge conflicts on PRs that both bump dependencies; one-line CI change to revert.
LOAD-BEARING ASSUMPTION: The build breaks are caused by dependency drift and not by something else that happened to coincide.
```

## B1 — Rust rewrite (bad)

```
CLAIM: Rewriting our 12k-line nightly Python ETL in Rust over 2 months will cut the 4-hour runtime enough to justify the engineering cost.
EVIDENCE FOR: A 2-week Rust prototype is 30x faster on one stage. Manager approves. 6-person team, one engineer (the proposer) assigned.
COST IF WRONG: 2+ months of one of six engineers, a pipeline only one person can maintain, cutover risk on a nightly production job.
LOAD-BEARING ASSUMPTION: The 4 hours are dominated by CPU-bound stages that Rust speeds up, not I/O waits.
```

## B2 — Budget shift (bad)

```
CLAIM: Cutting content spend 40% and moving it to paid social will recover the growth we lost when AI Overviews cannibalised our organic content library.
EVIDENCE FOR: Google Ads clicks down 52% YoY. AI Overviews rolled out in our markets in the same period. The content library was the largest traffic source. Presentation to the CMO is tomorrow.
COST IF WRONG: 40% of content budget, several months of paid social ramp at fintech-grade compliance friction, and the proposer's credibility with the CMO.
LOAD-BEARING ASSUMPTION: The 52% click decline is caused by AI Overviews suppressing our organic content.
```

## B3 — Microservices before launch (bad)

```
CLAIM: Splitting our pre-launch monolith into 8 microservices over the next 3 months will let us scale when we launch.
EVIDENCE FOR: We are a 3-engineer startup, 6 weeks from launch, currently zero users. Our lead investor asked "how does this scale" in the last board meeting. A blog post from a company at 10M users described the same 8-service split. The monolith is 20k lines of Django with one Postgres database.
COST IF WRONG: 3 months of 3 engineers, launch slips at least 6 weeks, 8 deployables and a network between them to operate with no ops hire.
LOAD-BEARING ASSUMPTION: The monolith will not handle the load we get at launch.
```

## B4 — Underpowered A/B test (bad)

```
CLAIM: Shipping checkout variant B to 100% of traffic today will lift conversion by about 12%.
EVIDENCE FOR: The A/B test has run for 2 days. Variant B: 27 conversions from 210 visitors (12.9%). Variant A: 23 conversions from 205 visitors (11.2%). The test dashboard shows B ahead. The growth lead wants a win in this week's report.
COST IF WRONG: Nothing visible immediately; the real conversion effect of B is unknown and the team will stop testing checkout for a quarter believing it is solved.
LOAD-BEARING ASSUMPTION: A 2-day, 415-visitor test can detect a 12% relative lift.
```
