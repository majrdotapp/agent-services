---
name: good-morning
description: Morning pickup routine. Reads last night's wind-down brief, re-checks what changed overnight (CI, PR reviews, merges, new git state) across your sessions, and presents a prioritized "Start here" list. Use when you invoke /good-morning or say "good morning", "wake up", "resume", "pick up where I left off", "morning brief", or "what was I working on".
---

# good-morning

Companion to `/wind-down`. Reads the consolidated brief written last night and tells
you where to start today, factoring in anything that changed overnight (CI finished,
a reviewer left comments, a PR merged).

The two skills share a **brief contract** — the section list `/wind-down` writes.
This skill **walks every section** and re-checks it, so nothing the evening captured
silently drops out of the morning. If you edit one skill's section list, edit the
other.

**Write it to be read.** Lead with the blocker/outcome, rank by impact, plain
language, scannable. Don't soften it into a neutral status dump.

## Requirements

Same portability as `/wind-down` (plain SKILL.md; runs in Claude Code, Codex, and
other agents). If your agent can't enumerate sessions, still read the brief and
re-check git/PR state for what it lists — just note that live session state couldn't
be refreshed (single-workspace mode).

## Workflow

### 1. Read the brief
Read `~/.wind-down/LATEST.md`. If it doesn't exist, tell the user there's no
wind-down brief yet (suggest running `/wind-down` at end of day) and offer to run a
fresh live sweep using the `/wind-down` logic instead. If the brief is several days
old, say so — the "overnight" deltas may span a longer gap.

Parse each contract section so you can re-check it below: **Done today**, **Dangling
work (by repo)**, **Monitoring watch** (if present, + its "Watch tonight" line),
**Resume tomorrow**, **Sessions**, **Uncommitted work**, **Cleanup candidates**.

### 2. Re-check overnight deltas
For each session listed under **Sessions**, check what changed since the brief was
written. If your agent supports subagents, spawn them in parallel; otherwise run the
read-only commands directly. For each:
- `git -C <dir> fetch` then `git -C <dir> status -sb` (new upstream commits? still dirty?)
- if a PR exists and `gh` is available:
  `gh pr view <n> --json state,statusCheckRollup,reviewDecision,url,comments` —
  surface **new review comments and bug reports** (from a human *or* an automated
  reviewer), CI pass/fail, and merge state. New review bugs route to *Needs
  attention* in step 3 and outrank a green CI check.
- re-list sessions once to see any new activity / state changes since the brief.

All commands are read-only. Do not modify any worktree.

### 2a. Re-sweep dangling work (by repo)
Don't rely only on the session list — the brief's **Dangling work** section may name
repos/branches not tied to a listed session. For each distinct repo in that section,
re-run the read-only sweep (`git -C <repo> fetch`, `status --porcelain`,
`log @{u}.. --oneline`, `worktree list`) and report what **changed**: a dirty
worktree now clean (or newly dirty), an unpushed branch now pushed/merged, a parked
branch that moved. Anything still dangling carries forward into "Start here."

### 2b. Re-check the monitoring watch (only if the brief had one)
If the brief has a **Monitoring watch** section, re-query the same surface and report
*what changed overnight* — render it back in the same table shape so it round-trips.
Read-only. Lead the morning output with anything that fired or regressed; it outranks
clean git state. **Close the loop on "Watch tonight"**: the brief named one specific
signal and what "healthy" looks like — check *that* signal and say whether it landed
healthy, is still pending, or missed. Skip this step entirely if the brief had no
monitoring section.

### 3. Present "Start here"
Output a prioritized list, blockers first. These tiers re-bucket the brief's **Resume
tomorrow** list by what changed overnight:
1. **🔴 Needs attention** — a regression from step 2b, failing CI, "changes
   requested", merge conflicts, a "Watch tonight" signal that missed, or **new bug
   reports / review comments overnight** (human *or* automated reviewer). Unresolved
   review bugs outrank a green CI check — a PR can be CI-green and still need work.
2. **▶ Ready to continue** — clean, next action known (carried from Resume tomorrow).
3. **✅ Newly done overnight** — PR merged / CI green *and no open review bugs* →
   candidate to wrap or archive (cross-reference the brief's **Cleanup candidates**
   and re-offer any now-eligible).

For each item give: title, repo/branch, **what changed overnight** (if anything), the
**next action**, and the `sessionId` + repo so you can jump back into that session.
Keep it scannable — one short block per session.

When 2a/2b ran, include a short **📟 Overnight watch** block: any monitoring table
(same shape as the brief), any repo whose dangling state changed, and the "Watch
tonight" verdict — so you see the system's health at a glance even when nothing broke.

## Notes
- This skill reports and prioritizes; it does not resume work automatically or switch
  sessions for you.
- It only reads. It never commits, stashes, or mutates a worktree.
