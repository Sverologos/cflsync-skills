<!--
Copyright (c) 2026 Sverologos BV.
This Source Code Form is subject to the terms of the Mozilla Public License,
v. 2.0. If a copy of the MPL was not distributed with this file, You can obtain
one at https://mozilla.org/MPL/2.0/.
-->

# Import Markdown files into an existing workarea

Import one or more Markdown files as new pages directly under the configured
workarea root. Creation makes an empty page visible remotely; the imported body
and attachments remain local. Do not push automatically at the end of import.
Publication is a separate requested operation using the
[synchronization guide](synchronization.md).

## Establish sources and destination

Locate the existing workarea and read `.cflsync/root` to retain its root page
ID as `ROOT_ID`, following the skill's identity and scope rules. Do not
initialize or re-anchor a workarea as part of import. Check
`cflsync page create --help` if the installed capability is not established.

Read the selected source files and identify their referenced local media before
creation. Preserve the source files and assets. Resolve relative media paths
against each source file's directory, not the workarea or current command
directory. Import only the selected Markdown files, with one new page per
file; do not recreate their source directory hierarchy as pages.

## Select a title and create each page

Process files individually, retaining the source path, selected title, new page
ID, installed path, and outcome for each file.

1. If the file contains a level-one Markdown heading, use the first such
   heading's plain text as the page title. Recognize both ATX (`# Title`) and
   Setext (`Title` followed by `=====`) headings; heading-like text in code
   blocks is not a title. Additional level-one headings remain body sections
   and are demoted during transfer below.
2. Otherwise, use the source filename without its `.md` extension. For example,
   `release.notes.md` gives `release.notes`.
3. Ensure the selected title is non-empty, single-line text without surrounding
   whitespace. Trim heading whitespace; resolve an unusable title before
   creating the page rather than inventing one.
4. From the workarea root, create the page with the root page as its parent:

   ```console
   cflsync page create ROOT_ID "Selected title"
   ```

   Follow [page creation](creation.md#create-once-and-retain-identity) for
   identity discovery, path clashes, and concurrency checks. Successful
   `page create` already pulls the new page, including its generated title
   heading and `_attachments/` directory; this satisfies the pull step.
   Retain the new ID as `NEW_ID` and its actual installed path before editing.
   If creation succeeds remotely but pull fails, use
   [creation recovery](creation.md#recover-after-partial-creation) to pull the
   existing page by ID, without creating it again.

## Copy the body and local media

1. Read the pulled `content.md` and check `cflsync page status NEW_ID` before
   editing. Preserve intervening edits using the authoring and synchronization
   guides. Transfer the source content into this managed file, not into a
   fabricated page directory. Arbitrary Markdown files cannot be pushed as
   managed pages directly, even if named `content.md`.
2. Keep the generated title heading as the first and only level-one heading.
   Remove the source heading selected as its title, wherever that heading
   occurred, to avoid duplicating the title in the body. If the source had no
   title heading, retain its entire body below the generated filename-based
   title. Demote additional level-one headings to body headings, adjusting
   their subordinate hierarchy as needed within levels two through six.
   Preserve other content, including code blocks; use the
   [authoring guide](authoring.md) for unsupported Markdown. Do not rename the
   page by replacing its generated heading.
3. Copy referenced local images and downloadable files into this page's
   `_attachments/` directory. Copy only referenced assets, preserving their
   sources; do not copy source directories, caches, or unrelated files. Choose
   non-conflicting attachment filenames when distinct source paths have the
   same basename or a destination file already exists. Repeated references to
   the same source asset can share one destination file. Each imported page
   needs its own copies of the media it references.
4. Rewrite the corresponding image and file-link targets in `content.md`,
   including reference-style definitions and supported HTML `<img src="…">`
   attributes, using the chosen attachment names. Preserve figure attributes
   and captions when present, following the authoring guide.
   For example, `![Diagram](images/diagram.png)` becomes
   `![Diagram](_attachments/diagram.png)`. Preserve labels, alt text, and link
   titles, and percent-encode attachment filenames containing spaces or
   punctuation. Leave external URLs and fragment-only links intact.
   Review other relative links because their base directory has changed;
   links to other Markdown documents are not media and do not automatically
   become attachments or links to imported pages.
5. Report missing or unresolved local media and leave that file's import
   incomplete rather than claim success with broken links. Do not overwrite
   managed or unmanaged files blindly or introduce traversal paths in
   `_attachments/` references. A copied file becomes managed only when
   `content.md` references it under `_attachments/`.
6. Re-read affected destination files before applying changes. Review the
   resulting content, attachment set, rewritten links, and any deletions;
   preserve unrelated edits, opaque ADF, and unmanaged files.

## Verify and report without publishing

Check that the generated title is retained, each rewritten local-media target
exists, and distinct source assets have not overwritten one another. Inspect
the resulting local changes and run:

```console
cflsync page status NEW_ID
```

Local content or attachment changes are the expected result; do not push to
obtain an unchanged status. Report each source file's new page ID and managed
path, local import outcome, unresolved assets, and failed or skipped steps.
Distinguish remote creation of the empty page from local preparation of its
body and attachments. Status does not verify Confluence browser rendering.

For partial batches, retain successful pages and resume failed files using
their known page IDs. Never repeat creation for a file whose page already
exists, automatically delete created pages as rollback, or retry creation
while its remote outcome or identity is uncertain. Stop further creation on
such uncertainty and use the creation recovery guide.
