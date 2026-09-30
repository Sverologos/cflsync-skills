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
5. Leave an editing-only result local. If publishing is requested, use
   `cflsync page push PAGE_ID`, then `cflsync page status PAGE_ID`. Handle a new
   conflict rather than forcing a stale candidate over intervening edits.

Report whether changes remain local, were published, or require resolution.

## Title and supported body content

The first and only level-one heading is the generated page title. Keep it
unchanged; use `##` through `######` for body headings. To change the title,
use `cflsync page rename PAGE_ID "New title"` on a synchronized page. Rename
updates the remote title, generated heading, and managed directory together;
then locate the page again by ID before further editing.

Supported body forms include paragraphs, hard line breaks, emphasis, links,
blockquotes, ordered/unordered/nested lists, task lists, fenced code with a
language, horizontal rules, and pipe tables with a header row. Inline forms
include `*italic*`, `**bold**`, `~~strikethrough~~`, inline code, and
`<u>underline</u>`, `<sub>subscript</sub>`, `<sup>superscript</sup>`.
Use two trailing spaces only where a Markdown hard line break is intended.

Panels use quoted alert syntax:

```markdown
> [!WARNING]
> Check the release status before publishing.
```

Supported panel tags are `NOTE`, `TIP`, `WARNING`, and `CAUTION`. Ordinary
tables use GFM pipe syntax. Pulled complex tables may use HTML for merged cells
or multiple blocks in a cell; preserve that representation as needed. Table
widths, cell colors, alignment, and other advanced presentation settings do
not survive conversion reliably.

## Attachments

Use relative references rooted at the page's `_attachments/` directory:

```markdown
![Architecture](_attachments/architecture.png)
[Download report](_attachments/report.pdf)
```

The remote manifest defines existing managed files. A new local file becomes
managed when `content.md` references it under `_attachments/`; an unreferenced
local file stays unmanaged. Links elsewhere remain ordinary links; external
images are supported as external media. Do not introduce traversal paths.

Content and the complete managed attachment set share one synchronization
unit. Changing an attachment can conflict even when Markdown is unchanged.
Removing a previously managed attachment file locally can delete it remotely
on push. Review intended deletions and repair or remove references accordingly;
do not discard unmanaged files. Removing a page directory manually does not
delete the remote page and is not a substitute for `page remove`.

## Special spans and opaque content

Pulled statuses, dates, and mentions may use cflsync-specific HTML spans.
Preserve their required attributes and edit only the intended values:

- Status: retain `cfl-type="status"` and `style="background-color: COLOR"`.
  Change the text and, if requested, the color to `gray`, `purple`, `blue`,
  `red`, `yellow`, or `green`.
- Date: retain `cfl-type="date"`. Prefer text such as
  `2026-04-01[Europe/Brussels]` with a symbolic timezone. Without the bracketed
  timezone, push uses the machine's local timezone. A `cfl-timestamp` attribute
  preserves the original timestamp unchanged; changing visible text alone
  does not change it. Preserve timestamp-backed spans unless a date change is
  requested; to author a different date, use the documented text-only form.
- Mention: retain `cfl-type="mention"`, the non-empty `cfl-id`, and existing
  account metadata unless the replacement account values are known.

Example text-only date:

```html
<span cfl-type="date">2026-04-01[Europe/Brussels]</span>
```

An email link such as `[Example User](mailto:example.user@example.com)` becomes
a mention only if cflsync finds exactly one accessible user matching both the
display-name search and email. Otherwise it remains an email link; do not
report successful mention creation from local syntax alone.

Other Confluence macros and unsupported content can appear as fenced
`atlas_doc_format` blocks containing original JSON. Leave the complete blocks
unchanged so push can restore them. They are opaque retention forms, not a
Markdown authoring format; do not normalize, invent, or delete their JSON as a
side effect of prose editing.

## Verification limits

Conversion can lose panel colors/icons, image dimensions/layout, and advanced
table presentation. Check the remote page when those features matter to the
request. A successful push and unchanged status verify synchronization, not
Confluence browser rendering, mention appearance, or app-macro behavior.

Markup source:
[cflsync writing guide](https://github.com/sverologos/cflsync/blob/69ebb39239c449ab9b5921e847df09cf7764a5c0/doc/MARKUP.md).
