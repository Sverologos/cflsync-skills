---
name: confluence-sync
description: >-
  Operate cflsync for shared human/AI authoring of Confluence Cloud pages in
  local workareas, including setup, synchronization, publishing, conflict
  resolution, creating pages, importing Markdown files or directories, and copying
  template pages. Use for requests mentioning
  confluence sync, cflsync, cflsync workarea, Confluence workarea,
  Confluence push, or Confluence pull. Applies to CLI workarea operations,
  rather than developing cflsync or general Confluence administration.
license: MPL-2.0
compatibility: >-
  Requires filesystem and command execution access, separately installed
  cflsync supporting workarea/cache format 3, Pandoc with JSON API
  1.23.1.2, network access to Confluence Cloud, and a configured cflsync
  authentication profile. See references/setup.md for capability checks.
metadata:
  author: Sverologos BV
  version: "0.1.0"
  cflsync-version: "0.5.7"
  cflsync-revision: "4ec3f3ecb3968017fc9fdde9e7140af4a228eafc"
  pandoc-json-api: "1.23.1.2"
---

<!--
Copyright (c) 2026 Sverologos BV.
This Source Code Form is subject to the terms of the Mozilla Public License,
v. 2.0. If a copy of the MPL was not distributed with this file, You can obtain
one at https://mozilla.org/MPL/2.0/.
-->

# Confluence sync

Use the existing cflsync CLI to maintain one rooted Confluence page tree and
its local Markdown and attachments. Local authoring is the central workflow;
setup, synchronization, publishing, page creation, Markdown import, and page
copy support it.

## Select guidance

- Read only the references required by the task. Paths below are relative to
  this skill directory; the package carries its operational guidance.

| Task | Read |
| --- | --- |
| Initialize a workarea, configure authentication, or resolve a missing/incompatible dependency | [Setup](references/setup.md) |
| Initial synchronization | [Setup](references/setup.md) and [Synchronization](references/synchronization.md) |
| Edit an existing page | [Authoring](references/authoring.md) and [Synchronization](references/synchronization.md) |
| Pull, push, inspect status, or resolve a conflict | [Synchronization](references/synchronization.md) |
| Recover missing page content/directories or diagnose missing/unreferenced attachments | [Recovery](references/synchronization.md#recover-missing-local-state) and [Attachment validation](references/authoring.md#attachment-validation) |
| Create a page from scratch | [Page creation](references/creation.md), then authoring/synchronization guidance as needed |
| Import one or more Markdown files into an existing workarea, leaving imported content local | [Markdown import](references/import.md), with creation/authoring/synchronization guidance as directed |
| Import a directory of Markdown files, selecting parents by subdirectory title matches | [Directory import](references/import.md#directory-import), then the shared single-page steps in that guide |
| Copy and customize a template page | [Template pages](references/template-pages.md), then authoring/synchronization guidance as needed |

- Reuse compatible dependencies during ordinary authoring. Load installation
  guidance only for missing or incompatible dependencies; do not trigger upgrades.
- Check the installed CLI's help for commands required by the task.

## Establish identity and scope

- Find the workarea by walking upward from the requested working directory to
  `.cflsync/profile`. Read that profile name and `.cflsync/root` to establish
  the configured root ID. `.cflsync/version` must hold workarea version `3`;
  use setup guidance for incompatible workareas. Private `.cflsync/cache/`
  state is not page content; do not fabricate or modify it to bypass an error.
  No file in `.cflsync` or its subdirectories may be altered except through
  the cflsync CLI.
- A managed page contains `content.md`, `_attachments/`, and cached child-page
  directories. Other entries are unmanaged; preserve them. Run commands from
  the workarea root when a page directory may move or be renamed.
- Every managed page directory, including the root page's, ends in `_PAGE_ID`.
  The CLI encodes and may truncate the title portion. Retain the installed path
  instead of constructing it from a title.
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
- When a page directory or `content.md` is missing, remote page content is
  authoritative; use the recovery guide instead of reconstructing or merging
  its missing body. Page-content links define required attachments. Missing
  referenced files are errors; unreferenced local attachments are warnings.
  Report either condition and consult the user about how to proceed.
- Never delete local attachments without explicit user consent covering the
  affected files, including deletion performed by a CLI command. Existing
  explicit authorization carries through; a generic sync/recovery request,
  force flag, or backup does not provide deletion consent.
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
  except a preserved-baseline refresh in the documented merge procedure or
  remote-authoritative recovery of a missing page body/directory. These
  exceptions do not waive the attachment-deletion consent requirement.
- `page create` and `page copy` create remote pages before local customization.
  Do not describe either operation as an unpublished local draft or repeat it with
  a known created ID or an uncertain remote outcome.
- Check command exits and resulting status. Report local edits, remote
  creation, publication, and verified synchronization as distinct outcomes,
  including failed or skipped pages. An unchanged status is evidence only at
  the time checked; local conversion does not verify browser rendering or
  app-macro behavior. Do not create Git commits implicitly.

Upstream application: [cflsync](https://github.com/sverologos/cflsync).
Copyright and license: [LICENSE.md](LICENSE.md).
