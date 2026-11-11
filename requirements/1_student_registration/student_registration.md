# Peak Sports Academy Student Registration PRD

**Version:** 1.1, 08 November 2026  
**Status:** Provisional draft for stakeholder review; not an approval or an implementation-readiness declaration.  
**Source:** [PSA Student Registration Requirements](PSA_Student_Registration_Requirements.docx), version 2.1, 08 November 2026, especially sections 1–16 and acceptance tests AT-01–AT-59. The source describes its Phase 1 rules as confirmed; this PRD carries that status as a source-reported decision, not independent evidence of stakeholder approval or validation.  
**Ledger:** Initial integrated ledger and decision log in this document, version 1.1. No separate project ledger was supplied. Owner and approval authority are unknown. The source references a prior decision log and original requirements, which were not provided as current inputs; source v2.1 takes precedence over older wording and the prototype.

> This draft captures the supplied Phase 1 requirements. Open design and operational decisions remain below. Proposed acceptance interpretations and implementation details are not additional approved product scope.

## 1. Product overview

Peak Sports Academy (PSA) needs a staff-operated application to register and maintain students, their contact and cricket information, documents, status and history. Staff need to find existing students, avoid duplicate Emirates IDs, export filtered lists, and recover local data. The source establishes the desired workflow but does not provide a baseline, customer research or measured impact.

The first release supports Cricket and is organized for later academy modules. It runs offline after installation on one compatible Windows PC, with a browser frontend, local backend and database. The planning volume is approximately 1,000 students. React with Vite, Python FastAPI and local PostgreSQL are confirmed technology constraints (ENV-01–02).

### Goals and measures

| ID | Desired outcome | Measure and evidence | Linked scope |
| --- | --- | --- | --- |
| G-01 | Register and maintain a complete student record with a unique identity and registration number. | Successful end-to-end registration and duplicate/validation scenarios; baseline and business target not supplied. | J-01, J-02, FLW, VAL, NUM |
| G-02 | Find students and export the intended filtered population. | List, dashboard and export criteria AT-27–31 and AT-45; usage and time targets open. | J-03, LST, EXP |
| G-03 | Preserve records, documents and accountable change history, and recover a consistent set. | Controlled backup, restore, change and failure tests AT-17–19, AT-32–42, AT-47–55. Recovery time target open. | J-02, J-04, DOC, HIS, SET, BAK, OPS |

## 2. Scope and users

**Phase 1 source-confirmed scope:** registration and editing, profile and status, identity checks, student documents, per-student and Settings histories, Students search/filtering, two dashboard cards, Excel/PDF exports, local backup/restore, Settings, installation and upgrade (SCP-01–02). The document does not assign relative MoSCoW priorities; treating all as release commitments preserves the source rather than inventing a smaller approved MVP. Dependencies and sequencing require technical planning.

**Excluded from Phase 1 (SCP-03–04):** Excel import/migration; global student history; extra export columns and Additional Fields selector; fees/payments, batches and locations, assignments, coaches, transport, attendance and sessions; parent/student accounts and self-registration; multiple roles, public SaaS, password recovery, and digital terms, declarations, signatures or paper submission date/time. Later academy modules and sports are roadmap areas without approved detailed rules or dates. No Phase 2 commitment is specified.

| Actor | Access and responsibility | Boundary |
| --- | --- | --- |
| U-01 Staff/admin | Sign in and operate all Phase 1 information and features, including student management, Settings and recovery. One local account starts as `admin` / `admin`; password change is optional. | No separate application role restrictions or parent/student login. Authentication applies to application and API access. |
| U-02 Windows installer administrator | Installs and upgrades the application and prerequisites. | Normal use must work under a standard Windows account without elevation; a shared PostgreSQL server is managed by its administrator. |

The source does not specify an accessibility standard, supported browser/version matrix, languages, or device targets beyond the compatible local Windows PC (Q-03, Q-04).

## 3. Journeys

- **J-01 Create:** From Students, open New Registration, complete the six-section form, optionally attach files, and Save. Validation and identity checks precede a successful creation. The application issues a number, creates one Active record and one Created history entry, then opens the profile with a success message and number. Cancel/navigation with unsaved changes warns before discard and creates no record (FLW-01–04; AT-01–04, AT-17).
- **J-02 Maintain:** Double-click a list row to open the profile, then edit fields, status or attachments. Save validates and commits fields, files and history consistently; Cancel discards pending changes. Deactivate/reactivate retains the record and its history and files. On duplicate Emirates ID, route to the existing student, including the option to reactivate an inactive record (FLW-05–10; AT-18–20, AT-35–37).
- **J-03 Find and export:** Search and filter Students or follow a dashboard card to a matching list. Return from profile with list context preserved. Export all matching rows across pages to Excel or PDF (LST-01–08; EXP-01–03; AT-27–31, AT-45).
- **J-04 Protect and recover:** Configure valid storage and database at setup; create and verify manual or required safety backups; restore a verified archive with prior-state recovery on failure. Upgrades require a safety backup and retain existing data. Restore uses the target PC's configured folders and current application password (BAK-01–11; OPS-01–06; AT-46–54).

