# Verification Fieldbook PWA
### Engineering Survey — Field Verification & As-Built Capture System

---

## Overview

A mobile-first, installable Progressive Web App (PWA) for capturing as-built survey verification readings in the field. Works fully offline after first load; all data stored locally in IndexedDB.

---

## Quick Start

### Option A — Local Server (Recommended)
```bash
# Python 3
python -m http.server 8080

# Node.js (npx)
npx serve .

# Then open in browser:
# http://localhost:8080
```

### Option B — Static Host (Nginx / Apache / Netlify / GitHub Pages)
Copy the three files to your web root:
```
/
├── index.html
├── sw.js
└── manifest.json
```

### Default Login
| Field    | Value            |
|----------|-----------------|
| Username | `admin`          |
| Password | `fieldbook2024`  |

> Change password immediately via **Settings → Change Password**.

---

## Workflow

```
1. Import Design CSV
       ↓
2. Verify / Capture: Lookup Index → Enter Readings → Save
       ↓
3. As-Built Table: Review, filter, edit notes
       ↓
4. Export: CSV or Excel (.xlsx)
```

---

## Feature Reference

### A. Authentication
- PBKDF2-salted password hashing (Web Crypto API)
- Session persists with configurable inactivity timeout (default 30 min)
- Password change in Settings

### B. PWA + Offline
- Service worker caches app shell and CDN assets
- Installable via browser "Add to Home Screen"
- All data in IndexedDB — no backend required

### C. Import Design CSV
- Accepts `.csv` and `.xlsx` / `.xls`
- Auto-detects column headers (CHAIN, Left X-Fall, CL Design Level, Right X-Fall)
- Column mapping UI for non-standard headers
- X-Fall normalization: |val| > 1 treated as percent (−2 → −0.02)
- Duplicate chain detection (keeps last row, logs count)
- Import summary with error reporting

**Expected CSV format:**
```
CHAIN,Left X-FALL,CENTRELINE DESIGN,Right X-FALL
0,−0.02,100.250,−0.02
25.5,−0.02,100.188,−0.02
50,−2,99.910,−2
```

### D. Setting Out / Stakeout
- **Purpose**: Calculate design elevations and required staff readings for setting out construction stakes at specified offsets from centerline
- **Workflow**:
  1. Lookup chainage (SV) — loads CL Design Level and cross-fall data
  2. Enter BM Height + Staff → calculates collimation automatically
  3. Add nails (left/right) with:
     - Travel (future use)
     - Offset (distance from CL in meters)
     - X-Fall % (cross-fall percentage)
  4. System auto-calculates:
     - **Design Elevation** = CL_Design + (Offset × X-Fall %)
     - **Staff Reading** = Collimation − Design Elevation
- **Example**:
  ```
  SV: 25, CL Design: 1542.602
  BM Height: 1549.000, Staff on BM: 2.300 → Collimation: 1551.300
  
  Left Nail 1: Offset 5.250m, X-Fall -2%
    Design = 1542.602 + (5.250 × -0.02) = 1542.497
    Staff = 1551.300 - 1542.497 = 8.803
  ```
- **Save**: Stores all nails (left + right) as one stakeout entry
- **Export**: CSV/XLSX with columns: SV, Side, Nail, Offset, X-Fall%, Design, Staff

### E. Index Lookup
- Two modes:
  - **Closest ≤** (default, VLOOKUP approximate): finds the highest CHAIN ≤ input value
  - **Exact match**: requires precise CHAIN value
- Toggle per lookup or set default in Settings

### F. Verification Points (1–8)
- Select number of verification points (1–8)
- Auto-generates symmetric profile:
  - Odd N: includes true centerline at 0%
  - Even N: straddles center with two near-center points
- Labels: L3, L2, L1, CL, R1, R2, R3, R4 (editable per row)
- Profile % editable per row
- BM Height / Collimation mode (toggle in Settings):
  - Enter BM Height + Staff on BM → collimation computed automatically
  - Staff reading → elevation auto-calculated
- Δ Design column: elevation − CL design level (green/red indicator)

