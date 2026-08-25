# IL-2 Korea Tools

Self-contained, offline HTML tools for IL-2 Sturmovik: Korea. Each runs entirely
in your browser — nothing is uploaded, and no internet connection is needed once
the page is open.

| Tool | File | What it does |
|---|---|---|
| Landing page | `index.html` | Index of the tools below |
| Graphics Configurator | `graphics.html` | Edits the `[KEY = graphics]` section of `startup.cfg` |
| Career File Viewer | `career.html` | Reads a pilot career `.db` and decodes it |
| Task Editor Missions | `taskeditor.html` | Edits a mission saved in the in-game Task Editor |
| Cooperative Converter | `coop.html` | Turns a generated mission into a co-op multiplayer one |

---

## Graphics Configurator — `graphics.html`

Edits the `[KEY = graphics]` section of an IL-2 Korea `startup.cfg` file, laid
out to visually match the in-game Graphics Settings menu, including the
Extended & Hidden Settings section.

1. Open `graphics.html` in any modern desktop browser (Chrome, Edge, Firefox).
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

## Task Editor Missions — `taskeditor.html`

Edits a mission built and saved in the game's in-game **Task Editor**. Those are
stored as small JSON files in

```
data\NSData\UserData\QuickMission\Presets\<mission name>_<date>_<time>.json
```

and they hold the design you authored — mission type, the friendly and enemy
formations, weather and time — rather than the generated mission itself. Editing
one here and loading it back in the Task Editor works normally; this was verified
by round-tripping a real mission through the game.

### What it edits

**Time, season and weather**, through the same strip the Task Editor puts along
the bottom of its screen: one button per setting showing its current value, and a
parchment panel above whichever you click — a slider for time in ten-minute steps,
the five season icons, wind speed and direction together, a list for cover and
cloud type. It reads as the game reads because a row of dropdowns bears no
resemblance to the thing being edited.

The mission's **name** is editable too. Everything else — formations, aircraft,
counts, AI levels, loadouts — is **read only**. Those interact with mission type in
ways that are not worth guessing at from outside the game, so the editor shows what
is set and leaves changing it to the Task Editor.

Formations are laid out as the game lays them out, in two columns that never
interleave: **Friendly** and **Enemy**, each formation one card holding however
many aircraft groups it has under a single start altitude, plus distance and
approach where the mission sets them. Approach reads *Face to face*, *Pursuit*,
*Escape*, *Aside* or *Scramble*, and a start altitude of `0` reads **On runway**,
which is what it means — not ground level.

Side and nationality are shown separately, because they are separate things — a
Skirmish will happily put DPRK aircraft on the side opposing you. The column says
which side an aircraft is on; a small red or blue badge on each aircraft says which
nation it belongs to. Colouring the card by nationality alone would put a red card
in the enemy column and read as a mistake.

The Advanced panel puts one column per formation group — `F1` your own flight, `F2`
a supporting one, `E1` and `E2` the opposing sides — with the loadout values listed
down the side.

### Values that do not mean what they look like

- **Wind direction is stored as the reciprocal of what you see.** A mission the
  Task Editor displays as `150°` holds `330` in the file. The editor shows you the
  in-game figure and converts on save.
- **Cloud type puts Random at −1, not at the top of the list.** The menu shows
  *Random* first, which makes it look like entry zero, but missions displaying
  Random store `-1` while a mission storing `0` displays as *Cumulus humilis* — the
  first named sky. So the ten real skies are 0–9. The generator then writes the
  chosen index into the mission's `CloudConfig` path, which is why
  `summer\00_Clear_00\sky.ini` is cover 0, variant 0.

- **Pilot AI level uses the same −1 for Random.** The named grades run 1–4 as
  Novice, Average, Veteran and Ace — Ace confirmed against a mission showing Ace
  selected while storing `4` — and a formation set to Random stores `-1`.

Two more values are named only as far as the evidence goes. `intercept_bombers`
is the one `subType` confirmed against a saved mission, and the rest of the menu
follows the same snake_case pattern. The three types with no submenu — Free
Flight, Duel and Skirmish — all store `subType: "default"` and are told apart by
`missionType` instead, of which only `701` (Skirmish) has been matched to a
screen; the others are reported as their number rather than guessed at.

Raw values never appear on screen next to the name they belong to. Where one
exists, the name carries a small **(i)** — as the graphics configurator does —
that opens a panel listing exactly what the file stores.

### What it does not touch

Everything outside the weather bar and the mission name is written back exactly as
it came in — formations, gun loadouts, belt choices, ammunition schemes, fuel, the
`content` array, the map coordinates. The tables that would give the loadout
numbers meaning live inside the packed `.gtp` archives, so there is no way to know
from disk which values are legal for a given aircraft.

The only difference between a file in and a file out is JSON number formatting:
`2.0` is rewritten as `2`. The game loads that without complaint.

### Where the reference data came from

Country names and the 62 callsigns with their country restrictions were read out of
`DefaultCountryNames.eng` and `DefaultCallsigns.eng` in `data\GUI`; the aircraft
roster from `EditorObjectsBlackList.cfg` alongside them. The season, turbulence,
cover, cloud type, pilot AI and mission type menus are transcribed from the Task
Editor's own screens. Nothing in this tool's tables is guessed — a value that could
not be named is shown as its number instead.

---

## Cooperative Converter — `coop.html`

