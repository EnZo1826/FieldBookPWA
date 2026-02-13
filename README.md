# Verification Fieldbook PWA

## Engineering Survey Verification Fieldbook

A mobile-first, installable Progressive Web App for field engineers to import design survey data, verify index points with 1-8 readings, and export as-built results.

---

## Workflow

```
Import Design CSV  -->  Verify Index  -->  Enter Readings  -->  Save  -->  Export
```

### 1. Import Design CSV
- Navigate to **Import** tab
- Drop or browse a `.csv` file containing: CHAIN, Left X-FALL, CENTRELINE DESIGN, Right X-FALL
- Map columns if headers differ from expected names
- Review import summary (row count, chain range, duplicates)

### 2. Verify Index Point
- Navigate to **Verify** tab
- Enter a chainage/index number
- App matches to closest design point (VLOOKUP-style) or exact match
- Selected design point card displays: Chain, Left X-Fall, CL Design Level, Right X-Fall

### 3. Enter Readings
- Select number of verification points (1-8)
- Auto-generated labels (L3, L2, L1, CL, R1, R2, R3) and target profile percentages
- Enter staff readings and/or elevations
- Optional: Enable BM Height + Staff auto-calculation in Settings

### 4. Save
- Confirmation modal previews all entered values
- Saves entry + child points to IndexedDB
- Auto-clears for next entry

### 5. Export
- **As-Built** tab shows all saved entries
- Export to CSV or XLSX (two sheets: Entries_Wide + Points_Long)
- Filter by chain range or search

---

## Quick Start

### Local Development
```bash
# Serve with any static HTTP server
cd fieldbook
python3 -m http.server 8080
# or
npx serve .
```

Open `http://localhost:8080` in your browser.

### Default Credentials
- **Username:** admin
- **Password:** fieldbook

### Deploy to Production
Copy all files to any static hosting (Nginx, Apache, Netlify, Vercel):
```
index.html
manifest.json
sw.js
icons/icon-192.png
icons/icon-512.png
```

### Nginx Example
```nginx
server {
    listen 443 ssl;
    server_name fieldbook.example.com;
    root /var/www/fieldbook;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

> **Note:** PWA installation requires HTTPS or localhost.

---

## Features

| Feature | Status |
|---------|--------|
| PBKDF2 password hashing | Yes |
| Session timeout (configurable) | Yes |
| Offline-first (IndexedDB) | Yes |
| Installable PWA | Yes |
| CSV import with column mapping | Yes |
| X-Fall normalization (% or decimal) | Yes |
| Approximate match (VLOOKUP) | Yes |
| Exact match toggle | Yes |
| 1-8 verification points | Yes |
| Dynamic target profile | Yes |
| BM Height / Collimation auto-calc | Yes |
| Save confirmation modal | Yes |
| As-Built history + detail view | Yes |
| CSV export | Yes |
| XLSX export (2 sheets) | Yes |
| Data backup/restore (JSON) | Yes |
| Responsive mobile UI | Yes |

---

## Tech Stack

- **UI:** Bootstrap 5.3 + Custom CSS
- **CSV:** PapaParse
- **XLSX:** SheetJS
- **Storage:** IndexedDB (native)
- **Auth:** PBKDF2 via Web Crypto API
- **PWA:** Service Worker + Web Manifest

---

## Data Model

### design_points
| Field | Type | Description |
|-------|------|-------------|
| id | uuid | Primary key |
| chain | number | Chainage value |
| left_xfall | number | Decimal slope (-0.02 = -2%) |
| cl_design_level | number | Centreline elevation |
| right_xfall | number | Decimal slope |
| raw | JSON | Original CSV row |
| import_batch_id | uuid | Batch identifier |
| imported_at | timestamp | Import timestamp |

### asbuilt_entries
| Field | Type | Description |
|-------|------|-------------|
| id | uuid | Primary key |
| created_at | timestamp | Save timestamp |
| user | string | Username |
| input_index | number | User-entered index |
| resolved_chain | number | Matched design chain |
| design_point_id | uuid | FK to design_points |
| cl_design_level | number | Design elevation |
| left_xfall | number | Left cross-fall |
| right_xfall | number | Right cross-fall |
| point_count | 1-8 | Number of readings |
| notes | string | Optional notes |

### asbuilt_points
| Field | Type | Description |
|-------|------|-------------|
| id | uuid | Primary key |
| entry_id | uuid | FK to asbuilt_entries |
| seq | 1-8 | Sequence number |
| position_label | string | L3, CL, R2, etc. |
| target_profile_pct | number | Target % |
| staff_reading | number | Staff reading |
| elevation | number | Computed elevation |
| delta_to_design | number | Elevation - design |
| comment | string | Optional comment |

---

## Settings

- **Session Timeout:** 5-480 minutes (default 30)
- **BM Height/Staff:** Enable auto-calculation of elevation from collimation
- **Match Mode:** Approximate (closest <=) or Exact
- **Default Profile:** Configurable symmetric profile (e.g., -10, -5, 0, -5, -10)

---

## File Structure
```
fieldbook/
  index.html       # Complete application (single file)
  manifest.json    # PWA manifest
  sw.js            # Service worker for offline caching
  icons/
    icon-192.png   # PWA icon 192x192
    icon-512.png   # PWA icon 512x512
  README.md        # This file
```
