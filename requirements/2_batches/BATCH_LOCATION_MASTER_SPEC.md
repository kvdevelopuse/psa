# Batch Master and Location Master — Build Specification (Phase 1 Prototype)

| | |
|---|---|
| **Product** | Peak Sports Academy (PSA) student and fee app |
| **Feature** | Location Master and Batch Master |
| **Build target** | Browser prototype — plain HTML, CSS and JavaScript, data in `localStorage`, opened from `file://` |
| **Version** | 1.2 · 11 November 2026 (status filter is a drop-down: Active / Inactive / All) |

This document is the single source of truth for building this feature. Build exactly what is written here. If something is not described here, do not add it; raise it as a question instead.

---

## 0. How to read this document

- **MUST** means required. Anything not stated as required is out of scope.
- Text shown in `code` or in quotes inside tables is **exact UI text** — use it character for character, including punctuation and curly quotes (“ ”).
- Section 2 lists every decision. Sections 4–10 describe the data, the screens and the behaviour. Section 13 is the acceptance test list.

---

## 1. Scope

### 1.1 In scope (build now)

| # | Item |
|---|------|
| S1 | Location Master: list (landing page), new location, edit location, Active/Inactive status |
| S2 | Batch Master: list (landing page), batch details, new batch, edit batch, Active/Inactive status |
| S3 | Class dates for any month, worked out from the batch's class days |
| S4 | Shared storage layer for both masters in `localStorage` |
| S5 | Shared top bar with navigation between the two masters |

### 1.2 Out of scope (do NOT build)

| # | Item | Note |
|---|------|------|
| X1 | Holidays (screen, storage, or skipping dates) | Class dates are the class days only |
| X2 | Assigning students to batches | Phase 2. Batch details shows a placeholder only |
| X3 | Attendance | Phase 3 |
| X4 | Fee collection, receipts, reports | Separate feature |
| X5 | Deleting a location or batch | Use Active/Inactive instead |
| X6 | Extra or make-up classes | Phase 3 |
| X7 | Search, filters, paging, sorting controls, export, print | Not designed |
| X8 | Fixed list of age categories | Age category is free text |
| X9 | "Unsaved changes" confirmation on Cancel/Back | Cancel discards silently |
| X10 | Login, users, roles | Not part of this feature |

---

## 2. Decisions

### 2.1 Decisions agreed with the owner

| ID | Decision |
|----|----------|
| D1 | A batch has **no start date, end date or month**. The same batch runs every month of the year. |
| D2 | There are **no monthly batch records**. A month's class dates are calculated from the batch's class days every time they are shown. |
| D3 | Locations are a **master list** maintained on screen. The batch form's Location drop-down is filled from it. |
| D4 | Monthly fee is a **whole number of dirhams (AED)**, no decimals, greater than 0. No − / + stepper buttons. |
| D5 | **Age category is free text.** |
| D6 | **No holidays in phase 1.** |
| D7 | Locations and batches have an **Active / Inactive status. Nothing is ever deleted.** |
| D8 | Build target is the **browser prototype**: plain HTML/CSS/JS, no frameworks, no build step, no libraries, `localStorage` only, must run from `file://`. |
| D9 | Visual style follows the **PSA Student Registry designs** (section 11). |
| D10 | Landing page of each master is its **list**. A list row opens on **double-click** or with its **Open** button. |

### 2.2 Rules fixed by this specification (review before building)

These were not explicitly discussed; this spec fixes them so nothing is left to guess. Change them here if you disagree.

| ID | Rule |
|----|------|
| R1 | Location names are unique, ignoring upper/lower case, across **all** locations (active and inactive). |
| R2 | Batch names are unique **within a location**, ignoring upper/lower case, across active and inactive batches. The same batch name may exist at two different locations. |
| R3 | A location **cannot be made Inactive while it has any Active batch**. |
| R4 | An Inactive location is **not offered** when choosing a location for a batch. |
| R5 | An Inactive batch **cannot be made Active** while its location is Inactive. |
| R6 | Each list has a status filter: a **drop-down** with the options **Active**, **Inactive** and **All** (same behaviour as the Coaches master filter, shown as a drop-down). It is set to **Active** when the page opens. Active and Inactive show only those records, sorted by name A→Z. All shows active records first, then inactive, each group A→Z. The choice is kept while the user moves between the list and the forms, and resets to Active when the page is reloaded. |
| R7 | Batches link to locations by **location ID**, so renaming a location updates every batch automatically. |
| R8 | On first run (no stored locations), two locations are created: `DCA` and `Repton`, both Active, empty description. No batches are created. |
| R9 | Maximum lengths: location name 60, batch name 60, age category 20, descriptions 500 characters. Monthly fee maximum 9,999,999. |
| R10 | Text inputs are trimmed (leading and trailing spaces removed) on save. Inner spaces are kept as typed. |
| R11 | The week runs Monday → Sunday everywhere (checkbox order, text order). |
| R12 | Validation runs when Save is pressed. After the first failed Save, each field re-validates as the user types/changes it. |
| R13 | Each master has its own HTML page. Views inside a page are switched with the URL hash, so the browser Back button works. |

---

## 3. Files and folder structure

