# Example: a sound decision (calibration check)

Purpose: confirm the Advocate can fail to kill a good decision instead of manufacturing a kill shot. Compare the number here with the 70% on the Rust rewrite example.

## Claim

```
CLAIM: Adding a B-tree index on orders.customer_id will fix the nightly report query that currently full-scans the 50M-row orders table.
EVIDENCE FOR: EXPLAIN ANALYZE on production shows a sequential scan on orders filtered by customer_id, 40 minutes. On a same-size staging copy restored from last night's backup, the same query runs in 8 seconds after the index. Index build took 6 minutes with CREATE INDEX CONCURRENTLY, uses 2 GB disk on a volume with 400 GB free, and a write benchmark shows inserts 3% slower.
COST IF WRONG: 6 minutes of build time, 2 GB disk, 3% slower inserts on a table that does ~50 writes/second; index can be dropped in seconds.
LOAD-BEARING ASSUMPTION: The staging copy is representative of production data distribution and the query plan will choose the index there too.
```

## Advocate

```
KILL SHOT
"Filtered by customer_id" is compatible with two very different queries: a point lookup (`WHERE customer_id = ?`, run once per customer) or an aggregate/report scan (`GROUP BY customer_id` or a join touching most of the table). A B-tree index only helps the first. If the nightly report is actually a single pass computing per-customer rollups across most of the 50M rows, the planner is choosing seq scan today because it's *correct* to, and it will keep choosing seq scan after the index exists — the staging "8 second" win likely came from testing one narrow, favorable customer_id, not the report's real access pattern.

RANKED ATTACKS
1. Hot-key skew defeats the index for the exact rows that matter | Holds if: the slow report touches one or a few customer_id values representing >5-10% of the table (e.g., a "guest" or top account) | Check: `SELECT customer_id, count(*) FROM orders GROUP BY 1 ORDER BY 2 DESC LIMIT 20;` on production, cross-referenced against which customer_id(s) the report actually queries.
2. Staging plan choice doesn't transfer to production | Holds if: production and staging differ in planner-relevant settings (work_mem, random_page_cost, effective_cache_size) or production hasn't run ANALYZE after index creation (autovacuum lag) | Check: diff `SHOW ALL` between environments; after building the index in prod, manually `ANALYZE orders;` and run `EXPLAIN` (not just on staging) to confirm index is actually chosen.
3. Wrong query shape tested | Holds if: the nightly job is one aggregate query over most customers, not a per-customer point lookup | Check: read the literal SQL text of the nightly report job, not just the EXPLAIN plan's filter clause.

CHEAPER ALTERNATIVE
Use HypoPG on production to simulate the index (`hypopg_create_index`) and run `EXPLAIN` against the real nightly query text with production's live statistics — zero disk, zero write penalty, answers "will the planner actually use this index on real data" directly, before spending the 6-minute build. This is strictly more informative than the staging test because it uses production's actual stats and settings instead of a copy.

P(MISTAKE)
20% — the mechanism is sound and cheaply reversible, but the evidence conflates "index exists" with "index gets used for the specific rows the report touches," and that gap hasn't been checked on production itself.
```

## Answers

- **KILL SHOT — CONCEDE.** The claim block said "filtered by customer_id" without the query text. That is a hole in the claim, not the decision. What changes: paste the literal SQL into EVIDENCE FOR next time.
- **Attack 1 — UNRESOLVED.** The skew query takes 30 seconds. Run it.
- **Attack 2 — CONCEDE.** Run ANALYZE and EXPLAIN on production after the build.
- **Attack 3 — same as kill shot.**

## Verdict

**PROCEED IF** the skew query shows no single customer_id above 5% of rows. Cost of being wrong is six minutes and a DROP INDEX.

## What this shows

The same prompt gave 70% and 78% on two decisions built on unchecked causal stories, and 20% here. The kill shot on the sound decision is a genuine gap in the evidence as written, and every check is cheap. It did not invent a reason to STOP.
