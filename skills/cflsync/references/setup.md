<!--
Copyright (c) 2026 Sverologos BV.
This Source Code Form is subject to the terms of the Mozilla Public License,
v. 2.0. If a copy of the MPL was not distributed with this file, You can obtain
one at https://mozilla.org/MPL/2.0/.
-->

# Setup and initial synchronization

## Dependencies and compatibility

These instructions follow cflsync source revision
`69ebb39239c449ab9b5921e847df09cf7764a5c0` (project version `0.4.5`). Required
capabilities include rooted workareas, page status/pull/push, and native
`page copy --parent` for template-page tasks. Do not infer command availability
from the version number alone. The documented revision can be selected for a
reproducible installation.

Offline CLI workflows were validated against this revision on Linux with
Python 3.13.15 and Pandoc 3.10 using isolated Confluence fixtures.

Check the existing commands without contacting Confluence:

```console
cflsync --help
cflsync page --help
cflsync page pull --help
cflsync page push --help
cflsync page status --help
pandoc --version
```

For copying, also check `cflsync page copy --help` and its `--parent` option.
Use command help rather than assuming a `cflsync --version` option exists.
Only check structural commands such as rename or move when required.

The converter requires Pandoc's native JSON API version **1.23.1.2**, not just
a particular Pandoc executable version. Pandoc **3.10** is the validation
baseline. Feed empty standard input to `pandoc --from=gfm --to=json` and inspect
its `pandoc-api-version` array: it must be `[1, 23, 1, 2]` for this cflsync
revision. Reuse an existing installation that satisfies this check. A different
cflsync revision may change the API requirement; use its reported requirement
rather than upgrade either tool speculatively.

If installation is necessary and covered by the requested setup, use the
application's supported distribution. Existing task authorization carries
through; environment execution permissions still apply.

On Windows, Scoop supplies cflsync, bundled CPython, and Pandoc as a dependency:

```console
scoop bucket add sverologos https://github.com/sverologos/scoop
scoop install cflsync
```

Reuse an existing bucket rather than add it again. On Linux/macOS or WSL, use
uv with Python 3.11 or later and install compatible Pandoc separately:

```console
uv tool install git+https://github.com/sverologos/cflsync@69ebb39239c449ab9b5921e847df09cf7764a5c0
```

From a suitable application checkout, `uv tool install .` is also supported.
Do not replace a compatible installed tool or add upgrade flags during ordinary
authoring. If an incompatible tool is already installed, establish the needed
replacement within the setup scope and follow uv/Scoop's supported procedure.
Verify command capabilities and Pandoc after installation, before workarea
operations. If execution or installation is unavailable or outside scope,
report the unmet dependency and applicable commands rather than claim setup
success.

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
   Read `.cflsync/profile` and `.cflsync/root`; preserve its cache and content.
   Reuse a matching workarea. Do not initialize inside another workarea or
   silently switch an existing root/profile.
4. Resolve the root as a numeric ID or exact title. `init` checks it remotely
   using the profile. If a title is ambiguous, use the reported candidates to
   obtain the intended ID; do not select an arbitrary candidate.
5. Run from the intended workarea root, substituting the established values:

   ```console
   cflsync init -p PROFILE ROOT_PAGE_REF
   ```

Initialization anchors the workarea; it downloads no pages. Report the selected
root ID/profile and whether synchronization was requested and completed.
Omitting `-p` selects `default`, including during re-anchoring.

Re-anchoring is a distinct requested operation: `init` permits it only from the
workarea root when the cache contains no page state. Do not clear a populated
cache to satisfy this restriction. A legacy unanchored workarea needs the
application's documented migration, preserving existing content first.

## Initial pull

For requested full-tree synchronization after initialization:

```console
cflsync pull
cflsync status
```

For selected pages, use `cflsync page pull PAGE_REF`; missing ancestors are
installed automatically, parents first. Follow with `page status PAGE_ID`.
Do not expand a selected-page request into a full-tree pull. Setup alone does
not imply a pull, and initial synchronization needs neither force nor deletion
flags. In an already populated workarea, preserve local edits and follow the
synchronization guide's state-dependent behavior. Check both exit statuses and
reported states; skipped or failed pages are not synchronized.

Application installation and migration details:
[cflsync README](https://github.com/sverologos/cflsync/blob/69ebb39239c449ab9b5921e847df09cf7764a5c0/README.md).
