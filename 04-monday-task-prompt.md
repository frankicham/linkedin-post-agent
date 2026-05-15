# Monday 9am Scheduled Task — Prompt

Paste this into a new Claude Cowork scheduled task.

**Schedule:** cron `0 9 * * 1` (every Monday at 9am local time).
**Pre-flight:** Click "Run now" once after saving, approve each tool that prompts for permission, then cancel the run. Future runs go through without interruption.

---

## Placeholders to replace (search and replace before saving)

| Placeholder | Replace with |
|---|---|
| `<BRAND_VOICE_PATH>` | Absolute path to your `brand-voice.md` file |
| `<SOURCES_PATH>` | Absolute path to your `sources.md` file |
| `<AD_HOC_FINDS_PATH>` | Absolute path to your `ad-hoc-finds.md` file |
| `<MASTER_LOG_PATH>` | Absolute path to your `scheduled-posts.json` master log |
| `<WEEKLY_SNAPSHOTS_DIR>` | Absolute path to a folder where weekly md snapshots go |
| `<MAKE_MCP_PREFIX>` | The Make MCP tool prefix (varies per install — find via Claude's MCP registry connection) |
| `<DATA_STORE_ID>` | The numeric ID of your `linkedin-queue` data store in Make |
| `<YOUTUBE_CHANNEL_URL>` | Your channel URL, e.g. `https://www.youtube.com/@YOURHANDLE/videos` |
| `<CHANNEL_NAME>` | Buffer channel label, e.g. `linkedin-yourhandle` |
| `<LOCAL_TIMEZONE>` | e.g. Australia/Sydney, America/New_York |
| `<TZ_OFFSET>` | e.g. +10:00, -05:00 |

---

## The prompt (copy from here)

```
You are running on behalf of [YOUR NAME] to generate the weekly LinkedIn batch. Today is Monday — deliver 5 reviewed/approved posts into Buffer's queue via Make's data-store path.

This session has no memory of prior weeks. Follow every step.

# STEP 0 — Idempotency check

Call: `<MAKE_MCP_PREFIX>data-store-records_list` with `dataStoreId: <DATA_STORE_ID>`.

If any records have `status: pending` or `queued` AND `scheduled_at` within the next 7 days, a batch already exists. STOP and tell the user: "Already a batch in queue for this week — skipping. Existing posts: [list titles + times]. Reply 'force new batch' if you want me to add another batch on top."

# STEP 1 — Read context

Read these files:
- `<BRAND_VOICE_PATH>` — voice rules
- `<SOURCES_PATH>` — source list with tier weighting
- `<AD_HOC_FINDS_PATH>` — recent finds (note any active entries)
- `<MASTER_LOG_PATH>` — last week's batch (avoid repeating angles)

If any file is missing, STOP and tell the user where it should be.

# STEP 2 — Research fresh angles

**Tier 1 (40% weight):** Use WebFetch on `<YOUTUBE_CHANNEL_URL>` — extract recent video titles + descriptions. Two of the 5 ideas should anchor on the user's own video content.

**Tier 2 (50% weight):** Use WebSearch on key sources from `<SOURCES_PATH>` — find 1-2 timely hooks from this week (new product launches, fresh newsletter pieces, industry announcements).

**Tier 3 (10% weight, fallback):** The user's own IP frameworks from the brand-voice doc.

**Ad-hoc:** Check active entries in `<AD_HOC_FINDS_PATH>`. Weave any unused ones in.

# STEP 3 — Generate 5 idea candidates

Each idea uses a different format. Mix:
1. **Channel-anchored** (Tier 1) — "In this week's video on X…"
2. **Channel-anchored** (Tier 1, different format from #1)
3. **Tool-led practitioner** (Tier 2) — anchored on a source's fresh news
4. **Framework opener** (Tier 3) — user's own IP
5. **Hot take / acknowledge-then-improve** (Tier 2) — riff on an industry trend

Each idea must include:
- Working title
- Format (pattern name)
- Source (where the angle came from)
- Hook line (actual opening sentence, in user's voice)
- Topics (tags)
- Hashtags (3-5, format matching brand-voice doc conventions)
- Why it works (1 sentence on what makes it on-brand and timely)

Apply ALL voice rules from `<BRAND_VOICE_PATH>` strictly.

# STEP 4 — Render review widget

Use `mcp__visualize__show_widget` to render the 5-idea review widget with 4 actions per idea:
- **Approve** (green) — use as-is
- **Edit** (amber) — modify with notes
- **Skip** (blue) — generate a replacement idea
- **Cancel** (red) — no post for this slot

Pattern: cards with format/source/topics/hashtags meta, hook line in serif italic, why-it-works small text, action buttons with state badges, sticky submit button. Submit fires sendPrompt with structured decisions.

# STEP 5 — Wait for review

The user's submit message contains decisions. Wait for it.

# STEP 6 — Process decisions

- **APPROVE** → write the full post (120-220 words) in user's voice
- **EDIT** → write the full post applying user's notes
- **SKIP** → generate 1 replacement idea with different angle, write it as draft
- **CANCEL** → no post for this slot

# STEP 7 — Assign random times

For each active post (non-cancelled), assign a random time between 08:00-09:55 in `<LOCAL_TIMEZONE>`.

Slots: Tue / Wed / Thu / Fri this week + Mon next week (5 slots).

Use bash + python for randomization:
```
python3 -c "import random; print(f'{random.randint(8,9):02d}:{random.randint(0,55):02d}')"
```
Don't pick round times. Aim for varied minutes (8:14, 9:37, 8:53).

# STEP 8 — Push to data store

For each active post, call `<MAKE_MCP_PREFIX>data-store-records_create`:
- `dataStoreId: <DATA_STORE_ID>`
- `data: { post_id, text, scheduled_at (ISO 8601 with <TZ_OFFSET>), status: "pending", channel: "<CHANNEL_NAME>", approved_at: now }`

# STEP 9 — Defensive verification

Wait 60 seconds. Then call `data-store-records_list` to verify.

For each post you created:
- If `status` is `queued` → great, Make scenario processed it
- If still `pending` after 60s → manually patch via `data-store-records_update` setting `status: "queued"` (Make Module 3 has a stale-record edge case)

# STEP 10 — Save logs

- Append new posts to `<MASTER_LOG_PATH>` (preserve existing entries)
- Write fresh `<WEEKLY_SNAPSHOTS_DIR>/week-of-YYYY-MM-DD.md`

# STEP 11 — Re-render calendar widget

Use `mcp__visualize__show_widget` to render a live calendar showing:
- This week's queued posts (date + time + title + tags + status)
- Any cancelled slots (greyed, strikethrough)
- Footer with ops used + summary count

# STEP 12 — Send summary

Brief message: "X posts queued, Y skipped (regenerated), Z cancelled." + link to weekly snapshot.

# Constraints

- Never invent quotes from real people. Use named sources only when verifiable.
- Never use surveillance framing or generic AI hype.
- Match the user's spelling convention from the brand-voice doc.
- If WebFetch on the channel URL fails, fall back to one extra Tier 2 idea.
- If `<BRAND_VOICE_PATH>` is missing, STOP and ask — do not invent voice rules.
```
