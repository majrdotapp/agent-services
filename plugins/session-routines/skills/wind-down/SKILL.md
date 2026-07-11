---
name: wind-down
description: End-of-day routine across ALL your coding-agent sessions. Captures where every active session stands, flags uncommitted work, and writes one consolidated brief so you can pick up cleanly tomorrow. Use when you invoke /wind-down or say "wind down", "I'm logging off for the night", "good night", "shutting down for the day", or "wrap up for today".
---

# wind-down

If you run several agent sessions in parallel across repos, this routine
sweeps them at the end of the day and writes one consolidated brief so nothing is
lost and you can resume cleanly tomorrow. Run it from whichever session you're in;
it inspects the *other* sessions and writes a single file.

Companion: `/good-morning` reads that brief back and re-checks what changed
overnight. The two share a **brief contract** — the section list in step 4.
`/good-morning` walks every section this skill writes, so if you edit one skill's
section list, edit the other.

**Write it to be read.** Lead with outcomes, rank by impact (blockers and broken
flows first), plain language, keep it scannable. Separate signal from noise. The
whole point of the brief is that it's fast to skim tomorrow — don't flatten it into
a neutral log.

## Requirements

These are plain [SKILL.md](https://developers.openai.com/codex/skills) skills and
run in any agent that reads that open standard (Claude Code, Codex, and others).

The routine shines when your agent can enumerate your *other* sessions — a
`list_sessions` / `archive_session` pair such as Claude Code's session-management
tools. Tool names vary by agent; discover them rather than hard-coding.

- If your agent has **no way to enumerate other sessions** (e.g. Codex today),
  run in **single-workspace mode**: capture just the current repo's git + PR state
  and write a one-workspace brief. Say so plainly.
- If session enumeration returns **only the current session**, say so and skip
  writing an empty brief.

## What "wind down" means here (read this first)

There is **no clean way to freeze another running session mid-thought** from
outside. So this routine does NOT send halt signals. "Wind down" = *capture each
session's state from disk + git so you can safely close everything, and flag
anything unfinished.* Be honest about this if asked.

- **WIP is flag-only.** NEVER run `git stash`, `git commit`, `git add`, or any
  mutating git command in any worktree. Only read state and report it.
- **Archiving is cleanup only** — used in step 5 on sessions that are already
  *done* (merged/closed PR or long idle), after explicit confirmation. Never use it
  to "pause" in-progress work.

## Workflow

### 1. Enumerate sessions
List the available sessions (e.g. `list_sessions`). Each entry typically has an id,
title, working directory, running flag, PR number/state, and a last-activity
timestamp. Exclude the current session.

### 2. Filter to the active working set
Keep a session only if **any** of these is true:
- it is running, OR
- it was active within the last ~24h, OR
- it has an open PR (state not `MERGED` and not `CLOSED`), OR
- (discovered in step 3) its worktree has uncommitted changes.

Everything else is noise — list it only under "Cleanup candidates" (step 4), not
the detailed summary. Cap the detailed set at ~12 sessions; if more qualify, keep
the most recently active and note how many were omitted. Group results by repo
(derive the repo name from the working directory).

### 3. Gather per-session state
Work through the active sessions and build a compact structured summary for each. If
your agent can spawn subagents, run **one per active session** concurrently (all in a
single message); otherwise gather them sequentially. For each session:

1. **Find the transcript** and read roughly the **last 60 lines** to see what it was
   doing. Storage differs by agent — adapt to wherever yours keeps session
   transcripts (Claude Code, for example, uses
   `~/.claude/projects/*/<sessionId>.jsonl`, one JSON object per line). In
   single-workspace mode there's only the current session, so skip the hunt.
2. **Derive the real working dir + branch** from the most recent entries — a
   session may have been working in a git worktree, not the main repo path.
3. **Read git + PR state** in that dir (read-only commands only):
   - `git -C <dir> status --porcelain` (dirty files)
   - `git -C <dir> branch --show-current`
   - `git -C <dir> log --oneline -5`
   - unpushed: `git -C <dir> log @{u}.. --oneline` (ignore error if no upstream)
   - if a PR exists and `gh` is available: `gh pr view <n> --json state,statusCheckRollup,reviewDecision,url`
4. **Return** this structure (omit fields it can't determine):
   ```
   { title, repo, branch, sessionId, lastActivity,
     whatItWasDoing,   // 1–2 sentences from the transcript tail
     whereItStopped,   // the last meaningful action / state
     nextAction,       // the single best next step to resume
     dirtyFileCount, dirtyFiles[],
     unpushedCommitCount,
     pr: { number, state, ci, review, url },
     openThreads[] }   // unresolved items near the end of the transcript
   ```
   Keep it grounded in what's actually in the transcript/git — don't invent a
   `nextAction` the evidence doesn't support.

### 4. Write the brief (the shared contract)

Assemble these sections **in this order**. `/good-morning` walks every one, so
don't drop a section — write it and note "none" if empty.

Write the full brief to both:
- `~/.wind-down/STATUS-YYYY-MM-DD.md` (dated, today's date)
- `~/.wind-down/LATEST.md` (overwrite — `/good-morning` reads this)

Use this structure:

```markdown
# Wind-down brief — <YYYY-MM-DD HH:MM>

<N active sessions captured · M with uncommitted work · K cleanup candidates>

## ✅ Done today
<3–6 concise bullets: what shipped and what moved forward across all sessions
today. Grouped by theme, outcomes first, PR numbers in parentheses, ordered by
impact. Final bullet = raw counts (PRs merged today · worktrees still dirty).>
- ...

## 🧵 Dangling work (by repo)
<Cross-repo inventory of loose ends, one line per repo, ordered by impact: open PRs
awaiting merge, dirty worktrees, unpushed commits, in-flight or parked branches,
half-done threads. Separate real WIP from artifacts (stray screenshots, tooling
files, ` 2.ext` duplicate-suffix cruft). Flag a checkout left dirty on `main`.>
- **<repo>** — <loose ends, real WIP vs. noise>
- ...
→ <one-sentence lead-in handing off to the plan below>

## 📟 Monitoring watch (overnight) — OPTIONAL
<Include this section ONLY if a repo in the active set has a monitoring or
observability tool connected (e.g. via an MCP server). Snapshot, read-only, what
will be watching while you're away — alerts and their state, a headline dashboard
value, recent errors — as a short table. If nothing is connected, omit the whole
section. Keep it vendor-neutral; there is no built-in monitoring here.>

**Watch tonight:** <the single signal most worth a glance in the morning, and what
"healthy" looks like for it — /good-morning closes the loop on this one.>

## ▶ Resume tomorrow (ranked by impact)
<One line per active session, ranked by impact — a broken/blocked flow outranks
internal cleanup regardless of CI state. Within equal impact, break ties by
urgency: failing CI / "changes requested" > dirty WIP / unpushed > clean & idle.
Say in one phrase why each lands where it does.>
- **<title>** (<repo>/<branch>) — Pick up: <nextAction>  · `<sessionId>`
- ...

## Sessions
### <title> — <repo> / <branch>
- **Last active:** <relative time>
- **Status:** <whatItWasDoing> <whereItStopped>
- **Next:** <nextAction>
- **PR:** #<n> <state> · CI <ci> · review <review> — <url>   (omit if none)
- **Open threads:** <openThreads, or "none">
- `<sessionId>`
### ... (repeat per session)

## ⚠ Uncommitted work (flag only — nothing was stashed or committed)
| Repo / branch | Dirty files | Unpushed commits | Session |
|---|---|---|---|
| ... | <count> | <count> | <title> |

## 🧹 Cleanup candidates
<Sessions with merged/closed PRs or multi-day idle — offered for archive next.>
- <title> (<repo>) — PR #<n> <MERGED/CLOSED> · idle <time> · `<sessionId>`
```

### 5. Offer cleanup
List the cleanup candidates and ask which (if any) to archive. Only on explicit
confirmation, archive each chosen session with a short reason (e.g. "PR #18
merged"). Never archive speculatively or in bulk without being told what to archive.

### 6. Recap
Print a short terminal summary: sessions captured, how many have dirty WIP, how many
archive candidates, and the path to the brief (`~/.wind-down/LATEST.md`).
Remind the user `/good-morning` will pick it back up.

## Notes
- The brief is global (spans all repos) by design — that's why it lives under
  `~/.wind-down/` in your home directory, not in any one project's directory.
