<!--
Copyright (c) 2026 Sverologos BV.
This Source Code Form is subject to the terms of the Mozilla Public License,
v. 2.0. If a copy of the MPL was not distributed with this file, You can obtain
one at https://mozilla.org/MPL/2.0/.
-->

# Synchronization, conflicts, and recovery

## Inspect the requested scope

For a cached page:

```console
cflsync page status PAGE_ID
```

The output distinguishes local content/attachment changes, remote changes,
and a possible directory relocation. A zero exit status does not mean the page
is unchanged; inspect both sides. `page status` resolves cached local state
only. For an unpulled page in the tree, establish local state with
`cflsync page pull PAGE_REF` before editing. Managed paths and exact titles may
resolve the first command; retain its reported ID for later commands.

For whole-workarea scope, use `cflsync status`. Its labels include `not in
local`, `remote removed`, `remote changed`, `local changed`, `conflict`, and
`unchanged`. A missing remote page can also be inaccessible or outside the
root; do not infer deletion from that label alone.

| Local / remote state | Action |
| --- | --- |
| Both unchanged | Edit if requested; otherwise synchronization is already verified at this check |
| Remote changed only | Pull the page before editing; push would conflict |
| Local changed only | Preserve edits and incorporate the request; push only if authorized; targeted pull would conflict |
| Both changed | Preserve both sides and reconcile before publication |
| Missing local state | Pull the requested in-tree page; preserve existing files if cflsync reports a clash |
| Remote not found, inaccessible, or outside root | Diagnose access, identity, or movement; preserve local work and do not implicitly recreate or bypass scope |

## Pull and push

Use targeted commands for a single page:

```console
cflsync page pull PAGE_ID
cflsync page push PAGE_ID
cflsync page status PAGE_ID
```

These are command examples, not an unconditional pull-then-push sequence.
Select the operation from the state table and requested scope. Pull installs
missing ancestors and can move page directories after remote renames/moves;
retain the ID and inspect the resulting path. `page pull --force` affects the
target only, not its ancestors.

Whole-tree `pull` processes parents first, fetches remote-only and missing
pages, skips local-only changes, and keeps pages no longer in the tree. Without
force, a conflict aborts before any page is pulled. Whole-tree `push` uploads
local-only changes, skips remote-only and missing pages, and also aborts on
conflict before mutation. Neither direction alone necessarily completes a
generic synchronization request; inspect status afterward and handle remaining
changes within scope. Publish existing local changes only when authorized.

Check exit codes and per-page summaries: whole-tree commands can continue
after individual failures and return non-zero. A successful command can still
skip pages. Use the resulting status to report exactly what remains. Read-only
status does not repair invalid caches or inaccessible roots.

Do not introduce `pull --delete` for ordinary synchronization. It deletes local
copies of pages outside the current tree, including unmanaged files; it deletes
nothing remotely. `page remove` deletes a page subtree remotely and locally.
Both require the corresponding deletion scope. If a confirmation requires a
terminal, use a suitable terminal; the suggested `--force` flag is not itself
authorization to bypass the prompt or discard content.

## Reconcile two-sided changes

cflsync detects conflicts but does not merge. The baseline cache contains
hashes and versions, not the original document or attachment bytes.

1. Preserve the page's current `content.md` and complete managed attachment
   set outside its managed directory, with hashes or another way to detect
   intervening local changes. Keep any merge candidate outside the managed
   tree while constructing it. Backing up the whole page directory is possible
   but must not lead to restoring cached child pages over newer work.
2. Obtain remote content separately using a temporary workarea outside the
   shared workarea. Reuse the same authentication profile and root ID:

   ```console
   cflsync init -p PROFILE ROOT_PAGE_ID
   cflsync page pull PAGE_ID
   ```

   Run those commands from the temporary directory. Do not force-pull the
   shared workarea simply to inspect the remote side. If the page is missing
   or outside the root, diagnose that condition before proceeding.
3. Compare content and attachments. Use a real common baseline when one is
   available, such as a Git revision known to represent the last successful
   synchronization. Do not assume an arbitrary commit or the cache is a merge
   base. Without a base, make unresolved differences explicit. Merge disjoint
   changes from the requested intent; obtain clarification where contradictory
   edits or attachment choices cannot be resolved from that intent.
4. Once the merged candidate is established, recheck both the original local
   files against the preserved snapshot and the temporary remote page's
   status. If either changed, preserve the candidate and reassess. Do not
   overwrite intervening human contributions or refresh from remote content
   that has not been considered in the merge.
5. For the authorized reconciliation, refresh the shared workarea's remote
   baseline. With preserved local state and a reviewed merge candidate, a
   targeted `cflsync page pull --force PAGE_ID` may be necessary. This is a
   deliberate replacement of preserved managed files, not an automatic
   conflict remedy. Compare the refreshed content and attachments with the
   remote version used to construct the candidate; if they differ, reassess
   before applying it. Locate the page at its current path after the pull.
6. Apply the merged body and selected attachment set at the current managed
   path, retaining the current generated title heading and preserving
   unrelated files. Review new attachment references and deletions. An
   editing-only request leaves this merged candidate local.
7. If publication is authorized, use a normal
   `cflsync page push PAGE_ID`, then `cflsync page status PAGE_ID`. The normal
   push retains concurrency checks. If another conflict occurs, preserve the
   candidate and reassess from current state; do not escalate to force push.
   Stop further mutation when a semantic choice remains unresolved or the
   remote creation/update outcome is uncertain.

An explicit request to retain only the remote side can use a targeted force
pull; retaining only the local side can use a targeted force push. State the
side selected and keep the operation inside the requested scope. Force push
still uses Confluence's current version and can conflict with concurrent edits.
Whole-tree force can affect unchanged pages as well; do not substitute it for
a targeted resolution.

## Partial failures

Local installation and remote operations cannot form one transaction. A push
uploads attachment changes, updates content, then performs managed attachment
deletions. A failure can leave a partly updated remote page while the old cache
remains. Preserve local work, inspect status, and compare the current remote
state before attempting reconciliation. A failed push is not proof that
nothing was published.

An interrupted pull can be resumed after diagnosing the reported condition.
Installed ancestors and successful pages remain. Rename or move failures after
remote success identify a targeted pull for completing local synchronization;
use that recovery rather than repeating the structural mutation. Do not retry
create/copy on an uncertain outcome, or automatically delete a partly created
page. For copy recovery, use the template-page guide.

Synchronization source:
[cflsync specification](https://github.com/sverologos/cflsync/blob/69ebb39239c449ab9b5921e847df09cf7764a5c0/doc/SPEC.md).
