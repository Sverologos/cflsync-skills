---
name: cflsync
description: >-
  Operate cflsync for shared human/AI authoring of Confluence Cloud pages in
  local workareas, including setup, synchronization, publishing, conflict
  resolution, and copying template pages. Use for requests mentioning
  Confluence sync, cflsync, cflsync workarea, Confluence workarea,
  Confluence push, or Confluence pull. Applies to CLI workarea operations,
  rather than developing cflsync or general Confluence administration.
license: MPL-2.0
compatibility: >-
  Requires filesystem and command execution access, separately installed
  cflsync with rooted workareas and native page copy, Pandoc with JSON API
  1.23.1.2, network access to Confluence Cloud, and a configured cflsync
  authentication profile. See references/setup.md for capability checks.
metadata:
  author: Sverologos BV
  version: "0.1.0"
  cflsync-revision: "69ebb39239c449ab9b5921e847df09cf7764a5c0"
  pandoc-json-api: "1.23.1.2"
---

<!--
Copyright (c) 2026 Sverologos BV.
This Source Code Form is subject to the terms of the Mozilla Public License,
v. 2.0. If a copy of the MPL was not distributed with this file, You can obtain
one at https://mozilla.org/MPL/2.0/.
-->

# cflsync

Use the existing cflsync CLI to maintain one rooted Confluence page tree and
its local Markdown and attachments. Local authoring is the central workflow;
setup, synchronization, publishing, and page copy support it.

## Select guidance

Read only the references required by the task. Paths below are relative to
this skill directory; the package carries its operational guidance.

| Task | Read |
| --- | --- |
| Initialize a workarea, configure authentication, or resolve a missing/incompatible dependency | [Setup](references/setup.md) |
| Initial synchronization | [Setup](references/setup.md) and [Synchronization](references/synchronization.md) |
| Edit an existing page | [Authoring](references/authoring.md) and [Synchronization](references/synchronization.md) |
| Pull, push, inspect status, or resolve a conflict | [Synchronization](references/synchronization.md) |
| Copy and customize a template page | [Template pages](references/template-pages.md), then authoring/synchronization guidance as needed |

Ordinary authoring reuses compatible dependencies; it does not trigger tool
upgrades or load installation instructions unless a dependency is missing or
incompatible. Check the installed CLI's help for commands required by the task.

## Establish identity and scope

- Find the workarea by walking upward from the requested working directory to
  `.cflsync/profile`. Read that profile name and `.cflsync/root` to establish
  the configured root ID. Private `.cflsync/cache/` state is not page content;
  do not fabricate or modify it to bypass an error. No file in .cflsync or its
  subdirectories may be altered except through the cflsync CLI.
- A managed page contains `content.md`, `_attachments/`, and cached child-page
  directories. Other entries are unmanaged; preserve them. Run commands from
  the workarea root when a page directory may move or be renamed.
- Page references are classified as an existing local path, then a numeric
  page ID, then an exact title. A local file must be managed `content.md`; a
  local directory must identify a cached page. Ordinary commands remain inside
  the root tree. Only copy may read an external source on the configured site.
- Titles prefer cached matches; ambiguous matches require a page ID or other
  clarification. Retain the resolved page ID for subsequent operations.
  Paths can change after pull, rename, move, or title disambiguation.
- `page status` resolves cached local state only. For an unpulled in-tree page,
  use a targeted pull to establish its local baseline before editing; do not
  treat inability to run page status as evidence of synchronization.

## Shared authoring contract

- Preserve unrelated human edits, attachment changes, and unmanaged files.
  Re-read affected files before applying an edit and incorporate intervening
  changes rather than replacing a stale snapshot.
- Keep an editing-only request local. An edit-and-publish request includes
  push; carry existing authorization through that task without repeated
  confirmation. A generic sync request with changes on both sides requires
  reconciliation rather than choosing a side implicitly.
- Prefer page-specific commands for page-specific requests. Whole-tree
  operations are appropriate only for whole-workarea scope.
- Preserve the generated title heading, attachment references, special spans,
  and opaque `atlas_doc_format` blocks. Use `page rename` for title changes and
  cflsync commands for page creation, copying, moving, and removal; do not
  create, rename, or move managed page directories manually.
- Inspect local and remote status before editing or publishing. Pull
  remote-only changes first, incorporate local-only changes, and resolve
  two-sided changes deliberately. Force options select one side; they do not
  merge. Deletion and force flags require the corresponding requested scope,
  except a preserved-baseline refresh in the documented merge procedure.
- A copy creates a remote page before local customization. Do not describe
  that operation as an unpublished local draft. Do not repeat a creation with
  a known created ID or an uncertain remote outcome.
- Check command exits and resulting status. Report local edits, remote
  creation, publication, and verified synchronization as distinct outcomes,
  including failed or skipped pages. An unchanged status is evidence only at
  the time checked; local conversion does not verify browser rendering or
  app-macro behavior. Do not create Git commits implicitly.

Upstream application: [cflsync](https://github.com/sverologos/cflsync).
Copyright and license: [LICENSE.md](LICENSE.md).