Create these files. Use **classic scripts** (`<script src="...">`), **not** ES modules — ES modules do not load from `file://` in Chrome/Edge.

```
psa-masters/
├── batches.html          Batch Master page (list, details, form)
├── locations.html        Location Master page (list, form)
├── css/
│   └── masters.css       All styles for both pages
└── js/
    ├── store.js          Storage layer (load, save, IDs, validation helpers)
    ├── dates.js          Class-date calculation and date/number formatting
    ├── ui.js             Shared UI helpers (escape, flash message, nav highlight)
    ├── locations.js      Location Master page logic
    └── batches.js        Batch Master page logic
```

Script order on both pages: `store.js`, `dates.js`, `ui.js`, then the page script.
Each script attaches what it exports to one global object `window.PSA` (for example `PSA.store`, `PSA.dates`, `PSA.ui`). No other globals.

If the project already has a folder for these pages, place the files there with the same names and relative paths.

---

## 4. Data model (localStorage)

### 4.1 Keys

| Key | Value |
|-----|-------|
| `psa.schemaVersion` | The string `"1"` |
| `psa.locations` | JSON array of Location records |
| `psa.batches` | JSON array of Batch records |

### 4.2 Location record

| Field | Type | Rules |
|-------|------|-------|
| `id` | string | Created once, never changed. Format: `loc_` + ID suffix (4.5) |
| `name` | string | Required, 1–60 chars after trim, unique (R1) |
| `description` | string | 0–500 chars after trim; empty string when not given |
| `isActive` | boolean | `true` when created |
| `createdAt` | string | ISO 8601 timestamp (`new Date().toISOString()`) when created |
| `updatedAt` | string | ISO 8601 timestamp, updated on every save |

Example:
```json
{ "id": "loc_lzx9k2_a81f", "name": "DCA", "description": "", "isActive": true,
  "createdAt": "2026-11-11T16:20:00.000Z", "updatedAt": "2026-11-11T16:20:00.000Z" }
```

### 4.3 Batch record

| Field | Type | Rules |
|-------|------|-------|
| `id` | string | Created once, never changed. Format: `bat_` + ID suffix (4.5) |
| `name` | string | Required, 1–60 chars after trim, unique within the location (R2) |
| `description` | string | 0–500 chars after trim; empty string when not given |
| `locationId` | string | Required. Must be the `id` of an existing location |
| `ageCategory` | string | Required, 1–20 chars after trim, free text (e.g. `U-13`) |
| `classDays` | number[] | Required, 1–7 items, ISO weekday numbers **1 = Monday … 7 = Sunday**, no duplicates, stored sorted ascending |
| `monthlyFee` | number | Required, integer, 1 – 9,999,999 (AED) |
| `isActive` | boolean | `true` when created |
| `createdAt` | string | ISO 8601 timestamp when created |
| `updatedAt` | string | ISO 8601 timestamp, updated on every save |

There are **no** date fields on a batch other than `createdAt` and `updatedAt`.

Example:
```json
{ "id": "bat_lzxa01_9c2e", "name": "Evening Juniors", "description": "Coach: [name]",
  "locationId": "loc_lzx9k2_a81f", "ageCategory": "U-11", "classDays": [1, 3],
  "monthlyFee": 650, "isActive": true,
  "createdAt": "2026-11-11T16:25:00.000Z", "updatedAt": "2026-11-11T16:25:00.000Z" }
```

### 4.4 Storage layer (`js/store.js`) — required functions

| Function | Behaviour |
|----------|-----------|
| `PSA.store.init()` | Called once on page load. If `psa.locations` is **missing**, write the two seed locations (R8), `psa.batches` as `[]`, and `psa.schemaVersion` `"1"`. If `psa.batches` is missing, write `[]`. Never overwrite existing data. |
| `PSA.store.getLocations()` | Read and parse `psa.locations`. Returns the array. |
| `PSA.store.getBatches()` | Read and parse `psa.batches`. Returns the array. |
| `PSA.store.saveLocation(record)` | Re-read the current array from storage, replace the record with the same `id` or append it, write back. Returns `true`/`false`. |
| `PSA.store.saveBatch(record)` | Same as above for batches. |
| `PSA.store.newId(prefix)` | Returns a new ID (4.5). |

Rules for every read and write:

1. **Always re-read from storage immediately before a save**, apply the one change, then write. Never write back an array that was loaded earlier (prevents overwriting changes made in another tab).
2. **Missing `isActive`** on a stored location or batch (for example from the earlier exported Location Master, which stored `{ id, name, description }`) is treated as `true`. Missing `createdAt`/`updatedAt` are treated as empty strings and filled on the next save of that record.
3. **Unreadable data:** if a key holds text that is not valid JSON, or is not an array, the page MUST show the error banner `Saved data could not be read. Nothing has been changed. Contact support before continuing.` and MUST disable every Save button on that page. It MUST NOT overwrite the key.
4. **Write failure:** wrap `localStorage.setItem` in `try/catch`. On failure, keep the user on the form with their input intact and show the error banner `Could not save. Your changes have not been stored.`
5. **Storage unavailable:** if `localStorage` itself throws on access, show the same banner as rule 3 and disable saving.

### 4.5 ID generation

