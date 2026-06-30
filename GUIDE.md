# BeLife Multi-Center Duplicate Detector — Google Drive Edition

`belife-multidc-detector.html` — one self-contained file. Each diagnostics
center (Amino, My Choice Lab, …) has its **own Google Drive folder** and its
**own isolated database**. A report is only ever checked against the *same
center's* history, and every checked PDF is saved back into that center's Drive
folder so it becomes part of the database.

There is **no server** and **no combined `master_db.json`** — each center's
database is built live, in your browser, from the files in its Drive folder.

---

## What you need before first use (one time, ~5 minutes)

Because a browser page cannot touch Google Drive without permission, you must
create **your own** Google OAuth credentials (free). This is Google's security
model, not an app limitation.

1. **OAuth Client ID** — In the
   [Google Cloud Console → Credentials](https://console.cloud.google.com/apis/credentials):
   create a project → **Create Credentials → OAuth client ID → Web application**.
   Under **Authorized JavaScript origins**, add the *exact* URL you'll open the
   page from — e.g. `http://localhost:8000` (local) or your hosting URL. Copy the
   **Client ID** (ends in `.apps.googleusercontent.com`).
2. **Enable APIs + API key** — In
   [APIs & Services → Library](https://console.cloud.google.com/apis/library),
   enable **Google Drive API** and **Google Picker API**. Then **Create
   Credentials → API key** and copy it.
3. Open the tool, paste both into the **one-time setup** panel, and click
   **Save & continue**. They're stored only in your browser (localStorage).

> **Scope:** the app requests `drive.file` — it can see and modify only the files
> it creates or that you explicitly open with it. It can **never** see your whole
> Drive.

---

## How to run

The page must be served over HTTP (not opened as a `file://`), because Google
sign-in requires a real origin.

```bash
# put belife-multidc-detector.html in a folder, then:
python3 -m http.server 8000
# open http://localhost:8000/belife-multidc-detector.html
```

Make sure `http://localhost:8000` is listed as an **Authorized JavaScript origin**
on your OAuth Client ID (step 1 above). For real deployment, host the single HTML
file on any static host (and add that URL as an authorized origin).

---

## Organizing each center's data on Drive

One **folder per center**, each containing that center's monthly export files:

```
Google Drive/
├── Amino/
│   ├── Amino_January.xlsx
│   ├── Amino_February.csv
│   └── Amino_March.xlsx
├── My Choice Lab/
│   ├── MyChoice_Jan.xlsx
│   └── MyChoice_Feb.xlsx
└── … (one folder per center)
```

- Files can be `.xlsx`, `.xls`, or `.csv`.
- The tool auto-detects the **month** from each filename (looks for
  January/Feb/Mar… or a `2026-03` style date). If it can't, it uses the filename
  as the label — still fine.
- **All centers must use the same export format** as your current files (the wide
  `value, Range, value, Range…` columns with the same parameter spellings). The
  built-in normalization map covers 150+ name variants → 77 canonical tests. If a
  center uses a *different* lab software with different column names, that format
  needs its own mapping added to the `CANON` table in the file.

---

## Daily workflow

1. **Sign in with Google** (top right) — once per session.
2. **Pick a center.** First time, click **“Add / link a center folder”**, choose
   that center's Drive folder in the Google Picker, and name it. The tool reads
   the folder, parses every file, and builds that center's database
   (progress shown). The built database is **cached locally** keyed by center, so
   re-opening is instant. To rebuild after adding new monthly files, you can
   re-link / re-open the center.
3. **Upload a report PDF** to check. The tool:
   - extracts its parameter values,
   - checks them against **only this center's** history,
   - shows suspicious/duplicate matches with bit-scores and a side-by-side
     comparison,
   - shows the **“How common are this report's values?”** panel (analytics link),
   - **uploads the PDF into this center's Drive folder**, and folds it into the
     center's database so future checks see it.
4. **Approve / confirm** each match (with an optional remark). Decisions are
   stored locally per center.

Switching centers (**← Switch center**) loads that center's separate database.
Amino's reports are never compared to My Choice Lab's.

---

## The detection engine (unchanged, validated)

Same information-theoretic matcher as the single-center tool:

- **Hard gate:** a report must share **≥2** parameter values with a historical
  record to be considered at all.
- **Weighting:** each shared value scores `-log₂(P(value))` — its rarity in
  *this center's* data. Rare values (a specific B12, an unusual platelet count)
  score high; common values near zero. A re-uploaded report scores hundreds of
  bits; coincidental overlaps fall below the threshold.
- **Panel-size aware:** thresholds scale with √(panel size), so a 40-test checkup
  and a 3-test report produce equally meaningful review queues.
- **Quality floor:** average ≥3 bits per matched parameter.

Validated against the real 26K-record corpus: a genuine re-upload scores in the
hundreds of bits (HIGH); coincidences are suppressed.

---

## Analytics tab (per center)

Each center's **📊 Common Parameter & Value Analytics** tab shows, for that
center's data:

- **High-Frequency Value Alerts** — continuous-test values repeating far more
  than expected (e.g. *HbA1c 5.3*, *Hemoglobin 14.2*), colour-coded by severity.
- **Top-20 parameters** bar chart + **Parameter Usage** table (total, month-wise,
  % usage).
- **Most Repeated Values** table (value, repeats, % of param, patients, months).
- **Month-wise value trend** line chart and **duplicate-value heatmap**.
- Filters (month / parameter / value-type — *Continuous only* by default / Top
  10·25·50 / value search) and **Excel + PDF export**.

> Alerts run on **continuous** parameters only by default: count-type tests
> (Basophils, Eosinophils, Neutrophils, ESR…) cluster on small integers by
> nature, so their repetition isn't meaningful. Switch the value-type filter to
> include them if needed.

---

## Privacy

All parsing, matching, and analytics happen **in your browser**. Patient data
moves only between your browser and **your own Google Drive** — never to any
third-party server. Built databases and review decisions are cached locally in
your browser (IndexedDB) and can be cleared anytime via the browser, or by
clearing the saved keys in the setup panel.

---

## Limitations / notes

- **Scanned image PDFs** (no text layer) can't be read in-browser — OCR them
  first.
- **Very large centers** (100k+ records in one folder) will be slow to build the
  first time, since parsing happens client-side; subsequent opens use the cache.
- **Different export formats** across centers need per-format column mappings
  (extend the `CANON` map).
- Rebuild a center's database after you add new monthly files to its folder
  (re-open / re-link the center).
