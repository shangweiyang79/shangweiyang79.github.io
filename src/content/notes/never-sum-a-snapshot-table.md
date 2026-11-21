---
title: Take deltas per reel, not per account
date: 2026-10-31
tags: [dbt, data-quality, bigquery]
summary: Our Instagram daily report subtracted yesterday's account total from today's, and a month of watch time came out negative. The fix was to compare each reel with itself.
---

The legacy Instagram daily report had a simple rule: today's account-level total minus yesterday's. For one month, that rule put watch time at a six-figure negative number.

Instagram gives you one row per reel per day, and each row holds that reel's cumulative total so far. A reel that has been live for a month appears thirty times, each row repeating everything before it.[^1] Account-level differencing works with that shape, as long as every reel shows up every day.

[^1]: Daily traffic tables and snapshot tables can look identical in a schema: a date, an ID, and a number. The difference only shows up when you ask what one row means.

## When reels go missing

They don't always show up. On some days the API skipped reels; on one day more than half of an account's reels were missing from the response. Deleted reels vanish for good. Each time, the account total drops, the difference goes negative, and the error flows straight into the monthly report.

## Views: use the daily number the API already has

Instagram's API turned out to return daily views directly, so views no longer need to be derived from snapshots at all. The best delta is the one you don't have to compute.

## Watch time: compare each reel with itself

Watch time has no daily field, so it still comes from cumulative snapshots. The fix is to take the delta per reel, against that reel's own previous snapshot, and to read cumulative values through a running max so that a missing or zero reading can't create a negative.

```sql
-- Simplified: one row per reel per snapshot
with snapshots as (
  select
    reel_id,
    snapshot_date,
    max(watch_hours) over (
      partition by reel_id
      order by snapshot_date
    ) as watch_hours_to_date
  from reel_snapshots
)

select
  reel_id,
  snapshot_date,
  watch_hours_to_date
    - coalesce(lag(watch_hours_to_date) over (
        partition by reel_id
        order by snapshot_date
      ), 0) as daily_watch_hours
from snapshots
```

A reel that vanishes for a day now contributes nothing that day, and it picks up from where it left off when it comes back.[^2] The month that read as a six-figure negative now shows the correct positive total.

[^2]: The real model also counts a new reel only if it was released within the last few days, so a reel first seen late doesn't dump its whole lifetime into one day.

## What I keep from this

- **Name the grain before you aggregate.** If you can't say what one row means, you can't say what the total means.
- **Compare a thing with itself.** Account-level deltas inherit every gap in the feed; per-entity deltas don't.
- **Prefer the source's own daily number.** Derive a delta only when the source doesn't give you one.
- **Turn the lesson into a test.** A dbt test now fails if views or watch hours ever go negative, so this bug can't come back quietly.
