---
name: doc-from-chat
description: Extracts how-it-works information, architecture explanations, and answered questions from the current conversation, verifies the claims against the code, then creates or updates documentation in the project's docs/ folder — customer-friendly support docs in docs/, developer-facing architecture docs in docs/internal/. This skill should be used when the user invokes /doc-from-chat, asks to "document this" or "save this to docs", or when significant how-the-app-works information has been discussed and should be captured as documentation.
---

# doc-from-chat

Scan the current conversation for information about how the app works, its architecture, or questions the user asked and the answers given. Verify each claim against the code, then write it as markdown files in the project's `docs/` folder.

## When to Run

- User invokes `/doc-from-chat`
- User says "document this", "save this to docs", "turn this into documentation", or similar
- End of a session where significant how-it-works information was discussed

**Run it close to the conversation, not hours later.** Long sessions get summarized and detail disappears from context. Run the skill when the explanation happens. If the session will run long, run a pass mid-session rather than waiting for an end-of-day routine like `/wind-down`.

## Workflow

### 1. Scan the Conversation

Read the recent conversation exchanges and identify content in these categories:

- **How it works** — explanations of features, flows, or behaviors
- **Architecture** — how components fit together, data flow, key design decisions
- **Q&A** — questions the user asked and the answers given about the app
- **Setup / configuration** — steps or requirements explained during the session

Ignore: code diffs, debugging back-and-forth, tool call output, git commands, unrelated chatter.

### 2. Verify Claims Against the Code

Chat answers are sometimes wrong, and some become stale during a session (a behavior discussed early may have been changed by a later fix). Before a claim is written to a doc:

- Spot-check each factual claim against the current code (read the relevant file, grep for the behavior)
- If a claim cannot be verified quickly, either drop it or mark it inline with `<!-- unverified -->` so a later docs-audit pass can catch it
- Never document a behavior that the session itself changed without reflecting the final state

### 3. Read the Docs Index, Then Related Docs

`docs/README.md` is the index: a title and one-line summary per doc, split into **Support** and **Internal** sections. Read it first to find related docs — do not rely on scanning filenames alone. Then read any docs the index shows as related.

Determine for each piece of extracted information:

- **Add to existing file** — if a doc on this topic already exists
- **Create new file** — if no doc covers this topic yet; prefer updating over creating a near-duplicate

If `docs/README.md` does not exist, create it from the current contents of `docs/` before writing anything else.

### 4. Route by Audience

Two audiences, two locations:

- **Support topics** (features, flows, setup, Q&A a customer would ask) → `docs/`
  Write for a curious end user: plain English, short sentences, active voice, no jargon without explanation.
- **Architecture topics** (component relationships, data flow, design decisions) → `docs/internal/`
  Write for a developer: precise terms are fine, but stay explanatory — these docs onboard a future session, not just a future human.

If one conversation produced both kinds of content, split it into separate files. Never write architecture content in the customer register or vice versa.

### 5. Write or Update Documentation

**File naming:** `<topic-slug>.md` — lowercase, hyphenated, descriptive
Examples: `docs/how-analysis-works.md`, `docs/getting-started.md`, `docs/internal/media-pipeline.md`

**Required structure for each doc:**

```markdown
---
topic: <short topic phrase>
audience: support | internal
last-updated: <YYYY-MM-DD>
source: <one-line note on the session that produced/updated this, e.g. "2026-08-20 session on video cache">
---

# <Title — plain English>

<One sentence that answers "what is this about?">

## Overview
<2–4 sentences of context>

## <Section per subtopic>
<Explanation. Use bullet lists for steps or options. Bold key terms on first use.>
```

The YAML frontmatter replaces any older `*Last updated:*` footer. When updating a doc that still has the footer, migrate it to frontmatter.

**When updating an existing file:**

- Append new sections or expand existing ones — do not rewrite content that is already accurate
- **If the conversation contradicts existing content, correct it.** Verified information from this session wins over stale text. Rewrite the inaccurate passage; do not append a conflicting section alongside it
- Update `last-updated` and `source` in the frontmatter

**After writing:** update `docs/README.md` — add a line for each new doc, and refresh the summary line for any doc whose scope changed.

### 6. Confirm

After writing, list:

- Files created or updated, with a one-line summary of what was added to each
- **Corrections made** — any existing content that was rewritten because the session contradicted it, with a one-line before/after
- Any claims marked `<!-- unverified -->`

## Notes

- If the `docs/` or `docs/internal/` folder does not exist, create it before writing any files
- Do not document implementation details only relevant to developers in a support doc; route them to `docs/internal/` instead. Skip internals with no explanatory value (e.g., actor isolation mechanics) unless the user explicitly asks
- Keep each doc focused on one topic — split into multiple files if the content spans unrelated areas
