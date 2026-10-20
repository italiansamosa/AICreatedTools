# AICreatedTools

Small, single-file browser tools for day-to-day cloud/platform admin work, built with AI assistance.

Each tool is one self-contained `.html` file: no build step, no dependencies, no CDN calls, no backend. Open it in a browser and it works offline. Nothing you load or paste leaves the tab.

| Tool | What it does | Persists data? |
|---|---|---|
| [AD Group Explorer](AD-Group-Explorer/) | Slice and audit an Entra ID / AD group-membership CSV export | No |
| [Diff Checker](Diff-Checker/) | Side-by-side or unified text diff with inline word/char highlighting | No |
| [SAS Token Tracker](SAS-Token-Tracker/) | Track Azure Storage SAS token metadata and expiry dates | Yes, `localStorage` |

## Usage

```
git clone https://github.com/italiansamosa/AICreatedTools.git
```

Then double-click the `.html` file for the tool you want, or open it via `file://` in any modern browser (Edge, Chrome, Firefox, Safari).

---

## AD Group Explorer

`AD-Group-Explorer/AD Group Explorer.html`

Drop in a flat CSV of group memberships (one row per user–group pair) and get filterable, sortable views plus KPIs.

**Views**

- **Memberships** – raw rows
- **Users** – one row per user: group count, direct vs nested, full group list
- **Groups** – one row per group: user count, direct/nested split, disabled accounts, guests
- **Matrix** – users × groups grid, ● direct / ○ nested

**Features**

- KPIs: memberships, unique users, groups, nested-only users, disabled accounts, guests (disabled/guests highlighted when non-zero)
- Dropdown filters for group, family, role, domain, membership type, department, user type, enabled
- Free-text search across name, UPN, mail, title, department, group, employee ID
- Click a group or UPN to drill into its memberships
- Export the current view (filters applied) to CSV
- Renders the first 5,000 rows; export gets everything

**Expected CSV columns**

| Column | Required | Notes |
|---|---|---|
| `GroupName` | Yes | Load fails without it |
| `MembershipType` | Recommended | `Direct` or `Nested` |
| `UserId` / `UserPrincipalName` / `DisplayName` | At least one | Used as the user key, in that order of preference |
| `Mail`, `Department`, `JobTitle`, `EmployeeId` | Optional | Searchable/displayed |
| `UserType` | Optional | `Member` / `Guest` |
| `AccountEnabled` | Optional | `True` / `False` |

UTF-8 with or without BOM; quoted fields supported.

**Group-name parsing.** `Family`, `Domain` and `Role` are derived from the group name and are tuned to one naming convention:

- `AAD.RBAC.ITDA_DLH_<...>` → family `AAD.RBAC`
- `DPF_ITDA_DLH_<...>` → family `DPF`
- anything else → `Other`
- Domain is the first underscore token in `FIN`, `HR`, `PLM`, `SN`, `TPM`, `COMMON`; the remaining tokens form the role

To reuse with a different convention, edit `DOMAINS` and `derive()` near the top of the script.

---

## Diff Checker

`Diff-Checker/diff-checker.html`

A local replacement for online diff sites — useful when the content (configs, exports, policy JSON, Terraform plans) shouldn't be pasted into a third-party website.

**Features**

- Paste text, pick a file, or drag a file onto either pane
- Side-by-side or unified view
- Inline highlighting by word, character, or off
- Ignore whitespace / ignore case
- Collapses unchanged regions to 3 lines of context (click to expand)
- Change stats: modified / removed / added line counts
- Jump between change blocks; swap and clear
- Excel ranges pasted in arrive as tab-separated rows, so spreadsheet columns diff cleanly
- Myers O(ND) diff, implemented inline

Binary formats (`.xlsx`, `.xls`, `.docx`, `.pdf`, `.zip`) are rejected — export to CSV/text first.

**Shortcuts**

| Keys | Action |
|---|---|
| `Ctrl/Cmd + Enter` | Compare |
| `Alt + ↓` / `Alt + ↑` | Next / previous change |

---

## SAS Token Tracker

`SAS-Token-Tracker/sas-token-tracker.html`

An inventory of SAS tokens you've issued, so expiries don't sneak up on app teams. Tracks **metadata only** — never the signature.

**Features**

- Paste a SAS query string or full SAS URL to auto-fill: storage account, container, path, resource type (`sr`, incl. `sdd` depth), permissions (`sp`), protocol (`spr`), start/expiry (`st`/`se`), signed version (`sv`)
- The `sig=` value is detected and discarded; the paste box is cleared ~100 ms after parsing. A copy of the query string with `sig=REDACTED` is kept for reference
- Status per token: **Active**, **Expiring soon** (≤ 30 days), **Expired**
- Stats bar, search (account / container / requester / notes), status filter, sortable columns
- Calendar view of expiry dates, colour-coded by status
- Requester/team and purpose fields per token
- Export / import JSON (import merges; matching IDs overwrite)

**Storage.** Records live in the browser's `localStorage` under `sas_token_tracker_v1`. That means:

- Data is per browser and per profile, and for `file://` pages may be per file path — moving or renaming the file can make existing data appear to vanish
- Clearing site data deletes it
- Use **Export JSON** for backups or to move between machines

---

## Security notes

- No network requests: no external scripts, fonts, analytics, or APIs
- Input is HTML-escaped before rendering
- AD Group Explorer and Diff Checker hold data in memory only; closing the tab clears it
- SAS Token Tracker never stores signatures, but the redacted query string and account/container names are saved in `localStorage` — treat exported JSON as internal

## Adding a tool

1. Create a folder named after the tool
2. Put a single self-contained `.html` file in it (inline CSS/JS, no external requests)
3. Add a row to the table at the top and a section below