## 4. Student fields and validation

The source field IDs are retained. Requiredness and validation apply on both create and edit. Trim text outer whitespace, preserve case and internal spaces; free-text student fields have a 255-character maximum except for the separately specified identity and phone rules (VAL-04). Mandatory fields are full name, gender, DOB, nationality, Emirates ID, passport number, joining date and at least one phone with its code (VAL-03).

| ID | Field | Requirement |
| --- | --- | --- |
| P01 | Registration Number | Issued on successful creation; editable later; configured format and case-insensitive uniqueness across current Active and Inactive students, subject to retained-number exception in NUM-10. |
| P02 | Full Name | Required; English letters and spaces only, maximum 255; first name determines folder component. |
| P03 | Nickname | Optional free text, maximum 255. |
| P04 | Gender | Required; exactly Male, Female or Others. |
| P05–06 | Date of Birth; Age | DOB required, date picker prevents future dates and server rejects them; Today permitted, no age eligibility limits. Age is read-only completed whole years calculated from DOB to UAE Today. |
| P07 | Nationality | Required free text, maximum 255. |
| P08 | Emirates ID Number | Required 15 digits in `XXX-XXXX-XXXXXXX-X` format; automatically insert hyphens while digits are typed on create and edit (VAL-06); identity duplicate key across both statuses. **Acceptance:** AT-58. |
| P09 | Passport Number | Required letters/digits only, 5–20 characters; duplicate passport alone does not block save. |
| P10 | Email | Optional; validate syntax when populated; not a duplicate key. |
| P11–12 | Phones 1 and 2 and their country codes | At least one number required, each entered number requires its own code. Local number digits only, maximum 20, no additional minimum, decimals, spaces or dashes; preserve leading zeros. Editable codes default to +971, support leading plus and preserve zeros; no steppers or numeric conversion that loses zeros (VAL-01). |
| P13–14 | Height cm; Weight kg | Optional positive whole numbers; reject zero, negatives and decimals. |
| P15–17 | Blood Group; School Name; Residential Address | Optional free text; address multiline; blood group is not a fixed dropdown. Maximum 255 each. |
| C01–04 | Previous Cricket Coaching and Details; Part of Cricket Team and Details | Each Yes/No radio is optional and may remain unanswered. Details are optional, always visible/editable, maximum 255 each. Changing an answer never clears details; Yes does not make details mandatory (VAL-02). |
| M01 | Medical Conditions | Optional free text for illness, injury, surgery or ongoing treatment, maximum 255. |
| G01–04 | Father Name and Occupation; Mother Name and Occupation | Optional free text, maximum 255 each. No separate guardian account, emergency-contact module, relationship management or sibling linkage. |
| D01–03, O04 | Passport, Emirates ID, Photo, Visa files | Each optional, one current file per slot. Passport, single Emirates ID slot and separate Visa accept JPG/JPEG/PNG/PDF; Photo accepts JPG/JPEG/PNG only. Per-file limit configurable, initially 10 MB. Photo displays only on profile, not list. |
| O01–03 | Joining Date; Student Status; Insurance Card Details | Joining Date required, defaults to UAE Today, editable, past/current/future allowed with no DOB-relative restriction. Status starts Active and is later Active or Inactive only. Insurance details are optional text, maximum 255, with no insurance upload. |

Show inline validation messages below invalid fields, highlight errors and focus the first invalid field; validate on client and server (VAL-05). Display dates DD-MMM-YYYY; use Asia/Dubai for Today, ages, defaults, filters, dashboard periods, history and backup timestamps (ENV-06).

## 5. Functional and business requirements

All IDs in this section refer to source-confirmed Phase 1 requirements, with the source document as the authoritative detailed specification. Criteria references identify verification coverage; they do not claim a passing implementation.

### Registration, editing and identity

