# Concrete Companion v1.0 — Concrete Site Inspection Companion

A installable, offline-capable PWA covering reinforcement reference/calculators and concrete
inspection tools, referenced to **AS 3600:2018**. Designed for one-tap access from an NFC tag.

> Folder/repo name can stay `sitecheck-rc` (or whatever you already created) — only the
> in-app name and icon changed, so your existing GitHub Pages URL and NFC tag still work.

## What's new in this version — PDF import bug fix

Fixed "Could not load that PDF. Buffer is already detached" when importing a PDF. pdf.js hands
the file's bytes to a Web Worker for parsing, and browsers **transfer** (not copy) that memory to
the worker for performance — which leaves the original buffer unusable in the main thread
afterward. The code was reading that same buffer again right after, to keep a copy of the original
PDF for native-size export, which is exactly what broke. Now the original bytes are captured
first, before pdf.js gets anywhere near them, and pdf.js is given an independent copy instead of
the original. Verified directly by genuinely detaching a buffer the same way a real browser's
worker transfer does, and confirming the captured copy still reconstructs the exact original file
byte-for-byte afterward — not just re-running the import and hoping.

## What's new in the previous version — Phase 2: full annotation toolset + native PDF markup

**Site markup now has a proper annotation toolset**, matching what was scoped for Phase 2:

- **Pen, Highlighter, Eraser, Text, Pin, Rectangle, Circle, Line, Arrow** — nine tools, all sharing
  the same colour palette (now 5 colours) and canvas. Highlighter draws a wide, semi-transparent
  stroke; Eraser removes whichever strokes/shapes/text/pins it touches.
- **Proper Undo/Redo** — a real action stack (not just "undo last stroke"). Every add and every
  eraser pass is one undo step; redo replays it. This applies per photo/drawing/PDF page.
- **Text tool** places a small on-canvas text box wherever you tap, independent from the numbered
  Pin tool (which keeps its own note-list legend underneath, for issue-tracking style markup).

**PDF import, with true native-size export** — the bigger piece of Phase 2:

- **"+ Add PDF"** alongside "+ Add photo/drawing". Pick a PDF (e.g. an exported drawing sheet) and
  every page is imported as its own markup entry, annotatable with the full toolset above.
- On export, native PDF pages are **not** redrawn onto an A4 page — the original page (A4, A3, A2,
  A1, whatever size and orientation it was) is copied through byte-for-byte, with your annotations
  burned onto a copy of it at the correct scale and position. An A3 landscape sheet exports as an
  A3 landscape page, not squashed onto A4.
- Any numbered pins on a PDF page get a "Markup notes" legend page appended straight after it
  (same pattern as the existing photo markup notes), since there's nowhere on a live engineering
  drawing to write a paragraph of note text.
