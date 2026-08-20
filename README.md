# agent-services

A simple way to begin and end your day for people who spend all day in agentic coding tools. Especially Claude. 

Two plugins so far.

**session-routines** — two routines:

- **`/wind-down`** — end of day, sweeps your sessions and writes one consolidated
  brief (`~/.wind-down/LATEST.md`) so nothing is lost overnight.
- **`/good-morning`** — start of day, reads that brief back and re-checks what
  changed overnight (CI, PR reviews, merges, new git state), then hands you a
  prioritized "Start here" list.

**doc-from-chat** — one routine:

- **`/doc-from-chat`** — turns the conversation you just had into documentation.
  It scans the session for how-it-works explanations, architecture, and answered
  questions, verifies each claim against the code, then writes or updates docs in
  the project's `docs/` folder — support docs for end users in `docs/`,
  developer-facing architecture docs in `docs/internal/`.

They're plain [SKILL.md](https://developers.openai.com/codex/skills) skills — an open
standard, so they run in Claude Code and other agents like ChatGPT Codex. No service,
no API, no accounts; everything is local to your machine and your git repos. The two
session routines share a "brief contract" so the morning routine walks every section
the evening one wrote.

## Install

In an interactive Claude Code session:

```
/plugin marketplace add majrdotapp/agent-services
/plugin install session-routines@agent-services
/plugin install doc-from-chat@agent-services
```

You can also point at a git URL or a local path:

```
/plugin marketplace add https://github.com/majrdotapp/agent-services.git
/plugin marketplace add /path/to/agent-services       # local clone, for development
```

Then start using it:

```
/wind-down       # at the end of the day
/good-morning    # the next morning
/doc-from-chat   # after a session that explained how something works
```

Or run `/plugin` to browse and install from the interactive menu.

## Update

Pull the latest version of the marketplace, then update the plugins:

```
/plugin marketplace update agent-services
```

Then update the plugins from the `/plugin` menu (or reinstall them). Because the
marketplace is backed by this git repo, updating just re-fetches whatever is on the
default branch.

## Uninstall

```
/plugin uninstall session-routines@agent-services
/plugin uninstall doc-from-chat@agent-services
/plugin marketplace remove agent-services
```

## Using these in Codex (or other SKILL.md agents)

The skills themselves are plain [SKILL.md](https://developers.openai.com/codex/skills) files — an open standard
— so they also work in [ChatGPT Codex](https://developers.openai.com/codex) and other
agents that read it. There's no marketplace step; Codex auto-discovers skills from a
skills directory. Clone the repo and copy the skill folders into your user-level
Codex skills directory (`~/.agents/skills`):

```
git clone https://github.com/majrdotapp/agent-services.git
cd agent-services
mkdir -p ~/.agents/skills
cp -R plugins/session-routines/skills/wind-down    ~/.agents/skills/
cp -R plugins/session-routines/skills/good-morning ~/.agents/skills/
cp -R plugins/doc-from-chat/skills/doc-from-chat   ~/.agents/skills/
```

Then invoke them in Codex with `/skills` (or type `$` to mention one, e.g.
`$wind-down`). See the [Codex skills docs](https://developers.openai.com/codex/skills)
for more.

**Scope on Codex:** Today, Codex works one workspace at a time and has no cross-session
enumeration like Claude Code's session harness, so the session routines run in
*single-workspace mode* — they capture the current repo's git/PR state and write/read
the brief (`~/.wind-down/LATEST.md`) rather than sweeping every parallel session.
`/doc-from-chat` is single-session by design, so it works the same everywhere.

## License

MIT — see [LICENSE](LICENSE).
