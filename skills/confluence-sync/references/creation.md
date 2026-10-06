<!--
Copyright (c) 2026 Sverologos BV.
This Source Code Form is subject to the terms of the Mozilla Public License,
v. 2.0. If a copy of the MPL was not distributed with this file, You can obtain
one at https://mozilla.org/MPL/2.0/.
-->

# Create a page from scratch

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
create a child. Every page directory ends in its page ID, including the root
page's. The new ID is known only after remote creation, so an unmanaged entry
occupying the new page's actual path is refused by the follow-up pull, after
the remote page exists. Preserve the entry and use creation recovery rather
than create again. Do not construct managed directories or change `.cflsync/`
manually.

The CLI creates an empty remote child, then pulls it into the workarea with a
generated title heading and attachment directory. Check the exit status before
editing. At the documented revision, successful creation does not print the
new page's ID or path. Inspect the newly installed directory under the parent,
then use its managed path to obtain the ID and actual title:

```console
cflsync page status "path/to/new/page/content.md"
```

Retain that ID as `NEW_ID` and the installed path. Do not select an arbitrary
title match or assume that the title alone identifies the installed path.
Read the generated `content.md` and check local/remote status before editing;
another contributor may already have changed the new page. Apply the authoring
and synchronization guides to any intervening changes.

## Author the body

Write the requested body below the generated title using the
[authoring guide](authoring.md). For a supplied draft with an explicitly
requested title or parent, reuse only the
[body and media transfer steps](import.md#copy-the-body-and-local-media)
from the import guide; retain this creation request's title, parent, and
publication scope. For the dedicated workflow that derives titles from Markdown
files and selects parents for individually selected files or directory imports,
use the [import guide](import.md).

If only creation and local authoring were requested, leave these changes
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
[cflsync specification](https://github.com/sverologos/cflsync/blob/4ec3f3ecb3968017fc9fdde9e7140af4a228eafc/doc/SPEC.md).
