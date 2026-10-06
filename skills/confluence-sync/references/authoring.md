<!--
Copyright (c) 2026 Sverologos BV.
This Source Code Form is subject to the terms of the Mozilla Public License,
v. 2.0. If a copy of the MPL was not distributed with this file, You can obtain
one at https://mozilla.org/MPL/2.0/.
-->

# Authoring pages and attachments

## Editing workflow

1. Resolve the page and retain its ID. Inspect `cflsync page status PAGE_ID`.
   Incorporate remote-only changes with a targeted pull; preserve local edits;
   resolve two-sided changes using the synchronization guide. Pull an unpulled
   in-tree page before editing it.
2. Locate the current managed directory after any pull. Read `content.md` and
   relevant attachments; do not assume the path derived from a title is stable.
3. Re-read files immediately before making focused edits. Preserve unrelated
   human contributions and use the supported forms below.
4. Review the content and attachment diff, including deletions and links. A
   Git diff is useful when available, but Git is not required. Preserve opaque
   JSON blocks and validate any deliberately edited special-span attributes.
   Apply attachment validation below before publication or attachment changes.
5. Leave an editing-only result local. If publishing is requested, use
   `cflsync page push PAGE_ID`, then `cflsync page status PAGE_ID`. Handle a new
   conflict rather than forcing a stale candidate over intervening edits.

6. Report whether changes remain local, were published, or require resolution.

## Title and supported body content

- Keep the generated page title as the first and only level-one heading; use
  `##` through `######` for body headings.
- To change a title:
  1. On a synchronized page, use `cflsync page rename PAGE_ID "New title"`.
     Rename updates the remote title, generated heading, and directory together.
  2. Locate the page again by ID before further editing.

Supported body forms include paragraphs, hard line breaks, emphasis, links,
blockquotes, ordered/unordered/nested lists, task lists, fenced code with a
language, horizontal rules, and pipe tables with a header row. Inline forms
include `*italic*`, `**bold**`, `~~strikethrough~~`, inline code, and
`<u>underline</u>`, `<sub>subscript</sub>`, `<sup>superscript</sup>`.

- Use two trailing spaces only where a Markdown hard line break is intended.

Panels use quoted alert syntax:

```markdown
> [!WARNING]
> Check the release status before publishing.
```

Alert panels map one to one: `NOTE` is Info, `IMPORTANT` is Note, `TIP` is
Success, `WARNING` is Warning, and `CAUTION` is Error. Custom panels and panels
with explicit colors or icons use `<div data-type="panel-…">` around Markdown.
Supported suffixes are `info`, `note`, `tip`, `success`, `warning`, `error`, and
`custom`.

- Preserve `data-color`, `data-icon`, `data-icon-id`, and `data-icon-text` when
  present.
- Keep tags on separate lines with blank lines around the body, for example:

  ```markdown
  <div data-type="panel-custom" data-color="#F4F5F7" data-icon=":dart:">

  Panel text with **formatting**.

  </div>
  ```

Ordinary tables use GFM pipe syntax. Pulled complex tables may use HTML for
merged cells or multiple blocks in a cell. Table widths, cell colors, alignment,
and other advanced presentation settings do not survive conversion reliably.

- Preserve the HTML representation of complex tables as needed.

Text colors and highlights use `<span style="color: #0747a6">text</span>` and
`<span style="background-color: #f8e6a0">text</span>`, with six-digit hex values.
They can combine with supported inline formatting; inline code cannot carry
colors. `<mark>text</mark>` becomes a `#FFFF00` highlight. GitHub previews may
strip these styles even though Confluence retains them.

## Structured body forms

Images with captions, layout, or size settings use a figure around an image:

```markdown
<figure data-type="media-single" data-layout="wrap-right" data-width="400" data-width-type="pixel">

<img src="_attachments/example.png" width="800" height="600" alt="Example" />

<figcaption>

An editable **caption**.

</figcaption>

</figure>
```

`data-width` is the displayed width; `data-width-type` is `pixel` or
`percentage` (the default). Width must be positive and finite; percentage
width cannot exceed 100. The `<img>` width and height are separate intrinsic
dimensions. Layouts are `center`, `wrap-left`, `wrap-right`, `wide`,
`full-width`, `align-start`, and `align-end`. Captions contain one paragraph
of inline content. Unsupported figures remain opaque ADF.

- Preserve supported figure attributes and captions; do not reduce a figure to
  an ordinary Markdown image.

Column layouts use `<section data-type="layout-section">` with one
`<div data-type="column" data-width="66.66">` per column.

- Preserve optional `data-breakout` (`wide` or `full-width`) and
  `data-breakout-width`.
- Keep layouts at the top level, without nesting or placement inside other
  containers; put all section content inside columns.

Expands use `<details>` with a plain-text `<summary>` and a Markdown body.
Ordinary expands can carry `data-breakout` and positive `data-breakout-width`;
nested expands cannot.

- Escape HTML characters in summaries.
- Use `<details data-type="nested-expand">` inside another expand or a table cell.
- Keep layout sections outside expands.
- Keep structural tags on separate lines with blank lines around Markdown
  content so the body is parsed correctly.

## Attachments

- Use relative references rooted at the page's `_attachments/` directory:

  ```markdown
  ![Architecture](_attachments/architecture.png)
  [Download report](_attachments/report.pdf)
  ```

