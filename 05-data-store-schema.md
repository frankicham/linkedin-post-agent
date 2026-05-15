# Make Data Store — Schema

In Make: **Data Stores → Add Data Store**. Name it `linkedin-queue`. Create a new Data Structure with these 6 fields:

| Field | Type | Required | Default | Notes |
|---|---|---|---|---|
| `post_id` | Text | ✅ | — | Your unique post identifier, e.g. `user-2026-05-12-001` |
| `text` | Text | ✅ | — | The full LinkedIn post copy (LinkedIn supports up to 3,000 chars) |
| `scheduled_at` | Date | ✅ | — | ISO 8601 datetime with timezone offset, e.g. `2026-05-12T09:18:00+10:00` |
| `status` | Text | ✅ | `pending` | One of: `pending` / `queued` / `posted` |
| `channel` | Text | — | `linkedin-yourhandle` | Buffer profile label — useful when you extend to X / multiple accounts |
| `approved_at` | Date | — | — | When you approved it in the review widget |

Set max size = 1 MB (plenty for hundreds of posts).

---

## Field semantics

- **post_id** — your own ID. Use a date-based scheme so it's sortable and human-readable.
- **text** — exactly what gets posted, including hashtags. LinkedIn renders `\n\n` as paragraph breaks.
- **scheduled_at** — must include the timezone offset, or Make/Buffer will interpret as UTC. Sydney = `+10:00`. US Eastern = `-05:00`.
- **status** — the scenario filters by `pending`, updates to `queued` after Buffer accepts the post.
- **channel** — purely informational for now. Becomes useful when you fan out to multiple platforms.
- **approved_at** — useful for analytics later ("how long between approval and publish?"). Optional.

---

## Why these 6 fields and not more?

You'll be tempted to add: `format`, `topics`, `hashtags` (as separate field), `source_url`, `predicted_engagement`, etc. Resist.

The data store should hold ONLY what Buffer needs to publish + what the agent needs to dedupe. Everything else (format, topics, why-it-works) lives in the master log (`scheduled-posts.json`) — that's your analytics layer.

The data store is a queue. Keep it lean.
