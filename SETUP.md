# Setup — 6 Steps

Follow in order. Each step ~5–15 min.

---

## Step 1 — Fill in `01-brand-voice-TEMPLATE.md`

The hardest and most important step. Open Claude and prompt:

> Build me a brand-voice doc for my LinkedIn presence. I'm pasting 3–5 of my recent posts. Cover: 3–5 tone adjectives with quoted examples from my posts, "things I'd say" with patterns, "things I'd never say," 5 sample post openings in my voice, hashtag patterns I use.

Paste your 3–5 recent posts. Take the output, drop it into `01-brand-voice-TEMPLATE.md`, replacing the placeholders.

Save the final file as `brand-voice.md` in a known location on your computer.

**Time: 10–15 min.**

---

## Step 2 — Fill in `02-sources-TEMPLATE.md` and `03-ad-hoc-finds-TEMPLATE.md`

Open the sources template, list the tools / newsletters / feeds you want monitored weekly. Heaviest weight (40%) goes to YOUR OWN channels — your YouTube, blog, podcast — because first-person practitioner content is highest-trust.

Open the ad-hoc-finds template and leave it empty. You'll capture week-by-week.

Save both to the same folder as your `brand-voice.md`.

**Time: 5 min.**

---

## Step 3 — Buffer

1. Sign up at buffer.com (free)
2. Connect your LinkedIn profile
3. That's it. Buffer is the publishing endpoint. We won't use its queue templates — Make controls timing.

**Time: 5 min.**

---

## Step 4 — Make data store + scenario

See `05-data-store-schema.md` and `06-make-scenario-walkthrough.md`.

Short version:
1. Sign up at make.com (free)
2. Create a Data Store called `linkedin-queue` with the 5 fields from the schema
3. Create a Scenario called `LinkedIn Queue → Buffer` with 3 modules (Search records → Buffer Create → Update record)
4. Set schedule = Every 15 minutes
5. Activate

**Time: 15 min.**

---

## Step 5 — Claude scheduled task

1. In Claude Cowork, connect the Make MCP from the registry (one click)
2. Create a new scheduled task
   - Schedule: cron `0 9 * * 1` (every Monday 9am local time)
   - Prompt: paste the full contents of `04-monday-task-prompt.md`
   - Replace ALL placeholders (paths, data-store ID, YouTube channel) with your own values
3. Click "Run now" once to pre-approve the tools the task uses, then cancel the run

**Time: 10 min.**

---

## Step 6 — Sunday test, then ship

1. Saturday or Sunday: manually trigger your Monday task ("Run now" in the Scheduled section)
2. Work through the review widget — approve / edit / skip / cancel each idea
3. Click submit, watch the calendar render, open Buffer to confirm posts are queued
4. Monday morning: first real auto-fire — same flow, no human trigger needed

**Time: 10 min for the first test. Zero after that.**

---

## Total: ~1 hour to build, ~5 min/week to run

If anything breaks, the most likely cause is a mapping in the Make scenario. See the pitfall callouts in `README.md`.
