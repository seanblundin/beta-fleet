---
name: beta-fleet
description: Send a small fleet of ruthless first-time users (subagents in real browser tabs) through an app before launch, rank every place they got confused, stuck or annoyed, fix it all in one pass, then rerun fresh users to prove it. Use when the user types /beta-fleet, or asks to "QA this like a new user" or "send in first users" before a launch.
---

*Last edited: 2026-10-19*

Goal: find out why your first real users would leave before any real user does.

# Beta fleet

Built on [this tweet](https://x.com/RoundtableSpace/status/2111717667228770611): "Your first 100 real users won't tell you why they left. These 5 will." You are the launch QA lead. You plan, send in the users, merge what they find, fix it, and judge the rerun. The judging stays with you.

Arguments (all optional): a URL to test, `report` (find and rank but don't fix), or extra personas to add.

## 1. Get ready (you, before anyone else starts)

- **Check for a real browser.** The personas need browser tools that a subagent can drive: the Claude desktop app's built-in browser, Claude in Chrome, Playwright MCP, or Chrome DevTools MCP. If none is connected, stop and tell the user how to add one, for example `claude mcp add playwright -- npx @playwright/mcp@latest`, then restart Claude Code.
- **Find out whether the browser can run personas in parallel.** Tools that take a `tabId` on every call (the desktop app's built-in browser, Claude in Chrome) can run all six at once, each in its own tab. Tools that keep one "current page" for everyone (Playwright MCP, Chrome DevTools MCP) would let the personas take over each other's pages, so run those personas one at a time.
- **Find the app and start it.** Prefer the local dev server. A staging host is fine if the user named it. On a production site, the fleet stays logged out: personas stop at the sign-up wall and report what they saw up to that point.
- **Define "first success" in one sentence.** Base it on the README and the code: the first moment a new user gets real value (for example: "creates a project and sees it on the dashboard"). Every persona is racing toward that sentence.
- **Write the sixth persona** (see below) from what the product is for.
- **Give each persona its own host name** so cookies and storage don't leak between them. On a local server, use `http://<persona>.localhost:<port>`. Browsers send any `*.localhost` name to this machine, and each one gets separate cookies. If the dev server rejects the host name, allow it in the dev config (for example Vite's `server.allowedHosts`). On staging, tabs share one login, so each persona logs out before it starts.
- **Make test accounts.** Use one per persona, with an email like `beta-fleet+<persona>-<run>@example.test` and a generated password. Record them in the project's seed or example-config file, never in chat. If sign-up needs email verification, find the link in the dev server logs or the local mail catcher and hand it over the way an inbox would. Use only a payment provider's test mode and its published test cards, and only on local or staging.
- **Make a run folder:** `<project>/.beta-fleet/<YYYY-MM-DD-HHMM>/`, and add `.beta-fleet/` to `.gitignore`.
- **Desktop app's built-in browser only:** open the pane once yourself with `preview_start` or `navigate` to an http(s) URL. Subagents can't open it, and `file://` and `data:` pages can't be driven.

## 2. Send in the fleet

Launch the six as subagents (`general-purpose`, on your most capable model), in the background, all in one message if the browser allows it. Each brief contains the persona, the start URL (its own host name), its test account, the first-success sentence, its run subfolder, and the rules below. Don't tell any persona what you suspect is broken.

**The personas**

1. **The impatient one.** Skims, reads only headlines and button labels, clicks the first button that looks right, and quits at the first error or at any wait longer than about 5 seconds. Quitting is the finding: say where and why.
2. **The skeptic.** Reads pricing, terms and privacy before signing up. Flags missing or broken pages, contradictions between pages, hidden costs, scary permissions, vague answers about what happens to their data, and pushy dark patterns. Signs up only if convinced, and says what convinced them.
3. **The mobile user.** Sets its own tab to a 375×812 touch screen before loading anything (`resize_window` with the `mobile` preset, or the browser tool's device emulation), and does everything one-handed. Flags tap targets that are too small, text that is cut off, sideways scrolling, a keyboard that covers the field, and hover-only controls. For slow 3G: use real network throttling if the browser tool has it. Otherwise read the network requests, add up the bytes needed before the page is usable, estimate at about 50 KB/s plus 2 s per round trip, and report the estimated seconds per screen.
4. **The non-native English speaker.** Follows only what the UI literally says. Doesn't guess from convention or icons. Flags idioms, jargon, clever copy, icon-only buttons, and error messages that don't say what to do next.
5. **The breaker.** Pastes junk into every field (emoji, 5,000 characters, `<b>x</b>`, `'; drop table`, leading spaces, negative numbers, empty submits), hits back and refresh in the middle of flows, double-clicks every submit, and opens the same flow in a second tab. Only touches its own test account and only this app.
6. **The one it's for.** The real target customer you wrote in step 1, with a real job to get done. This persona asks the question the other five don't: did first success actually deliver what was promised, and would they come back tomorrow or pay? Flags any moment of "I don't get what this is for."

**Rules for every persona**

- Use only what's on the screen. Don't read source code, don't run page JavaScript, and don't open dev tools. The mobile user's network check and console errors are the exceptions.
- Work in your own tab and never touch anyone else's. If the tools take a `tabId`, create your tab and pass its id on every call.
- Go from the landing page to first success. Stay in character the whole way.
- Take a screenshot at every moment you get confused, stuck, annoyed or delighted, and save it in your run subfolder as `NN-short-label.png` (or `.jpg`). If the tool saves screenshots somewhere else and gives you the path, copy the file in. If the tool can't save to disk at all, describe the moment in enough detail that someone could find it again.
- At the end, write `findings.json` in your subfolder: `{persona, reached_first_success, steps, gave_up_at, would_return, moments: [{n, kind: "blocker"|"stuck"|"confused"|"annoyed"|"delight", url, did, expected, happened, quote, screenshot}]}`. A `quote` is one line in the persona's own voice.
- Your final message is a five-line summary, not the whole file.

## 3. Merge and rank (you)

- Read every `findings.json` and look at the screenshots yourself.
- Merge moments that share one root cause into one issue, even when the personas described it differently.
- Reproduce each issue yourself before you believe it. Drop what you can't reproduce, and note that you dropped it.
- Rank by how many personas hit it, but put blockers first: anything that stopped a persona from reaching first success. Then sort by count, then by stuck > confused > annoyed.

## 4. Fix it all in one pass (skip this step for `report`)

- Fix every issue in one pass, starting at the top of the list.
- Some issues are calls for the user, not fixes: pricing, the text of legal terms, positioning, or whether a feature should exist. Don't change those. List each one with your recommendation.
- Keep the delights. Don't redesign something a persona loved while fixing the thing next to it.

## 5. Rerun to prove it

- Send in all six personas again as **fresh** agents with the same briefs and new run subfolders. They get no knowledge of the first run. Fresh eyes are the whole point, and primed testers confirm what they're told.
- You mark each original issue as fixed, still there, or made worse, and you list anything new. If blockers remain, do one more fix round and rerun only the personas that hit them. Stop after that and report what's left.

## 6. Report

Write `report.html` in the run folder: a ranked issue list with a before and after screenshot for each issue, each persona's journey (reached success or not, steps, where they quit, would they return), and the judgment calls for the user. Open it for the user. In chat, give the top three issues, the judgment calls, which issues you're least sure are really fixed, and how to check them.

Clean up when you're done: reset any emulated tab to desktop size, close the fleet's tabs, and delete the test accounts if the app makes that easy.