Build a mission in the in-game Task Editor and fly it once; the game writes the
result to `data\Missions\_gen.Mission`. This turns that into a cooperative
multiplayer mission in `data\Multiplayer\Cooperative\`.

Point it at your **data** folder and it finds the mission and its language files,
and writes the converted set back — no moving files around by hand. Opening a
single `.Mission` file works too, but the briefing can't come along that way.

### What it changes

Almost nothing, which is the point. On a real 21 MB generated mission it rewrites
a handful of lines out of 1,121,380:

- `MissionType` → `1` (cooperative)
- one `MultiplayerPlaneConfig` line per flyable aircraft type
- `CoopStart = 1` on each aircraft that becomes a player slot, and back to `0` on
  any that no longer is

`AILevel` is deliberately left alone — it is the AI skill setting and the mission
already carries whatever the Task Editor asked for.

Converting the same file twice produces a byte-identical result, so re-running it
after changing your mind about which flights are flyable is safe.

### Which mission is this?

`_gen.Mission` is the same anonymous filename every time and the game overwrites
it on every flight, recording nothing about which Task Editor mission produced
it. So the converter works it out by fingerprinting the generated file against
your saved missions in `data\NSData\UserData\QuickMission\Presets`: your
aircraft type and flight size, the clock, the sky, the cloud altitude, and each
enemy formation that turned up as expected. Saving shortly before the mission was
generated counts as corroboration, never as proof on its own.

It names the mission it found, lists the evidence, and uses that name for the
output file. If nothing scores clearly — or if two saved missions tie, which they
will when they differ only in ways the generated file can't show — it says so
and lets you pick from a list rather than putting a confident wrong name on your
work.

**You can only convert the mission you flew most recently.** The game overwrites
`_gen.Mission` on every flight, so having four saved missions does not mean four
missions available to convert — only the last one flown exists on disk. Naming
this file as one of the others does not make it that mission, so the picker
checks: assert a mission whose formations aren't in the file and it says which
ones are missing and tells you to go fly that mission first, instead of quietly
renaming the output.

Knowing which mission it is also reveals **which flights you actually placed**, and
those are the only ones listed — the generator's background traffic is folded away
behind a toggle, since a mission with three formations lands as ten flights on
disk. A formation you dialled down to zero correctly produces no flight at all. A
generator flight you tick stays visible even when the rest are hidden.

If the mission can't be identified, every flight is listed instead, because there
is then no basis for calling any of them yours.

### Choosing the flights

Your own flight is found by the `Desc = "PLAYERSQUAD"` marker the generator puts
on every aircraft in it, and is ticked by default. Every other flight is listed
too — the generator surrounds an authored mission with a lot of extra traffic, so
a mission with three formations in the editor can land as ten flights on disk.

Ticking a flight on the *other* side gives you a two-sided co-op. That isn't a
hack; the Korea installation's own `_test_cooperative_basic.Mission` is built
exactly that way, with four flyable aircraft at country 501 and four at 601.

### Reading the summary

The header decodes the values rather than showing them raw, because the raw ones
mean nothing on their own:

- **Sky** comes from `CloudConfig`, whose middle path segment is already
  `<index>_<name>_<variant>` — `summer\00_Clear_00\sky.ini` is simply "Clear".
- **Mission type** is `MissionType`, confirmed against every mission shipped with
  the game: `0` single player, `1` cooperative, `2` dogfight, and `703` / `1101`
  for the Quick Mission and Task Editor generators respectively — the same values
  a saved Task Editor mission stores.
- **Flights and aircraft** count what your mission placed, with the generator's
  additions shown separately. A three-formation, fourteen-aircraft mission
  typically lands as ten flights and fifty aircraft, and reporting the second pair
  as though they were yours is just noise.

### Cleaning up

Converting again overwrites `.Mission` and the language files but removes nothing,
so leftovers accumulate: a language file from a run when you had different
languages, or the `.msnbin` and `.list` the resaver produced from an older
version. The game keeps reading all of it.

So the converter lists what is actually in `Multiplayer\Cooperative`, grouped by
mission, with every extension each one owns. If files already under your chosen
output name would survive the conversion untouched, it names them before you
convert. Deleting a mission removes its whole set, and asks first with the list in
front of you.

`StartType` also differs between generated and shipped co-op missions, but
uniformly across every aircraft in the file — that is the mission's air start, not
a co-op requirement, so it is left alone.

### The one manual step

The converter writes the `.Mission` and the language files, which is what the game
reads. It cannot produce the `.msnbin` or `.list` companions: the `.list` hashes
are not CRC-32, Adler-32 or FNV-1a (tested against the raw bytes, BOM-stripped and
re-encoded as UTF-8), so there is no honest way to generate one in a browser. The
game ships the tool that does — `bin\resaver\MissionResaver.exe`. Try the mission
without it first; the shipped Korea co-op example has no `.msnbin` at all.

---

## Live version

GitHub Pages serves the tools directly — nothing to download, nothing to set up:

- <https://ddrake1984.github.io/IL-2-Korea/> — index of all tools
- <https://ddrake1984.github.io/IL-2-Korea/graphics.html> — Graphics Configurator
- <https://ddrake1984.github.io/IL-2-Korea/career.html> — Career File Viewer
- <https://ddrake1984.github.io/IL-2-Korea/taskeditor.html> — Task Editor Missions
- <https://ddrake1984.github.io/IL-2-Korea/coop.html> — Cooperative Converter

## Repository

<https://github.com/ddrake1984/IL-2-Korea>

To run them locally instead, clone the repo (or download the files you want) and
open the HTML. The Career File Viewer needs `sql-wasm.wasm` kept alongside
`career.html`; the Graphics Configurator is a single file on its own.