`newId(prefix)` returns `prefix + Date.now().toString(36) + '_' + four random base-36 characters`, for example `loc_lzx9k2_a81f`. Do **not** use `crypto.randomUUID()` (it is not available on `file://` pages in every browser). Before returning, check the ID is not already used in the target array; if it is, generate again.

---

## 5. Class-date calculation (`js/dates.js`)

### 5.1 Rule

The class dates of a batch in a given month are **every date in that calendar month whose ISO weekday is in the batch's `classDays`**. Nothing is skipped in phase 1 (no holidays).

`Classes this month` = the number of class dates in the **current** calendar month (the month of the PC's local date).

### 5.2 Required functions

| Function | Behaviour |
|----------|-----------|
| `PSA.dates.isoWeekday(date)` | JavaScript `getDay()` returns 0 = Sunday … 6 = Saturday. Return `getDay() === 0 ? 7 : getDay()`. |
| `PSA.dates.classDates(classDays, year, monthIndex)` | Returns an array of local `Date` objects, ascending, for every day of the month (`monthIndex` 0–11) whose ISO weekday is in `classDays`. Build dates with `new Date(year, monthIndex, day)` (local time, never parse strings, never use UTC). |
| `PSA.dates.monthLabel(year, monthIndex)` | `November 2026` — `toLocaleDateString('en-GB', { month: 'long', year: 'numeric' })` |
| `PSA.dates.classDateLabel(date)` | `Mon 2 Nov` — `toLocaleDateString('en-GB', { weekday: 'short', day: 'numeric', month: 'short' })` |
| `PSA.dates.daysText(classDays)` | Short names in Mon→Sun order joined with `, ` — e.g. `[1,3]` → `Mon, Wed`. Names: `Mon Tue Wed Thu Fri Sat Sun`. |
| `PSA.dates.feeText(amount)` | `AED ` + `amount.toLocaleString('en-US')` — e.g. `AED 1,250` |

### 5.3 Worked example

Batch class days Mon and Wed (`[1, 3]`), month November 2026:
Mondays 2, 9, 16, 23, 30 and Wednesdays 4, 11, 18, 25 → **9** class dates:
`Mon 2 Nov, Wed 4 Nov, Mon 9 Nov, Wed 11 Nov, Mon 16 Nov, Wed 18 Nov, Mon 23 Nov, Wed 25 Nov, Mon 30 Nov`.

---

## 6. Shared layout (both pages)

### 6.1 Page frame

- Page background: Canvas `#F3F3F7`. Font: Barlow (section 11).
- **Top bar** (white, full width, bottom shadow `0 1px 0 #E3E4EC`):
  - Left: eyebrow `MASTERS` (12px, 700, letter-spacing .14em, uppercase, red `#D7262E`), below it the page title (Barlow Condensed 800, 30px, uppercase): `Batches` or `Locations`.
  - Right: the page's create button (red), shown **only on the list view**: `New batch` / `New location`, with a plus icon (inline SVG, 18px, stroke 2.2, `aria-hidden="true"`).
  - Below: navigation with two links `Batches` (→ `batches.html`) and `Locations` (→ `locations.html`). The current page's link has `aria-current="page"`, blue text `#2A3A9E` and a 3px red bottom bar; the other link is Slate `#5E6280`. Each link is at least 44px tall.
- **Content area:** max width 1240px, padding 32px (16px below 640px viewport width), vertical gap 20px.

### 6.2 Flash message

A green pill shown at the top of the content area after a save: background `#E3F4EA`, text `#17663A`, 14px bold, a 7px dot before the text, `role="status"`. It disappears on the next navigation (any hash change or page load).

### 6.3 Error banner

For storage errors (4.4): a full-width box at the top of the content area, background `#FDF1F1`, 2px inner outline `#D7262E`, text `#A3141B` 15px bold, `role="alert"`.

### 6.4 Status chips

| Status | Text | Style |
|--------|------|-------|
| Active | `Active` | background `#E3F4EA`, text `#17663A`, 7px dot, pill, 13px bold |
| Inactive | `Inactive` | background `#ECECF1`, text `#4E5170`, 7px dot, pill, 13px bold |

### 6.5 Routing (R13)

Each page reads `location.hash` on load and on `hashchange`, then shows exactly one view and hides the others (`hidden` attribute). Unknown hash → list view. A hash containing an ID that does not exist → list view with flash text replaced by the error banner text `That record was not found.` (shown once).

| Page | Hash | View | Document title |
|------|------|------|----------------|
| locations.html | `#/` or empty | Location list | `Locations — PSA` |
| locations.html | `#/new` | New location form | `New location — PSA` |
| locations.html | `#/<locationId>` | Edit location form | `<name> — Locations — PSA` |
| batches.html | `#/` or empty | Batch list | `Batches — PSA` |
| batches.html | `#/new` | New batch form | `New batch — PSA` |
| batches.html | `#/<batchId>` | Batch details | `<name> — Batches — PSA` |
| batches.html | `#/<batchId>/edit` | Edit batch form | `Edit <name> — PSA` |

---

## 7. Location Master (`locations.html`)

### 7.1 Location list — landing page

**Card:** white, radius 14px, outline `0 0 0 1px #E3E4EC`.

**Card header** (padding 22px 28px, bottom border `#ECEDF3`):
- Heading `ALL LOCATIONS` (Barlow Condensed 700, 20px, letter-spacing .16em, uppercase).
- Count badge next to it: `1 location` / `<n> locations` (records shown under the current filter).
- Right side hint (13px Slate): `Double-click a row to open it.`

**Status filter** (right side of the card header, before the hint):

| Item | Specification |
|---|---|
| Control | A native `<select id="status-filter">` drop-down (not buttons, not a custom menu) |
| Visible label | `Status`, a `<label for="status-filter">` placed to the left of the drop-down, 13px, weight 600, colour `#3B3F5C` |
| Options, in this order | `Active` (value `active`), `Inactive` (value `inactive`), `All` (value `all`). No other options and no empty option |
| Default | `Active` is selected every time the page is opened or reloaded |
| Changing it | Selecting an option updates the list straight away (on the `change` event). There is no Apply button. Focus stays on the drop-down |
| What each option shows | `Active`: only records with `isActive = true`. `Inactive`: only records with `isActive = false`. `All`: every record, active first then inactive (R6) |
| Remembered | The choice is kept, separately for each list, while the user opens records or forms and comes back, and when switching between the Batches and Locations tabs. It is not saved to storage; a reload sets it back to `Active` |
| Count badge | Shows the number of records listed under the current choice |
| Style | Height 44px, minimum width 150px, no border, radius 6px, background `#ECEAF2`, 15px, weight 600, text `#15172B`; focus outline as in A4 |
| Keyboard | Reachable with Tab; arrow keys and Enter change the option (native drop-down behaviour) |

**Filter has no match** (records exist but none match): heading `NO ACTIVE LOCATIONS` or `NO INACTIVE LOCATIONS`, text `Change the filter to see other locations.`


**Table columns:**

| # | Header | Content |
|---|--------|---------|
| 1 | `Location` | Name in a blue-tint chip (Barlow Condensed 700, 17px, text `#2A3A9E`, background `#E9ECFA`, radius 4px) |
| 2 | `Description` | Description text, or `—` when empty |
| 3 | `Active batches` | Number of **Active** batches with this `locationId` (right-aligned) |
| 4 | `Status` | Status chip (6.4) |
| 5 | *(visually hidden header `Actions`)* | `Open` button (tonal style), `aria-label="Open <name>"` |

- Order: R6.
- Inactive rows: all cell text in Slate `#5E6280`.
- Row hover: background `#F7F7FA`; cursor pointer.
- **Double-click** anywhere on a row → `#/<locationId>`. **Open** button click → same.
- Table wrapper scrolls horizontally when narrower than 640px.

**Empty state** (no locations at all — only possible if the data was emptied): heading `NO LOCATIONS YET`, text `Add a location before creating batches.`, button `New location`.

### 7.2 New / edit location form

**Back link** above the card: `Back to locations` with a left-chevron icon → `#/`.

**Form header** (padding 28px 32px, bottom border):
- Eyebrow (red): new → `PEAK SPORTS ACADEMY`; edit → `LOCATION DETAILS`.
- Title (Barlow Condensed 800, 44px, uppercase, blue): new → `New location`; edit → the location's saved name.
- Sub-text (14px Slate): `Fields marked * are required.` (the `*` in red bold).

**Section A — band `Location details`:**

| Field | Control | Required | Max | Placeholder |
|-------|---------|----------|-----|-------------|
| `Location *` | `<input type="text" id="loc-name">` | Yes | 60 | `e.g. Al Barsha` |
| `Description` | `<textarea id="loc-desc" rows="3">`, full width | No | 500 | `Address, ground details or directions` |

**Section B — band `Status`:**
Two radio buttons styled as option buttons, side by side, legend `Status`: `Active` and `Inactive`. New location: `Active` selected. Selected option: background `#E9ECFA`, inner outline 2px `#2A3A9E`, text `#1B2466` bold. Unselected: background `#ECEAF2`.

**Bottom bar** (sticky to the bottom of the card, white, top shadow): left status text; right buttons `Cancel` (secondary) and save (primary, with a check icon): new → `Save location`; edit → `Save changes`.
Left status text: new → `The new location is added to the list.`; edit → `Used by <n> active batch(es).` (`1 active batch` / `<n> active batches`).

### 7.3 Location validation (on Save, then live per R12)

| Check | Message under the field |
|-------|------------------------|
| Name empty after trim | `Enter a location name.` |
| Name longer than 60 | Not possible — `maxlength="60"` |
| Name equals another location's name, ignoring case (R1) | `A location with this name already exists.` |
| Status set to Inactive while the location has ≥ 1 Active batch (R3) | Under the Status options: `This location has 1 active batch. Make that batch inactive first.` or `This location has <n> active batches. Make those batches inactive first.` |

When any check fails: field background `#FDF1F1` with 2px inner outline `#D7262E`, message 12px bold `#A3141B`, field gets `aria-invalid="true"` and `aria-describedby` pointing at its message, bottom-bar text becomes `Fix the highlighted fields to save.` in `#A3141B` bold, focus moves to the first invalid field. Nothing is saved.

### 7.4 Location save behaviour

| Case | Result |
|------|--------|
| New, valid | Create record (4.2), save, go to `#/`, flash `Location “<name>” added` |
| Edit, valid | Update `name`, `description`, `isActive`, `updatedAt`; save; go to `#/`, flash `Location “<name>” saved` |
| Cancel or Back link | Go to `#/` without saving, no flash |

Renaming a location needs no other change: batches store `locationId` (R7).

---

## 8. Batch Master (`batches.html`)

### 8.1 Batch list — landing page

Card and header as in 7.1, with heading `ALL BATCHES`, count `1 batch` / `<n> batches` (under the current filter), hint `Double-click a row to open its details.`

**Status filter** (right side of the card header, before the hint):

| Item | Specification |
|---|---|
| Control | A native `<select id="status-filter">` drop-down (not buttons, not a custom menu) |
| Visible label | `Status`, a `<label for="status-filter">` placed to the left of the drop-down, 13px, weight 600, colour `#3B3F5C` |
| Options, in this order | `Active` (value `active`), `Inactive` (value `inactive`), `All` (value `all`). No other options and no empty option |
| Default | `Active` is selected every time the page is opened or reloaded |
| Changing it | Selecting an option updates the list straight away (on the `change` event). There is no Apply button. Focus stays on the drop-down |
| What each option shows | `Active`: only records with `isActive = true`. `Inactive`: only records with `isActive = false`. `All`: every record, active first then inactive (R6) |
| Remembered | The choice is kept, separately for each list, while the user opens records or forms and comes back, and when switching between the Batches and Locations tabs. It is not saved to storage; a reload sets it back to `Active` |
| Count badge | Shows the number of records listed under the current choice |
| Style | Height 44px, minimum width 150px, no border, radius 6px, background `#ECEAF2`, 15px, weight 600, text `#15172B`; focus outline as in A4 |
| Keyboard | Reachable with Tab; arrow keys and Enter change the option (native drop-down behaviour) |

**Filter has no match** (records exist but none match): heading `NO ACTIVE BATCHES` or `NO INACTIVE BATCHES`, text `Change the filter to see other batches.`



**Table columns:**

| # | Header | Content |
|---|--------|---------|
| 1 | `Batch` | Name (bold) and, on a second line, the description in 13px Slate (omit the line when empty) |
| 2 | `Location` | Location name in a blue-tint chip (Barlow Condensed 700, 15px) |
| 3 | `Age category` | Age category text |
| 4 | `Class days` | `PSA.dates.daysText(classDays)` |
| 5 | `Monthly fee` | `PSA.dates.feeText(monthlyFee)`, right-aligned, tabular numbers |
| 6 | `Classes this month` | Count of class dates in the current month (5.1), right-aligned, bold |
| 7 | `Status` | Status chip |
| 8 | *(hidden header `Actions`)* | `Open` button, `aria-label="Open <name>"` |

- Order: R6.
- Inactive rows in Slate text.
- Double-click a row or click **Open** → `#/<batchId>`.

**Empty state:** heading `NO BATCHES YET`, text `Create the first batch. Students are assigned to it in the next phase.`, button `New batch`.
If there are **no active locations**, the `New batch` buttons are still shown; the form shows the message in 8.3.

### 8.2 Batch details

**Back link:** `Back to batches` → `#/`.

**Header block** (white card top, padding 28px 32px, bottom border):
- Eyebrow `BATCH DETAILS` (red).
- Batch name (Barlow Condensed 800, 44px, uppercase, blue).
- Chips row: location name chip, age category chip (both blue-tint), status chip.
- Right side: a box (background `#F3F3F7`, radius 8px) with label `THIS MONTH` (11px, uppercase, Slate), value `<n> classes` (`1 class` when one) in Barlow Condensed 800 26px, and below it the month label, e.g. `November 2026`.
- Button `Edit details` (secondary) → `#/<batchId>/edit`.

**Section A — band `Details`.** Grid of read-only tiles (background `#F7F7FA`, radius 10px, label 12px Slate, value 16px bold):
`Location`, `Age category`, `Class days`, `Monthly fee`, `Status`, and a full-width `Description` tile (value `—` when empty).

**Section B — band `Class dates`.**
- Month bar: previous-month button (left chevron, `aria-label="Previous month"`), month label (Barlow Condensed 800, 26px, `aria-live="polite"`), next-month button (`aria-label="Next month"`). When the shown month is not the current month, a text button `This month` appears after the arrows and returns to the current month. Buttons 44×44px, tonal style.
- Badge on the right: `<n> classes` (`1 class`).
- Explanation text (14px): `Worked out from the class days. These are the dates that will get an attendance record for each student in the batch.`
- Table with columns `#` and `Date`: one row per class date (5.2), numbered 1, 2, 3 …, date label `Mon 2 Nov`.
- No limit on how far back or forward the user can move. The shown month resets to the current month each time the details view is opened.

**Section C — band `Students`.**
A dashed-feel placeholder box (background `#F7F7FA`, inner outline 2px `#E3E4EC`, centred): heading `NO STUDENTS ASSIGNED`, text `Assigning students to this batch comes in the next phase.` No buttons.

### 8.3 New / edit batch form

**Back link:** new → `Back to batches` → `#/`; edit → `Back to batch details` → `#/<batchId>`.

**Form header:**
- Eyebrow: new → `PEAK SPORTS ACADEMY`; edit → `EDIT BATCH DETAILS`.
- Title: new → `New batch`; edit → the batch's saved name.
- Sub-text: `Fields marked * are required. A batch runs every month; there are no start or end dates.`

**Section A — band `Batch details`:**

| Field | Control | Required | Max | Placeholder / options |
|-------|---------|----------|-----|-----------------------|
| `Batch name *` | text input, full width | Yes | 60 | `e.g. Evening Juniors` |
| `Location *` | `<select>` | Yes | — | First option `Select location` (value `""`, disabled, selected on a new form). Then every **Active** location, sorted A→Z, value = location `id`, text = name. When editing a batch whose location is Inactive, that location is also listed, as `<name> (inactive)`, and is selected. |
| `Age category *` | text input | Yes | 20 | `e.g. U-13` |
| `Batch description` | textarea, 3 rows, full width | No | 500 | `Coach, skill level or anything parents should know` |

If there are no Active locations at all, show under the Location field (12px Slate, not an error): `No active locations. Add one in Locations first.` with `Locations` as a link to `locations.html`.

**Section B — band `Class days & fee`:**
- Fieldset, legend `Class days *`: seven checkboxes in order `Mon Tue Wed Thu Fri Sat Sun`. Each is a `<label>` wrapping an `<input type="checkbox">` plus the short name, styled as a 46px-tall option button that grows to fill the row (min width 96px, wraps on narrow screens). `aria-label` on each checkbox is the full day name (`Monday` …). Ticked: background `#E9ECFA`, inner outline 2px `#2A3A9E`, text bold `#1B2466`. Unticked: background `#ECEAF2`. Checkbox accent colour `#2A3A9E`.
- `Monthly fee *`: a 46px field with the input on the left and the suffix `AED` (13px bold Slate) inside the field on the right.
  - `<input type="text" inputmode="numeric" autocomplete="off" maxlength="7">` — **not** `type="number"` (it accepts `e`, `.` and `-` and shows spinner arrows).
  - Typing: on the `beforeinput` event, if the inserted text contains any character other than `0–9`, call `preventDefault()` and show the field message `Whole dirhams only, no decimals.` Typing a digit clears that message.
  - Paste: same rule — a pasted value containing anything other than digits (e.g. `650.50`, `1,250`, `AED 650`) is **rejected whole** and the message is shown. The value is **never** altered by removing characters.
  - Helper text under the field (12px Slate): `Whole dirhams only, no decimals.`

**Section C — band `Status`:** as in 7.2 (Active / Inactive option buttons; new batch defaults to Active).

**Bottom bar:**
- Left text: new → `The new batch is added to the list.`; edit → `Changes apply to every month.`
- Buttons: `Cancel` (secondary); save (primary, check icon): new → `Save batch`; edit → `Save changes`.

### 8.4 Batch validation (on Save, then live per R12)

| Check | Message |
|-------|---------|
| Batch name empty after trim | `Enter a batch name.` |
| Another batch at the same location has the same name, ignoring case (R2) | `A batch with this name already exists at this location.` |
| No location selected | `Choose a location.` |
| Age category empty after trim | `Enter an age category.` |
| No class day ticked | `Select at least one day.` (under the day buttons) |
| Fee empty, `0`, starts with `0`, or not 1–7 digits (regex `^[1-9][0-9]{0,6}$`) | `Enter a whole amount above 0.` |
| Status Active while the chosen location is Inactive (R5) | Under Status: `<location> is inactive. Choose an active location or make the location active first.` |

Error styling, focus and bottom-bar text are the same as 7.3, with bottom-bar text `Fix the highlighted fields to save this batch.`

### 8.5 Batch save behaviour

| Case | Result |
|------|--------|
| New, valid | Create record (4.3) with `classDays` sorted ascending and `monthlyFee` as a number; save; go to `#/`; flash `Batch “<name>” created` |
| Edit, valid | Update all editable fields and `updatedAt`; save; go to `#/<batchId>`; flash `Details saved` |
| Cancel / Back (new) | Go to `#/`, no save |
| Cancel / Back (edit) | Go to `#/<batchId>`, no save |

---

## 9. Behaviour rules across both masters

| ID | Rule |
|----|------|
| B1 | Changing a batch's class days changes the class dates and counts of every month immediately (nothing is stored per month). |
| B2 | Changing a batch's fee changes only the batch record. (Payments are a separate feature and keep their own amount.) |
| B3 | Making a batch Inactive keeps it in the list (with the Inactive chip) and keeps all its data. |
| B4 | Making a location Inactive is allowed only when it has no Active batches (R3). Inactive locations stay in the location list and are hidden from the batch form's drop-down (R4). |
| B5 | Every screen reads fresh data from storage when it is shown (on load and on every hash change). |
| B6 | All user text is inserted into the page as text, never as HTML. If HTML strings are built, every value MUST pass through `PSA.ui.escape()` (escapes `& < > " '`). |
| B7 | Nothing in this feature uses the network except the optional Google Fonts stylesheet (11.2). The app MUST work fully offline. |

---

## 10. Accessibility

| ID | Requirement |
|----|-------------|
| A1 | Every input, select and textarea has a visible `<label for>`; checkbox and radio groups use `<fieldset>` + `<legend>`. |
| A2 | Every row action is also a real `<button>` (`Open`), so double-click is never the only way. Icon-only buttons have `aria-label`. Decorative SVGs have `aria-hidden="true"`. |
| A3 | Interactive targets are at least 44px tall (row `Open` buttons at least 36px, inside a 44px+ row). |
| A4 | Visible focus outline on all controls: 2px solid `#2A3A9E`, offset 2px (`:focus-visible`). |
| A5 | Errors: `aria-invalid="true"` + `aria-describedby`; the bottom-bar status uses `role="status"`; the storage banner uses `role="alert"`. |
| A6 | Text contrast at least 4.5:1 (the palette in 11.1 meets this on white and on its stated backgrounds). |
| A7 | `<html lang="en">`; each view sets `document.title` (6.5). |

---

## 11. Visual design (PSA style)

### 11.1 Colour tokens (define as CSS variables in `masters.css`)

| Token | Hex | Use |
|-------|-----|-----|
| `--blue` | `#2A3A9E` | Primary buttons, titles, section bands, links |
| `--night` | `#1B2466` | Text on tint backgrounds |
| `--red` | `#D7262E` | Create buttons, eyebrows, required `*`, section letter tiles, error outline |
| `--red-dark` | `#A3141B` | Error text |
| `--ink` | `#15172B` | Body text |
| `--label` | `#3B3F5C` | Field labels |
| `--slate` | `#5E6280` | Secondary text, hints, inactive rows |
| `--fill` | `#ECEAF2` | Input backgrounds, unselected options |
| `--tint` | `#E9ECFA` | Chips, selected options, tonal buttons |
| `--canvas` | `#F3F3F7` | Page background, info boxes |
| `--row` | `#F7F7FA` | Table header, row hover, read-only tiles |
| `--line` | `#E3E4EC` | Card outlines |
| `--line-2` | `#ECEDF3` | Row and section dividers |
| `--ok-bg` / `--ok` | `#E3F4EA` / `#17663A` | Active chip, flash |
| `--off-bg` / `--off` | `#ECECF1` / `#4E5170` | Inactive chip |
| `--err-bg` | `#FDF1F1` | Invalid field background, error banner |

### 11.2 Typography

- Body: `'Barlow', 'Segoe UI', Arial, sans-serif` — 15–16px, weights 500/600/700.
- Display (titles, bands, chips, big numbers): `'Barlow Condensed', 'Arial Narrow', 'Segoe UI', Arial, sans-serif` — weights 700/800, uppercase for titles and bands.
- Load the fonts with `<link href="https://fonts.googleapis.com/css2?family=Barlow:wght@400;500;600;700&family=Barlow+Condensed:wght@600;700;800&display=swap" rel="stylesheet">`. The academy PC has no internet, so the fallbacks above MUST give a usable layout. Do not add font files to the project unless the owner approves.

### 11.3 Components

| Component | Specification |
|-----------|---------------|
| Card | White, radius 14px, `box-shadow: 0 0 0 1px var(--line)`, `overflow: hidden` |
| Section band | Height 44px. Left: 52px red tile with the white section letter (A, B, C — Barlow Condensed 800, 22px), `clip-path: polygon(0 0,100% 0,80% 100%,0 100%)`. Right: blue band, `margin-left: -8px`, `clip-path: polygon(12px 0,100% 0,100% 100%,0 100%)`, white text, Barlow Condensed 700, 18px, letter-spacing .18em, uppercase, padding `0 20px 0 26px`. Rendered as an `<h3>` inside the band. |
| Text input / select | Height 46px, no border, radius 6px, background `--fill`, padding 0 14px, 16px, weight 500 |
| Textarea | Same as input, padding 12px 14px, line-height 1.5, vertical resize |
| Field label | 13px, weight 600, `--label`; required `*` in `--red` |
| Field grid | `display:grid; grid-template-columns: repeat(auto-fit, minmax(240px, 1fr)); gap: 20px 24px` |
| Primary button | Background `--blue`, white, min-height 48px, padding 0 26px, radius 8px, 15px bold |
| Create button (top bar) | Background `--red`, white, min-height 44px, padding 0 18px, radius 8px, plus icon |
| Secondary button | White, `inset 0 0 0 1px #DADBE6`, text `--ink`, min-height 48px |
| Tonal button | Background `--tint`, text `--night`, min-height 36px, radius 6px, 14px bold |
| Table | Full width, collapsed borders, 15px. Header row background `--row`, 12px bold uppercase Slate, letter-spacing .1em. Cells padding 14px 16px (first/last column 28px side padding), bottom border `--line-2` |
| Bottom bar | `position: sticky; bottom: 0`, white, `box-shadow: 0 -1px 0 var(--line)`, padding 18px 32px, space-between |

---

## 12. Build order

1. `css/masters.css` — tokens and components (section 11).
2. `js/store.js` — storage layer with seed, error handling and IDs (section 4).
3. `js/dates.js` and `js/ui.js` (sections 5, 6.2–6.4, B6).
4. `locations.html` + `js/locations.js` — list, form, validation, save, routing (sections 6, 7).
5. `batches.html` + `js/batches.js` — list, details, form, validation, save, routing (sections 6, 8).
6. Run the acceptance tests in section 13 in Chrome **and** Edge, opening the files directly from disk (`file://`).

---

## 13. Acceptance tests

Run each test from a fresh start (clear `localStorage` for the page) unless it says otherwise.

| # | Steps | Expected |
|---|-------|----------|
| T1 | Open `locations.html` | List shows `DCA` and `Repton`, both Active, 0 active batches; count `2 locations` |
| T2 | New location → Save with empty name | `Enter a location name.`; nothing saved |
| T3 | New location `dca` → Save | `A location with this name already exists.` |
| T4 | New location `Al Barsha`, description `Near the metro` → Save | Back on list; flash `Location “Al Barsha” added`; row shows description |
| T5 | Open `batches.html` → New batch | Location drop-down lists `Al Barsha`, `DCA`, `Repton` (A→Z) |
| T6 | Save the empty batch form | Messages: `Enter a batch name.`, `Choose a location.`, `Enter an age category.`, `Select at least one day.`, `Enter a whole amount above 0.`; bottom bar `Fix the highlighted fields to save this batch.` |
| T7 | In Monthly fee, type `6`, `5`, `0`, then `.` | The `.` is refused: the field still shows `650` and the message `Whole dirhams only, no decimals.` appears. Typing another digit clears the message. |
| T8 | Clear Monthly fee, paste `650.50` | Paste refused; field stays empty; message `Whole dirhams only, no decimals.` |
| T9 | Create `Evening Juniors`, DCA, `U-11`, Mon + Wed, fee `650` | List flash `Batch “Evening Juniors” created`; row shows `Mon, Wed`, `AED 650`, Classes this month = number of Mondays + Wednesdays in the current month |
| T10 | Double-click the Evening Juniors row | Details view; `THIS MONTH` matches T9; Class dates table lists each Monday and Wednesday of the month, numbered |
| T11 | On details, press Next month twice, then `This month` | Month label and dates change; `This month` returns to the current month and the button disappears |
| T12 | Create another batch at DCA named `evening juniors` | `A batch with this name already exists at this location.` |
| T13 | Create `Evening Juniors` at Repton | Saves (same name allowed at another location) |
| T14 | Edit Evening Juniors (DCA): change fee to `700` → Save | Details view, flash `Details saved`, fee `AED 700` |
| T15 | Locations → open DCA → set Inactive → Save | Blocked: `This location has 1 active batch. Make that batch inactive first.` |
| T16 | Make Evening Juniors (DCA) Inactive, then make DCA Inactive | Both save. Under `Active` neither appears; under `Inactive` each appears with the Inactive chip; under `All` inactive rows come after active ones |
| T16a | Open either list | The `Status` drop-down shows `Active` and lists the options `Active`, `Inactive`, `All` in that order; the count matches the rows shown |
| T16b | Choose `All` in the drop-down, open a record, press Cancel; then switch tabs and back | The drop-down still shows `All` each time; the other tab's drop-down is unaffected |
| T16c | With only active batches, choose `Inactive` in the batch list drop-down | `NO INACTIVE BATCHES` and `Change the filter to see other batches.` |
| T16d | Choose `All` in the drop-down, reload the page | The drop-down is back on `Active` |
| T17 | New batch | DCA is not in the drop-down |
| T18 | Edit the inactive Evening Juniors (DCA) | Drop-down shows `DCA (inactive)` selected; setting Status Active → `DCA is inactive. Choose an active location or make the location active first.` |
| T19 | Rename location `Repton` to `Repton School` | Batch list shows `Repton School` for the Repton batch |
| T20 | Close and reopen both pages | All data still there |
| T21 | Open `batches.html#/does-not-exist` | List view with banner `That record was not found.` |
| T22 | In DevTools set `localStorage['psa.batches'] = 'oops'`, reload | Banner `Saved data could not be read. Nothing has been changed. Contact support before continuing.`; Save buttons disabled; the key still contains `oops` |
| T23 | Put a location named `<b>x</b>` | Shown literally as `<b>x</b>`, not bold |
| T24 | Use only the keyboard: Tab to a row's `Open`, press Enter; fill and save a form | Every action reachable; focus outline visible |
| T25 | Disconnect from the internet, reload | Pages work; fonts fall back without breaking the layout |
| T26 | Use the browser Back button after opening a batch's details | Returns to the list |

---

## 14. For later phases (do not build — context only)

These explain why the data model looks the way it does.

- **Phase 2 — assigning students:** a `studentBatches` list with `{ id, studentId, batchId, joinedOn, leftOn }` (`leftOn` empty while the student is still in the batch). A student can be in only one batch on any day; moving batch closes the old record and opens a new one. The Students section of batch details will list current students.
- **Phase 3 — attendance:** one record per student per **class date** (section 5), e.g. `S1 · B1 · 2026-11-02`, with `status` = `present`, `absent`, `excused`, or empty for "not marked". Records are created when the attendance sheet for a batch and month is opened; marked records are never changed automatically. The batch ID is stored on each record so history stays correct after a student changes batch.
- **Holidays** (when added): a list of closed dates for all locations, one location or one batch; class dates covered by a holiday are skipped in section 5.
