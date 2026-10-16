# pdfedit — Edit the words inside a PDF you didn't make

**Paid tool: $149 once (plus sales tax where it applies), then a private GitHub repository you're invited to, with every update. Buy: https://dominares.org/tools/pdfedit**

Commercial license v1.3 — see https://dominares.org/license · Support: support@dominares.org

A Python library, a command and a browser canvas for changing prices, dates, addresses and headings in existing PDFs. It cuts the old words out of the page and draws the new ones in the file's own embedded font, so nothing else moves. The canvas, pdfedit studio, runs on your computer only.

## Get it

Buy pdfedit at https://dominares.org/tools/pdfedit: one payment of $149, plus sales tax where it applies, and every future update. After checkout you are invited to a private GitHub repository with the source and every version published to it. The repository includes the Python library, the pdfedit command and pdfedit studio, the browser canvas.

Requirements: Installs pypdf, pdfplumber, pypdfium2, reportlab, fonttools, Pillow and numpy.

This page documents version 1.1.2. Install commands and file paths in the documentation below run inside the private repository you are invited to; the source is not in this repository.

## Documentation

`pdfedit` is a Python library, a command-line tool and a browser canvas (the studio) for changing text in existing PDFs: a price, a date, an address, a phone number, a heading. It reads each page into real objects (text runs with their own font, size, colour and position, plus images and vector shapes), lets you change them, and writes the page back by cutting the changed text out of the page's content stream and drawing the new text in the file's own embedded font.

Most tools that "edit" a PDF paint a white box over the old words and type new ones on top. The old words are still in the file: they can be selected, copied, searched and read back by anyone. `pdfedit` takes them out.

### Who it is for

Developers who need to update wording in PDFs that came from somewhere else: price lists and menus from a designer, notices exported from a browser, invoices and forms from an office suite, a batch of files that all say last year's date. You get a library to call from your own code and a `pdfedit` command for one-off jobs and shell scripts.

### What you get

