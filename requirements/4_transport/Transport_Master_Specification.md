# Transport Master

**Feature specification · Peak Sports Academy fee app**

Version 1.1 · 11 November 2026

## 1. Purpose

The Transport Master holds the academy's transport routes. For each route it records a route number, a location, a landmark, a description, remarks, the driver's name, the driver's mobile number and whether the route is active.

## 2. Scope

### 2.1 In this feature

- Route list (landing page) with an Active / Inactive / All filter.
- New route form.
- Edit route form, including changing a route between Active and Inactive.
- Field validation, messages and empty states.
- Storage in browser localStorage (prototype) and PostgreSQL (production).

### 2.2 Not part of this feature

- Deleting routes. Routes no longer used are set to Inactive instead.
- Assigning students to routes, and transport fees.
- A separate driver master. Driver details are stored on each route.
- Searching the route list.
- Importing or exporting routes.
- User roles or permissions.

## 3. Users and environment

These are the same as for the Location Master, Batch Master and Coaches Master.

- **Users:** academy admin staff, using one shared login. No role-based permissions.
- **Device:** the academy office PC, in Chrome or Edge. The office PC has no internet connection.
- **Prototype:** plain HTML, CSS and JavaScript, with no frameworks and no build step. Data is stored in browser localStorage. The app opens by double-clicking index.html.
- **Production:** a local web app backed by PostgreSQL, using the table in section 8.
- **Fonts:** Barlow and Barlow Condensed. With no internet, the app must either bundle the font files locally or fall back to system fonts without breaking the layout.

## 4. Confirmed decisions

These decisions were confirmed by the product owner on 11 November 2026.

| ID | Decision |
| --- | --- |
| D1 | Route number is required, with a maximum of 100 characters. |
| D2 | Two routes cannot have the same route number. The comparison ignores upper and lower case and extra spaces, so "R1", "r1" and " R1 " are the same. The check covers all routes, active and inactive. |
| D3 | Location is required. It is free text (not a drop-down from the Location Master), with a maximum of 255 characters. |
| D4 | Landmark is optional free text, with a maximum of 255 characters. |
| D5 | Description and Remarks are optional, with a maximum of 500 characters each. |
| D6 | Driver name is required, with a maximum of 100 characters. Two routes may have the same driver name. |
| D7 | Driver mobile number is optional and follows the same rule as the coach phone number: only digits, spaces and one leading + are allowed; if entered, it must contain 9 to 15 digits; maximum 20 characters. Two routes may have the same driver mobile number. |
| D8 | Routes are never deleted. Each route has a status of Active or Inactive, the same as the Coaches Master. |
| D9 | A new route is always saved as Active. The status switch appears only when editing an existing route. |
| D10 | The route list has a Status drop-down filter with Active, Inactive and All, the same as in the Coaches Master. Active is selected by default. |
| D11 | The route list is sorted by route number, A to Z, in natural order (Route 2 comes before Route 10), ignoring upper and lower case. |

## 5. Screens

### 5.1 Route list (landing page)

| Element | Specification |
| --- | --- |
| Header | Small red label "MASTERS" above the title "TRANSPORT". A red **New route** button with a + icon sits at the top right. The button shows only on the list, not on the forms. |
| Success message | A green pill above the list shows "Route “<route number>” added" or "Route “<route number>” saved" after a save. It disappears when a form is opened. |
| Panel heading | The heading "ALL ROUTES" is followed by a count badge, such as "1 route" or "5 routes". The count is the number of routes shown under the current filter. |
| Status filter | A drop-down next to the count, labelled "Status", the same as in the Coaches Master. Options, in this order: **Active**, **Inactive**, **All**. Active is selected by default when the screen opens. Choosing an option updates the list straight away (no Apply button). The choice is kept while the user moves between the list and the forms. |
| Hint | "Double-click a row to open it." on the right of the panel heading. |
| Table columns | 1. **Route number**: a navy circle with a bus icon, then the route number in bold. 2. **Location**. 3. **Landmark**. 4. **Driver name** (semi-bold). 5. **Driver mobile**. 6. **Description**. 7. **Remarks**. 8. **Status**: a green "Active" or grey "Inactive" badge. 9. An **Open** button. |
| Empty values | A blank landmark, driver mobile, description or remark shows as "—". |
| Long text | Location, Landmark, Driver name, Description and Remarks show at most two lines in the list. Longer text is cut off with an ellipsis (…). |
| Sort order | By route number, A to Z, in natural order, ignoring upper and lower case (decision D11). |
| Row actions | Double-clicking a row, or pressing its Open button, opens the Edit route form for that route. Rows highlight on hover. |
| Empty state: no routes at all | In place of the table, show "NO ROUTES YET", the line "Add the academy's transport routes here." and a navy **New route** button. The filter stays visible. |
| Empty state: filter has no match | In place of the table, show "No active routes" or "No inactive routes" (depending on the filter), with the line "Change the filter to see other routes." |
| Narrow screens | The table scrolls sideways inside its panel. |

