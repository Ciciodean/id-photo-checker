# MonochromeVibe AI

A fully **client-side** tool that pre-checks a **US** driving licence, state ID, passport, or
permanent resident (green) card photo before you submit it for identity verification. It runs
**entirely in your browser** — your document is never uploaded or stored anywhere.

> **Scope:** US documents only, checked **by upload or live camera**. There is **no selfie and
> no face scanning** — this tool is purely about the document. The verifier's own selfie/liveness
> step happens later, on their side.

Everything lives in one self-contained file: **`id-photo-checker.html`**. There is no
backend, no database, no build step.

## What it does

A **Persona-style verification flow**, run entirely on-device — **for the document only**:

- **Guided live capture** — a frame overlay plus a live guidance prompt and **auto-capture**
  when the shot is sharp, evenly lit, and glare-free (meters update ~4×/second).
- **Upload or live camera** — front + back of a US licence/state ID or permanent resident (green)
  card, or the photo page of a US passport. (No selfie / face slot.)
- **Verification report** — a formal result page styled like a product: a decision banner
  (**Ready to submit / Review needed / Not ready**), **three stage cards** (Capture → Extract
  → Validate), an **on-device confidence score**, and a summary of pass/review/block counts.
- **Grouped checklist breakdown** — under the summary, every check is bucketed by status with a
  count per group: **Blockers** (fail), **To review**, **Passed**, **Pending** (not checked yet),
  and **Verifier side** (decided by a human or the issuing state). Each blocker / to-review row
  keeps its specific reason inline, so you can triage at a glance without losing the why.
- **Actionable view by default** — a "Show" filter sits above the report and defaults to
  **"Blockers & to review"** so you see only what needs action; anything that passed is hidden.
  Switch to **"Everything"** to see the full list again. The **"Not ready — will be refused"** decision
  banner is its own sticky panel at the top of the report, so it stays in view while you scroll the checks.
- **Read-the-text first** — the **"Read the text"** OCR panel appears **at the top** of the results
  section, ahead of the "Will this get through?" verdict, so the raw extracted strip / barcode data
  is the first thing you see.
- **Check sequence table** — the exact sequence a verifier runs: capture quality, document
  classification, data extraction (PDF417 + MRZ), data validation (ICAO 9303 check digits,
  AAMVA/REAL ID fields, expiry, age, format), and cross-side consistency.
- **Extracted-data review** — the name, DOB, document number, expiry, state and sex it read,
  so you can eyeball it before submitting.
- **Configurable rules** — toggle which checks are required (capture, data decode, expiry,
  minimum age).
- **Data censoring** — mask sensitive fields (numbers, dates) in the on-screen report and export.
- **Export report** — download a standalone HTML report with the decision and every check.
- **How a genuine US document looks** — a real reference library for driving licences, state IDs,
  passports, passport cards and green cards: dimensions, where the data physically lives,
  validity, number format and the genuine-print security features to look for. Once a document
  is identified it is shown as the reference, with the structure compared against real format
  expectations (issuer code, number length, A-number / green-card number, document type code).
- **Genuineness / authenticity context** — image-forensics signals for recapture/editing, plus an
  honest statement of what a browser genuinely can (structure + machine-readable data) and cannot
  verify (holograms, UV ink, chip, issuing-authority records).
- **Per-issue fix guidance** — each flagged reason is listed with how to fix it, inline in the report.
- **Cross-field intelligence** — a "detective" layer that compares each field against an *independent*
  source of the same field (the printed visual zone, the MRZ strip, the PDF417 barcode) and only flags
  real contradictions: a given name, date of birth, surname, document number or expiry that disagrees
  between two sources; a physically impossible date (date of birth after today, a card issued after it
  expires); and the same photo dropped into both the front and back slot. Shown as its own
  **"Cross-field validation"** stage in the report.
- **Real per-state number formats** — the licence/ID number is now checked against its issuing state's
  actual shape (e.g. California = letter + 7 digits, New York = 9 digits, Florida = letter + 12 digits,
  Illinois = letter + 11 digits), and for states that build the number with the surname's first letter
  up front (FL/IL/MD/MI/MN/NJ/WI) a mismatch is flagged as a soft "review". Each format note also
  carries a **REAL ID indicator** (the gold star that every US state now puts on compliant cards).
  For **Ohio**, the final digit of the 8-digit licence number is checked as a **mod-10 (Luhn) check
  digit** over the first seven — flagged as advisory, never a hard block, since Ohio's rule has
  changed over the years.
- **Verdict** — one summary you can act on, with each reason listed and how to fix it.

## How to run it

The quality checks run offline with no network. The OCR / barcode engines
are loaded from a CDN **the first time** you press their buttons, and are cached afterwards.

- **Easiest:** open `id-photo-checker.html` in **Chrome / Edge / Safari** (double-click it, or
  drag it into a browser tab). This is the recommended way to use camera + OCR features.
