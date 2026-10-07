<!--
Copyright (c) 2026 Sverologos BV.
This Source Code Form is subject to the terms of the Mozilla Public License,
v. 2.0. If a copy of the MPL was not distributed with this file, You can obtain
one at https://mozilla.org/MPL/2.0/.
-->

# Import Markdown files into an existing workarea

Import individually selected Markdown files under the configured workarea root.
A request to import a directory uses the multi-page procedure below to select
each file's parent. Creation makes an empty page visible remotely; the imported
body and attachments remain local.

- Do not push automatically at the end of import. Publish only as a separate
  requested operation using the [synchronization guide](synchronization.md).

## Establish sources and destination

1. Locate the existing workarea and read `.cflsync/root`; retain its root ID as
   `ROOT_ID` under the skill's identity and scope rules. Do not initialize or
   re-anchor during import.
2. Read selected source files and identify referenced local media before
   creation. Preserve source files and assets; resolve relative media paths
   against each source file's directory, not the workarea or command directory.
3. Import Markdown files only, one new page per file. Copy other files only as
   referenced local media; do not create pages to represent source directories.

## Directory import

1. For a requested directory, enumerate its Markdown files recursively.
2. Process top-level files first, then each subdirectory's files before its
   children. Pages created earlier in the batch can parent later files.
3. Select `PARENT_ID` for each file:
   - Directly inside the requested directory: use `ROOT_ID`, regardless of the
     requested directory's name.
   - Inside a subdirectory: resolve the immediate containing directory's
     basename as an exact page title. Prefer cached exact-title matches; without
     one, look among remote pages inside the configured root tree. Include pages
     already created in the batch. Retain the matching ID as `PARENT_ID` and
     verify that it remains a valid in-tree parent.
   - With no matching page: use `ROOT_ID`. Check the immediate directory's own
     basename; do not inherit a matching ancestor directory's parent.
4. Pass resolved IDs to `page create`. Directory names are titles, not managed
   directory names, paths, or numeric IDs; passing a raw name could cause the CLI
   to interpret it as a path or ID.
   - For ambiguity or lookup failure, leave the file pending and report candidate
     IDs or the error. Root fallback requires confirmed absence of a match.
   - Preserve unrelated parent edits; child creation does not require parent
     publication.
5. Apply every single-page step below with the selected `PARENT_ID`, including
   media transfer and local verification. Do not push the batch.

For example, when `Architecture` is a matching page and `misc` is not:

| Source path relative to the requested directory | Parent |
| --- | --- |
| `overview.md` | Workarea root |
| `Architecture/decision.md` | The `Architecture` page |
| `misc/notes.md` | Workarea root |

## Single-page import

- Process files individually; retain each source path, selected title, new page
  ID, parent ID, installed path, and outcome.
- For individually selected files, set `PARENT_ID` to `ROOT_ID`. For directory
  import, use the parent selected above; the remaining steps are shared.

1. Select the page title:
   - With a level-one Markdown heading, use the first such heading's plain text.
     Recognize ATX (`# Title`) and Setext (`Title` followed by `=====`); ignore
     heading-like text in code blocks. Demote additional level-one headings
     to body sections during transfer below.
   - Without one, use the filename minus `.md`; `release.notes.md` gives
     `release.notes`.
2. Trim heading whitespace from the selected title; if the CLI rejects the
   derived title as unusable, resolve it with the user rather than inventing
   one.
3. From the workarea root, create the page under the selected parent:

   ```console
   cflsync page create PARENT_ID "Selected title"
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
5. Report missing or unresolved local media as errors and consult the user
   about recovery; leave that file's import incomplete while advice is pending.
   Do not overwrite managed or unmanaged files blindly or introduce traversal
   paths in `_attachments/` references. A copied file becomes managed only when
   `content.md` references it under `_attachments/`.
6. Re-read affected destination files before applying changes. Review the
   resulting content, attachment set, rewritten links, and any deletions;
   preserve unrelated edits, opaque ADF, and unmanaged files.

## Verify and report without publishing

1. Check the generated title, existence of each rewritten local-media target,
   and absence of overwrites between distinct source assets.
2. Apply [attachment validation](authoring.md#attachment-validation). Consult
   the user about missing referenced files or unreferenced local attachments;
   retain files without explicit deletion consent.
3. Inspect the resulting local changes and run:

   ```console
   cflsync page status NEW_ID
   ```

   Local content or attachment changes are expected; do not push to obtain an
   unchanged status.
4. Report each source file's new page ID, parent ID, managed path, local import
   outcome, unresolved assets, and failed or skipped steps. Distinguish remote
   creation of the empty page from local preparation of body and attachments.
   Status does not verify Confluence browser rendering.
5. For partial batches, retain successful pages and resume failed files by their
   known IDs. Do not repeat creation for an existing page or automatically delete
   created pages as rollback. Stop further creation while remote outcome or
   identity is uncertain and use the creation recovery guide.