### 5.2 New route form

| Element | Specification |
| --- | --- |
| Back link | "Back to transport", with a left-arrow icon, above the form. It returns to the list without saving. |
| Form heading | Red label "PEAK SPORTS ACADEMY" above the title "NEW ROUTE". Below it: "Fields marked * are required." |
| Section A · Route details | Route number * and Location * side by side, then Landmark, then Description across the full width. |
| Section B · Driver details | Driver name * and Driver mobile number side by side. |
| Section C · Remarks | Remarks across the full width. |
| Focus | When the form opens, the cursor is in Route number. |
| Bottom bar | Fixed to the bottom of the form. Left: "The new route is added to the route list as Active." Right: **Cancel** (white) and **Save route** (navy, with a tick icon). |

### 5.3 Edit route form

This is the same as the New route form, with these differences:

| Element | Specification |
| --- | --- |
| Form heading | Red label "ROUTE DETAILS". The title is the route number and updates as it is typed. If the route number field is cleared, the title shows the saved route number. |
| Status switch | An extra field in Section A, labelled "Status", shown after Landmark. It is a switch showing **Active** or **Inactive**, set to the route's current status. The change is saved only when the user presses Save changes. |
| Bottom bar | Left: "Changes show straight away in the route list." Right: **Cancel** and **Save changes**. |

## 6. Fields

| Field | Type | Required | Max length | Rules |
| --- | --- | --- | --- | --- |
| Route number | Text, one line | Yes | 100 characters | Leading and trailing spaces are removed, and repeated spaces inside are reduced to one. Must not match another route's number, ignoring upper and lower case (D2). Placeholder: "Route number". |
| Location | Text, one line | Yes | 255 characters | Free text. Spaces are cleaned as for Route number. Placeholder: "Location covered by this route". |
| Landmark | Text, one line | No | 255 characters | Free text. Spaces are cleaned as for Route number. Placeholder: "Nearby landmark". |
| Description | Text, several lines (3 rows, resizable) | No | 500 characters | Leading and trailing spaces are removed. Placeholder: "Description of the route". |
| Driver name | Text, one line | Yes | 100 characters | Spaces are cleaned as for Route number. Duplicates are allowed. Placeholder: "Driver's full name". |
| Driver mobile number | Phone (type="tel"), one line | No | 20 characters | Only digits, spaces and one + as the first character can be typed. Any other character is removed as it is typed or pasted. If entered, it must contain 9 to 15 digits (spaces and + are not counted). Spaces are cleaned as for Route number. Duplicates are allowed. The value is stored as entered, after cleaning. Placeholder: "e.g. 050 123 4567". Help text: "Digits, spaces and a leading + only." |
| Remarks | Text, several lines (3 rows, resizable) | No | 500 characters | Leading and trailing spaces are removed. Placeholder: "Internal notes for the office". |
| Status | Switch: Active / Inactive | Set by the system | — | Set to Active when the route is created. Shown and editable only on the Edit route form. |

### 6.1 Validation messages

| Field | Condition | Message under the field |
| --- | --- | --- |
| Route number | Blank, or spaces only | Enter the route number. |
| Route number | Matches another route's number (D2) | Another route already has this route number. |
| Location | Blank, or spaces only | Enter the location. |
| Driver name | Blank, or spaces only | Enter the driver's name. |
| Driver mobile number | Entered, with fewer than 9 or more than 15 digits | Enter a valid mobile number (9 to 15 digits). |

## 7. Functional requirements

