Commissioning — Full Feature Plan
### Tab: "Commissioning" (new 4th tab on DashboardPage)

---

### 1. Camera-by-Camera Checklist

One row per camera. Each row has:

|Camera|IP|Online|Added to NVR|Angle OK|Image OK|Label OK|Notes|Snapshot|Status|
|---|---|---|---|---|---|---|---|---|---|

- **Online** — live ping dot (reuse existing)
- **Added to NVR** — checkbox: camera is recording on the NVR
- **Angle OK** — checkbox: physical camera angle verified on-site
- **Image OK** — checkbox: image quality acceptable (no blur, IR working)
- **Label OK** — checkbox: physical label/sticker matches system label
- **Notes** — inline text field per camera
- **Snapshot** — button that captures a MJPEG snapshot and saves it to disk, thumbnail shown inline
- **Status** — auto-calculated: `Pass` (all 4 checked) / `Fail` (manually overridden) / `Pending` (incomplete)

---

### 2. Site-Level Progress Bar

Top of the tab:

```
12 / 15 cameras commissioned  ██████████░░░  80%
```

Breakdown: `3 passing · 9 pending · 0 failing`

---

### 3. Snapshot Storage

- Capture snapshot per camera via `/stream/snapshot`
- Save to `snapshots/<site_id>/<camera_id>_<timestamp>.jpg` on disk
- Show thumbnail inline in the row
- Click thumbnail → full-size preview modal
- Keep only the latest snapshot per camera (overwrite)

---

### 4. Bulk Actions

- **Ping All** — refresh all online dots at once
- **Capture All Snapshots** — sequential snapshot of every camera
- **Mark All Passed** — one-click pass everything (for when client is watching)
- **Reset** — clear all commissioning data for the site

---

### 5. PDF Report Generation

Button: **"Export Report"** — generates a PDF with:

- Site name, date, technician name
- Summary table: camera label, IP, MAC, model, port, status
- Per-camera snapshot thumbnails (2 per row)
- Checklist tick marks
- Notes per camera
- Footer: company name, report generated timestamp

Uses `reportlab` (Python) or `weasyprint`. Saved as `<SiteName>_commissioning_<date>.pdf`

---

### 6. Additional DB columns needed

Current schema is missing:

- `commissioned_at` — timestamp when camera was marked passed
- `commissioned_by` — technician name/username
- `snapshot_taken_at` — timestamp of last snapshot
- `fail_reason` — text field for why it failed

---

### 7. Quick-Commission Flow (for speed on-site)

Click a camera row → side panel opens with:

- Live MJPEG stream preview
- All 4 checkboxes
- Notes
- Snapshot button
- Pass / Fail buttons

This lets you walk around with a laptop, click each camera, verify it live, tick boxes, snap, move on.

---

### Build order:

1. **Commissioning tab + checklist UI** — the table with checkboxes, autosave
2. **Snapshot capture + storage** — the most visually impressive part
3. **Progress bar + status calculation**
4. **Bulk actions**
5. **PDF report** — the deliverable

Want to start with step 1?