- **FLW-01–10:** Use a single registration page with Personal, Cricket, Medical, Parents, Documents and Office Use sections in a clean two-column layout; wider fields may span both columns. Save and Cancel are the only form actions. Age is read-only; other ordinary fields including registration number are editable. Save must persist student changes and history together; failure or cancellation is not a completed modification. Folder/file changes and database references must be one recoverable logical operation. There is no student deletion; inactive records, current files, versions and history are retained indefinitely. Reject another student's Emirates ID across both statuses but exclude the record being edited from its own duplicate check. Provide a route to the matching profile. **Acceptance:** AT-01–20, AT-37, AT-55–57. Supports G-01, G-03 and J-01–02.

### Registration numbers

- **NUM-01–04, NUM-10:** Configure an English alphanumeric prefix preserving chosen case and a blank, hyphen or underscore separator. Generate a numeric sequence padded to at least four digits and growing beyond 9999, continuing after the highest issued sequence rather than deliberately filling gaps. Current numbers are unique case-insensitively across both statuses. An unused formerly assigned number may be manually reused if it matches the current configured format; associations remain with the correct student. After a format change with existing numbers retained, an unchanged old-format number remains valid during unrelated edits; a changed number must match current format. **Acceptance:** AT-21–24, AT-56.
- **NUM-05–09:** Show prefix, separator and initially unchecked Update existing registration numbers inline in Settings. Unchecked preserves existing values and applies new configuration to future numbers. Checked previews old/new values, preserves numeric parts and padding, and blocks the entire operation on conflicts. Before applying, create a successful safety backup; update numbers, folders, all current/previous filenames, references and student/Settings histories consistently with recovery on failure. After an older restore where latest issued sequence is unknown, require the last issued number or a previously unused prefix before issuing new automatic numbers. **Acceptance:** AT-24–26, AT-51.

### List, dashboard and navigation

- **LST-09:** Show Dashboard, Students and Settings sidebar icons matching `protoype/student_registration/index_v1.html` alongside their labels, without missing-character placeholders. **Acceptance:** AT-59.

- **LST-01–08:** Students shows exactly Registration Number, Full Name, Phone 1, Phone 2 and Date of Joining; phones include their codes. Default to Active; support status, any joining month/year, and case-insensitive partial search across registration number, name, Emirates ID, passport and both phones, combining filters without extra normalization. Sort registration number ascending with numeric part compared numerically. Page size is 20 initially, configurable only to 20 or 50. Double-click opens profile, with search, filters and page position retained on return. No photo, status, ID or row-level View/Edit controls in the listing. **Acceptance:** AT-27–29.
- **Dashboard (LST-07–08):** Exactly two cards: Total Students counts Active only; Recently Registered counts Active students whose Joining Date is in the current UAE calendar month. A card click clears search, applies corresponding Active and optional month/year filters, starts at page one and yields the matching population. No global student history action; history is accessed through the individual profile and Settings history through Settings. **Acceptance:** AT-30–31, AT-48.

### Documents and student history

- **DOC-01–10:** Store file content in the configured local Student Documents base folder and metadata/references in PostgreSQL. Folder is `RegistrationNumber_FirstName`, where FirstName is the first token of trimmed full name; if the component makes the folder too long, use its first 20 characters in the folder only. Validate paths and reject a change if remaining filesystem work fails. Current file name is `registrationnumber_documenttype` plus its actual accepted extension. Replacement retains prior file as sequential `_v1`, `_v2`, etc., with its original extension; current file has no suffix. Preview and download current and previous versions. Remove physically deletes the current file on successful Save, clears its slot and leaves prior versions; it does not archive the removed current file. Stage additions, replacements and removals until Save; Cancel or failed save preserves committed files. Number changes rename the folder and current/previous files and update references; first-name component changes rename the folder. Changing the base path requires the administrator to move the existing tree manually; the application must not move it automatically and must validate access and references. **Acceptance:** AT-32–38.
- **HIS-01–08:** Per-student history shows current registration number once above a newest-first list; retain stable student association despite renumbering/reuse. Record date/time in UAE, changed field/event, Action, From, To and Remarks, and persist actor. Actions are Create, Modify, Deactivate, Activate. Exactly one Created/Create entry for creation, including initial attachments. An ordinary save creates one Modify row per changed field with one operation timestamp; status transitions get one Activate/Deactivate entry and no duplicate status Modify. Attachment changes after creation use Modify with type and actual change in Remarks; new/replacement uses “New image attached”, removal describes removal, and rename alone is not a new attachment. Bulk renumbering logs each changed student. Preserve old audit values. One case-insensitive partial search covers displayed row values; no extra filters or history export. No false Modify entries for unchanged saves. **Acceptance:** AT-18–19, AT-35, AT-39–40, AT-57.

### Settings and session

