# Coaches Master

**Feature specification · Peak Sports Academy fee app**

Version 1.0 · 11 November 2026

## 1. Purpose

The Coaches Master holds the academy's coaches. For each coach it records a name, an optional phone number, a description, remarks and whether the coach is active.

## 2. Scope

### 2.1 In this feature

- Coach list (landing page) with a Status drop-down filter (All, Active, Inactive).
- Coach details page (read-only), opened from the list, with Edit details and Activate / Deactivate buttons.
- New coach form.
- Edit coach form.
- Field validation, messages and empty states.
- Storage in browser localStorage (prototype) and PostgreSQL (production).

### 2.2 Not part of this feature

- Deleting coaches. Coaches who leave are set to Inactive instead.
- Assigning coaches to batches.
- Searching the coach list.
- Importing or exporting coaches.
- User roles or permissions.

## 3. Users and environment

These are the same as for the Location Master and Batch Master.

- **Users:** academy admin staff, using one shared login. No role-based permissions.
- **Device:** the academy office PC, in Chrome or Edge. The office PC has no internet connection.
- **Prototype:** plain HTML, CSS and JavaScript, with no frameworks and no build step. Data is stored in browser localStorage. The app opens by double-clicking index.html.
- **Production:** a local web app backed by PostgreSQL, using the table in section 8.
- **Fonts:** Barlow and Barlow Condensed. With no internet, the app must either bundle the font files locally or fall back to system fonts without breaking the layout.

## 4. Confirmed decisions

These decisions were confirmed by the product owner on 11 November 2026 (D8 and the drop-down in D7 added the same day).

| ID | Decision |
| --- | --- |
| D1 | Name is required. Two coaches may have the same name. Maximum 100 characters. |
| D2 | Phone number is optional. If entered, it must contain 9 to 15 digits. Only digits, spaces and one leading + are allowed. |
| D3 | Two coaches may have the same phone number. |
| D4 | Description and Remarks are optional, with a maximum of 500 characters each. |
| D5 | Coaches are never deleted. Each coach has a status of Active or Inactive. |
| D6 | A new coach is always saved as Active. Status is changed only on the coach details page, with an Activate or Deactivate button. The edit form has no status field. |
| D7 | The coach list has a Status drop-down with the values All, Active and Inactive, in that order. Active is selected by default. |
| D8 | Double-clicking a row on the coach list opens the coach details page. |

## 5. Screens

### 5.1 Coach list (landing page)

| Element | Specification |
| --- | --- |
| Header | Small red label "MASTERS" above the title "COACHES". A red **New coach** button with a + icon sits at the top right. The button shows only on the list, not on the forms. |
| Success message | A green pill above the list shows "Coach “<name>” added" or "Coach “<name>” saved" after a save. It disappears when a form is opened. |
| Panel heading | The heading "ALL COACHES" is followed by a count badge, such as "1 coach" or "5 coaches". The count is the number of coaches shown under the current filter. |
| Status filter | A drop-down labelled "Status", next to the count badge, with the options **All**, **Active** and **Inactive** in that order. Active is selected every time the screen opens. The choice is kept while the user moves between the list, the details page and the forms. |
| Hint | "Double-click a row to open it." on the right of the panel heading. |
| Table columns | 1. **Coach**: a navy circle with the coach's initials (first letters of up to two words), then the name in bold. 2. **Phone number**. 3. **Description**. 4. **Remarks**. 5. **Status**: a green "Active" or grey "Inactive" badge. 6. An **Open** button. |
| Empty values | A blank phone number, description or remark shows as "—". |
| Long text | Description and Remarks show at most two lines in the list. Longer text is cut off with an ellipsis (…). |
| Sort order | By name, A to Z, ignoring upper and lower case. |
| Row actions | Double-clicking a row, or pressing its Open button, opens the coach details page (section 5.3) for that coach. Rows highlight on hover. |
| Empty state: no coaches at all | In place of the table, show "NO COACHES YET", the line "Add the academy's coaches here." and a navy **New coach** button. |
| Empty state: filter has no match | In place of the table, show "No active coaches" or "No inactive coaches" (depending on the filter), with the line "Change the Status filter to see other coaches." |

### 5.2 New coach form

| Element | Specification |
| --- | --- |
| Back link | "Back to coaches", with a left-arrow icon, above the form. It returns to the list without saving. |
| Form heading | Red label "PEAK SPORTS ACADEMY" above the title "NEW COACH". Below it: "Fields marked * are required." |
| Section A · Coach details | Name * and Phone number side by side, then Description across the full width. |
| Section B · Remarks | Remarks across the full width. |
| Bottom bar | Fixed to the bottom of the form. Left: "The new coach is added to the coach list as Active." Right: **Cancel** (white) and **Save coach** (navy, with a tick icon). |

### 5.3 Coach details page

A read-only page showing one coach. It opens when a row on the list is double-clicked or its Open button is pressed.

