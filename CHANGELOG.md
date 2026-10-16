# Changelog

## 1.1.2

No change to the editing engine or the studio.

- The test that starts `pdfedit studio` as a real command now resets SIGINT to its default in the child, so it passes when the suite is run from a background job (a job started with `&` hands SIGINT down ignored, and the studio could not be stopped).

## 1.1.1

Fixes to the studio found in review of 1.1.0.

- A reloaded tab (or a second tab) after turning to another page no longer saves the first page's text over the page the file is on. A page load shows the page the open file is on now, and every request carries its page number; a tab showing a different page is refused with "Reload the tab" instead of being merged in.
- Resize and crop are refused for a project opened with `--flatten` (they were accepted on the canvas, and the save returned the original). The message says to use Flatten page first.
- The README and the flatten message no longer say everything on a flattened page can be edited: that needs a text recogniser, which `pdfedit studio` does not use. Without one the page becomes a picture you can add layers on top of.
- The studio asks before opening another file, a blank page or a flattened page over a document with unsaved changes, warns before the tab is closed, and lets the server forget the document it replaced.
- The save message says text was removed from the file only for a PDF save where that happened.
- A refused request (oversize upload, unknown route, any error reply) closes the connection so its unread body is not read as the next request.
- Tests added for each of the above and for the project JSON round trip.

## 1.1.0

New: `pdfedit studio`, a visual canvas in the browser. The editing engine is unchanged.

- `pdfedit studio [FILE] [--port N] [--no-browser]` serves a canvas on `127.0.0.1` (standard library only; no new dependencies, no outside network calls). Open a PDF, PNG/JPEG or project; retype, move, restyle, add and delete layers; undo and redo; page picker; find and replace across one page or every page; true preview drawn by the saving code; flatten a page; resize or crop blank pages and pictures; save as PDF, PNG, JPEG or project.
- `pdfedit.studio`: sessions kept on the server, so the information that makes an edit a real removal never reaches the browser. `open_bytes`, `open_path`, `open_blank`, `open_project`, `open_page`, `operate` (`find-replace`, `restyle`, `resize-page`, `crop-page`), `flatten`, `export`, `preview`, `close`. At most 12 sessions are kept.
- `pdfedit.bridge`: the flat canvas view of a page and the merge of the canvas's edits back onto it.
- `pdfedit.server`: `make_server` and `serve`. Only answers requests that name the local address in `Host`, and refuses a `POST` from another origin.
- Resizing or cropping a picture now moves its pixels with the page (cropped, or letterboxed; never stretched). A PDF page edited in place keeps its size; flatten it first. A picture with lifted type cannot be resized or cropped.
- `restyle`, `resize_page` and `crop_page` (already in `pdfedit.edit`) now have tests.
- The package ships the page files (`studio_ui`).
- Two changelog lines for 1.0.1 reworded so the repository passes its own brand-free test.

## 1.0.2

Documentation and packaging only. The code is unchanged.

- The README now opens with a one-line description and a three-line example you can paste,
  and the licence and access notes moved to the end.
- The package version (`pip show pdfedit`, `pdfedit.__version__`) now matches the release tag.
  Release 1.0.1 reported 1.0.0.

## 1.0.1

Licence and contact details only. The code is unchanged.

- Licence text updated to version 1.3.
- One public contact address in the licence and the README.

## 1.0.0

First release.

- Read any page of a PDF into text runs (font, weight, size, colour, tracking, position), images and vector shapes, with ids that stay the same every time the file is read (`p3-t2`).
- Replace, restyle, move or delete text. Changed text is cut out of the page's content stream and the surviving text is re-anchored, so nothing else on the line or the page moves.
- New text is drawn in the file's own embedded TrueType font. A Unicode character map is rebuilt from the PDF's own mapping (its `/ToUnicode` map, or its `/Encoding` when there is none, as in files printed by macOS) so subsets from any producer can be reused; missing letters fall back to a full copy of the family from this computer's font folders or a built-in font, with a warning that names them. A redrawn run's font keeps its original name, so it reads back as the same family and weight.
- Text state that outlives a rewritten block (the leading set by `TD`, the spacing set by `"`) is put back, so later blocks that move with `T*` or `'` stay where they were.
- Rotated text is read as one run at its real size and angle. It can be deleted; `replace` leaves it unchanged with a note, and any retype is refused rather than drawn upright elsewhere.
- Protected files: a PDF that needs a password is refused in one line; one whose permissions forbid changes is listed but never saved edited; one that allows changes is edited, and the save says the copy is not protected.
- Saving a file nobody edited returns the original bytes. Pages nobody edited keep their content streams; outlines, form fields and other document-level parts are carried through.
- Objects painted on top of a change are redrawn with it, so stacking order is kept.
- `pdfedit.open()` and `Project`: open a PDF, PNG/JPEG or saved project; find and replace across pages; save as PDF, PNG, JPEG or a JSON project (schema `pdfedit/1`) that keeps the original file and reopens for further in-place edits.
- Raster path for images and scanned pages: measure lifted type, paint it out of the pixels, redraw it. The text recogniser is pluggable; macOS Vision is used when the `macos` extra is installed.
- `pdfedit` command: `list`, `replace` (with `--dry-run` and `--pages`), `set`, `delete`, `export`, `--version`. Errors are one line; a phrase split across runs is explained rather than reported missing.
- Full fonts are found in the operating system's font folders (every face of a `.ttc`, nested folders included), then reportlab's Vera family; `PDFEDIT_FONT_DIRS` replaces that list.
- `examples/price-list.pdf` and the script that makes it, used by the README quickstart.