- **SET-01–07:** Academy Name is mandatory; address, phone, email and logo optional. PDF displays all supplied academy details. Logo accepts JPG/JPEG/PNG within configurable upload limit and replacement retains numbered prior versions with actual extensions. Settings also include password change, inactivity timeout (initially 30 minutes), numbering, document and backup folders, manual backup and restore, upload limit (initially 10 MB), page size and Windows startup On/Off. Desktop shortcut remains available. Save/Cancel stages configuration and logo changes; commit values, related files and history only on successful Save. Record one Modify row per changed setting and no initial Created row. Password history shows Hidden/Hidden with “Password changed”; never record password or hash in history. Backup/restore operation metadata is separate from Settings change history. **Acceptance:** AT-41–44, AT-57.
- **SES-01–03:** Warn one minute before inactivity expiry with Stay Signed In and Logout. On expiry block session/API access but retain unsaved form work in application memory for the same administrator to sign in and resume. Work remains unsaved and subject to validation/Save/Cancel. This is not a persistent draft or browser-restart recovery feature. **Acceptance:** AT-44.

### Exports

- **EXP-01–03:** Excel and PDF include all students matching the current search and filters across all pages, with exactly five columns in order: Registration Number, Full Name, Phone 1, Phone 2, Date of Joining. Include phone codes, preserve zeros and display dates DD-MMM-YYYY. PDF includes supplied academy details and optional logo in the header, separate from student data. No additional column selector. **Acceptance:** AT-43, AT-45.

### Backup, restore and installation

- **BAK-01–06:** Manual Back Up Now and a separate backup folder; no scheduled backup or automatic purge. Retain backups until manually removed from the folder. Create a single self-contained `PSA_Backup_YYYYMMDD_HHMMSS` archive containing a consistent recoverable set of database, student records/history, settings/history, account and numbering state, current/previous student files and current/retained logos. Archive format is an engineering choice; encryption/password are not required. Validate destination access/capacity, reject unsafe paths, warn when on live-data drive and verify contents before success. Show date/time, size and location. Incomplete backup is not a successful restore source. Dashboard warns immediately if no successful backup exists or the last is more than 30 days old. **Acceptance:** AT-47–48.
- **BAK-07–11:** Restore a selected verified backup after warning of overwrite and creating a successful safety backup; failed safety backup blocks restore. Use currently configured destination folders, including on a replacement compatible PC; restore matching files and records consistently while retaining the target installation's current admin password. Invalid, failed or interrupted restore must preserve or recover the prior consistent state and explain the failure. Apply NUM-09's older-backup numbering safeguard. **Acceptance:** AT-49–51.
- **OPS-01–06:** Windows installer configures the app, backend services, PostgreSQL and shortcut; reuse an existing PostgreSQL installation with a separate PSA database, otherwise install/configure PostgreSQL. First-run setup configures academy details, database, documents folder, backup folder and numbering; registration is blocked until database and folders work. Internet is permitted during installation, but normal operation, previews, file access and exports are offline. Detect existing installation, offer upgrade, make a safety backup and preserve all records, versions, histories and Settings without wiping. On DB failure retry and provide troubleshooting; do not restart a shared PostgreSQL server. Show clear operational errors, not stack traces, and log unexpected technical failures locally. **Acceptance:** AT-46, AT-52–55.

## 6. Acceptance and verification

The source's **AT-01–AT-59** are the detailed acceptance criteria and form part of this PRD by reference; their original wording and expected results govern. Use synthetic records, controlled failures and a compatible Windows environment. The requirement-group links above map the criteria to journeys and goals. In particular, verify server-side uniqueness under repeated/racing saves, file/database rollback, status and history integrity, archive integrity, portable restoration and backup-before-upgrade. No scenario is asserted to have passed.

The following implementation details remain incomplete rather than silently becoming acceptance commitments: supported environment versions and performance thresholds (Q-03), detailed session and filesystem recovery design (Q-04), archive format and compatibility (Q-05), and ownership of product approval (Q-01). The source's engineering handover explicitly requires these to be designed and tested.

## 7. Data, constraints and nonfunctional outcomes