- Under the hood this uses two more libraries bundled locally for offline use: **pdf.js** (renders
  PDF pages so you can see and mark them up) and **pdf-lib** (merges the final document while
  preserving each page's exact original size). Total app size is now ~3.5 MB — still a one-time
  download, fully cached for offline use afterward.

**Signature upload bug also fixed as part of this pass**: PNG uploads went through the same
JPEG-flattening step as regular photos, silently destroying transparency — signatures now use a
dedicated PNG-preserving path.

**What's still not covered** (flagged rather than glossed over):
- Only the specific page(s) you import become markup entries — there's no in-app "browse a
  multi-hundred-page drawing set and pick one sheet" viewer; export the relevant sheet(s) to PDF
  first, the way the FDC example workflow already does.
- Shapes and text don't support resize/move-after-placement yet — delete (via eraser) and redraw
  if you need to adjust one.
- No pinch-zoom while annotating; you're working at the canvas's fitted display size. Fine for
  typical phone/tablet screens, but very fine linework on a dense A1 sheet may be easier to place
  precisely at a larger physical screen size.

## What's new in the previous version — Disclaimer flow + sidebar shape

The two known gaps from the last round are closed:

- **Disclaimer paragraph** now flows as one continuous wrapped paragraph exactly like the
  template — the bold-italic lead sentence ("Site safety remains...") runs directly into the
  plain-italic body text, instead of sitting on its own line above it. jsPDF has no built-in
  support for mixed-style text wrapping, so this required a small manual word-by-word layout
  routine (measures each word at its own style, packs them onto lines, draws each word in its
  correct style at the right position) — verified by OCR'ing the rendered PDF (confirms it reads
  as one paragraph) and by measuring ink density (the lead sentence is ~40% denser than the rest,
  confirming it's actually bold and not just visually similar).
- **Sidebar shape** now has the rounded top-right corner from the template, instead of a plain
  rectangle. jsPDF's built-in rounded-rectangle only supports rounding all four corners equally,
  so this is built from a manual path (straight edges + one bezier-curve corner). Verified by
  scanning the actual pixel boundary of the rendered corner to confirm it's a smooth curve, not a
  square corner.
- Added the extra bold-italic weight of your brand font (Figtree SemiBold Italic) needed for the
  disclaimer's lead sentence — four font files are now embedded in exported PDFs in total.

## What's new in the previous version — Header overlap bug fix

Found and fixed the cause of the missing address / "100 Ebley Street": the recipient block's
left column (addressee/company/address) had no width limit, so a long address was drawn straight
across the page with no wrapping — it visually collided with the date/job number/project name in
the right column, garbling both into unreadable overlapping text. That's why the project address
appeared to vanish. The left column now wraps properly within its own space, confirmed with a
real typed-input-through-to-PDF test (not just pre-set data), plus OCR on the rendered output to
verify the exact text landing correctly with no overlap.

## What's new in the previous version — Template fidelity pass

The PDF export was rebuilt to match your actual Northrop template precisely, instead of an
approximation:

- **Real template colours**: exact cream table shading (`#FFF2DF`) and rule-line grey (`#B0B0B0`)
  pulled directly from your `.docx`, not eyeballed.
- **Real brand font**: your template's actual typeface (Figtree) is now embedded and used
  throughout the PDF — previously it silently fell back to a generic system font.
- **Unified table layout**: distribution table, report info, and signature are now one continuous
  bordered table (cream label shading, thin grey rules between rows, no vertical grid lines) —
  matching the template exactly instead of being three separate disconnected blocks.
- **Signature moved into the table**: there's now a single SIGNATURE row in that table (matching
  the template), instead of a second signature floating near "Yours sincerely,".
- **Checkboxes**: now drawn as an outlined box with an X mark when checked, matching the
  template's ☒/☐ style, instead of a plain filled/empty square.
- **Report No. / Rev** moved out of the sidebar into its correct row in the info table.
- **Signature bugs fixed**:
  - Uploaded PNG signatures were silently being flattened to JPEG, which destroyed transparency
    (any background behind the signature showed as a solid block). Uploads now stay PNG end to
    end, so transparency is preserved.
  - Both drawn and uploaded signatures are now measured and scaled to fit the signature box
    without distortion, cropping or stretching, and land in the same position regardless of
    which method was used.

**Known remaining gaps** (flagged rather than silently glossed over):
- The disclaimer paragraph's bold lead sentence now sits on its own line above the italic body,
  rather than flowing inline as one paragraph like the template — true mixed bold/italic
  word-wrapping in a single paragraph needs more involved manual text layout than a straight port
  could justify in this pass. Say the word if you want that tightened up.
- The sidebar's rounded-corner shape is still a plain rectangle rather than the template's
  rounded-top pill shape.

## What's new in the previous version — Phase 1: Site Inspection Report

The **Checklist** tab is now **Report** — a full Northrop-style Site Inspection Report builder:

- **Report header**: addressee, company, address, project/job address, date, job number, and a
  new Report No. / Rev field.
- **Distribution table**: To/Copy checkboxes, company & attn per recipient, add/remove rows freely.
- **Report information**: site visit requested by, reason for visit (auto-fills from the
  location/grid reference field until you edit it directly), inspector, sent via.
- **Observations** (formerly the checklist): statuses are now **Observed / Attention Required /
  Hold** instead of Pass/Fail/Hold. Deleting an item is now instant with an **Undo** toast instead
  of a confirmation dialog. Each item can be marked **"Hide from report"** — it stays visible in
  the app but is left out of the PDF/text export. Adding or removing an item no longer jumps you
  back to the top of the page.
