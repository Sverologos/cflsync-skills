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
`cflsync page pull PAGE_REF` before editing. For cached pages missing a directory
or `content.md`, use the recovery procedure below; normal targeted pull fails
and page status may be unavailable. Managed paths and exact titles may
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
| Uncached page | Normal targeted pull, then attachment validation and status |
| Cached directory missing | Remote content is authoritative; targeted force pull under the recovery procedure |
| Cached `content.md` missing | Remote content is authoritative; inspect attachment effects before a guarded targeted force pull |
| Referenced attachment missing | Error: consult the user about recovery; do not push |
| Local attachment unreferenced | Warning: consult the user; retain the file without explicit deletion consent |
| Remote not found, inaccessible, or outside root | Diagnose access, identity, or movement; preserve local work and do not implicitly recreate or bypass scope |

## Pull and push

Apply [attachment validation](authoring.md#attachment-validation) before push
and after pull. Resolve missing-reference errors and unreferenced-file warnings
with the user before proceeding with affected attachment work or publication.

Before a pull that can replace existing managed files, inspect its attachment
effects. A normal or force pull can delete local managed files absent from the
new remote manifest. Check the current cached manifest, existing local files,
and the incoming managed manifest: existing cached files omitted from the
incoming manifest would be deleted. Obtain explicit consent for those local
deletions before running the command; a backup or force flag is not consent.
For whole-tree operations, apply this check to every affected page.

Use targeted commands for a single page:

```console
cflsync page pull PAGE_ID
cflsync page push PAGE_ID
cflsync page status PAGE_ID
```

Push preflights conversion and managed page links before uploading attachments.
If it reports broken page links, inspect the target IDs, access, and root-tree
membership using the [authoring guide](authoring.md#links-between-managed-pages);
repair the links within the requested scope. Force does not bypass this check.

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
5. Apply attachment validation to the candidate and check the refresh's local
   attachment deletions. Resolve errors/warnings with the user and obtain any
   missing deletion consent before proceeding. For the authorized
   reconciliation, refresh the shared workarea's remote baseline. With
   preserved local state and a reviewed merge candidate, a
   targeted `cflsync page pull --force PAGE_ID` may be necessary. This is a
   deliberate replacement of preserved managed files, not an automatic
   conflict remedy. Compare the refreshed content and attachments with the
   remote version used to construct the candidate; if they differ, reassess
   before applying it. Locate the page at its current path after the pull.
6. Apply the merged body and selected attachment set at the current managed
   path, retaining the current generated title heading and preserving
   unrelated files. Validate new attachment references and ensure any local
   attachment deletion has explicit user consent. An editing-only request
   leaves this merged candidate local.
7. If publication is authorized, use a normal
   `cflsync page push PAGE_ID`, then `cflsync page status PAGE_ID`. The normal
   push retains concurrency checks. If another conflict occurs, preserve the
   candidate and reassess from current state; do not escalate to force push.
   Stop further mutation when a semantic choice remains unresolved or the
   remote creation/update outcome is uncertain.

An explicit request to retain only the remote side can use a targeted force
pull after the attachment checks and required local-deletion consent;
retaining only the local side can use a targeted force push. State the
side selected and keep the operation inside the requested scope. Force push
still uses Confluence's current version and can conflict with concurrent edits.
Whole-tree force can affect unchanged pages as well; do not substitute it for
a targeted resolution.

## Recover missing local state

Use a page ID from the existing cache or established remote identity, rather
than a missing local path. Run commands from the workarea root. Missing page
content or a missing directory makes the remote body authoritative; no body
merge or reconstruction is required. This does not authorize deletion of local
attachments or discarding surviving attachment edits. Keep all `.cflsync/`
changes under CLI control and diagnose inaccessible/out-of-tree pages or invalid
caches instead of recreating pages or editing cache state.

### Uncached page

```console
cflsync page pull PAGE_ID
cflsync page status PAGE_ID
```

Use normal targeted pull, preserve any unmanaged path clash, and validate the
pulled attachments using the authoring guide before further attachment work or
publication.

### Cached page directory missing

```console
cflsync page pull --force PAGE_ID
cflsync page status PAGE_ID
```

The targeted force pull restores the remote page and required missing ancestors.
Check that the directory is actually absent; an existing directory with a
missing body uses the next procedure. Do not fabricate directories or clear
cache entries. Cached descendants are not restored by this targeted operation;
recover only requested descendants separately, parent before child. Locate
the installed path and validate its attachments after recovery.

### Cached directory present, `content.md` missing

Inspect the remote version separately in a temporary workarea outside the
shared workarea, using the same profile and root. Run there:

```console
cflsync init -p PROFILE ROOT_PAGE_ID
cflsync page pull PAGE_ID
```

Use that remote body as the authoritative candidate. Inventory surviving local
attachments, preserve their edits, and apply attachment validation to the
candidate and the files the refresh would leave. Referenced files unavailable
from either the surviving files or the incoming remote copy are errors;
unreferenced local files are warnings. Consult the user about either condition.
Check for local deletions using the manifests as described above. Do not proceed
until advice is resolved and any local deletion has explicit consent.

Once these checks permit the refresh, run from the original workarea root:

```console
cflsync page pull --force PAGE_ID
cflsync page status PAGE_ID
```

This restores the remote body and refreshes the baseline through the CLI. A
force pull also replaces managed attachment bytes; preserve and reapply any
surviving local attachment edits covered by the user's advice. Locate the
current path, revalidate attachments, and inspect status after reapplication.
If the remote version or local files changed since inspection, reassess the
attachment effects before refreshing. Do not use a whole-tree force pull for
this single-page recovery.

### Referenced attachments missing or local attachments unreferenced

Keep the existing body authoritative for its attachment requirements. A missing
referenced attachment is an error: report it and consult the user about recovery,
even if a remote copy or backup appears available. An absent `_attachments/`
directory is an error only when the body references attachments. Do not
force-pull the whole page to repair missing attachments.
Unreferenced local attachments are warnings: consult the user and retain them
until deletion is explicitly authorized. Apply the agreed action to the
affected files and references, validate again, and inspect page status.

Recovery ends with status inspection and reporting unresolved errors/warnings.
It does not include an automatic push, and retained local edits can legitimately
leave a local-changed status. Synchronization does not establish that attachment
validation passed or that Confluence rendering was verified.

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
page. For creation recovery, use the [page creation guide](creation.md); for
copy recovery, use the [template-page guide](template-pages.md).

Synchronization source:
[cflsync specification](https://github.com/sverologos/cflsync/blob/4ec3f3ecb3968017fc9fdde9e7140af4a228eafc/doc/SPEC.md).
