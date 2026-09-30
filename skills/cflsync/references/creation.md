<!--
Copyright (c) 2026 Sverologos BV.
This Source Code Form is subject to the terms of the Mozilla Public License,
v. 2.0. If a copy of the MPL was not distributed with this file, You can obtain
one at https://mozilla.org/MPL/2.0/.
-->

# Create from scratch or import a Markdown draft

## Establish destination and creation scope

Establish the workarea, destination parent, new title, and whether the requested
body and attachments should be published or left local. Check
`cflsync page create --help` if the installed capability has not been established.
Use the setup guide only for missing dependencies or a missing workarea.

The parent must be a page inside the configured root tree, including the root
itself. Resolve it using a managed path, numeric ID, or exact title; clarify
ambiguity and retain its ID. The title must be non-empty, single-line text
without surrounding whitespace. Creation uses the parent's space and has no
offline or deferred-creation mode.

A request to create a Confluence page authorizes remote creation. If the request
requires only a local draft or no remote visibility before review, prepare the
draft outside the managed page tree and leave creation pending. `page create`
creates an empty remote page immediately; later local edits are separate from
that creation and require a push to appear remotely.

## Create once and retain identity

Run from the workarea root with the established parent ID and title:

```console
cflsync page create PARENT_ID "New page title"
```

Before remote creation, the CLI installs the parent and any missing ancestors.
It preserves existing local ancestor edits; do not push those edits merely to
create a child. An unmanaged entry occupying the proposed page path is refused
before creation. Preserve it and resolve the clash deliberately. A cached
sibling using the directory name causes the new directory to receive a page-ID
suffix. Do not construct managed directories or change `.cflsync/` manually.

The CLI creates an empty remote child, then pulls it into the workarea with a
generated title heading and attachment directory. Check the exit status before
editing. At the documented revision, successful creation does not print the
new page's ID or path. Inspect the newly installed directory under the parent,
then use its managed path to obtain the ID and actual title:

```console
cflsync page status "path/to/new/page/content.md"
```

Retain that ID as `NEW_ID` and the installed path. Do not select an arbitrary
title match or assume a title-derived path when a suffix or clash is possible.
Read the generated `content.md` and check local/remote status before editing;
another contributor may already have changed the new page. Apply the authoring
and synchronization guides to any intervening changes.

## Author or import the body

For creation from scratch, write the requested body below the generated title
using the authoring guide. For a standalone Markdown draft:

1. Read the source draft and its referenced local assets. Preserve the source
   files; import content into the CLI-created page, rather than move the draft
   into a fabricated managed directory. Arbitrary Markdown files cannot be
   pushed directly as managed pages, even if named `content.md`.
2. Keep the generated first and only level-one heading in the destination.
   Omit a draft heading that only repeats its document title; retain meaningful
   body headings at levels two through six, adjusting their hierarchy as
   needed. Do not overwrite the generated title with the draft's title. Use
   `page rename` on a synchronized page if a title change is requested.
3. Transfer the requested body into the current managed `content.md`, adapting
   unsupported Markdown using the authoring guide. Preserve unrelated edits
   and opaque ADF already present in the destination. Relative links from the
   draft must be reviewed because their base directory has changed.
4. Copy the requested local images and downloadable files into this page's
   `_attachments/` directory and rewrite their references, for example
   `images/diagram.png` to `_attachments/diagram.png`. Resolve source assets
   relative to the draft, choose non-conflicting attachment names, and update
   image/link references, including reference-style definitions. Do not
   overwrite existing managed or unmanaged files blindly. External URLs may
   remain external when intended; report unresolved assets rather than leave
   broken local references or introduce traversal paths. A copied file is
   managed only when referenced under `_attachments/` in `content.md`.
5. Re-read affected destination files before applying changes and review the
   resulting content, attachment set, references, and any deletions. Do not
   copy the draft's directories, cache, or unrelated files into the page tree.

If only creation and local authoring/import were requested, leave these changes
local and report that the empty page already exists remotely. When publication
of the body and attachments is authorized, use only the new page's ID:

```console
cflsync page push NEW_ID
cflsync page status NEW_ID
```

Use normal concurrency checks and resolve conflicts with the synchronization
guide; do not force a stale draft over newer remote content. A whole-workarea
push could publish unrelated edits and is outside a single-page request.
Report remote creation, local body/attachment changes, publication, and final
status separately. Synchronization does not verify browser rendering.

## Recover after partial creation

If creation succeeded remotely but the follow-up pull failed, the error reports
the created ID. Retain it, preserve any local work and installed ancestors, and
resolve the reported clash, access failure, or other cause. Complete local
installation of that existing page with:

```console
cflsync page pull NEW_ID
```

Do not create again or automatically delete the remote page as rollback. If a
recovery pull reports existing local changes, preserve and reconcile them using
the synchronization guide rather than apply force automatically.

If the response was lost or the created ID is unknown, inspect Confluence for
the attempted creation under the intended parent before retrying. A non-zero
exit alone does not establish that nothing was created; parent installation may
also have succeeded. Stop further creation while the remote outcome or new
identity remains uncertain, to avoid duplicate pages.

Creation behavior source:
[cflsync specification](https://github.com/sverologos/cflsync/blob/69ebb39239c449ab9b5921e847df09cf7764a5c0/doc/SPEC.md).