- **Report body**: a rich text editor (bold/italic/underline, bullet & numbered lists, undo/redo)
  for the inspection write-up, under an auto-generated "Dear [Addressee]," line.
- **Sign-off**: choose a closing phrase, name/position/company, and a signature — drawn on-screen
  or uploaded as an image.
- **Settings** (gear icon, top right): save your name, position, company and signature once — it
  pre-fills every new report, and can still be edited per-report without changing what's saved.
- **Export PDF report** now generates a branded, Northrop-letterhead-style cover sheet (logo,
  contact sidebar, distribution table, report info, body, sign-off, disclaimer) followed by the
  observations and any site markup — built directly from your uploaded template.

**What's next (Phase 2, not in this version):** upgraded annotation tools (highlighter, eraser,
text boxes, shapes) and true PDF import/markup/export at native page size for large drawings —
the current markup tool still works on photos/images only, downscaled on import.

## What's new in the previous version
- **Ast Check** now accepts reinforcement as either "bar size @ spacing" or "number of bars"
  (e.g. `2/N20`), and the comparison target can be a raw Ast value **or** a design bar size &
  spacing/count — matching how reinforcement is usually called up on drawings.
- **Spacing Calculator** now takes its target as a bar size & spacing by default (Ast is derived
  automatically), with the option to switch back to a raw Ast value.
- **Light theme is now the default** (better for direct sunlight), with a toggle in the header to
  switch to dark. Your choice is remembered on the device.
- **Pre-pour checklist** now supports adding your own custom items per section, and **deleting
  any item — built-in or custom** (built-in ones can be restored via a "Restore removed items"
  button per section).
- **Site markup** — attach a photo or drawing, mark it up with freehand lines (pick from 4
  colours) and numbered pins with notes, and it's automatically included in a combined
  **PDF report** alongside the checklist.

## What's in this folder

```
index.html                    the whole app (HTML/CSS/JS, no build step)
manifest.json                  PWA manifest (name, icons, theme colour)
sw.js                           service worker — caches the app shell for offline use
vendor/jspdf.umd.min.js         builds the cover sheet/observations PDF, bundled locally
vendor/pdf.min.mjs               pdf.js — renders imported PDF pages for markup, bundled locally
vendor/pdf.worker.min.mjs        pdf.js's worker (required alongside it)
vendor/standard_fonts/           pdf.js's fallback fonts, for PDFs that don't embed their own
vendor/pdf-lib.min.js            merges the final PDF while preserving native page sizes
vendor/Figtree-*.ttf             your template's actual brand font, embedded in exported PDFs
assets/logo.png                 Northrop logo, used in the PDF cover sheet
assets/brand-mark.png            Northrop circular mark, used in the PDF sidebar
icons/icon-192.png
icons/icon-512.png
icons/icon-512-maskable.png
```

Everything runs client-side. There's no server, database, or account — it just needs to be
**hosted somewhere reachable over HTTPS** (required for service workers / installability), and
opened once online so the service worker can cache it.

## 1. Host it (pick one — all free)

**GitHub Pages** (recommended, simplest to keep updating):
1. Create a new GitHub repo, e.g. `sitecheck-rc`.
2. Upload all files in this folder, keeping the `icons/` subfolder.
3. Repo Settings → Pages → Deploy from branch → `main` / root.
4. Your app will be live at `https://<username>.github.io/sitecheck-rc/`.

**Netlify / Vercel (drag-and-drop):**
1. Sign in, choose "deploy manually" / drag-and-drop deploy.
2. Drag this whole folder in.
3. You'll get a URL like `https://sitecheck-rc.netlify.app`.

**Your own server / company intranet:**
Just copy the folder to any static file host — it's plain HTML/JS, no build step required.

## 2. Install it as an app (once hosted)

