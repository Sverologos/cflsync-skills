<!--
Copyright (c) 2026 Sverologos BV.
This Source Code Form is subject to the terms of the Mozilla Public License,
v. 2.0. If a copy of the MPL was not distributed with this file, You can obtain
one at https://mozilla.org/MPL/2.0/.
-->

# Synchronization, conflicts, and recovery

## Inspect the requested scope

1. Resolve the requested page or workarea scope:
   - For a cached page:

     ```console
     cflsync page status PAGE_ID
     ```

     Managed paths or exact titles can resolve the first command; retain the
     reported ID for later commands. `page status` resolves cached state only.
   - For an unpulled in-tree page, establish local state with
     `cflsync page pull PAGE_REF` before editing.
   - For cached pages missing a directory or `content.md`, use recovery below.
     Normal targeted pull fails and page status may be unavailable.
   - For whole-workarea scope, use `cflsync status`.
2. Inspect both local and remote state, including content/attachment changes and
   possible directory relocation. Zero exit status does not mean unchanged.
   Whole-workarea labels include `not in local`, `remote removed`, `remote changed`,
   `local changed`, `conflict`, and `unchanged`. Do not infer remote deletion from
   a missing-page label alone; the page may be inaccessible or outside the root.
3. Select the action from the state table:

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

1. Select the operation from the state table and requested scope. Publish existing
   local changes only when authorized; do not run an unconditional pull-then-push
   sequence.