| ID | Requirement | Acceptance criteria |
| --- | --- | --- |
| TM-01 | Route list is the landing page | Opening the Transport Master shows the list with Active selected in the Status drop-down, sorted by route number in natural order, with a count such as "2 routes". |
| TM-02 | Status filter | Choosing Active, Inactive or All in the Status drop-down updates the rows and the count straight away. The choice is kept when the user opens a route and comes back. |
| TM-03 | Empty states | With no routes saved, the list shows "No routes yet" and a New route button, whatever the filter. With routes saved but none matching the filter, it shows "No active routes" or "No inactive routes". |
| TM-04 | Create a route | New route opens a blank form with no Status switch. Saving a valid form adds the route as Active and returns to the list with the message "Route “<route number>” added". |
| TM-05 | Open a route | Double-clicking a row, or pressing its Open button, opens the Edit route form filled with that route's details and status. |
| TM-06 | Edit a route | Saving a valid form updates the route and returns to the list with the message "Route “<route number>” saved". The list shows the change straight away. |
| TM-07 | Make a route inactive or active again | On the Edit route form, switching Status and saving changes the route's status. An inactive route is kept, appears under the Inactive and All filters, and no longer appears under Active. It can be switched back to Active the same way. |
| TM-08 | Required fields | Saving with a blank Route number, Location or Driver name (or spaces only) shows that field's message from section 6.1, and the field is highlighted in red. |
| TM-09 | Unique route number | Saving a route number that matches another route, ignoring upper and lower case and extra spaces, shows "Another route already has this route number." This applies when the other route is inactive too. Saving an existing route without changing its own route number is allowed. |
| TM-10 | Driver mobile validation | A blank driver mobile number is accepted. A number with fewer than 9 or more than 15 digits shows "Enter a valid mobile number (9 to 15 digits)." and the field is highlighted in red. |
| TM-11 | Driver mobile characters | Typing or pasting letters or symbols other than a leading + into Driver mobile number removes them straight away. A + typed anywhere but first is removed. |
| TM-12 | When errors show | Errors appear only after the first Save attempt. After that, each error updates as the user types. While any error shows, the bottom bar says "Fix the highlighted field to save." (one error) or "Fix the highlighted fields to save." (two or more errors), the cursor moves to the first field with an error, and nothing is saved. |
| TM-13 | Length limits | Route number and Driver name stop accepting input at 100 characters, Location and Landmark at 255, Driver mobile number at 20, and Description and Remarks at 500 each. |
| TM-14 | Duplicates allowed | Saving a route with the same driver name or driver mobile number as another route succeeds, with no warning. |
| TM-15 | Cancel | Cancel or "Back to transport" returns to the list without saving any changes. |
| TM-16 | Saved data | Routes are kept after the browser is closed and reopened. Blank optional fields are saved as empty (NULL), not as empty text. |
| TM-17 | Keyboard and accessibility | All buttons, the Status drop-down, the status switch and all fields can be reached with Tab and used with the keyboard. Pressing Enter in any one-line field saves the form. Every field has a visible label. Buttons and controls are at least 44 px high. Text meets 4.5:1 contrast. |

## 8. Data

### 8.1 PostgreSQL table

| Column | Type | Null | Rules |
| --- | --- | --- | --- |
| route_id | integer, identity | No | Primary key, generated by the database. |
| route_number | varchar(100) | No | Must not be blank. Unique, ignoring upper and lower case (the app cleans spaces before saving). |
| location | varchar(255) | No | Must not be blank. |
| landmark | varchar(255) | Yes | NULL when not entered. |
| description | varchar(500) | Yes | NULL when not entered. |
| remarks | varchar(500) | Yes | NULL when not entered. |
| driver_name | varchar(100) | No | Must not be blank. |
| driver_mobile | varchar(20) | Yes | NULL when not entered. If present: only digits, spaces and a leading +, with 9 to 15 digits. |
| is_active | boolean | No | Default true. |

```sql
CREATE TABLE transport_route (
  route_id      integer GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  route_number  varchar(100) NOT NULL CHECK (btrim(route_number) <> ''),
  location      varchar(255) NOT NULL CHECK (btrim(location) <> ''),
  landmark      varchar(255) NULL,
  description   varchar(500) NULL,
  remarks       varchar(500) NULL,
  driver_name   varchar(100) NOT NULL CHECK (btrim(driver_name) <> ''),
  driver_mobile varchar(20)  NULL
                CHECK (driver_mobile IS NULL OR (
                  driver_mobile ~ '^\+?[0-9 ]+$'
                  AND length(regexp_replace(driver_mobile, '[^0-9]', '', 'g')) BETWEEN 9 AND 15)),
  is_active     boolean      NOT NULL DEFAULT true
);

-- D2: one route number per route, ignoring case
CREATE UNIQUE INDEX transport_route_number_uq
  ON transport_route (lower(route_number));

-- D11: natural, case-insensitive sort order
CREATE COLLATION IF NOT EXISTS route_natural
  (provider = icu, locale = 'en-u-kn-true-ks-level2', deterministic = false);
```

- Before saving, the app removes leading and trailing spaces and reduces repeated spaces to one in Route number, Location, Landmark, Driver name and Driver mobile number. Description and Remarks only have leading and trailing spaces removed.
- Optional fields that are left blank are saved as NULL, not as empty text.
- The list query sorts with ORDER BY route_number COLLATE route_natural. The filter uses is_active (Active = true, Inactive = false, All = no condition).
- The unique index is the final guard for D2. If an insert or update fails on it, the app shows "Another route already has this route number." under Route number.