- **Android (Chrome):** open the URL → menu → "Install app" / "Add to Home screen".
- **iPhone (Safari):** open the URL → Share → "Add to Home Screen".
- After installing once online, the app shell (tables, calculators, checklist) works **offline**
  — useful on sites with poor signal. Checklist entries are stored on-device (localStorage) and
  aren't shared between devices.

## 3. Write it to an NFC tag

Use any NFC-writing app (e.g. **NFC Tools** on Android/iPhone):
1. Open NFC Tools → Write → Add a record → **URL / URI**.
2. Enter your hosted URL, e.g. `https://<username>.github.io/sitecheck-rc/`.
3. Write to the tag, hold your phone to it once to confirm.

Tapping the tag will open the installed app directly (or the site in-browser if not yet
installed — from there the person can install it themselves).

**Sticker placement tip:** laminate/heat-shrink the tag and mount it somewhere it'll survive a
site environment — inside a site office door, on a tablet/hard-hat case, or on a laminated card
in the site folder.

## Site markup & PDF report

On the **Report** tab, in the Site markup card:
1. Tap **"+ Add photo/drawing"** for a photo (camera or gallery) or scanned drawing image, or
   **"+ Add PDF"** to import a PDF sheet — every page in the PDF becomes its own markup entry.
2. Pick a tool: **Pen**, **Highlighter**, **Eraser**, **Text**, **Pin**, **Rectangle**, **Circle**,
   **Line**, **Arrow** — pick a colour first for anything except Eraser. **Pin** is separate from
   **Text**: pins are numbered and get a note field in the list underneath (good for issue
   tracking); Text places a label directly on the drawing.
3. **Undo**/**Redo** step through your annotation history; **Clear** wipes all markup on that item.
4. Add as many photos/drawings/PDF pages as the inspection needs; each has its own caption field.
5. Tap **Export PDF report** — generates one PDF: project details, the full observations list,
   every photo/drawing markup, and every PDF page **at its original size** with annotations burned
   in (an A3 sheet stays A3, not squashed onto A4). Downloads straight to the device — no internet
   needed once the app's been opened online at least once (everything's bundled in `vendor/`).

**Export text** (below the PDF button) gives a quick plain-text copy of just the observations, for
pasting into an email or chat — it doesn't include markup images or PDF pages.

Photos and PDF page previews are automatically scaled on import to keep on-device storage
reasonable — fine for a report, not intended as a high-resolution archive. The original PDF bytes
are kept alongside so the *exported* page is still full quality at its native size regardless.

## Updating the content later

Edit `index.html` (all the reference data lives in the `BARS`, `SQUARE_MESH`, `RECT_MESH`,
`COVER_TABLE`, `COVER_ENV`, `DEFECTS` and `CHECKLIST_GROUPS` arrays near the top of the
`<script>` block) and re-upload. Bump `CACHE_NAME` in `sw.js` (e.g. `sitecheck-rc-v2`) so
installed devices pick up the change next time they're online.

## Important — verify before relying on this in the field

This tool is a **field reference and QA aid**, not a design tool or a substitute for the project
structural drawings and specification:

- Bar/mesh areas are per AS/NZS 4671 (D500N) and the reinforcement reference table you supplied.
- Cover values are simplified from AS 3600:2018 Section 4 / Table 4.10.3.2 for common cases —
  always confirm exact exposure classification and nominal cover against the project durability
  specification and structural drawings.
- Development length and lap length calculators implement the **basic** (unrefined) formulas of
  Cl 13.1 and Cl 13.2 for straight D500N bars only. They do **not** cover hooked/cogged bar
  reductions, the transverse-steel refinement (k4/k5), bundled bars, or epoxy-coated/lightweight
  concrete multipliers — for those cases, or any non-standard condition, refer to the project
  engineer.
- The current uploaded copy of AS 3600:2018 could not be read when this app was built (only the
  reinforcement area reference table was accessible), so clause values were sourced from general
  engineering knowledge and cross-checked against multiple independent references rather than
  transcribed directly from the standard. **Cross-check the cover table and clause formulas
  against your own copy of AS 3600:2018 (with current amendments) before relying on this for
  sign-off**, and treat this as a fast reference/estimate tool rather than a certified
  calculation record.
