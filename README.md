# cflsync-skills

The [cflsync skill](skills/cflsync/SKILL.md) supports shared human/AI authoring
of Confluence pages in local [cflsync](https://github.com/sverologos/cflsync)
workareas. The skill provides guidance for operating the existing cflsync CLI,
with local authoring as the central workflow.

## Workflows

- Set up a workarea rooted in a selected Confluence page.
- Pull pages for initial synchronization.
- Edit existing pages, synchronize changes, resolve conflicts, and publish
  when requested.
- Copy an ordinary Confluence source page identified by title, then customize
  the new page. This workflow uses the remote page as a content template, but
  does not instantiate a native Confluence template.

The instructions preserve existing human edits, unmanaged files, attachments,
and opaque page content. They distinguish local editing from remote creation
and publishing, respect the configured workarea boundary, and require
deliberate conflict resolution rather than automatic force overwrites.

## Installation

Install the `cflsync` skill from `skills/cflsync/` in a repository checkout or
from the `cflsync/` directory extracted from a ZIP on
[GitHub Releases](https://github.com/sverologos/cflsync-skills/releases).

Install cflsync and compatible Pandoc separately. The agent needs filesystem
and command execution access, network access to Confluence, and a configured
cflsync authentication profile; installing the skill does not supply these.
See the [cflsync installation instructions](https://github.com/sverologos/cflsync#installation).

Copy the complete `cflsync/` directory, including `SKILL.md`, its license, and
references, into one of these locations:

| Agent | Project installation | Personal installation | Official documentation |
| --- | --- | --- | --- |
| Claude Code | `.claude/skills/cflsync/` | `~/.claude/skills/cflsync/` | [Claude Code skills](https://code.claude.com/docs/en/skills) |
| Codex | `.agents/skills/cflsync/` | `~/.agents/skills/cflsync/` | [Codex skills](https://learn.chatgpt.com/docs/build-skills) |
| GitHub Copilot | `.agents/skills/cflsync/` or `.github/skills/cflsync/` | `~/.agents/skills/cflsync/` or `~/.copilot/skills/cflsync/` | [Copilot skills](https://docs.github.com/en/copilot/concepts/agents/about-agent-skills) |
| pi.dev | `.agents/skills/cflsync/` | `~/.agents/skills/cflsync/` | [Pi skills](https://pi.dev/docs/latest/skills) |

Project paths are relative to the target workarea or repository root.
Personal installations apply across projects. A single `.agents/skills/`
installation can serve Codex, Copilot, and pi; Claude Code needs its own
location. Choose one location per agent to avoid duplicate discovery.

For example, on Linux, macOS, or WSL, install personally for Codex, Copilot,
and pi with:

```sh
skill_source="/absolute/path/to/cflsync-skills/skills/cflsync"
skill_destination="$HOME/.agents/skills/cflsync"
mkdir -p "$skill_destination"
cp -R "$skill_source/." "$skill_destination/"
```

For Claude Code, use
`skill_destination="$HOME/.claude/skills/cflsync"` instead. For a
project installation, use an absolute destination under the target workarea,
such as `/absolute/path/to/workarea/.agents/skills/cflsync` or
`/absolute/path/to/workarea/.claude/skills/cflsync`.

Start the agent in the target workarea after copying. Claude Code supports
`/cflsync`, Codex supports `$cflsync`, and pi supports
`/skill:cflsync` (or `/reload` to refresh an existing session).
In Copilot, request use of the `cflsync` skill for the task.

## License

Copyright © 2026 Sverologos BV.

This project is licensed under the Mozilla Public License 2.0
(`MPL-2.0`). See [LICENSE.md](LICENSE.md) for the license text.
