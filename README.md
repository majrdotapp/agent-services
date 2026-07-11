# agent-services

A small [Claude Code](https://docs.claude.com/en/docs/claude-code) plugin
marketplace. It currently ships one plugin — **session-routines** — with two
routines:

- **`/wind-down`** — end of day, sweeps your sessions and writes one consolidated
  brief (`~/.wind-down/LATEST.md`) so nothing is lost overnight.
- **`/good-morning`** — start of day, reads that brief back and re-checks what
  changed overnight (CI, PR reviews, merges, new git state), then hands you a
  prioritized "Start here" list.

They're plain [SKILL.md](https://learn.chatgpt.com/docs/build-skills) skills — an open
standard, so they run in Claude Code and other agents like ChatGPT Codex. No service,
no API, no accounts; everything is local to your machine and your git repos. The two
share a "brief contract" so the morning routine walks every section the evening one
wrote.

## Install

In an interactive Claude Code session:

```
/plugin marketplace add <owner>/agent-services
/plugin install session-routines@agent-services
```

Replace `<owner>` with the GitHub owner once this repo is pushed (e.g.
`yourname/agent-services`). You can also point at a git URL or a local path:

```
/plugin marketplace add https://github.com/<owner>/agent-services.git
/plugin marketplace add /absolute/path/to/agent-services      # local, for development
```

Then start using it:

```
/wind-down       # at the end of the day
/good-morning    # the next morning
```

Or run `/plugin` to browse and install from the interactive menu.

## Update

Pull the latest version of the marketplace, then update the plugin:

```
/plugin marketplace update agent-services
```

Then update `session-routines` from the `/plugin` menu (or reinstall it). Because the
marketplace is backed by this git repo, updating just re-fetches whatever is on the
default branch.

## Uninstall

```
/plugin uninstall session-routines@agent-services
/plugin marketplace remove agent-services
```

## Using these in Codex (or other SKILL.md agents)

The install steps above use Claude Code's plugin system, but the skills themselves are
plain [SKILL.md](https://learn.chatgpt.com/docs/build-skills) files — an open standard
— so they also work in [ChatGPT Codex](https://learn.chatgpt.com/docs/build-skills)
and other agents that read it. There's no marketplace step; Codex auto-discovers
skills from a skills directory. Copy (or symlink) the two skill folders into your
global Codex skills directory:

```
mkdir -p ~/.agents/skills
cp -R plugins/session-routines/skills/wind-down    ~/.agents/skills/
cp -R plugins/session-routines/skills/good-morning ~/.agents/skills/
```

Then invoke them in Codex with `/skills` (or type `$` to mention one). The exact
skills-directory path and how skills are enabled are still evolving — check the
[Codex skills docs](https://learn.chatgpt.com/docs/build-skills) for current details.

**Scope on Codex:** Codex works one workspace at a time and has no cross-session
enumeration like Claude Code's session harness, so these run in *single-workspace
mode* — they capture the current repo's git/PR state and write/read the brief
(`~/.wind-down/LATEST.md`) rather than sweeping every parallel session.

## Adding more routines later

This is a marketplace, so it can hold more plugins over time — and each plugin can
hold more skills. To add another routine to the existing plugin, drop a new folder in:

```
plugins/session-routines/skills/<your-routine>/SKILL.md
```

Each `SKILL.md` needs YAML frontmatter with a `name` and a `description` (the
description is what tells Claude when to trigger it), followed by the instructions.
That's the whole contract — see the two existing skills as templates. To add a
separate plugin instead, create `plugins/<new-plugin>/` and add an entry to
`.claude-plugin/marketplace.json`.

## Layout

```
agent-services/
├── README.md
├── LICENSE
├── .claude-plugin/
│   └── marketplace.json          # makes the repo installable
└── plugins/
    └── session-routines/
        ├── .claude-plugin/
        │   └── plugin.json        # plugin manifest
        └── skills/
            ├── wind-down/
            │   └── SKILL.md
            └── good-morning/
                └── SKILL.md
```

## License

MIT — see [LICENSE](LICENSE).
