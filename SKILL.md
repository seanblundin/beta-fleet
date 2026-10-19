---
name: beta-fleet
description: Send five first-time users (subagents in real browser tabs) through an app before launch, fix what they hit, rerun fresh users to prove it, and report back as one clear HTML page. Use when the user types /beta-fleet or asks to beta test an app like a new user.
---

*Last edited: 2026-10-19*

# Beta fleet

Built on [this tweet](https://x.com/RoundtableSpace/status/2111717667228770611): your first real users won't tell you why they left, but these five will.

1. Start the app (local dev server if there is one). Write one sentence for "first success": the first moment a new user gets real value.
2. Launch five subagents at once, each on Opus with high effort (`model: "opus"`, `effort: "high"`), each a different first-time user:
   1. The impatient one: skims, clicks the first button, quits at the first error
   2. The skeptic: reads pricing, terms and privacy before signing up
   3. The mobile user: does everything on a 375px screen on slow 3G
   4. The non-native English speaker: only follows what the UI literally says
   5. The breaker: pastes junk into forms, hits back mid-flow, double-clicks everything

   Each opens the app in its own browser tab, goes from the landing page to first success, and screenshots every moment it gets confused, stuck or annoyed. On a local server, give each one its own host name (`http://<name>.localhost:<port>`) so their logins don't mix. If the browser tool can only drive one page at a time, run them one after another instead.
3. Merge everything into one list ranked by how many users hit it, and fix it all in one pass. Leave pricing, legal text and positioning to the user.
4. Rerun five fresh subagents that know nothing about the first run, to prove it's fixed.
5. Report back with one HTML page, opened for the user. Keep it short and plain. Use headers that say exactly what's below them ("Fixed", "Still broken", "Your call"), and put a before and after screenshot next to each issue.