- **Serve it locally** (optional) if you want a URL:
  ```
  python3 -m http.server 8000
  # then open http://localhost:8000/id-photo-checker.html
  ```

> Note: the browser's in-app HTML preview is a sandboxed iframe with **no network access**, so
> the CDN-powered OCR / barcode / camera features won't run inside that preview. File-upload
> quality checks (sharpness, lighting, glare) do work there. For the full feature set, open the
> file in a normal browser tab.

### Deploy on GitHub Pages

This is a single static file, so it hosts for free on GitHub Pages:

1. Create a repository and push this folder to it (see below).
2. In the repo, go to **Settings → Pages → Source**, pick the `main` branch. GitHub
   builds the site at `https://<your-user>.github.io/<repo>/`.
3. The `index.html` at the root is a one-line redirect to `id-photo-checker.html`, so the
   site root opens the app directly.

> The CDN OCR/barcode engines and live camera need a normal browser tab. When served over
> Pages the page is the same file, so those features behave exactly as when opened from disk.

### Ways to add a photo

First pick what you're uploading so the right slots appear — or just scan it: once the scan
identifies the document, the layout **auto-switches** to match. A **passport / passport card**
carries everything on one photo page (portrait + the machine-readable strip), so it shows a
**single** upload slot — put the whole page in shot. A **licence / state ID** and a **green card**
each show **two** slots (Front + Back) because their machine-readable data lives on the back.

Each slot (Front / Back) accepts the image three ways:

1. **Browse files** — the on-page button (or clicking the box) opens the native file picker.
2. **Drag & drop** — drop a JPG/PNG onto the slot.
3. **Paste** — press `Ctrl+V`/`Cmd+V` with a copied image, into the current slot.

A **status line** appears under the slots the moment you add a photo, so you're never left
wondering: it says *"Reading front.jpg…"*, then either *"Got it — the front is in"* (in green)
or a specific reason it couldn't be read (in red). Once a slot is filled it turns green and
shows **"Front added / Back added"** with a **Replace** button.

## What changed to make this ours

Rebranded clone of the original tool, then upgraded into a full Persona-style product. Specifically:

| Original | This copy |
|---|---|
| Title / heading: "CICIO ID CONFIGURATION" | "MonochromeVibe AI" |
| Guidance file: `CICIOXL.html` | `id-photo-checker.html` |
| Netlify AI-builder marketing comments/meta | removed |
| Cloudflare analytics beacon + challenge script | removed (not ours to ship) |
| Selfie / face slot + face detection | **removed** (document-only check) |
| US driving licences, state IDs, passports only | **Permanent resident (green) cards added** (TD1 machine-readable zone: C1/C2 document code, USA issuer, A-number + 13-char green-card number) |
| No "what a genuine one looks like" reference | **Document reference library** (real dimensions, data location, validity, number format, security features) + structural format checks |
| Single "will it get through" verdict | **Persona-style verification report** (decision banner, stage cards, confidence, extracted-data review, configurable rules, data censoring, export) |
| Dark-gray GitHub-style theme | **Modern indigo-violet theme** (gradient cards, gradient report banner, soft shadows) |
| Upload: click the drop zone only | **Robust upload** — Browse button, click, drag-&-drop, and paste, with a green "added" state and clear error messages |
| Two fixed Front/Back slots | **Document-type selector** (licence / state ID, passport, green card) — a passport shows a single slot because its portrait and MRZ are on one page |
| Full checklist table | **Actionable view by default** — only blockers + to-review shown, with a toggle to reveal everything; a sticky "Not ready — will be refused" banner |
| Generic "5–25 characters" length check | **Real per-state number formats** (CA letter+7, NY 9, FL letter+12, IL letter+11, …), a surname-initial cross-check, a REAL ID gold-star note, and an Ohio mod-10 check digit |
| Side-by-side consistency only | **Cross-field anomaly engine** — independent-source contradictions (given name / DOB / surname / number / expiry), impossible dates, and same-photo-twice detection |

The original verification **engine** (capture-quality measurement, OCR, AAMVA/PDF417 parsing,
ICAO 9303 validation, genuine-ness signals) was preserved and wrapped in the new report/flow.
All selfie/face-scanning code and UI was removed. The whole app is a single file (~170 KB).
A backup of the pre-upgrade rebranded version is kept at `id-photo-checker.backup.html`.

## Privacy

Every check runs in this browser tab. Your document and everything read from it stay on your
device. The recognition engines are downloaded from a CDN on first use but your images are
never uploaded — recognition runs in a local Web Worker.

## Modifying it

Edit `id-photo-checker.html` directly — CSS variables at the top of the `<style>` block
(`--bg`, `--accent`, etc.) control the theme. The single inline `<script>` holds all logic
and is heavily commented.
