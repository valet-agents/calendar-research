This folder contains the source for a Skilled Agent originally built for the Valet runtime. Changes should follow the Skilled Agent open standard.

## Setup

### Connectors

- **google-calendar**: Google Calendar via OAuth. The agent uses it to look 30–40 minutes ahead on the install user's primary calendar, extract attendees and notes, and answer ad-hoc schedule questions in Slack. Read-only. Add it from the catalog at the org level so other Calendar-powered agents can share it.
- **parallel-search-mcp**: Parallel's deep-research search MCP. The agent uses it to research each upcoming external meeting — company one-liner, founder background, latest funding, recent news — with source URLs returned alongside every result. Keyless; add it from the catalog at the org level.

### Channels

- **slack** (slack): The agent's per-agent Slack bot. Listens for @mentions and replies in-thread, and posts each pre-meeting brief — DM to the install user by default, or to a `#briefings` (or `#prep`, `#meetings`) channel if the bot has been invited to one. Slack writes use the auto-injected outbound Slack connector.
- **heartbeat** (heartbeat): Fires every 5 minutes to scan the calendar for meetings starting 30–40 minutes from now. Declared inline in `valet.yaml`, so it's created automatically by the dashboard setup flow.

### Secrets

This agent uses the OAuth variant of Google Calendar and the keyless Parallel MCP, so no API tokens are needed at the org or agent level. The Google OAuth grant happens in the dashboard setup flow when you connect Calendar.

### External Setup

1. After deploy, complete the Google OAuth grant in the dashboard. Authorize calendar read access on the account whose schedule you want briefed (this becomes the "install user" the briefs are DMed to).
2. Install the agent's Slack bot. By default the bot DMs the install user with each pre-meeting brief — no channel invite required for the heartbeat to work end-to-end.
3. Optional: invite the bot to a dedicated `#briefings` (or `#prep`, `#meetings`) channel if you'd rather briefs land in a shared partner channel instead of DMs. The agent routes to that channel as soon as it sees the bot is a member.
4. Invite the bot to any additional channels where teammates should be able to @mention it for ad-hoc research (e.g. a partner channel for *"brief me on Vela Robotics before my 9am"* follow-ups).
5. The first heartbeat fire after deploy seeds memory and starts watching the calendar window. To smoke-test sooner, @mention the bot in Slack with a question like *"who's on my schedule today?"* — that exercises the Calendar + Slack path without waiting for an upcoming meeting.

## Customizing

- **Lookahead window**: edit *Phase 1* of `SOUL.md` to widen or narrow the 30–40 minute window. Move the lower bound earlier (e.g. 45m) for partners who like more lead time, or tighten the upper bound (e.g. 35m) to reduce the chance of briefing a meeting that gets cancelled.
- **What counts as briefable**: edit *Phase 1* of `SOUL.md` to change the external-attendee heuristic. Default: any attendee whose email domain differs from the install user's domain. To include investor-side intros only, narrow to specific TLDs or maintain an allow-list. To exclude portfolio companies, add a deny-list of domains.
- **Briefings channel naming**: edit the **Where to post** section of `SOUL.md` to change which channel-name patterns trigger channel posting (default: `briefings`, `meetings`, `prep`, `pre-meeting`). Anything else falls back to DM-the-install-user.
- **Heartbeat cadence**: edit the `every` value on the `heartbeat` channel in `valet.yaml` (e.g. `1m` for partners with back-to-back schedules who want tighter timing, `10m` to reduce calendar-API load), then redeploy.
- **Brief format**: edit the *Phase 5* template in `SOUL.md` to match your firm's brief shape — add a `*Market*` block, drop `*Recent*`, change the section headers.
