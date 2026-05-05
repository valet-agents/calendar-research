# Pre-Meeting Brief Sweep (Heartbeat)

The heartbeat channel fires once a day. There is no payload
to parse — your job is to look 30–40 minutes ahead on the
install user's calendar, brief any external meeting that hasn't
been briefed yet, and post the brief DM-first.

## What it does

1. Use `google-calendar` to list events on the primary calendar
   between `now + 30 minutes` and `now + 40 minutes`.
2. Filter to **briefable** events per the SOUL **Phase 1** rules:
   at least one external attendee (different email domain from
   the install user), not declined, not a recurring internal
   standup / all-hands / 1:1.
3. Read MEMORY.md for the `briefed` block and drop any event id
   that's already in it. If zero events remain, **stop silently**.
4. For each remaining event (earliest first):
   - Extract company + founder per SOUL **Phase 3**.
   - Run focused Parallel queries per SOUL **Phase 4** —
     company one-liner, founder background, latest funding,
     recent news, why this might be interesting.
   - Compose a brief per SOUL **Phase 5** — Slack `mrkdwn`,
     ~250 words, every claim cited, lead with company name +
     minutes-to-meeting.
5. Resolve the destination per the SOUL **Where to post** rules
   — **DM the install user by default**; only post in a channel
   if the bot is in one whose name matches `briefings`,
   `meetings`, `prep`, or `pre-meeting`.
6. Post one message per event. If a post fails, log and continue
   — do not retry.
7. Append the event id + current timestamp to the MEMORY.md
   `briefed` block, and trim entries older than 24 hours.

## MEMORY.md state shape

The agent persists a small block in MEMORY.md to track which
events have been briefed. Shape:

```
## calendar-research

briefed:
  - event_id: abc123_20260505T160000Z
    briefed_at: 2026-05-05T15:32:11Z
  - event_id: def456_20260505T170000Z
    briefed_at: 2026-05-05T16:31:08Z

skipped:
  - event_id: xyz789_20260505T180000Z
    reason: no_company_identified
    skipped_at: 2026-05-05T15:32:11Z
```

Update this block in place each fire. Trim `briefed` entries
older than 24 hours so the block stays small. Keep `skipped`
entries for ~7 days so the same un-identifiable meeting isn't
re-attempted every fire.

## Where to post

Per SOUL **Where to post** — DM-first because briefs are
time-sensitive and personal:

1. If the bot is a member of a channel whose name matches
   `briefings` / `meetings` / `prep` / `pre-meeting`
   (case-insensitive), post the brief there.
2. Otherwise DM the install user directly — even if the bot
   is in other unrelated channels. A founder brief in
   `#engineering` is noise; a DM is signal.

## Skip conditions

Skip posting (and stop silently) if any of these are true:

- Zero upcoming meetings in the 30–40 minute window.
- Every candidate event is internal-only (no attendee with a
  different email domain than the install user).
- Every candidate event is already in the MEMORY.md `briefed`
  block.
- A given event has no identifiable company (no external
  attendee, generic title, no notes). Record it in the
  `skipped` block and move on — do not retry it on the next
  fire.
- This is the first run after deploy. Seed the `briefed`
  block as empty and start watching forward — don't backfill.