### G. Save Flow
- Save button enabled only when: design point selected + ≥1 reading entered
- Confirmation modal previews all entered values
- On confirm: writes `asbuilt_entries` + `asbuilt_points` records
- Auto-clears for next entry (preserves point count setting)

### H. As-Built Table
- Filter by: date range, chain range
- Click row eye icon → detail view with full point table
- Edit notes inline
- Delete entry with confirmation

### I. Export
| Export | Format | Description |
|--------|--------|-------------|
| Cross-Section Matrix | XLSX | **Field-grade format**: chainages as rows, cross-fall positions as columns with Actual/Design/Diff sub-columns |
| Setting Out / Stakeout | CSV / XLSX | One row per nail: SV, Side, Offset, X-Fall%, Design, Staff |
| Entries Wide | CSV / XLSX | One row per entry; P1–P8 columns |
| Points Long | CSV / XLSX | One row per point |
| Full Workbook | XLSX | All four sheets in one file |

**Cross-Section Matrix format:**
```
SV    | -10.0           | -5.0            | 0.0             | 5.0             | 10.0
      | Act  Des  Diff  | Act  Des  Diff  | Act  Des  Diff  | Act  Des  Diff  | Act  Des  Diff
------+----------------+----------------+----------------+----------------+----------------
100   | 1545 1544 0.72 | 1542 1542 -0.03| 1542 1542 -0.03| 1542 1542 -0.03| 1542 1542 -0.03
120   | ...
```
This format is ideal for visual inspection of cross-sections and quality control in the field.

**Entries Wide columns:**
`created_at, user, input_index, resolved_chain, cl_design_level, left_xfall, right_xfall, point_count, notes, P1_label … P8_delta`

**Points Long columns:**
`entry_id, created_at, user, resolved_chain, seq, label, target_pct, staff_reading, elevation, delta_to_design, comment`

---

## Data Model

```
design_points
  id, chain, left_xfall, cl_design_level, right_xfall
  raw, import_batch_id, imported_at

asbuilt_entries
  id, created_at, user, input_index, resolved_chain
  design_point_id, cl_design_level, left_xfall, right_xfall
  point_count, notes

asbuilt_points
  id, entry_id, seq, position_label, target_profile_pct
  staff_reading, elevation, delta_to_design, comment

stakeout_entries
  id, created_at, user, input_index, resolved_chain
  design_point_id, cl_design_level, left_xfall, right_xfall, notes

stakeout_nails
  id, entry_id, seq, side, label, travel, offset
  xfall_pct, design, staff
```

---

## Settings

| Setting | Default | Description |
|---------|---------|-------------|
| Session Timeout | 30 min | Inactivity auto-logout |
| BM Height / Staff | Off | Auto-calc elevation from collimation |
| Profile Left | −10% | Default left cross-fall |
| Profile Right | −10% | Default right cross-fall |
| Match Mode | Closest ≤ | Index lookup default |

---

## Technology Stack

| Library | Version | Purpose |
|---------|---------|---------|
| Bootstrap | 5.3.2 | UI framework |
| Bootstrap Icons | 1.11.3 | Icons |
| Dexie.js | 3.2.4 | IndexedDB wrapper |
| PapaParse | 5.4.1 | CSV parsing |
| SheetJS (xlsx) | 0.20.0 | Excel export |
| Web Crypto API | native | Password hashing |
| Service Worker | native | Offline caching |

---

## Security Notes

- Authentication is **client-side only** — suitable for single-user or trusted device deployments
- Passwords hashed with PBKDF2 (100,000 iterations, SHA-256, random salt)
- Session tokens stored in localStorage with expiry
- For multi-user or sensitive deployments, add a server-side auth layer

---

## Browser Compatibility

| Browser | Support |
|---------|---------|
| Chrome / Edge (desktop + Android) | Full PWA |
| Safari (iOS 16.4+) | Full PWA (install from Share menu) |
| Firefox | Runs, limited PWA install |

---

## File Structure

```
fieldbook/
├── index.html      # Complete single-file application
├── sw.js           # Service worker (offline caching)
├── manifest.json   # PWA manifest (installability)
└── README.md       # This file
```

---

*Version 1.0.0 — Engineering Survey Verification Fieldbook*
