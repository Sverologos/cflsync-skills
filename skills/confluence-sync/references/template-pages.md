<!--
Copyright (c) 2026 Sverologos BV.
This Source Code Form is subject to the terms of the Mozilla Public License,
v. 2.0. If a copy of the MPL was not distributed with this file, You can obtain
one at https://mozilla.org/MPL/2.0/.
-->

# Create and customize from a template page

A template here is an ordinary Confluence source page, not a native Confluence
template. Use cflsync's native single-page copy rather than constructing a
new managed directory or implementing a separate Confluence client.

## Resolve the source and destination

Establish the source title or ID, destination parent, new title, and requested
customization. Check `cflsync page copy --help` if the installed capability has
not been established. A request to copy and customize includes remote creation;
copy has no mode that defers creation until publishing. If only a local draft
is requested, do not use page copy without establishing the intended outcome.

Source references use existing managed paths, numeric IDs, or exact titles.
For title lookup:

1. Cached exact-title matches take precedence; ambiguity is an error.
2. Without a cached match, current in-tree exact-title matches take precedence
   over external candidates; ambiguity at this stage is also an error.
3. Only without an in-tree match are external exact-title candidates eligible.
   External sources may be in another space but must be on the configured site.

Use reported IDs to clarify an ambiguous selected stage. Do not fall back to
external candidates when a cached or in-tree title is ambiguous. A cached
source moved outside the tree is refused, not reclassified as an external
template. Paths select page identity, not local content to upload.

An in-workarea source must have unchanged local and remote body, title, parent,
and managed attachments. For an unpulled in-tree source, pull it first, then
check status. For existing source changes, preserve them and use the
synchronization guide: do not silently discard or publish changes merely to
make copy possible. If publishing source edits is needed but outside the
request, obtain that decision or leave the copy pending. Uncached external
sources are copied directly from remote state without a local baseline.

The destination parent must currently be a page inside the managed tree.
External sources and the root page require an explicit `--parent`; otherwise
omitting it chooses the source's current parent. Missing destination ancestors
are installed automatically. Existing local ancestor edits are retained.
Folder, cross-site, recursive, and force-copy operations are unsupported.

## Copy once, then customize

```console
cflsync page copy --parent "Destination page" "Template title" "New page title"
```

When IDs are established, use them for source and parent to retain identity.
Copy creates the remote page immediately and pulls it into the workarea.
Retain the reported new ID, actual title, and installed path; Confluence may
disambiguate the requested title. Do not assume that the requested title is
the new title heading or local directory name.

Read the installed page, then customize the requested fields or sections using
the authoring guide. Preserve unrelated source content, opaque ADF, and
attachment references. Confluence template variables are not automatically
substituted; replace only placeholders covered by the requested customization.

For authorized publication of customization:

```console
cflsync page push NEW_ID
cflsync page status NEW_ID
```

If only copying and local customization were requested, leave customization
local and report that the copied source content already exists remotely. Keep
remote creation and publication of later edits distinct.

## What copy retains

Copy includes one page's body, current attachments, and labels. It excludes
descendants, comments, history, source restrictions, properties, app-owned
custom content, and unmanaged local files. Confluence enforces permissions;
do not invent a manual fallback to circumvent a rejected copy. Labels remain
remote metadata without local synchronization or label-only change detection.

Structured source-owned media resolves to copied attachment IDs. Relative
`_attachments/` references use the new page's attachment manifest. Ordinary
download URLs, Smart Links, version parameters, and foreign-page media remain
unchanged and may depend on source access. Qualifying ordinary page links use
the normal local/remote page-link conversion described in the
[authoring guide](authoring.md#links-between-managed-pages); copy does not
retarget them to a different page. Remote attachments sharing a filename are
handled by the normal duplicate-attachment rules: one is managed and the
others remain untouched, with their media retained as opaque ADF. App macros
can rely on excluded metadata. Verify rendering and source independence when
the request requires them; a successful copy or no-op push alone does not
establish either.

## Recover without another creation

If remote creation succeeded but local installation failed, retain the new
ID and resolve the reported local clash, access failure, or other cause. Use
`cflsync page pull NEW_ID` to install that existing page. Preserve installed
ancestors and unmanaged entries; do not delete them merely to retry.

If the creation response was lost or unusable, the remote outcome is uncertain.
Inspect Confluence for the attempted creation, including its parent and actual
title, before any further copy. Do not retry automatically or assume that a
failed command created nothing. Do not automatically delete a created page as
rollback; unresolved identity or outcome is a stopping condition.

Copy behavior source:
[cflsync page copy specification](https://github.com/sverologos/cflsync/blob/4ec3f3ecb3968017fc9fdde9e7140af4a228eafc/doc/SPEC.md#page-copy).
