# Make Scenario — Walkthrough

Scenario name: `LinkedIn Queue → Buffer`

Three modules, in order. Schedule = every 15 minutes.

---

## Module 1 — Data Stores → Search records

- **Data store:** `linkedin-queue` (the one you created from `05-data-store-schema.md`)
- **Filter:** add one rule → field `status` · operator `Equal to` · value `pending` (type literal text — don't use the variable picker)
- **Sort:**
  - **Key:** `scheduled_at`
  - **Order:** `Ascending`
- **Continue execution even if no results:** **Yes** ← CRITICAL. Otherwise Make emails you a "no results" error every 15 min.
- **Limit:** `10`

---

## Module 2 — Buffer → Create a Status Update

- **Connection:** your Buffer connection
- **Profiles:** open dropdown → uncheck Select All → check ONLY your target LinkedIn profile
- **Text:** click into field → in right-side variable picker, click `1. Record → text`
- **Publication:** `Post an update at scheduled date and time`
- **Date scheduled:** click into field → click `1. Record → scheduled_at`
- **Attach media to the update:** `No`

---

## Module 3 — Data Stores → Update a record

- **Data store:** `linkedin-queue`
- **Key:** click into field → in variable picker, click `1. Record → Key` (NOT `post_id` — this is the data store's internal record key)
- **Insert missing record:** `No` ← CRITICAL. Prevents accidental ghost records.
- **Overwrite an existing array in the record:** `No`
- **Record:** find the `status` field, set it to literal text `queued`. Leave other fields blank — Make's update module won't overwrite blank fields.

---

## Schedule

1. Bottom-left of canvas → click the **scheduling clock**
2. Set: **Every 15 minutes** (this is Make Free's minimum interval)
3. Save the scenario (floppy-disk icon)
4. **Activate** with the green toggle in the top-right

---

## What the activated scenario does

Every 15 min, Make runs the scenario:

1. Module 1 searches the data store for records where `status = pending`, sorted by `scheduled_at` ascending, max 10
2. For each matching record (Make iterates), Module 2 sends it to Buffer with the text and scheduled time
3. Buffer accepts and queues the post for the exact `scheduled_at` moment
4. Module 3 updates the record's `status` from `pending` to `queued` so it doesn't get re-processed

When the queue is empty, Module 1 returns 0 records, the route stops cleanly (because of "Continue if no results = Yes"), and no error fires.

---

## Pitfalls we hit so you don't have to

### "Whoops, you've posted that one recently" from Buffer

Cause: Module 3 failed to update `status` on a previous run, so the same record got picked up again and submitted to Buffer twice.
Fix: The Monday task's defensive verification step manually patches stuck records to `queued` after 60 sec.

### Scenario errors with "Validation failed for 2 parameters"

Cause: Module 2's `Text` or `Date scheduled` mapping got broken (chip went red/empty) when you saved the module.
Fix: Open Module 2, verify both fields show blue/green chips like `1. Record: text` and `1. Record: scheduled_at`. If empty, re-map from the variable picker.

### Make scenario runs but Buffer queue stays empty

Cause: Module 2's Profile field doesn't have a real LinkedIn profile selected.
Fix: Open Module 2 → Profiles dropdown → make sure your LinkedIn profile is checked (the display might show an alphanumeric ID — that's normal, just confirm via the dropdown).

---

## Testing the scenario in isolation (no Claude required)

You can sanity-test the scenario without the Claude side:

1. In Make, open your data store → click "Add record" → fill in test values manually
2. Set `status: pending`, `scheduled_at: [some future time, at least 30 min out]`, `text: "Test from Make UI"`
3. Click "Run once" on the scenario
4. Watch all 3 modules turn green
5. Check Buffer queue for the test post
6. Delete the test post from Buffer before its scheduled time

If all 3 modules go green and Buffer receives the post, the publishing layer is solid. Then connect Claude.
