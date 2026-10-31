---
title: Never sum a snapshot table
date: 2026-10-31
tags: [dbt, data-quality, bigquery]
summary: Summing Instagram's raw table inflated views 100× or more, and the old fix produced negative days. The grain of a table decides what you can add up.
---

A plain `SUM` over our raw Instagram table returns views more than 100 times too high. Nothing is broken. The table was simply never meant to be added up.

Instagram gives you one row per reel per day, and each row holds that reel's cumulative total so far. A reel that has been live for a month appears thirty times, each row repeating everything before it.[^1]

[^1]: Daily traffic tables and snapshot tables can look identical in a schema: a date, an ID, and a number. The difference only shows up when you ask what one row means.

## The obvious fix has its own bug

The legacy report took a smarter route: today's account total minus yesterday's. That works until the API skips some reels on a given day. The account total drops, the difference goes negative, and a month of watch time can come out below zero.

## Compare each reel with itself

The fix is to take the delta per reel, against that reel's own previous snapshot, and to read cumulative values through a running max so that a missing or zero reading can't create a negative.

```sql
-- Simplified: one row per reel per snapshot
with snapshots as (
  select
    reel_id,
    snapshot_date,
    max(views) over (
      partition by reel_id
      order by snapshot_date
    ) as views_to_date
  from reel_snapshots
)

select
  reel_id,
  snapshot_date,
  views_to_date
    - coalesce(lag(views_to_date) over (
        partition by reel_id
        order by snapshot_date
      ), 0) as daily_views
from snapshots
```

A reel that vanishes for a day now contributes nothing that day, and it picks up from where it left off when it comes back.[^2]

[^2]: The real model also counts a new reel only if it was released within the last few days, so a reel first seen late doesn't dump its whole lifetime into one day.

## What I keep from this

- **Name the grain before you write the `SUM`.** If you can't say what one row means, you can't say what the total means.
- **Compare a thing with itself.** Account-level deltas inherit every gap in the feed; per-entity deltas don't.
- **Turn the lesson into a test.** A dbt test now fails if views or watch hours ever go negative, so this bug can't come back quietly.