| Element | Specification |
| --- | --- |
| Back link | "Back to coaches", with a left-arrow icon. It returns to the list. |
| Success message | A green pill below the back link shows "Coach “<name>” saved", "Coach “<name>” activated" or "Coach “<name>” deactivated" after those actions. |
| Page heading | Red label "COACH DETAILS" above the coach's name as the title. Below the name: a green "Active" or grey "Inactive" badge. |
| Buttons | At the top right of the heading: **Edit details** (white, with a pencil icon), which opens the Edit coach form; and **Deactivate** (white with a red outline and red text) when the coach is Active, or **Activate** (white with a green outline and green text) when the coach is Inactive. |
| Section A · Coach details | Name and Phone number side by side, then Description across the full width. Each shows a label above its value. A blank value shows as "—". Line breaks in Description are kept. |
| Section B · Remarks | Remarks, shown the same way. |

### 5.4 Edit coach form

This is the same as the New coach form, with these differences:

| Element | Specification |
| --- | --- |
| Back link | "Back to coach details". It returns to the details page without saving. |
| Form heading | Red label "EDIT COACH". The title is the coach's name and updates as the name is typed. If the name field is cleared, the title shows the saved name. |
| Status | Not shown. Status is changed only on the details page. |
| Bottom bar | Left: "Changes show straight away on the details page and in the coach list." Right: **Cancel** and **Save changes**. |

## 6. Fields

| Field | Type | Required | Max length | Rules |
| --- | --- | --- | --- | --- |
| Name | Text, one line | Yes | 100 characters | Leading and trailing spaces are removed, and repeated spaces inside are reduced to one. Duplicate names are allowed. Placeholder: "Coach's full name". |
| Phone number | Phone (type="tel"), one line | No | 20 characters | Only digits, spaces and one + as the first character can be typed. Any other character is removed as it is typed. If entered, it must contain 9 to 15 digits (spaces and + are not counted). Spaces are cleaned as for Name. Duplicates are allowed. The value is stored as entered, after cleaning. Placeholder: "e.g. 050 123 4567". Help text: "Digits, spaces and a leading + only." |
| Description | Text, several lines (3 rows, resizable) | No | 500 characters | Leading and trailing spaces are removed. Placeholder: "Specialism, qualifications or experience". |
| Remarks | Text, several lines (3 rows, resizable) | No | 500 characters | Leading and trailing spaces are removed. Placeholder: "Internal notes for the office, e.g. availability". |
| Status | Active / Inactive | Set by the system | — | Set to Active when the coach is created. Shown as a badge on the list and the details page. Changed only with the Activate / Deactivate button on the details page. |

## 7. Functional requirements

| ID | Requirement | Acceptance criteria |
| --- | --- | --- |
| CM-01 | Coach list is the landing page | Opening the Coaches Master shows the list with the Active filter selected, sorted by name, with a count such as "2 coaches". |
| CM-02 | Status filter | The Status drop-down lists All, Active and Inactive, in that order, with Active selected when the screen opens. Choosing a value updates the rows and the count straight away. The choice is kept when the user opens a coach and comes back to the list. |
| CM-03 | Empty states | With no coaches saved, the list shows "No coaches yet" and a New coach button, whatever the filter. With coaches saved but none matching the filter, it shows "No active coaches" or "No inactive coaches". |
| CM-04 | Create a coach | New coach opens a blank form with no status field. Saving a valid form adds the coach as Active and returns to the list with the message "Coach “<name>” added". |
| CM-05 | Open a coach | Double-clicking a row, or pressing its Open button, opens the coach details page for that coach, showing all its fields and its status. |
| CM-06 | Edit a coach | Edit details on the details page opens the Edit coach form filled with the coach's details. Saving a valid form updates the coach and returns to the details page with the message "Coach “<name>” saved". The list shows the change straight away. |
| CM-07 | Deactivate or activate a coach | On the details page, Deactivate sets an Active coach to Inactive, and Activate sets an Inactive coach to Active. The change is saved straight away, with no confirmation step, and the page stays open showing "Coach “<name>” deactivated" or "Coach “<name>” activated". The badge and the button change to match. An inactive coach is kept, appears under the Inactive and All filters, and no longer appears under Active. |
| CM-08 | Name validation | Saving with a blank name (or spaces only) shows "Enter the coach's name." under the field, and the field is highlighted in red. |
| CM-09 | Phone validation | A blank phone number is accepted. A phone number with fewer than 9 or more than 15 digits shows "Enter a valid phone number (9 to 15 digits)." under the field, and the field is highlighted in red. |
| CM-10 | Phone characters | Typing or pasting letters or symbols other than a leading + into Phone number removes them straight away. A + typed anywhere but first is removed. |
| CM-11 | When errors show | Errors appear only after the first Save attempt. After that, each error updates as the user types. While any error shows, the bottom bar says "Fix the highlighted field to save." (one error) or "Fix the highlighted fields to save." (two errors). Nothing is saved. |
| CM-12 | Length limits | Name stops accepting input at 100 characters, Phone number at 20, and Description and Remarks at 500 each. |
| CM-13 | Duplicates allowed | Saving a coach with the same name or phone number as another coach succeeds, with no warning. |
| CM-14 | Cancel | On the New coach form, Cancel or "Back to coaches" returns to the list without saving. On the Edit coach form, Cancel or "Back to coach details" returns to the details page without saving. |
| CM-15 | Keyboard and accessibility | All buttons, the Status drop-down and all fields can be reached with Tab and used with the keyboard. Pressing Enter in Name or Phone number saves the form. Every field has a visible label. Buttons and controls are at least 44 px high. Text meets 4.5:1 contrast. |

