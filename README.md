# IL-2 Korea Tools

Two self-contained, offline HTML tools for IL-2 Sturmovik: Korea. Both run
entirely in your browser — nothing is uploaded, and no internet connection is
needed once the page is open.

| Tool | File | What it does |
|---|---|---|
| Graphics Configurator | `index.html` | Edits the `[KEY = graphics]` section of `startup.cfg` |
| Career File Viewer | `career.html` | Reads a pilot career `.db` and decodes it |

---

## Graphics Configurator — `index.html`

Edits the `[KEY = graphics]` section of an IL-2 Korea `startup.cfg` file, laid
out to visually match the in-game Graphics Settings menu, including the
Extended & Hidden Settings section.

1. Open `index.html` in any modern desktop browser (Chrome, Edge, Firefox).
2. Load your `startup.cfg` file (usually in your IL-2 Korea installation's `data` folder).
3. Adjust settings using the same layout and options as the in-game menu.
4. Save the file back out — only the graphics section is modified; every other
   section of your `startup.cfg` is preserved byte-for-byte.

---

## Career File Viewer — `career.html`

Opens an IL-2 Korea career file and presents it as a readable career record:
pilot and squadron overview, full roster, kill board and victory log, mission
log with briefings, and a decoded career timeline.

Career files live in your installation's `data\Career` folder, one `.db` per
pilot. They are plain, unencrypted SQLite databases — the viewer opens them
with [sql.js](https://github.com/sql-js/sql.js), a build of SQLite compiled to
WebAssembly.

**This tool is read-only.** It never writes to your career file.

### Requires two files

`career.html` needs `sql-wasm.wasm` sitting next to it. Download both, keep them
in the same folder, and open `career.html`.

### Opening it from disk

Browsers will not let a page loaded over `file://` fetch its own WebAssembly
engine. So on first run from disk, the page asks you to point at `sql-wasm.wasm`
once — it is remembered from then on. To skip that entirely, serve the folder
over http instead:

```bash
python -m http.server
```

then open <http://localhost:8000/career.html>. The GitHub Pages link below has
no such step.

### What it can and cannot tell you

Most of the file decodes cleanly. The career format stores a lot of meaning in
opaque integers and in URL-encoded `key=value` blobs packed inside text columns;
all of that was worked out by cross-referencing five complete career files, and
every pilot in those five reconciles — their stored kill totals match the
independent sum of their own sortie records.

Where something could not be established from that evidence, it is shown with a
dotted underline and its raw value on hover, rather than guessed at silently.

Two things genuinely cannot be recovered from a career file at all:

- **Medal names.** Awards are stored as bare IDs. They look country-prefixed
  (`502002` in PLAAF careers), but the pattern breaks — PLAAF careers also carry
  `503002` and `601980` records — so the viewer only names the air force when
  the prefix matches the pilot's own country.
- **Rank names.** `rankId` clearly tracks seniority, but only as a number.

The display strings for both live inside `Interface.gtp`, a ~1.7 GB packed
archive the game never unpacks to disk.

---

## Live version

Both tools are served by GitHub Pages — nothing to download, nothing to set up:

- <https://ddrake1984.github.io/IL-2-Korea/> — Graphics Configurator
- <https://ddrake1984.github.io/IL-2-Korea/career.html> — Career File Viewer

## Repository

<https://github.com/ddrake1984/IL-2-Korea>

To run them locally instead, clone the repo (or download the files you want) and
open the HTML. The Career File Viewer needs `sql-wasm.wasm` kept alongside
`career.html`; the Graphics Configurator is a single file on its own.
