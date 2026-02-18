# Simple, Low-Cost Plan: Media Codes → PPT for Executives

This repository is currently empty, so this guide is a practical blueprint you can implement in stages without buying new software.

## 1) Goal (Plain Language)
Executives should be able to:
1. Select media codes (or paste a list of codes).
2. Click **Generate PPT**.
3. Download a branded proposal presentation with correct photos and location details.

## 2) Lowest-Cost Architecture (Use What You Already Have)
Use free/low-cost tools with your existing LAN-hosted app:

- **Google Sheets** = master metadata table (media code, location, type, size, lat/lng, owner brand, drive file id)
- **Google Drive** = photo storage (one image per media code, or folder per city/type)
- **LAN Web App (your existing app)** = UI + PPT generation engine
- **PPTX library** = generate slides automatically from selected codes

> Why this works: business users can maintain data in Sheets, photos are centralized in Drive, and executives use one simple internal URL.

## 3) Recommended Data Model (Google Sheet)
Create one sheet named `MediaInventory` with these columns:

- `media_code` (unique, required)
- `company` (SBA / OUTSPACE / YUVA)
- `location_name`
- `city`
- `media_type` (Hoarding / Unipole / Gantry / Arch)
- `size`
- `latitude`
- `longitude`
- `drive_file_id` (critical: stable identifier for image)
- `status` (active/inactive)
- `updated_at`

### Rules
- Make `media_code` unique.
- Use dropdown validation for `company`, `media_type`, `status`.
- Never depend on Drive filename for matching; use `drive_file_id`.

## 4) Drive Folder Strategy (Keep it Clean)
Use one shared Drive root:

`/MediaAssets/`
- `/SBA/`
- `/OUTSPACE/`
- `/YUVA/`

Within each, either:
- Flat storage (simpler), or
- Optional subfolders by city.

### Naming convention
- Recommended filename: `<media_code>.<ext>`
- But matching should still use `drive_file_id` from Sheet.

## 5) User Workflow (Executive-Simple)
In your LAN app, keep only 3 steps:

1. **Select brand mode**
   - Presenting brand: SBA / OUTSPACE / YUVA
   - Optional: No Logo mode
2. **Select assets**
   - Search/filter list OR paste bulk media codes
3. **Generate**
   - Click one button → app returns `.pptx`

That is all an executive should see.

## 6) Backend Integration Pattern (Simple + Reliable)

### Option A (Best long-term)
- Scheduled sync job pulls Sheet rows into local DB every 5–15 minutes.
- App reads from local DB for fast performance.
- On PPT generation, app fetches required Drive images by `drive_file_id` and caches locally.

### Option B (Fast MVP)
- App reads Sheet API directly on request.
- App fetches Drive images on the fly.

**Recommendation:** Start with Option B for MVP, move to Option A when usage grows.

## 7) PPT Generation Rules (Business-Critical)
For each selected code:
- Resolve row by `media_code`
- Pull image from Drive via `drive_file_id`
- Apply internal-slide logo from `company`
- Render:
  - Top-left: owner logo
  - Top-right: location text
  - Center: photo fit-to-box
  - Bottom-left: size/type/city/coordinates
  - Bottom-right: Google Maps link using lat/lng

Before and after asset slides:
- Add welcome + thank-you slides using selected presenting brand
- If No Logo mode: remove all brand marks

## 8) Cost Control (Important)
To keep cost near zero:
- Use existing Google Workspace (Sheets + Drive)
- Host app in existing LAN server
- Use open-source libraries only (e.g., Apache POI)
- Add local image cache to reduce repeated Drive downloads
- Use pagination and lazy loading to avoid heavy infrastructure

## 9) Security & Governance
- Restrict Sheet edit rights to operations/admin team only
- Give executives read-only app access
- Use service account for Drive/Sheets API
- Log who generated PPT and which media codes were used
- Keep daily backup export of sheet data

## 10) Implementation Roadmap (Practical)

### Week 1: Foundation
- Finalize Sheet columns and validation
- Organize Drive folders
- Upload sample 200 assets

### Week 2: MVP
- Build/read API integration for Sheets + Drive
- Implement “paste media codes” + generate PPT
- Add branding/no-logo toggles

### Week 3: Hardening
- Add caching, pagination, and audit logs
- Add error report for missing codes/images
- UAT with executives

### Week 4: Rollout
- Train users (30 min session)
- Go live in LAN
- Freeze template + establish data ownership

## 11) Minimal API Design (for your LAN app)
- `POST /api/ppt/generate`
  - Input: presentingBrand, noLogo, codes[]
  - Output: `.pptx` file
- `POST /api/media/resolve-codes`
  - Input: codes[]
  - Output: found rows + missing codes list
- `GET /api/media/search?query=&company=&type=&page=`
  - For inventory UI

## 12) Operational SOP (Keep Quality High)
- New media entry checklist:
  - media_code added
  - correct company/type
  - drive image uploaded
  - drive_file_id pasted
  - lat/lng verified
- Daily checks:
  - missing image IDs
  - duplicate codes
  - invalid coordinates

## 13) Biggest Risks and Fixes
- **Risk:** Wrong image mapped to code
  - **Fix:** use `drive_file_id`, not filename matching
- **Risk:** Slow generation with large code lists
  - **Fix:** local cache + batched image fetch
- **Risk:** Inconsistent branding
  - **Fix:** fixed template + controlled logo assets

## 14) Success Metrics
Track weekly:
- Average time to generate proposal
- Number of proposals generated
- Error rate (missing code/image)
- Manual rework incidents

Target outcome: reduce proposal preparation time from hours to minutes.

---

## Quick Start (If You Want to Start Today)
1. Create `MediaInventory` Google Sheet with mandatory columns.
2. Upload sample photos to Drive and capture `drive_file_id` in sheet.
3. Build one endpoint that accepts 10 media codes and returns PPT.
4. Validate output with 3 executives.
5. Expand to full inventory.
