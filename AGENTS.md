# CRC

This repository is a collection of **Microsoft Outlook email templates** (`.oft`
files) for multiple Great Plains Communications locations (Batesville, Blair,
Chadron, Greensburg, Kearney, Liberty, McCook, Omaha, Sunman). There are two
template types per location: a *General Feedback Template* and a
*New Customer Email Template*.

There is **no application to build, serve, or run** — the deliverables are the
binary `.oft` files, opened by an end user in the Outlook desktop client.

## Cursor Cloud specific instructions

- There is no app/server/test suite here. "Working on this repo" means editing
  the binary `.oft` templates; there is nothing to `build`/`run`/`serve`.
- `.oft` files are **OLE2 / Compound File Binary (CFB)** documents holding a
  MAPI message. You cannot edit them as plain text. The pieces that matter:
  - The rendered **HTML body** lives in the `PR_RTF_COMPRESSED` stream
    (`__substg1.0_10090102`) as **LZFu-compressed**, `\fromhtml1`
    RTF-encapsulated HTML. Image `src` URLs appear intact inside `\*\htmltag`
    groups, so a literal find/replace works *after* decompression.
  - There is also a plain-text alternative body in `PR_BODY_W`
    (`__substg1.0_1000001F`), stored as **UTF-16LE** (no trailing null; its
    length in `__properties_version1.0` is byte-length + 2).
  - `__properties_version1.0` stores a 4-byte size field per variable-length
    property (32-byte header, then 16-byte entries; size at entry offset +8).
    For `PtypString` (`001F`) the size = stream bytes + 2; for `PtypBinary`
    (`0102`) the size = exact stream bytes.
- **Editing recipe** (what was used to fix the broken image links): decompress
  the RTF, replace URLs, recompress (LZFu), replace URLs in the UTF-16LE plain
  body, update the matching size fields in `__properties_version1.0`, then
  rebuild the compound file. Because URL replacement changes stream sizes, the
  CFB must be re-laid-out — preserve every directory entry's metadata
  (name/type/color/sibling+child links/CLSID/timestamps) and only recompute
  starting sectors + sizes so the directory tree stays valid.
- **Tooling**: Python 3 with `olefile` (read CFB streams) and `compressed_rtf`
  (LZFu (de)compression) are sufficient to read and edit. `RTFDE` is handy to
  de-encapsulate the RTF back to HTML for previewing in a browser. These are
  installed by the startup update script.
- To preview a template, de-encapsulate its RTF body to HTML and open the HTML
  in a browser; the images are remote, so they load from their hosting URLs.
