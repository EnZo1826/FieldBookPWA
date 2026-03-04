# Fieldbook PWA — Updates

## Version 1.1 — Setting Out / Stakeout Module

### New Feature: Setting Out / Stakeout

Complete surveying stakeout module for calculating design elevations and required staff readings at construction offsets.

**Workflow:**
1. Lookup chainage (SV) → loads CL Design Level + cross-fall data
2. Enter BM Height + Staff → calculates collimation
3. Add left/right nails with offset distances
4. System auto-calculates:
   - **Design Elevation** = CL_Design + (Offset × X-Fall %)
   - **Staff Reading** = Collimation − Design
5. Save all nails as one stakeout entry
6. Export to CSV/XLSX

**Example Calculation:**
```
SV: 25, CL Design: 1542.602
BM: 1549.000, Staff: 2.300 → Collimation: 1551.300

Left Nail at 5.250m offset, X-Fall -2%:
  Design = 1542.602 + (5.250 × -0.02) = 1542.497
  Staff = 1551.300 - 1542.497 = 8.803
```

**Database Schema:**
- `stakeout_entries` — one per SV lookup
- `stakeout_nails` — one per nail (left/right)

**Export Formats:**
- CSV/XLSX: columns = SV, Side, Nail, Offset, X-Fall%, Design, Staff
- Included in Full Workbook export (4 sheets total)

**UI Features:**
- Split left/right nail tables
- Add/remove nails dynamically
- Real-time design + staff calculation
- Save button enabled when ≥1 nail entered
- Dashboard quick action button

---

## Version 1.0.1 — Cross-Section Matrix Export

### New Export Format: Cross-Section Matrix

Added field-grade matrix export matching reference format:

```
SV    | -10.0           | -5.0            | 0.0             | 5.0             | 10.0
      | Act  Des  Diff  | Act  Des  Diff  | Act  Des  Diff  | Act  Des  Diff  | Act  Des  Diff
------+----------------+----------------+----------------+----------------+----------------
100   | 1545 1544 0.72 | 1542 1542 -0.03| 1542 1542 -0.03| 1542 1542 -0.03| 1542 1542 -0.03
120   | ...
```

**Matrix Calculation Logic:**
- **Actual** = elevation reading
- **Diff** = delta_to_design (stored)
- **Design** = Actual - Diff (back-calculated)

**Excel Formatting:**
- Position headers merged across 3 sub-columns
- Two-row headers: position % + Actual/Design/Diff
- Auto-column widths

---

## Files Updated

- `index.html` — main application (added Stakeout module, updated exports, dashboard)
- `README.md` — comprehensive stakeout documentation
- Database schema v1: added `stakeout_entries` and `stakeout_nails` tables

---

*Current version: 1.1 — Full surveying fieldbook with verification capture and construction stakeout*

