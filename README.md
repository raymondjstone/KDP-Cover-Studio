# KDP Cover Studio

A Windows desktop app for designing print-ready Amazon KDP covers — paperback or case laminate hardcover, back, spine and front as a single wrap — and exporting the PDF that KDP actually accepts.

**[Download the latest release »](https://github.com/raymondjstone/KDP-Cover-Studio/releases/latest)**

![The cover editor](docs/screenshots/01-editor.png)

---

## What it does

You set the binding, the trim size, the paper and the interior page count. The app derives the wrap geometry from those — spine width, bleed, safe areas, fold tolerance, hinges, barcode reserve — and draws it to scale. You drag artwork onto a panel, set the type, and export.

The geometry is not estimated. It is derived from Amazon's own cover template PDFs and verified continuously against a corpus of 22 of them, covering both bindings, five paper stocks and two trim sizes.

### Getting the spine right

`spine width = page count × paper caliper`. For KDP's black-and-white white paper that caliper is **0.002252 inches per page**, identical to seven decimal places across all 18 B&W-white templates in the corpus.

Several popular online spine calculators add a "+0.06 inch cover allowance". Amazon's own templates do not, and adding it shifts spine text visibly off-centre. This app does not add it.

The binding constraint on spine text is the **safe** width, not the spine width. A 200-page paperback has a 0.450" spine but only 0.325" you can actually set type in. The status bar shows both, always.

### Hardcover is a different shape, not a bigger paperback

![Hardcover geometry with hinges](docs/screenshots/02-hardcover.png)

A case laminate cover wraps rigid boards, so the printed sheet is larger than the book in every direction and gains a **hinge** each side of the spine where the case flexes. Nothing readable may go there — it is not merely unsafe like the spine's fold tolerance, it is a crease with no board behind it, in a known place, on every copy. The editor draws the hinges and preflight enforces them.

**KDP specifies hardcover in millimetres**, and that is not a presentational detail. Every distance came out of the template as a round metric figure — 15 mm of wrap, 10 mm of hinge, 5 mm of board inset, 6 mm of overhang. Expressing them as rounded inches instead put the spine fold out by 0.007 pt, which the acceptance gate caught.

One honest gap: the split of the spine between paper and boards is **inferred from a single template**, since one template is one equation with two unknowns. The app says so where you can see it, rather than presenting a derived figure as a measured one.

### Trim sizes have names

The default is 5.06" × 7.81" — the **UK B-format paperback**, 129 × 198 mm. The picker says so, because an author looking for a standard UK paperback is looking for "B-format", not for an inch figure that is an artefact of KDP being a US service. A5, UK Royal and A4 are named on the same basis.

Every dimension on screen carries both units: `0.450" (11.4 mm)`. There is no units toggle — a cover is designed against inch-denominated rules and printed to a paper size named in millimetres, so both numbers are wanted.

---

## Features

### Page count without guessing

| Source | Accuracy |
|---|---|
| Interior PDF | Exact, authoritative — read straight from the file |
| EPUB estimate | A **range**, never a bare number |
| Manual | Whatever you enter |

An EPUB does not contain the typography that determines page count, so an estimate carries ±20%. On a 300-page book that is ±0.135" of spine — more than double the fold tolerance. The app shows a range, keeps words-per-page editable and calibratable, and never lets an estimate look settled.

**A book over KDP's 828-page maximum can be split into volumes.** The splitter proposes page ranges and can build a series from them; it does not touch your manuscript, because where a book should break is an editorial judgement. Given an EPUB it splits at document boundaries — most EPUBs put one chapter in one document, so the breaks are real ones. Volumes come out deliberately uneven: a reader notices a chapter cut in half and never notices thirty pages of difference.

### Your own ISBN, as a real barcode

![A barcode rendered from an ISBN](docs/screenshots/04-barcode.png)

Only needed if you are using your own ISBN — KDP prints its own barcode for the free one.

A barcode is the one thing on a cover with a right answer that nobody can check by eye, so the tests **decode the rendered pixels back into digits** the way a scanner does and assert the ISBN comes out. Asserting that bars were drawn would pass on a transposed digit, an inverted parity pattern or a missing quiet zone.

| Rule | Why it is not a style choice |
|---|---|
| The check digit is recomputed, never trusted | It exists to catch a mistyped digit, and the failure is a barcode that scans cleanly and returns somebody else's book |
| ISBN-10 is validated **before** conversion | Converting first turns a typo into a valid ISBN-13 for a different book |
| Nothing is hyphenated, ever | Correct hyphenation needs the ISBN range table; guessing at fixed positions prints a plausible number grouped wrong |
| Bars drawn with **antialiasing off** | At 300 DPI a module is ~4 px, so a half-pixel of grey each side is a real fraction of a bar width |
| The white ground covers the quiet zones | They only exist as white. Cropping to "just the bars" is the commonest reason a printed barcode fails |
| Below 0.8× magnification is an **error** | A barcode has no spectrum of quality — it reads at the till or it does not |
| Not black on white is an **error** | Many scanners are red lasers, and red ink under red light is white |

### Design templates and series

![Templates picker](docs/screenshots/03-templates.png)

Six built-in designs, plus any you save yourself. Every preview shows the design **applied to your book** — its binding, trim size and page count, your artwork, your title — rendered through the same engine as the final PDF. A stock picture of a design cannot answer the only question worth asking.

Applying a template is a re-placement, not a copy: every layer is repositioned relative to its own panel. All six built-ins cover **all three panels**, unlike most cover templates in the wild, which are a front and nothing else — leaving you the two genuinely hard parts: spine type inside the fold tolerance, and back-cover copy clear of the barcode.

**Series mode** shares a binding, trim size, paper and design across volumes; only the title, artwork and page count differ — and the page count is what makes each spine a different width. Batch export refuses one volume and still writes the rest.

### Type over artwork, done properly

A **scrim** is a gradient fading from one edge to nothing — the professional answer to type over busy artwork, where the amateur answers are a hard-edged bar (looks stuck on) and a heavy outline (muddies letterforms at thumbnail size). The ramp is eased rather than linear, because a straight alpha ramp leaves a visible corner where it lands on zero.

A scrim is translucent, so it must sit **below** the type or the whole cover loses its vector text. Three things make sure it does: new ones are inserted below text, preflight warns by name if one ends up above type, and the properties panel says what it costs.

**Spine sections** let you place type in a third of the spine — and the thirds are thirds of the *safe* length, not the spine, because thirds of the spine run into the trim.

### Artwork at 300 DPI, automatically

Artwork short of KDP's minimum is resampled up to the pixels it needs at the size it is placed, and the enlargement comes back off when it is no longer needed. Always exactly one step from the original, and it stops at 4× and says the file is simply too small rather than inventing further. Preflight reports every enlarged layer, because resampling adds pixels, not detail.

Layers **snap to the guides** as you drag them — hold Alt to place something deliberately off a guide.

### Colour

Background, text, text outline and colour blocks are all editable, each with a picker, a hex box and an eyedropper. The hex box matters: a cover often has to match a series or an imprint's palette, and those arrive as `#1E6FD9`, not as a position on a wheel.

Each picker shows a **CMYK gamut estimate inline**, at the point of decision. The eyedropper samples the **rendered cover with guides off**, not the screen, so you cannot click your artwork and get the colour of a guide overlay. A magnifier loupe follows the pointer while it is armed.

A **palette** travels with the document rather than living in settings — it is what keeps volume three matching volumes one and two, and it has to survive the project being handed on. Eyedropper samples are kept automatically, since those are the colours with no other record anywhere.

### The shop-size preview

A pane showing the **front panel, trimmed, at about 40 mm** — the size a shop actually shows a cover, and the size at which you should judge it. Trimmed rather than with bleed, because the bleed is guillotined off before anybody sees it and including it flatters the design. It renders through the same exporter as the print file, so it cannot show you something kinder than what prints.

### Live preflight

Checked continuously, not just at export: artwork below 300 DPI at the size it is placed; spine text outside the safe width or below legible size; content in a hardcover hinge; content colliding with the barcode reserve; barcodes too small or not black on white; a scrim above the type; and **missing glyphs** — Skia substitutes fonts per *face*, never per glyph, so a face without an em dash draws `.notdef` and carries on. Raised as an error: there is no version of this that prints acceptably.

### Export

**Print PDF** — one page at the full wrap size, meeting KDP's cover rules:

| KDP rule | How it is met |
|---|---|
| Single PDF, back + spine + front combined | One page at the full wrap size |
| Flatten all transparencies | Split by z-order — see below |
| Embed fonts | Subset and embedded |
| No crop marks, colour bars or template text | Export builds its own options with guides off; callers cannot enable them |
| Minimum 300 DPI | Clamped up to 300 even if something asks for less |
| Optimise file size | Artwork as JPEG, text as vector — typically ~300 KB, not tens of MB |
| No file security | Nothing encrypts the output |

Transparency is flattened **without losing the type**. Rasterising the whole cover would flatten it and also destroy the text — at 300 DPI a 10pt spine title is ~42 px, permanently. Instead the stack is split by z-order: everything from the first non-opaque layer downwards composites into one opaque JPEG, and opaque text above that is drawn as real PDF text on top.

*Worth knowing:* a translucent layer at the very top of the stack forces the whole cover to be flattened. If a design loses its sharp type, look for a translucent element sitting over everything.

**Ebook cover JPEG** — a standalone front cover at the aspect ratio the store wants, with each option explained in terms of what it costs on this book.

**Proof checklist** — written for a **bound** copy, which is what KDP sends. Half the useful measurements only exist once the thing is folded.

### Your work is kept safe

Autosaves go to `%LOCALAPPDATA%\KDPCoverStudio\recovery\`, **never to your project file**. An autosave that writes into the project destroys "close without saving" — which is how people back out of an experiment, and they only find it is gone after relying on it. If the app closes unexpectedly you are offered the copy back, and you can decline. Recovered covers open with no file path, so a reflexive Save cannot overwrite the on-disk version with a different cover.

The copy is a full package including artwork, because autosaving just the document recovers a cover with every image missing — which *looks* like it worked.

A log is written to `%LOCALAPPDATA%\KDPCoverStudio\logs\`, readable while the app runs.

### Help that is drawn to your book

![How a KDP cover works](docs/screenshots/05-how-covers-work.png)

The anatomy diagram is rendered from your cover's own geometry. A 90-page paperback shows a spine sliver captioned "usable 2.0 mm"; a 700-page one shows a spine wide enough to design into. Amazon's illustration shows one example and always will.

---

## Installing

1. Download `KDPCoverStudio.msi` from the [latest release](https://github.com/raymondjstone/KDP-Cover-Studio/releases/latest).
2. Run it. It installs to Program Files for all users, adds a Start menu shortcut, and registers the `.kdpcoverstudio` file type so projects open on double-click.

**Requirements: 64-bit Windows 10 or 11. Nothing else.** The installer is self-contained — the .NET runtime and all native libraries ship inside it.

Windows SmartScreen may warn on first run, as it does for any installer that is not code-signed. Choose *More info* → *Run anyway*.

The app follows your Windows light/dark setting, or you can pin either. It can check GitHub once a day for a newer release — it never downloads or installs anything, it tells you and you decide — and the setting says in plain words that it contacts github.com. Turn it off and nothing is sent.

### Project files

Projects are saved as `.kdpcoverstudio` — a ZIP holding the document plus a copy of every image used, named by content hash. Your artwork is copied in, so a project survives its source images being moved, renamed or deleted, and it is one file to back up or send. The format is detected from the file's contents, not its extension, so renamed files still open. Projects from every earlier version load unchanged.

---

## Current release

**Version 1.3.0** — `KDPCoverStudio.msi`, 40.9 MB. See the [release notes](https://github.com/raymondjstone/KDP-Cover-Studio/releases/latest) for what changed.

Verified at build: 547/547 tests and 220/220 template assertions.

### Known limitations

- **Colour is sRGB; KDP converts to CMYK.** The in-app gamut check is an estimate, deliberately generous, and is not a soft proof — that needs the printer's ICC profile.
- **The hardcover spine allowance is inferred from one template**, not measured. A second case laminate template at a different page count would settle it. The app flags it as unverified.
- **Only the black-and-white white paper caliper is corpus-verified.** Other stocks use Amazon's documented figures, and the app tells you which is which.
- **No proof has been physically printed.** Everything is verified numerically against Amazon's templates, but the final check on colour and on how the spine fold actually lands is a printed copy.

---

## Built with

C# · .NET 10 · [Avalonia 11](https://avaloniaui.net/) · [SkiaSharp](https://github.com/mono/SkiaSharp) · WiX v5

One renderer draws the editor, the shop-size preview, the raster export and the print PDF — they differ only in the canvas they are handed, so the preview cannot drift from what the printer receives.