Links in `content.md` define the attachments required by the page. The CLI's
remote manifest tracks existing managed files; it can include files that the
body no longer references. A referenced local file joins the managed set,
while a new unreferenced file stays unmanaged. Management status does not
determine whether a file is required or authorize its deletion. Links elsewhere
remain ordinary links; external images are supported as external media. New
files referenced by HTML `<img src="_attachments/…">` also become managed,
including inside figures and HTML table cells.

- Do not introduce traversal paths.
- Percent-encode filenames in targets, for example `_attachments/Pasted%20image.png`,
  or enclose targets containing spaces in Markdown angle brackets.

When remote attachments share a filename, pull manages one and reports the
others. Those duplicates remain untouched in Confluence, and media referring
to an unmanaged duplicate remains opaque.

- Do not remove or overwrite unmanaged duplicates as an attachment repair.

Content and the complete managed attachment set share one synchronization
unit. Changing an attachment can conflict even when Markdown is unchanged.
Removing a previously managed attachment file locally can delete it remotely
on push. Removing a page directory manually does not delete the remote page.

- Delete local attachments only with explicit consent for the affected files;
  removing references alone does not authorize file deletion.
- Repair references after authorized deletion and keep remote deletion within
  requested scope.
- Use `page remove` for page deletion; do not substitute manual directory removal.

## Attachment validation

1. Read current `content.md` and compare local attachment targets with files in
   `_attachments/`. Include inline and reference-style image/file links and
   supported HTML `<img src="…">`; decode URL-encoded filenames. Ignore external
   URLs and links to other pages as local attachment requirements.
2. Classify discrepancies:
   - Missing referenced attachment: report an error with page, link, and expected
     filename; consult the user about recovery. Do not silently remove the link,
     fetch a replacement, or push the incomplete page.
   - Unreferenced local attachment, managed or unmanaged: report a warning with
     the filename; consult the user about its disposition. Retain it until
     deletion is explicitly authorized.
3. Resolve the user's advice before continuing affected attachment operations
   or publication. Any deletion requires explicit consent covering local files.
4. Apply the advice and check again. Preserve opaque ADF and existing references
   while investigating uncertain usage.

- Run validation after body editing/import, pull, and recovery, and before
  publication or attachment changes.
- Also validate a proposed body before a refresh affecting existing attachments.

## Links between managed pages

Pull converts qualifying same-site Confluence page links within the root tree
to relative links to the target's `content.md`, with percent-encoded path
segments. Push recognizes managed page links by the target directory's page-ID
suffix, even when a rename or move has left the path stale. The resolved target
must still be visible and inside the tree; otherwise push refuses the page
before any remote change, including attachment uploads.

- Diagnose or correct a reported broken page link; do not force the push.

Rename, move, and pull do not rewrite links in other local files. Fragments are
preserved byte for byte; Confluence and Markdown heading anchors may differ.
External URLs, fragment-only links, and links such as `../notes.md` remain
ordinary links. Page-link conversion does not turn arbitrary draft filenames
into links between imported pages. Smart Links are separate from page links.

## Special spans and opaque content

Pulled statuses and mentions use HTML spans; dates use `<time>` elements.

- Preserve required attributes and edit only intended values.
- Status: use `data-type="status"`; change the text and, if requested,
  `data-color` to `neutral`, `purple`, `blue`, `red`, `yellow`, or `green`.
  Preserve `data-status-style` (`bold` or `mixedCase`) when present. Color and
  style are optional; an omitted color defaults to `neutral`.
- Date: change `datetime="YYYY-MM-DD"` to the intended calendar date. The
  visible text inside `<time>` is ignored on push; update it to match. Dates
  represent UTC midnight, without the old local-timezone interpretation.
- Mention: retain `cfl-type="mention"`, the non-empty `cfl-id`, and existing
  account metadata unless the replacement account values are known.

Example status and date:

```html
<span data-type="status" data-color="green" data-status-style="bold">Done</span>
<time datetime="2026-04-01">April 1, 2026</time>
```

- Rewrite old `cfl-type="status"` and `cfl-type="date"` spans using these forms;
  push rejects obsolete markup.
- Review old panel alerts against the current panel mapping before publication.

Inline Smart Links use
`<a href="https://example.atlassian.net/browse/ABC-123" data-card-appearance="inline">…</a>`.
The visible text is ignored on push. Other HTML links are unsupported.

- Change `href` to edit the target.
- Keep cards outside inline formatting.
- Use ordinary Markdown links for other links.

An email link such as `[Example User](mailto:example.user@example.com)` becomes
a mention only if cflsync finds exactly one accessible user matching both the
display-name search and email. Otherwise it remains an email link.

- Do not report successful mention creation from local syntax alone.

Other Confluence macros and unsupported content can appear as fenced
`atlas_doc_format` blocks containing original JSON. These are opaque retention
forms, not a Markdown authoring format.

- Leave complete blocks unchanged so push can restore them.
- Do not normalize, invent, or delete their JSON as a side effect of prose editing.

## Verification limits

Conversion can lose image borders, inline-image dimensions, and advanced
table presentation. Supported panels and image figures preserve their settings;
unsupported forms remain opaque. A successful push and unchanged status verify
synchronization, not Confluence browser rendering, mention appearance, or
app-macro behavior.

- Check the remote page when those features matter to the request.

Markup source:
[cflsync writing guide](https://github.com/sverologos/cflsync/blob/4ec3f3ecb3968017fc9fdde9e7140af4a228eafc/doc/MARKUP.md).
