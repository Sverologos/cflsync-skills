<!--
Copyright (c) 2026 Sverologos BV.
This Source Code Form is subject to the terms of the Mozilla Public License,
v. 2.0. If a copy of the MPL was not distributed with this file, You can obtain
one at https://mozilla.org/MPL/2.0/.
-->

# Create a page from scratch

## Establish destination and creation scope

1. Establish the workarea, destination parent, new title, and whether the body
   and attachments should be published or left local. Check
   `cflsync page create --help` if the installed capability is not established.
   Use setup guidance only for missing dependencies or a missing workarea.
2. Resolve the parent using a managed path, numeric ID, or exact title. It must
   be inside the configured root tree, including the root itself. Clarify
   ambiguity and retain its ID; creation uses the parent's space.
3. Validate a non-empty, single-line title without surrounding whitespace.
4. Establish remote creation scope:
   - A request to create a Confluence page authorizes immediate creation of an
     empty remote page. Later local edits require a push to appear remotely.
   - For a local-only draft or no remote visibility before review, prepare the
     draft outside the managed tree and leave creation pending. Creation has
     no offline or deferred mode.

## Create once and retain identity

1. Run from the workarea root with the established parent ID and title:

   ```console
   cflsync page create PARENT_ID "New page title"
   ```

   - The CLI installs the parent and missing ancestors before creation,
     preserving existing local ancestor edits. Do not publish those edits
     merely to create a child.
   - Every page directory, including the root's, ends in its page ID. The new
     ID is known only after remote creation; a clash at the actual path is
     refused by the follow-up pull after the remote page exists. Preserve the
     entry and use creation recovery rather than create again.
   - Do not construct managed directories or change `.cflsync/` manually.
2. Check the exit status before editing. The CLI creates an empty remote child,
   then pulls its generated title heading and attachment directory.
3. Inspect the newly installed directory under the parent. Successful creation
   at this revision does not print the new ID or path; obtain them using the
   managed path:

   ```console
   cflsync page status "path/to/new/page/content.md"
   ```

   Retain the ID as `NEW_ID` and the installed path. Do not select an arbitrary
   title match or assume that the title alone identifies the path.
4. Read the generated `content.md` and inspect local/remote status before
   editing. Apply the authoring and synchronization guides to intervening
   changes from other contributors.

## Author the body

1. Write the requested body below the generated title using the
   [authoring guide](authoring.md).
   - For a supplied draft with an explicit title or parent, reuse only the
     [body and media transfer steps](import.md#copy-the-body-and-local-media).
     Retain this creation request's title, parent, and publication scope.
   - For title derivation from Markdown and parent selection for selected files
     or directories, use the dedicated [import guide](import.md).
2. Follow the requested publication scope:
   - For creation and local authoring only, leave changes local and report that
     the empty page already exists remotely.
   - For authorized publication, use only the new page's ID:

     ```console
     cflsync page push NEW_ID
     cflsync page status NEW_ID
     ```

     Use normal concurrency checks and resolve conflicts through the
     synchronization guide; do not force a stale draft over newer content.
     A whole-workarea push is outside a single-page request.
3. Report remote creation, local body/attachment changes, publication, and final
   status separately. Synchronization does not verify browser rendering.

## Recover after partial creation

1. If remote creation succeeded but pull failed, retain the created ID reported
   in the error. Preserve local work and installed ancestors; resolve the clash,
   access failure, or other reported cause.
2. Complete local installation of the existing page:

   ```console
   cflsync page pull NEW_ID
   ```

   Preserve and reconcile existing local changes through the synchronization
   guide instead of applying force automatically. Do not create again or
   automatically delete the remote page as rollback.
3. If the response was lost or the ID is unknown, inspect Confluence under the
   intended parent before retrying. A non-zero exit does not prove that nothing
   was created; parent installation may also have succeeded. Stop further
   creation while the remote outcome or new identity remains uncertain.

Creation behavior source:
[cflsync specification](https://github.com/sverologos/cflsync/blob/4ec3f3ecb3968017fc9fdde9e7140af4a228eafc/doc/SPEC.md).
