<!--
Copyright (c) 2026 Sverologos BV.
This Source Code Form is subject to the terms of the Mozilla Public License,
v. 2.0. If a copy of the MPL was not distributed with this file, You can obtain
one at https://mozilla.org/MPL/2.0/.
-->

# Create and customize from a template page

A template here is an ordinary Confluence source page, not a native Confluence
template.

- Use cflsync's native single-page copy; do not construct managed directories
  or implement a separate Confluence client.

## Resolve the source and destination

1. Establish the source title or ID, destination parent, new title, and requested
   customization. Copy and customization authorize remote creation; copy cannot
   defer creation until publishing. For a local-only draft, establish the
   intended outcome before using copy. Use setup guidance only when a copy
   attempt fails for a missing or incompatible dependency.
2. Resolve the source using a managed path, numeric ID, or exact title. Paths
   select page identity, not local content to upload. For title lookup:
   1. Prefer cached exact-title matches; ambiguity is an error.
   2. Without a cached match, prefer current in-tree exact-title matches over
      external candidates; ambiguity at this stage is also an error.
   3. Only without an in-tree match, consider external exact-title candidates.
      They may be in another space but must be on the configured site.
3. Clarify ambiguity using reported IDs; do not fall back to external candidates
   from an ambiguous cached or in-tree stage. A cached source moved outside
   the tree is refused, not reclassified as an external template.
4. Copy reflects the source's remote state; local source edits are not
   included. Do not discard or publish source edits merely to enable copy — if
   the copy must include them, publish or reconcile them as a separate
   requested operation. A refused copy is a failure to investigate with the
   synchronization guide. Uncached external sources copy directly from remote
   state without a local baseline.
5. Resolve a destination parent currently inside the managed tree. External
   sources and the root page require explicit `--parent`; otherwise omission
   chooses the source's current parent. Missing ancestors install automatically,
   retaining local ancestor edits. Do not request unsupported folder, cross-site,
   recursive, or force-copy operations.

## Copy once, then customize

1. Run the copy command, substituting established source and parent IDs when
   available to retain identity:

   ```console
   cflsync page copy --parent "Destination page" "Template title" "New page title"
   ```

2. Retain the reported new ID, actual title, and installed path. Copy creates
   the page remotely and pulls it immediately; Confluence may disambiguate the
   title. Do not assume the requested title is the heading or directory name.
3. Read the installed page and customize the requested fields or sections using
   the authoring guide. Preserve unrelated source content, opaque ADF, and
   attachment references. Template variables are not automatically substituted;
   replace only placeholders within the requested customization.
4. Follow publication scope:
   - For authorized publication of customization:

     ```console
     cflsync page push NEW_ID
     cflsync page status NEW_ID
     ```

   - For copy and local customization only, leave customization local and report
     that copied source content already exists remotely.
5. Report remote creation and publication of later edits as distinct outcomes.

## What copy retains

Copy includes one page's body, current attachments, and labels. It excludes
descendants, comments, history, source restrictions, properties, app-owned
custom content, and unmanaged local files. Confluence enforces permissions.
Labels remain remote metadata without local synchronization or label-only
change detection.

- Do not invent a manual fallback to circumvent a rejected copy.

Structured source-owned media resolves to copied attachment IDs. Relative
`_attachments/` references use the new page's attachment manifest. Ordinary
download URLs, Smart Links, version parameters, and foreign-page media remain
unchanged and may depend on source access. Qualifying ordinary page links use
the normal local/remote page-link conversion described in the
[authoring guide](authoring.md#links-between-managed-pages); copy does not
retarget them to a different page. Remote attachments sharing a filename are
handled by the normal duplicate-attachment rules: one is managed and the
others remain untouched, with their media retained as opaque ADF. App macros
can rely on excluded metadata.

- Verify rendering and source independence when requested; successful copy or
  no-op push alone does not establish either.

## Recover without another creation

1. If remote creation succeeded but local installation failed, retain the new
   ID and resolve the reported clash, access failure, or other cause.
2. Install the existing page with `cflsync page pull NEW_ID`. Preserve installed
   ancestors and unmanaged entries; do not delete them merely to retry.
3. If the creation response was lost or unusable, inspect Confluence for the
   attempted creation, including parent and actual title, before further copy.
   Do not retry automatically or assume that a failed command created nothing.
4. Stop while identity or remote outcome remains uncertain. Do not automatically
   delete a created page as rollback.

Copy behavior source:
[cflsync page copy specification](https://github.com/sverologos/cflsync/blob/4ec3f3ecb3968017fc9fdde9e7140af4a228eafc/doc/SPEC.md#page-copy).