## 8. Data

### 8.1 PostgreSQL table

| Column | Type | Null | Rules |
| --- | --- | --- | --- |
| coach_id | integer, identity | No | Primary key, generated by the database. |
| name | varchar(100) | No | Must not be blank. |
| phone | varchar(20) | Yes | NULL when not entered. If present: only digits, spaces and a leading +, with 9 to 15 digits. |
| description | varchar(500) | Yes | NULL when not entered. |
| remarks | varchar(500) | Yes | NULL when not entered. |
| is_active | boolean | No | Default true. |

```sql
CREATE TABLE coach (
  coach_id    integer GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  name        varchar(100) NOT NULL CHECK (btrim(name) <> ''),
  phone       varchar(20)  NULL
              CHECK (phone IS NULL OR (
                phone ~ '^\+?[0-9 ]+$'
                AND length(regexp_replace(phone, '[^0-9]', '', 'g')) BETWEEN 9 AND 15)),
  description varchar(500) NULL,
  remarks     varchar(500) NULL,
  is_active   boolean      NOT NULL DEFAULT true
);
```

- Optional fields that are left blank are saved as NULL, not as empty text.
- The list query sorts by lower(name). The filter uses is_active (Active = true, Inactive = false, All = no condition).

### 8.2 Prototype storage (localStorage)

- Key: psa_coaches.
- Value: a JSON array of objects: { "id": string, "name": string, "phone": string | null, "description": string | null, "remarks": string | null, "isActive": boolean }.
- The same cleaning and validation rules apply as in production.

## 9. Visual design

The screen uses the same look as the Location Master and Batch Master.

### 9.1 Colours

| Use | Colour |
| --- | --- |
| Primary (headings, Save buttons, section band, count badge text, links) | #2A3A9E (navy); hover #1B2466 |
| Accent (New coach button, red labels, section letter tab, required *, error outline) | #D7262E (red) |
| Page background | #F3F3F7 |
| Header, panels and bottom bar | #FFFFFF |
| Main text | #15172B |
| Secondary text, hints, column headings | #5E6280 |
| Field labels | #3B3F5C |
| Field background | #ECEAF2 |
| Field background with error | #FDF1F1, with a 2 px #D7262E inside outline |
| Error message text | #A3141B |
| Success pill, Active badge | Background #E3F4EA, text #17663A |
| Inactive badge | Background #ECEDF3, text #5E6280 |
| Deactivate button | White, 2 px #D7262E inside outline, text #A3141B; hover background #FDF1F1 |
| Activate button | White, 2 px #17663A inside outline, text #17663A; hover background #E3F4EA |
| Count badge, Open button | Background #E9ECFA, text #2A3A9E / #1B2466 |
| Lines and borders | #E3E4EC and #ECEDF3; table header row #F7F7FA |

### 9.2 Type

| Element | Font |
| --- | --- |
| Body text, fields, buttons | Barlow, 15–16 px, weights 400–700 |
| Page title "COACHES" | Barlow Condensed 800, 30 px, upper case |
| Form title | Barlow Condensed 800, 44 px, upper case |
| Panel heading, section band titles | Barlow Condensed 700, 18–20 px, upper case, wide letter spacing |
| Small labels ("MASTERS", column headings) | Barlow 700, 12 px, upper case, wide letter spacing |

### 9.3 Layout

- White header bar, at least 76 px high, with 32 px side padding.
- Content up to 1240 px wide with 32 px padding. Panels have 14 px rounded corners.
- Section bands: a red 52 px tab with the section letter (A, B), joined to a navy 44 px bar with the section title.
- Fields are 46 px high with 6 px rounded corners and no border. The form fields sit in a grid that drops to one column on narrow screens.
- Buttons are 44–48 px high with 8 px rounded corners.
- On narrow screens the table scrolls sideways inside its panel, and the header and bottom bar wrap.

## 10. Prototype

A clickable prototype is on the "Coaches Master" design canvas. Where the prototype and this specification differ, follow this specification. The prototype and the downloadable index.html both follow this specification. The prototype starts with three sample coaches (one inactive) to show the filter; index.html starts with no coaches.