2. Before push, apply [attachment validation](authoring.md#attachment-validation).
   Resolve missing-reference errors and unreferenced-file warnings with the user
   before affected attachment work or publication.
3. Before a pull that can replace managed files, inspect attachment effects:
   - Compare the cached manifest, existing local files, and incoming managed
     manifest. Normal and force pull delete existing cached files omitted from
     the incoming manifest.
   - Obtain explicit consent for those local deletions before running the command;
     a backup or force flag is not consent.
   - For whole-tree operations, check every affected page.
4. Run the selected operation within scope:
   - Single-page pull: `cflsync page pull PAGE_ID`. Retain the ID and inspect the
     resulting path; pull installs missing ancestors and can relocate directories
     after remote renames/moves. `page pull --force` affects only the target,
     not its ancestors.
   - Single-page push: `cflsync page push PAGE_ID`. Push preflights conversion and
     managed page links before uploading attachments. For broken page links,
     inspect target IDs, access, and tree membership using the
     [authoring guide](authoring.md#links-between-managed-pages), then repair
     within scope. Force does not bypass this check.
   - Whole-tree pull: `cflsync pull`. It processes parents first, fetches
     remote-only and missing pages, skips local-only changes, and keeps pages
     no longer in the tree. Without force, conflict aborts before any page pull.
   - Whole-tree push: `cflsync push`. It uploads local-only changes, skips
     remote-only and missing pages, and aborts on conflict before mutation.
5. After pull, apply attachment validation and resolve errors/warnings with the
   user before further affected attachment work or publication.
6. Check exit codes, per-page summaries, and resulting status:
   - Use `cflsync page status PAGE_ID` for a single page or `cflsync status` for
     whole-workarea scope.
   - Whole-tree commands can continue after individual failures and return
     non-zero; successful commands can still skip pages. Report what remains.
   - Neither direction alone necessarily completes generic synchronization;
     handle remaining changes within scope. Read-only status does not repair
     invalid caches or inaccessible roots.

- Do not introduce `pull --delete` for ordinary synchronization. It deletes local
  copies outside the tree, including unmanaged files, and nothing remotely.
  `page remove` deletes a subtree remotely and locally; both require the
  corresponding deletion scope.
- Use a suitable terminal when confirmation requires one. A suggested `--force`
  flag does not authorize bypassing the prompt or discarding content.

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

- For an explicit request to retain only remote state, use targeted force pull
  after attachment checks and required local-deletion consent.
- For an explicit request to retain only local state, use targeted force push.
  It still uses Confluence's current version and can conflict with concurrent edits.
- State the selected side and keep the operation within scope. Do not substitute
  whole-tree force for targeted resolution; it can also affect unchanged pages.

## Recover missing local state

- Use a page ID from the cache or established remote identity, not a missing
  local path. Run commands from the workarea root.
- Treat remote body content as authoritative when `content.md` or its directory
  is missing; do not merge or reconstruct the missing body. Preserve surviving
  attachment edits and require consent for local attachment deletion.
- Keep `.cflsync/` changes under CLI control. Diagnose inaccessible/out-of-tree
  pages or invalid caches instead of recreating pages or editing cache state.

### Uncached page

1. Use normal targeted pull, preserving any unmanaged path clash:

   ```console
   cflsync page pull PAGE_ID
   ```

2. Validate pulled attachments using the authoring guide before further
   attachment work or publication.
3. Inspect `cflsync page status PAGE_ID`.

### Cached page directory missing

1. Check that the directory is absent. If it exists but lacks the body, use the
   next procedure. Do not fabricate directories or clear cache entries.
2. Restore the remote page and required missing ancestors:

   ```console
   cflsync page pull --force PAGE_ID
   ```

3. Locate the installed path and validate its attachments.
4. Recover only requested cached descendants separately, parent before child;
   targeted pull does not restore them.
5. Inspect `cflsync page status PAGE_ID`.

### Cached directory present, `content.md` missing

1. Inspect remote state in a temporary workarea outside the shared workarea,
   using the same profile and root. Run there:

   ```console
   cflsync init -p PROFILE ROOT_PAGE_ID
   cflsync page pull PAGE_ID
   ```

2. Use the remote body as the authoritative candidate. Inventory surviving local
   attachments, preserve their edits, and validate the candidate against files
   the refresh would leave:
   - Referenced files unavailable from surviving files or the incoming remote
     copy are errors.
   - Unreferenced local files are warnings.
   - Consult the user about either condition.
3. Check local deletions using the manifests as described above. Resolve advice
   and obtain explicit consent for any local deletion before proceeding.
4. Recheck remote state and local files against the inspection. If either changed,
   reassess attachment effects before refreshing.
5. Once checks permit the refresh, run from the original workarea root:

   ```console
   cflsync page pull --force PAGE_ID
   ```

   This restores the remote body and refreshes the baseline through the CLI;
   force pull also replaces managed attachment bytes. Do not use whole-tree
   force pull for single-page recovery.
6. Locate the current path and reapply preserved local attachment edits covered
   by the user's advice. Revalidate attachments after reapplication.
7. Inspect `cflsync page status PAGE_ID`.

### Referenced attachments missing or local attachments unreferenced

1. Derive attachment requirements from the existing body:
   - Missing referenced attachment: report an error and consult the user about
     recovery, even if a remote copy or backup is available. An absent
     `_attachments/` directory is an error only when the body references files.
   - Unreferenced local attachment: report a warning, consult the user, and
     retain the file until deletion is explicitly authorized.
2. Apply the agreed action to affected files and references; do not force-pull
   the whole page to repair missing attachments.
3. Validate again and inspect page status.

- End recovery with status inspection and report unresolved errors/warnings.
  Do not push automatically; retained edits may leave a local-changed status.
- Report attachment validation and rendering verification separately from
  synchronization; synchronization does not establish either.

## Partial failures

Local installation and remote operations cannot form one transaction. A push
uploads attachment changes, updates content, then performs managed attachment
deletions. A failure can leave a partly updated remote page while the old cache
remains. A failed push is not proof that nothing was published.

- Failed push: preserve local work, inspect status, and compare current remote
  state before reconciliation.
- Interrupted pull: diagnose the reported condition before resuming; installed
  ancestors and successful pages remain.
- Rename/move with remote success but local failure: use the reported targeted
  pull to complete local synchronization; do not repeat the structural mutation.
- Uncertain create/copy outcome: do not retry or automatically delete a partly
  created page. Use the [page creation guide](creation.md) or
  [template-page guide](template-pages.md) for recovery.

Synchronization source:
[cflsync specification](https://github.com/sverologos/cflsync/blob/4ec3f3ecb3968017fc9fdde9e7140af4a228eafc/doc/SPEC.md).