- **Identity and integrity:** A stable student identity must keep history, files and associations attached to the correct person even when display numbers change or are reused. Persistence must enforce Emirates ID and case-insensitive current registration number uniqueness, including concurrent saves (NUM-03, FLW-10, OPS-06). Physical schema and API contracts are engineering decisions.
- **Security and privacy:** Authentication protects local application/API access; all authenticated users share the same Phase 1 authority. Password values and hashes must not appear in history or ordinary diagnostics. The supplied default credentials and optional change are a confirmed constraint; assess operational exposure before deployment without silently changing that rule (ENV-03–04, SET-04, SES-02).
- **Reliability:** Student/Settings saves, file operations, bulk conversion, restore and upgrade must not acknowledge partial business changes. Retain/recover the prior consistent state on specified failures. Unexpected failures go to local technical logs (FLW-07–08, DOC-08–09, SET-07, NUM-08, BAK-10, OPS-06).
- **Offline and environment:** Local operation after installation, including previews and exports; approximately 1,000 students is a planning volume, not a performance guarantee. Benchmark response, export, backup and restore with realistic file/history volumes; targets and measurement conditions are delegated to technical design, not fixed in the source (ENV-01–02, ENG-01).
- **Accessibility, scale and support:** Accessibility standard and measurable targets, workload distributions, supported Windows/browser/PostgreSQL versions, support owner and recovery objectives are open (Q-03–04). AI is not in scope; no AI requirements are inferred.
- **Dependencies:** Working local database and accessible document/backup folders before registration; filesystem permissions and capacity; administrator-managed relocation when the document base changes; safety backup before bulk conversion, restore and upgrade. Backup format, file-content validation, path safety, case-only renames and interruption recovery need design and testing.

## 8. Decisions, assumptions, risks and open questions

| ID | State | Item and impact | Owner / next action |
| --- | --- | --- | --- |
| D-01 | Source-confirmed | Phase 1 scope, technology and local/offline environment are as in SCP and ENV. Prior source decisions are consolidated in v2.0. | Source stakeholder authority not identified; confirm approver. |
| D-02 | Source-confirmed | Editable/reusable registration numbers use stable student ownership; Emirates ID is the duplicate identity key. | Engineering: design constraints and test races and reuse. |
| D-03 | Source-confirmed | Replacement retains versions; removal deletes current file on Save. No global history or extra export columns. | Engineering: implement and verify the effective rules. |
| D-04 | Confirmed | User requested automatic Emirates ID hyphens during entry and sidebar icons from the earlier prototype. Affects P08, VAL-06, LST-09, AT-58 and AT-59. | User request, 08 November 2026; verify during implementation review. |
| A-01 | Assumed | This PRD is saved in the requirements folder, with the supplied document remaining the detailed source and acceptance-test authority. If source authority changes, reconcile IDs and scope. | Product owner to validate. |
| Q-01 | Open | Who owns the PRD and can approve Phase 1 and resolve ambiguities? Affects release approval, all requirements. | Owner unknown; identify approver. |
| Q-02 | Open | What business baselines and success targets should be measured for registration, retrieval and recovery? Affects G-01–03; not needed to draft, needed for outcome evaluation. | Product owner; establish baseline and targets. |
| Q-03 | Open | Which Windows, browser and PostgreSQL versions and benchmark thresholds constitute supported operation? Affects ENV, OPS, ENG-01 and release readiness. | Engineering and product; select, measure and approve conditions. |
| Q-04 | Open | What accessibility target, session behavior details, support ownership and interruption/recovery procedures apply where the source is silent? Affects SES, DOC, BAK and OPS verification. | Product and engineering; specify and validate without changing confirmed outcomes. |
| Q-05 | Open | What backup archive format and application/database compatibility strategy will be used? Affects BAK-03, BAK-10 and upgrade testing. | Engineering; choose and test portability and recovery. |
| Q-06 | Open | How are configured destination paths and restored Settings reconciled when restoring on a new PC? BAK-08 requires current destination folders while BAK-02 includes Settings. | Engineering/product; preserve target folder destinations for restore and define other Settings restoration behavior; validate against source. |

The source explicitly supersedes earlier instructions to lock individual numbers, clear cricket details, archive removed current attachments, use full names in folders, show global history, export extra fields, use a 500-student planning volume, and create an initial Settings history entry. These are historical conflicts resolved by source v2.0, not open alternatives. Source decisions are not evidence of empirical validation.

## 9. Readiness and revision history

The source covers the main student journey, fields, permissions, failure outcomes and acceptance scenarios. The PRD is suitable for stakeholder review, but approval and implementation readiness remain open until Q-01 and the relevant technical design, supported environments, recovery/compatibility decisions and verification targets are resolved. Verify the end-to-end criteria and the dependency checks on the selected environment before release; no test results or stakeholder approval were supplied.

| Version | Ledger | Change |
| --- | --- | --- |
| 1.0, 08 November 2026 | Integrated ledger 1.0 | Initial provisional PRD from source v2.0; retained source IDs, decisions and test references. |
| 1.1, 08 November 2026 | Integrated ledger 1.1 | Added user-confirmed Emirates ID input formatting and sidebar icons (D-04, VAL-06, LST-09); added acceptance scenarios AT-58 and AT-59. |
