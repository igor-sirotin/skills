# skills

Personal [Claude Code](https://claude.com/claude-code) skills. Logos-specific ones live in [skills-logos](https://github.com/igor-sirotin/skills-logos).

Each top-level directory is one skill (a `SKILL.md`, plus any supporting files).
The repo lives anywhere; `link.sh` symlinks each skill into `~/.claude/skills/<name>`,
where Claude Code discovers it. `~/.claude/skills/` itself stays unversioned, so
Claude Code's own `synced/` and `.trash/` never touch this repo.

## Skills

- **writing-pr-descriptions** — write a short, list-style PR description.

## Install

```sh
git clone https://github.com/igor-sirotin/skills.git ~/src/skills
~/src/skills/link.sh
```

Re-run `link.sh` after adding, renaming or removing a skill: it links new ones,
prunes links to removed ones, and skips any real directory already in the way.