- **Real removal.** Changed text is cut out of the content stream. The rest of the page keeps its exact bytes, and every other character stays at exactly the same position, including neighbours on the same line.
- **The file's own type.** New text is drawn with the font program embedded in the PDF. When the embedded subset lacks a letter you need, you get a warning that names the letters and the font used instead, never a blank box.
- **Untouched means untouched.** Open a file, change nothing, save: you get the same bytes back. Edit page 2 of 40: the other 39 pages are carried through as they were.
- **Stacking kept.** If you recolour a panel, anything painted on top of it is redrawn on top of it again.
- **Every page, one pass.** Replace a word across a whole file, see each line that changed, then save once.
- **Stable ids.** `p3-t2` is the second text run on page 3, every time the file is read, so `pdfedit list` followed by `pdfedit set` works.
- **Projects.** Save your work as a JSON project (original file included), reopen it later, and your edits are still real edits to the original page.
- **A visual canvas.** `pdfedit studio` opens the file in your browser: click a line to retype it, drag layers, add text, shapes and pictures, find and replace across every page, and see exactly how the saved file will draw the page. See [The studio](#the-studio).
- **Pictures too.** A PNG or JPEG, or a scanned PDF page, can be edited through the raster path: the old words are painted out of the pixels and the new ones drawn in their place. This needs a text recogniser; see [Images and scanned pages](#images-and-scanned-pages).

### Install

*The install commands are in the README of the private repository you are invited to when you buy. The notes on them are kept here.*

This installs the `pdfedit` command and the `pdfedit` Python package, with its dependencies (pypdf, pdfplumber, pypdfium2, reportlab, fontTools, Pillow, numpy).

### Quickstart (60 seconds)

The folder you received has a sample to practise on: `examples/price-list.pdf`, a one-page price list with an embedded subset font ("Ledger Sans"), built-in fonts, a picture and a coloured panel. (`examples/make_price_list.py` is the code that made it.) Every command below also works on any PDF of your own.

```bash
cd pdfedit/examples
```

See what is on the page. Each run has an id, its font, weight, size and colour, its position (points from the top left) and its text:

```console
$ pdfedit list price-list.pdf
price-list.pdf: 1 page(s), PDF

page 1: 612 x 792 pt
  p1-t1     LedgerSans 700 28.0pt #21543d          at 72,71     'Price list 2025'
  p1-t2     Helvetica 400 12.0pt #1c2b36           at 72,106    'Valid from 1 March 2025'
  p1-t3     LedgerSans 400 14.0pt #1c2b36          at 72,155    'Workshop hire: $40 per hour\nBench hire: $15 per day'
  p1-t4     Helvetica 700 14.0pt #1c2b36           at 72,241    'OPEN DAILY'
  p1-t5     Helvetica 400 12.0pt #1c2b36           at 72,266    'Members save ten percent on every booking'
  p1-t6     Helvetica 400 12.0pt #1c2b36           at 72,302    'Call'
  p1-t7     Helvetica 700 12.0pt #1c2b36           at 94,302    '555-0100'
  p1-t8     Helvetica 400 12.0pt #1c2b36           at 142,302   ' to book a bench'
  p1-t9     Times 400 11.0pt #1c2b36               at 72,353    '14 Orchard Lane\nOpen 9am to 5pm'
  p1-t10    Helvetica 400 8.0pt #1c2b36            at 72,390    'Prices include tax'
```

A run is text set in one font, size and colour. "Call 555-0100 to book a bench" is three runs because the number is bold.

Ids describe the file as it is. After you save an edited file, list the new file again before using ids from it.

Preview a change across every page, then make it:

```console
$ pdfedit replace price-list.pdf "2025" "2026" --dry-run
  page 1 p1-t1: 'Price list 2025' -> 'Price list 2026'
  page 1 p1-t2: 'Valid from 1 March 2025' -> 'Valid from 1 March 2026'
2 replacement(s) in 2 run(s) on 1 page(s).
Dry run: nothing was written.

$ pdfedit replace price-list.pdf "2025" "2026" --out price-list-2026.pdf
  page 1 p1-t1: 'Price list 2025' -> 'Price list 2026'
  page 1 p1-t2: 'Valid from 1 March 2025' -> 'Valid from 1 March 2026'
2 replacement(s) in 2 run(s) on 1 page(s).
Saved price-list-2026.pdf: 2 run(s) rewritten, 2 original drawing instruction(s) removed.
```

The same thing from Python:

```python
import pdfedit

project = pdfedit.open("price-list.pdf")
for change in project.replace("2025", "2026"):
    print(change.page, change.before, "->", change.after)
report = project.save("price-list-2026.pdf")
print(report.warnings)
```

Change one run by id, or take one off the page:

```bash
pdfedit set price-list.pdf p1-t10 --text "Prices exclude tax" --color "#8a1c1c" --out updated.pdf
pdfedit delete price-list.pdf p1-t10 --out trimmed.pdf
```

Save a project, carry on later, then build the PDF:

```console
$ pdfedit replace price-list.pdf "2025" "2026" --out work.json
  ...
Saved a project to work.json. Reopen it with any pdfedit command.
$ pdfedit replace work.json "Bench hire" "Table hire" --out work.json
  page 1 p1-t3: 'Bench hire: $15 per day' -> 'Table hire: $15 per day'
1 replacement(s) in 1 run(s) on 1 page(s).
Saved a project to work.json. Reopen it with any pdfedit command.
$ pdfedit export work.json --out final.pdf
Saved final.pdf: 3 run(s) rewritten, 4 original drawing instruction(s) removed.
```

Replace works within one run. A phrase that changes font, size or colour part way is found and explained, and you replace each part on its own:

```console
$ pdfedit replace price-list.pdf "Call 555-0100" "Call 555-0199" --dry-run
No single run in price-list.pdf contains 'Call 555-0100', but it is there on page 1 across p1-t6, p1-t7, p1-t8: the wording changes font, size or colour part way. Replace each part on its own. Nothing was written.
$ pdfedit replace price-list.pdf "555-0100" "555-0199" --out updated.pdf
```

### The studio

```bash
pdfedit studio price-list.pdf           # opens http://127.0.0.1:8765/ in your browser
pdfedit studio --port 8800 --no-browser # a different port, and no browser window
pdfedit studio                          # start empty; open a file, or a blank page, from the page
```

The studio is part of the same package and needs nothing extra: it uses only Python's standard library for the server and plain JavaScript in the page, with no outside scripts, fonts or network calls.

What you can do in it:

- **Open** a PDF, a PNG or JPEG, or a saved project (drop the file on the window or use Open). Several pages are shown one at a time with a page picker; edits to a page you leave are kept.
- **Change a line.** Click it, retype it in the inspector (or double-click), and the old text is removed from the saved file. Move, resize, rotate, recolour, change font, size, weight, alignment and spacing; delete, duplicate, lock, reorder.
- **Add** text, rectangles, ellipses, lines and PNG/JPEG pictures.
- **Find and replace** across the page or every page, with a list of each line that changed and a note for any rotated line that was left alone.
- **True preview.** The canvas in your browser is an approximation (the browser may not own the file's embedded font). True preview shows the page drawn by the same code that writes the saved file.
- **Flatten page.** Turns the page into a picture, at the cost of sharp vector type. With a text recogniser (see [Images and scanned pages](#images-and-scanned-pages); on a Mac, the `macos` extra), the words are lifted and can be edited like any other line. Without one, which is how `pdfedit studio` starts, you get a picture you can add layers on top of. It replaces the open document with a one-page picture of this page, after asking; save the original first if you need it.
- **Resize or crop** a blank page or a picture (the picture is cropped or letterboxed with its contents, not stretched). A PDF page edited in place, or a project opened with `--flatten`, keeps its size: use Flatten page first.
- **Save as** PDF (the whole file), PNG or JPEG (the page on screen, 200 dpi), or a project that reopens with its edits still real edits. Undo and redo cover the changes you make on the page you are on (they start fresh when you turn to another page or run find and replace).

The studio asks before it replaces a document you have changed but not saved, and the browser warns before you close the tab. A reloaded tab shows the page the file is on; a tab that is showing a different page from the one the studio has open is refused instead of being merged in.

A layer outlined in amber dashes is one that cannot be taken out of the original (text inside a reusable group, for example). Changing it draws over the old marks, and the page says so. The status bar and the save message tell you whether the file was changed at all: saving without changes returns your original bytes.

How it stays honest: the document stays in the `pdfedit` process on your computer. The browser is sent a flat view of one page (positions, colours, text, pictures), edits it, and sends the view back; the information needed to remove the original marks never goes to the browser, so a reload or a stray edit cannot lose it. The server listens only on `127.0.0.1`, refuses requests that name any other host, and refuses a `POST` from another site.

The same pieces are importable if you want to put the canvas inside your own tool: `pdfedit.studio` (sessions: `open_bytes`, `open_page`, `operate`, `flatten`, `export`, `preview`, `close`), `pdfedit.bridge` (canvas view and merge) and `pdfedit.server` (`make_server`, `serve`).

### Command reference

Every command takes a PDF, a PNG/JPEG, or a project `.json`, and `--flatten` to treat each PDF page as a picture (for scanned pages).

| Command | What it does |
|---|---|
| `pdfedit list FILE [--pages 1,3-5] [--all] [--json]` | List the text runs on each page, with ids. `--all` adds images and shapes; `--json` prints machine-readable rows. |
| `pdfedit replace FILE OLD NEW (--out PATH \| --dry-run) [--pages …] [--match-case] [--whole-word]` | Replace wording on every page (or the pages listed). Prints each changed line, and a note for any rotated run that contains the text but was left unchanged. Exits 1 if nothing matched. |
| `pdfedit set FILE ID [--text …] [--color #hex] [--size PT] --out PATH` | Change one run. Use `\n` in `--text` for a line break. Refused for rotated text. |
| `pdfedit delete FILE ID [ID …] --out PATH` | Take runs, images or shapes off the page. |
| `pdfedit export FILE --out PATH [--page N] [--dpi N]` | Write a project or file as `.pdf`, `.json`, `.png` or `.jpg`. |
| `pdfedit studio [FILE] [--port N] [--no-browser]` | Edit on a visual canvas in your browser (default port 8765; `--port 0` picks a free one). |

`--out` picks the format from its suffix: `.pdf` (the edited file), `.json` (a project), `.png`/`.jpg` (one page as a picture, `--dpi`, default 200). Commands that write report how many runs were rewritten, how many original drawing instructions were removed, and whether the output is the original file unchanged. Any warning is printed as a `note:` line. A problem is one `error:` line on stderr and exit status 1. `pdfedit --version` prints the version.

### Python reference

```python
import pdfedit

project = pdfedit.open(source, *, flatten=False, recognizer=None)   # path, bytes, or a project .json
```

`source` can be a PDF, a PNG/JPEG or a saved project; the type is decided from the bytes, not the file name. Page numbers count from 1.

| `Project` | |
|---|---|
| `page(n)` | The `Document` for page `n`, read the first time it is asked for. |
| `pages(numbers=None)` | `[(n, Document), …]` for the pages asked for, or all of them. |
| `text_runs(numbers=None)` | `[(n, TextObj), …]` in paint order. |
| `find(id)` | `(n, object)` for an id like `p3-t2`. |
| `replace(old, new, *, pages=None, match_case=False, whole_word=False)` | Returns a list of `Change(page, id, before, after, count)`. Rotated runs that contain `old` are left unchanged and described in `project.not_changed`. |
| `changed_pages` | Page numbers with edits. |
| `export()` | An `ExportReport` with the PDF bytes; writes nothing to disk. |
| `save(path, *, page=1, dpi=200)` | Write `.pdf`, `.json`, `.png` or `.jpg`. Returns the `ExportReport` (`None` for `.json`). |
| `save_project(path)` | Write a project file, original bytes included. |

`ExportReport` has `pdf` (bytes), `redrawn` (ids drawn again), `excised_ops` (drawing instructions removed from the original), `warnings` (sentences to show a person), `rebuilt` (True when the page was rebuilt over a picture rather than edited in place) and `byte_identical` (True only when `pdf` is the original file).

A `Document` is one page: `width`, `height` (points), `objects` in paint order, `warnings`, and `changed_ids()`. Edit its objects directly:

| Object | Fields you can change |
|---|---|
| `TextObj` | `text` (use `\n` for lines), `size`, `color` (RGB 0 to 1, or use `pdfedit.hex_to_rgb("#8a1c1c")`), `letter_spacing`, `line_height`, `align` (`left`, `center`, `right`), `x`, `y`, `font` (`family`, `weight`, `italic`). Read-only: `angle` (degrees; text read at an angle has `retypeable` False and can only be deleted). |
| `ShapeObj` | `fill`, `stroke`, `stroke_width`, `x`, `y`, `width`, `height` |
| `ImageObj` | `x`, `y`, `width`, `height`, `crop` |

Remove an object by taking it out of `doc.objects`; add one by appending a new `TextObj`. Changes are measured by diffing against the page as it was read, so there is no dirty flag to forget, and a colour that has been through a hex field and back does not count as a change.

Lower-level functions, for one page at a time: `pdfedit.read_pdf(data, page_number=0)`, `pdfedit.export_pdf(doc, original=data, page_number=0)`, `pdfedit.export_pages(data, {page_index: doc, …})` (0-based page indexes) and `pdfedit.read_image(data, recognizer=None)`.

### Configuration

| Setting | Default | What it does |
|---|---|---|
| `PDFEDIT_FONT_DIRS` (environment) | this computer's font folders, then the Bitstream Vera fonts that reportlab installs | Folders of full fonts with TrueType outlines (`.ttf`, `.ttc`, and `.otf`/`.woff` files built on TrueType outlines), searched with their subfolders, separated by `:` (`;` on Windows). When an embedded subset lacks a letter, or a font is not embedded at all, a full copy of the same family found here is used instead, silently and correctly. Setting it replaces the default; an empty string means no folders. Fonts are matched by the family name, weight and slant inside the file, not by file name. The default folders are `~/Library/Fonts`, `/Library/Fonts`, `/System/Library/Fonts` and its `Supplemental` folder on macOS; `%LOCALAPPDATA%\Microsoft\Windows\Fonts` and `%WINDIR%\Fonts` on Windows; `~/.local/share/fonts`, `~/.fonts`, `/usr/local/share/fonts` and `/usr/share/fonts` elsewhere. Helvetica, Times and Courier are never taken from these folders, because every PDF reader has them built in. |
| `recognizer=` (Python) | macOS Vision when installed, otherwise none | A callable that takes image bytes and returns a list of `pdfedit.TextLine(text, x0, y0, x1, y1, confidence)` in pixels from the top left. Used for PNG/JPEG input and `flatten=True`. |
| `flatten=` / `--flatten` | off | Treat each PDF page as a picture: for scanned pages, or pages whose text cannot be taken apart. The edited page is saved as a picture with the new words drawn over it. |

Project files carry the schema id `pdfedit/1`. Files written by `pdfedit` have `/Producer` set to `pdfedit` in the document information dictionary; every other field there, including the title, is kept. An XMP metadata stream, if the file has one, is carried through unchanged, so it still names the original producer.

### Images and scanned pages

A picture has no text objects in it, so editing one means finding the words, measuring how they were set (size, weight, colour, a best-match font), painting them out of the pixels, and drawing the new words in their place. Only lines you change are painted out; a picture nobody edited is saved exactly as it came in.

Finding the words needs a text recogniser. On a Mac, `pip install "./pdfedit[macos]"` enables the built-in Vision framework, which runs on the machine with no network calls. Anywhere else, pass your own. For example, with Tesseract via `pytesseract` (neither is a dependency of pdfedit, and this snippet is an illustration rather than part of the test suite):

```python
import io
import pytesseract
from PIL import Image
import pdfedit

def tesseract(data: bytes):
    image = Image.open(io.BytesIO(data))
    rows = pytesseract.image_to_data(image, output_type=pytesseract.Output.DICT)
    lines = {}
    for i, word in enumerate(rows["text"]):
        if word.strip():
            key = (rows["block_num"][i], rows["par_num"][i], rows["line_num"][i])
            lines.setdefault(key, []).append(i)
    out = []
    for idx in lines.values():
        x0 = min(rows["left"][i] for i in idx)
        y0 = min(rows["top"][i] for i in idx)
        x1 = max(rows["left"][i] + rows["width"][i] for i in idx)
        y1 = max(rows["top"][i] + rows["height"][i] for i in idx)
        text = " ".join(rows["text"][i] for i in idx)
        out.append(pdfedit.TextLine(text, x0, y0, x1, y1, confidence=0.9))
    return out

project = pdfedit.open("poster.png", recognizer=tesseract)
project.replace("Saturday", "Sunday")
project.save("poster-sunday.pdf")
```

Without any recogniser, a picture still opens (and can be saved as a PDF), with a warning that no text was lifted.

### Honest limits

- **Subset fonts only carry the letters the original used.** A PDF usually embeds just the glyphs its text needed. If your new wording needs a letter that is not in the subset, `pdfedit` draws that run in a full installed copy of the same family if it can find one (see `PDFEDIT_FONT_DIRS`), otherwise in the closest built-in font (Helvetica, Times or Courier), and says so in a warning that names the missing letters. The whole run is drawn in that font, including its other lines, so a run never mixes two faces; the warning says so. Install the family to keep the look.
- **Only TrueType font programs are reused.** Embedded CFF and Type 1 fonts are common in files from some design tools; text set in them is redrawn in the closest installed or built-in font, with a warning.
- **Scanned PDFs and images need the raster path.** There is no text in a scan to edit. With `--flatten` and a recogniser the words can be painted out and redrawn, but the result is a picture with type on it, not selectable text in the original's layout, and the typeface is a best match, not a certainty.
- **Replace works within one run.** A run is text in one font, size and colour. A phrase whose middle word is bold is three runs; replace each part on its own (the "no match" message tells you when this is the case).
- **Rotated text is listed and can be deleted, but not retyped.** Text set at an angle (a spine, a table header, a chart axis) is read as one run with its angle, and `list` marks it `(rotated 90°, delete only)`. `replace` leaves it unchanged and says so in a note, `set` refuses it, and a save that would redraw it stops with an error rather than drawing it upright somewhere else.
- **Text is not reflowed.** A changed run keeps its position and its left edge. Longer wording runs wider; paragraphs are not re-wrapped or re-justified. Check the result when the new text is much longer than the old.
- **Some text cannot be edited in place.** Text inside reusable groups (form XObjects) is listed and marked with `*`; an edit to it is drawn over the old marks with a warning, because the old marks cannot be taken out. Pages stored with a `/Rotate` are listed but editing them is refused; save an upright copy first. Right-to-left scripts, vertical writing mode and Type 3 fonts are not supported.
- **Protected files.** A PDF that needs a password to open is refused. A PDF that opens freely but whose permission settings forbid changes can be listed, and saving an edited copy is refused. A PDF that opens freely and allows changes is edited, and the saved copy does not carry the original's password protection or permission settings (the save says so in a note).
- **The studio edits one page at a time and one person at a time.** It is a local tool for one user on one computer: no accounts, no sharing, and it keeps at most 12 open files in memory (the oldest is forgotten; open it again). It is not meant to be put on a network, and refuses to bind to anything but the local address.
- **A page edited inside the original PDF keeps its size.** Resize and crop work on blank pages and pictures; for a PDF page, flatten it first. A picture whose type has been lifted into editable text cannot be resized or cropped, because the paint-out is tied to its pixels.
- **The canvas is a close drawing, not the file.** Fonts the browser does not have are drawn in a stand-in on the canvas; use True preview to see the saved result.
- **Not a redaction tool.** `pdfedit` changes wording on the page. It removes the drawing instructions it edits, but it does not look for other copies of the same words a PDF can carry: metadata, bookmarks, annotations, form fields, alternative text, or other pages you did not edit. Do not use it to hide information.

Use it on documents you have the right to edit.

### How it works

1. **Two readers, joined by position.** pdfminer (through pdfplumber) resolves what each character is and how it is set. A small content-stream parser resolves which operator painted it. The two are joined on the exact point each character was painted at, not by counting, so files that group glyphs differently still line up.
2. **Excise and re-anchor.** PDF text positions are relative: removing one string would slide everything after it on the line. So when a text block is rewritten, every surviving string is given its absolute position explicitly first. Removal is then exact, with no font metrics involved. Blocks with nothing removed are copied through byte for byte.
3. **Redraw in the file's own font.** The embedded font program is lifted out of the file, given a Unicode character map built from the PDF's own mapping, and used to draw the new text, which is merged onto the page. Coverage is checked against the glyph outlines, not the character map, before anything is drawn.
4. **Closed under overlap.** Anything painted after a changed object that overlaps it is redrawn too, in the original order, so stacking never changes.

### Running the tests

```bash
cd pdfedit
pytest
```

The suite builds its own test documents from four different producers (reportlab with an embedded TrueType subset, a hand-written content stream, a page printed by headless Chrome and a page printed by macOS, the last two committed under `tests/fixtures`) and asks every question of the finished PDF: is an untouched file the same bytes, is replaced text really gone, and did anything else on the page move. It needs no network, browser or Mac to run. The studio is tested through its sessions, its server (real HTTP requests to a server on a local port, including requests that must be refused) and its page (every element the script uses exists, nothing is loaded from outside). It was also driven in a real browser: open, retype a line, true preview, save, and the old text was gone from the saved file.

---

Docs mirror of the private repository Dominares-Tools/pdfedit; updated with each release.
