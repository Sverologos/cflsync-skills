# cflsync-skills

The [confluence sync skill](skills/confluence-sync/SKILL.md) supports shared human/AI authoring
of Confluence pages in local [cflsync](https://github.com/sverologos/cflsync)
workareas. The skill provides guidance for operating the existing cflsync CLI,
with local authoring as the central workflow.

The tool reference targets cflsync 0.5.7 (`v0.5.7`), workarea/cache format 3,
and Pandoc JSON API 1.23.1.2. See the skill's
[setup guidance](skills/confluence-sync/references/setup.md) for compatibility and older
workarea or markup transitions.

## Workflows

- Set up a workarea rooted in a selected Confluence page.
- Pull pages for initial synchronization.
- Edit existing pages, synchronize changes, resolve conflicts, and publish
  when requested.
- Create an empty remote page, then author its body from scratch; publish those
  changes when requested.
- Import one or more Markdown files as new pages under an existing workarea's
  root, deriving titles from level-one headings or filenames. Copy referenced
  local media and rewrite attachment links. Empty pages are created remotely;
  imported content and attachments remain local without an automatic push.
- Import a directory of Markdown files, including nested subdirectories.
  Top-level files use the workarea root; files in a subdirectory use the page
  whose title matches that subdirectory's name, falling back to the root when
  no page matches. Each file follows the same import procedure without a push.
- Copy an ordinary Confluence source page identified by title, then customize
  the new page. This workflow uses the remote page as a content template, but
  does not instantiate a native Confluence template.

The instructions preserve existing human edits, unmanaged files, attachments,
and opaque page content. They distinguish local editing from remote creation
and publishing, respect the configured workarea boundary, and require
deliberate conflict resolution rather than automatic force overwrites.

## Installation

Install the confluence sync skill (`confluence-sync`) from
`skills/confluence-sync/` in a repository checkout or from the
`confluence-sync/` directory extracted from a ZIP on
[GitHub Releases](https://github.com/sverologos/cflsync-skills/releases).

Install cflsync and compatible Pandoc separately. The agent needs filesystem
and command execution access, network access to Confluence, and a configured
cflsync authentication profile; installing the skill does not supply these.
See the [cflsync installation instructions](https://github.com/sverologos/cflsync#installation).

Copy the complete `confluence-sync/` directory, including `SKILL.md`, its license, and
references, into one of these locations:

| Agent | Project installation | Personal installation | Official documentation |
| --- | --- | --- | --- |
| Claude Code | `.claude/skills/confluence-sync/` | `~/.claude/skills/confluence-sync/` | [Claude Code skills](https://code.claude.com/docs/en/skills) |
| Codex | `.agents/skills/confluence-sync/` | `~/.agents/skills/confluence-sync/` | [Codex skills](https://learn.chatgpt.com/docs/build-skills) |
| GitHub Copilot | `.agents/skills/confluence-sync/` or `.github/skills/confluence-sync/` | `~/.agents/skills/confluence-sync/` or `~/.copilot/skills/confluence-sync/` | [Copilot skills](https://docs.github.com/en/copilot/concepts/agents/about-agent-skills) |
| pi.dev | `.agents/skills/confluence-sync/` | `~/.agents/skills/confluence-sync/` | [Pi skills](https://pi.dev/docs/latest/skills) |

Project paths are relative to the target workarea or repository root.
Personal installations apply across projects. A single `.agents/skills/`
installation can serve Codex, Copilot, and pi; Claude Code needs its own
location. Choose one location per agent to avoid duplicate discovery.

For example, on Linux, macOS, or WSL, install personally for Codex, Copilot,
and pi with:

```sh
skill_source="/absolute/path/to/cflsync-skills/skills/confluence-sync"
skill_destination="$HOME/.agents/skills/confluence-sync"
mkdir -p "$skill_destination"
cp -R "$skill_source/." "$skill_destination/"
```

For Claude Code, use
`skill_destination="$HOME/.claude/skills/confluence-sync"` instead. For a
project installation, use an absolute destination under the target workarea,
such as `/absolute/path/to/workarea/.agents/skills/confluence-sync` or
`/absolute/path/to/workarea/.claude/skills/confluence-sync`.

Start the agent in the target workarea after copying. Claude Code supports
`/confluence-sync`, Codex supports `$confluence-sync`, and pi supports
`/skill:confluence-sync` (or `/reload` to refresh an existing session).
In Copilot, request use of the confluence sync skill for the task.

## License

Copyright © 2026 Sverologos BV.

This project is licensed under the Mozilla Public License 2.0
(`MPL-2.0`). See [LICENSE.md](LICENSE.md) for the license text.
