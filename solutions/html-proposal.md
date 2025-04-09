# Proposal: allow optional use of raw HTML for staff slips

<!-- md2toc -l 2 html-proposal.md -->
* [Background](#background)
* [Proposed solution](#proposed-solution)
* [Technical implementation](#technical-implementation)
    * [Back-end](#back-end)
    * [User interface](#user-interface)
* [Security considerations](#security-considerations)



## Background

At present, staff slips (hold slips, pick slips, etc.) are maintained using a WYSIWYG editor ([React Quill](https://github.com/zenoamaro/react-quill)) in the Circulation Settings area. This editor supports basic formatting -- bold, italic, numbered and bulletted lists, indenting, alignment, font size -- but not more advanced features such as layout out a table.

This kind of advanced layout is required by the National Library of Sweden (NLS): for example, their workflow includes placing pick slips inside books in such a way that the bottom third protrudes, displaying three bar-codes (item, user and instance HRID) all on the same row.

We have determiend experimentally that the staff-slip editor currently in use stores the slips as HTML, and that the various uses of the slips (e.g. display in a user's Loans page) simply render that HTML with substitution of the `{{token}}` strings. We have also shown that HTML is sufficiently expressive for the NLS to format their staff slips as required.



## Proposed solution

We propose providing a means for staff slips to be _optionally_ maintained as raw HTML, when library staff have the necessary skills, rather than using the simpler but more limited WYSIWYG editor. The choice of how to maintain slips would pertain only at edit time: either way, the resulting slips would be saved as HTML (as they are now) and their use elsewhere in the system would be unaffected.

The choice to maintain a given slip as HTML would be indicated by a checkbox in the Settings &rarr; Circulation &rarr; Staff slips &rarr; Edit page.



## Technical implementation


### Back-end

The staff-slip structure would need to be enhanced by a new boolean `isRawHtml` field, defaulting to false, so that the staff-slip editor page known which form of editing to provide.

THe relevant data structure is part of `mod-circulation-storage` module's [`staff-slip` schema](https://github.com/folio-org/mod-circulation-storage/blob/master/ramls/staff-slip.json), and the change made here would affect records obtained from the `/staff-slips-storage/staff-slips` WSAPI endpoint and sent to `/staff-slips-storage/staff-slips/{id}`.

We think that addition to the schema is the only back-end change required: no part of `mod-circulation-storage`'s actual functionality would need to inspect or act upon the value of the new field.


### User interface

Two change are required to `ui-circulation`, which provides the Settings &rarr; Circulation &rarr; Staff slips section.

First, when [displaying a staff slip](https://folio-snapshot.dev.folio.org/settings/circulation/staffslips/e6e29ec1-1a76-4913-bbd3-65f4ffd94e04), it must include an indication of whether that slip is maintained as raw HTML.

Second, when [editing a staff slip](https://folio-snapshot.dev.folio.org/settings/circulation/staffslips/e6e29ec1-1a76-4913-bbd3-65f4ffd94e04?layer=edit), it must:
* Show a checkbox indicating whether the slip is maintained as raw HTML
* When the checkbox is clear (the default), provide the existing WYSIWYG editor
* When the checkbox is checked, instead provide a simple `<TextArea>` for editing the HTML, enhanced with token-insertion and previewing with sample data



## Security considerations

When users are invited to provide HTML, the concern is always that this could provide a vector for accidental or deliberate damage, most obviously through the inclusion of `<script>` tags to run JavaScript, but also via more benign errors such as leaving a `<b>` or `<i>` open at the end of the text.

The `stripes-template-editor` library already uses [DOMPurify](https://github.com/cure53/DOMPurify) to prevent such attacks. At present it is used when setting up the editor to display content, and when previewing a template. It would be prudent (and easy) to also have it purify the content of an edited form before returning it to the calling code.



