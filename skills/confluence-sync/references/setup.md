<!--
Copyright (c) 2026 Sverologos BV.
This Source Code Form is subject to the terms of the Mozilla Public License,
v. 2.0. If a copy of the MPL was not distributed with this file, You can obtain
one at https://mozilla.org/MPL/2.0/.
-->

# Setup and initial synchronization

## Dependencies and compatibility

These instructions follow cflsync **0.5.7**, source revision
`4ec3f3ecb3968017fc9fdde9e7140af4a228eafc` (tag `v0.5.7`). Required
capabilities include rooted workareas with workarea/cache format 3 and page
status/pull/push. Creation and import require `page create`; template-page
tasks require native `page copy --parent`. The documented tag supports
reproducible installation.

The installed 0.5.7 package was checked against the tagged source. CLI and
markup behavior were validated on Linux with Python 3.13 and Pandoc 3.10 using
isolated Confluence fixtures; these checks did not contact a live site.

A command failure routes here: an unavailable executable, an unsupported
command or option, or a Pandoc API mismatch is diagnosed with these steps.
Compatible installations are reused without speculative upgrades.

1. Diagnose the failed command without contacting Confluence:

   ```console
   cflsync --help
   cflsync page --help
   cflsync page pull --help
   cflsync page push --help
   cflsync page status --help
   pandoc --version
   ```

   - Use command help; do not infer capabilities from a version number or assume
     that `cflsync --version` exists.
   - For copying, check `cflsync page copy --help` and `--parent`.
   - For creation or import, check `cflsync page create --help` and its
     `PARENT_PAGE_REF TITLE` arguments.
   - Check structural commands such as rename or move only when required.
2. Feed empty standard input to `pandoc --from=gfm --to=json`; inspect
   `pandoc-api-version`. This revision requires **1.23.1.2**, represented as
   `[1, 23, 1, 2]`; the executable version alone is insufficient. Pandoc **3.10**
   is the validation baseline.
3. Reuse compatible installations. For a different cflsync revision, use its
   reported API requirement; do not upgrade either tool speculatively or add
   upgrade flags during ordinary authoring.
4. Install only when necessary and within the requested setup. Existing task
   authorization carries through; environment execution permissions still apply.
   - On Windows, Scoop supplies cflsync, bundled CPython, and Pandoc. Reuse an
     existing bucket; otherwise add it before installing:

     ```console
     scoop bucket add sverologos https://github.com/sverologos/scoop
     scoop install cflsync
     ```

   - On Linux/macOS or WSL, use uv with Python 3.11 or later and install
     compatible Pandoc separately:

     ```console
     uv tool install git+https://github.com/sverologos/cflsync@v0.5.7
     ```

     From a suitable application checkout, `uv tool install .` is also supported.
   - If an incompatible tool is installed, establish the needed replacement
     within setup scope and follow uv/Scoop's supported procedure.
5. Verify command capabilities and Pandoc after installation, before workarea
   operations. If execution or installation is unavailable or outside scope,
   report the unmet dependency and applicable commands; do not claim setup success.

## Authentication and workarea boundary

1. Establish the intended directory, configured site/profile, and root page
   from the request and existing configuration. List profile names with
   `cflsync auth --list`; do not print credential configuration or tokens.
2. Reuse an existing profile. When credentials must be configured, run
   `cflsync auth -p PROFILE` through its interactive prompts in a suitable
   terminal. Credentials belong in cflsync's configuration, never in skill
   files, page content, or command arguments. A noninteractive environment
   without a usable profile is an unmet setup dependency.
3. Inspect the intended directory and its ancestors for an existing workarea.
   Read `.cflsync/profile`, `.cflsync/root`, and `.cflsync/version`; preserve
   its cache and content. Version `3` is required for this cflsync release.
   Reuse a matching workarea. Do not initialize inside another workarea or
   silently switch an existing root/profile.
4. Resolve the root as a numeric ID or exact title. `init` checks it remotely
   using the profile. If a title is ambiguous, use the reported candidates to
   obtain the intended ID; do not select an arbitrary candidate.
5. Run from the intended workarea root, substituting the established values:

   ```console
   cflsync init -p PROFILE ROOT_PAGE_REF
   ```

   Initialization anchors the workarea and records version `3`; it downloads no
   pages. Omitting `-p` selects `default`, including during re-anchoring.
6. Report the selected root ID/profile and whether synchronization was requested
   and completed.

- Re-anchor only as a distinct requested operation. `init` permits it only from
  the workarea root when the cache contains no page state; do not clear a
  populated cache to satisfy this restriction.

## Existing workareas and older markup

cflsync 0.5.7 refuses populated workareas created by 0.4 or earlier, including
format-1 and format-2 cache entries.

1. For a requested migration, preserve local content, attachments, and unmanaged
   files first.
2. Publish old local changes using the version that created the workarea only
   when requested; otherwise retain them for deliberate reapplication.
3. Initialize a fresh workarea in an empty directory with the established
   profile and root; pull the requested pages.
4. Transfer preserved changes and needed unmanaged files into the new managed
   paths. Do not edit `.cflsync/version`, rewrite entries, or clear the old cache
   to bypass compatibility checks.

- Re-anchor an empty older workarea through the CLI only within an explicit
  setup/migration request.
- In compatible workareas, rewrite date spans from 0.5.3 or earlier and status
  spans from 0.5.6 or earlier using the
  [authoring guide](authoring.md#special-spans-and-opaque-content). Preserve
  intended values; 0.5.7 rejects the old spans on push.
- Review older panel alerts against the guide's current one-to-one mapping;
  they still push but may represent a different panel type.
- For a requested migration needing fresh remote markup, use targeted
  `page pull --force` only after preserving and reconciling changes under the
  synchronization guide. Normal pull does not regenerate unchanged files; do
  not force-pull the whole tree merely because the tool was upgraded.

## Initial pull

1. Establish requested synchronization scope. Setup alone does not imply a pull;
   initial synchronization needs neither force nor deletion flags. In a populated
   workarea, preserve local edits and follow the synchronization guide's
   state-dependent behavior.
2. Pull only the requested scope:
   - For full-tree synchronization after initialization:

     ```console
     cflsync pull
     cflsync status
     ```

   - For selected pages, use `cflsync page pull PAGE_REF`, then
     `cflsync page status PAGE_ID`. Missing ancestors install automatically,
     parents first; do not expand scope to a full-tree pull.
3. Check exit statuses and reported states; skipped or failed pages are not
   synchronized.

Application installation and migration details:
[cflsync README](https://github.com/sverologos/cflsync/blob/4ec3f3ecb3968017fc9fdde9e7140af4a228eafc/README.md).