### 8.2 Prototype storage (localStorage)

- Key: psa_transport_routes.
- Value: a JSON array of objects: { "id": string, "routeNumber": string, "location": string, "landmark": string | null, "description": string | null, "driverName": string, "driverMobile": string | null, "remarks": string | null, "isActive": boolean }.
- The same cleaning and validation rules apply as in production. Sorting uses localeCompare with { numeric: true, sensitivity: "base" }.
- If the browser cannot write to localStorage, the bottom bar shows "Could not save. Browser storage is not available." and the form stays open with nothing saved.

## 9. Visual design

The screen uses the same look as the Location Master, Batch Master and Coaches Master.

### 9.1 Colours

| Use | Colour |
| --- | --- |
| Primary (headings, Save buttons, section band, count badge text, links, route icon circle) | #2A3A9E (navy); hover #1B2466 |
| Accent (New route button, red labels, section letter tab, required *, error outline) | #D7262E (red) |
| Page background | #F3F3F7 |
| Header, panels and bottom bar | #FFFFFF |
| Main text | #15172B |
| Secondary text, hints, column headings | #5E6280 |
| Field labels | #3B3F5C |
| Field background, status switch background, Status drop-down background | #ECEAF2; drop-down arrow #2A3A9E |
| Field background with error | #FDF1F1, with a 2 px #D7262E inside outline |
| Error message text | #A3141B |
| Success pill, Active badge, switch track when Active | Background #E3F4EA, text #17663A; switch track #17663A |
| Inactive badge | Background #ECEDF3, text #5E6280; switch track when Inactive #B9BBCB |
| Count badge, Open button | Background #E9ECFA, text #2A3A9E / #1B2466 |
| Lines and borders | #E3E4EC and #ECEDF3; table header row #F7F7FA |

### 9.2 Type

| Element | Font |
| --- | --- |
| Body text, fields, buttons | Barlow, 15–16 px, weights 400–700 |
| Page title "TRANSPORT" | Barlow Condensed 800, 30 px, upper case |
| Form title | Barlow Condensed 800, 44 px, upper case |
| Panel heading, section band titles | Barlow Condensed 700, 18–20 px, upper case, wide letter spacing |
| Small labels ("MASTERS", column headings) | Barlow 700, 12 px, upper case, wide letter spacing |

### 9.3 Layout

- White header bar, at least 76 px high, with 32 px side padding.
- Content up to 1240 px wide with 32 px padding. Panels have 14 px rounded corners.
- Section bands: a red 52 px tab with the section letter (A, B, C), joined to a navy 44 px bar with the section title.
- Fields are 46 px high with 6 px rounded corners and no border. The form fields sit in a two-column grid that drops to one column on narrow screens.
- Buttons are 44–48 px high with 8 px rounded corners. The Status drop-down is 44 px high and at least 150 px wide, with 6 px rounded corners, no border and a navy down-arrow on the right; its label "Status" sits to its left.
- On narrow screens the table scrolls sideways inside its panel, and the header and bottom bar wrap.

## 10. Prototype

The clickable prototype is the file **index.html** supplied with this specification. It follows every rule in this document. It starts with no routes; add routes with New route. Routes are kept in the browser under the key psa_transport_routes. To start again, clear the site data for the file in the browser.

The "Transport Master" design canvas was made before decisions D8 to D11 and does not have the status switch, the Status drop-down or natural sorting. Where the canvas differs from this specification, follow this specification and index.html.

## 11. Acceptance checklist

1. Open index.html with no saved data: "No routes yet" shows, with Active selected in the Status drop-down.
2. Press Save route on an empty form: Route number, Location and Driver name show their messages, and the bar says "Fix the highlighted fields to save."
3. Add route "Route 10" with a location and driver: it appears under Active with the message "Route “Route 10” added".
4. Add "Route 2": it is listed above "Route 10".
5. Try to add "route 2" or "  Route   2 ": the duplicate message shows and nothing is saved.
6. Type "05a0-12" into Driver mobile number: the letter and the dash are removed and the field shows "05012". Save with 8 digits in the field: "Enter a valid mobile number (9 to 15 digits)." shows.
7. Leave Landmark, Driver mobile, Description and Remarks blank and save: the list shows "—" in those columns.
8. Open a route, switch Status to Inactive and save: it disappears from Active and appears under Inactive and All with a grey badge.
9. Add a new route with the same number as the inactive route: the duplicate message shows.
10. Close and reopen the browser: all routes, statuses and details are still there.
