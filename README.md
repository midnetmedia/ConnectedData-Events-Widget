# ConnectedData-Events-Widget
Information on connecting events widget to connected data

# Events Collection — CSV Import Guide

A ready-to-use CSV template with 9 sample events covering all field variations (featured/non-featured, with/without images, with/without URLs, single-day and multi-day).

**File:** `events-collection-template.csv`

## Column Reference

| Column | Type | Required | Notes |
|---|---|---|---|
| `title` | Text | Yes | Event name |
| `start_date` | DateTime (ISO 8601) | Yes | Format: `YYYY-MM-DDTHH:MM:SS`. Example: `2026-05-10T10:00:00` |
| `end_date` | DateTime (ISO 8601) | No | Leave blank for single-moment events |
| `location` | Text | No | City/venue or "Zoom" / "Online" |
| `image` | Image URL | No | Full URL to a hosted image. 16:9 recommended, min 800×450 |
| `description` | Text | No | 1–2 sentences. Long descriptions get truncated to 2 lines in cards |
| `category` | Text | No | Free-form tag (e.g., "Workshop", "Fundraiser") |
| `url` | URL | No | External link (Eventbrite, Zoom, partner site). Leave blank for no link |
| `featured` | Boolean | No | `TRUE` or `FALSE`. Featured events get hero treatment at wide layouts |

**Column names must match exactly** — the widget's sub-input variable names (`title`, `start_date`, etc.) are what Duda's Connected Data binding auto-maps to. Rename a column to `Title` or `event_title` and the mapping breaks.

---

## Setup Option 1 — Google Sheets

1. Create a new Google Sheet
2. **File → Import → Upload**, select `events-collection-template.csv`
3. On the import dialog, choose **Replace spreadsheet** and **Comma** as separator
4. Once imported, click **File → Share → Publish to web**
5. Choose **Entire document** and **Comma-separated values (.csv)** format
6. Click **Publish** and copy the generated URL
7. In Duda: **Content → Collections → New Collection → External → Google Sheets**, paste the published URL
8. Duda will poll the sheet every 1–2 hours for updates

**Pro tip:** Rename the sheet tab to `Events` before publishing — Duda uses the tab name as the default collection label, and "Sheet1" is a giveaway that it was a quick setup.

**Gotcha with dates in Google Sheets:** Sheets sometimes auto-converts the ISO datetime strings into its own date format, which can break Duda's parsing. Before publishing, select the `start_date` and `end_date` columns → **Format → Number → Plain text**. This preserves the ISO format as-is.

---

## Setup Option 2 — Airtable

1. Open Airtable, create a new base
2. **Add or import → CSV file**, upload `events-collection-template.csv`
3. Airtable auto-detects field types. Verify each one:
   - `title` → Single line text
   - `start_date`, `end_date` → Date with time (toggle "Include time" ON)
   - `location` → Single line text
   - `image` → Change to **Attachment** field, or keep as URL text field (see note below)
   - `description` → Long text
   - `category` → Single line text (or **Single select** if you want a controlled vocabulary)
   - `url` → URL
   - `featured` → Checkbox
4. Rename the table to `Events`
5. **Share → Create a shared view link**, or set up a view and copy its share link
6. In Duda: **Content → Collections → New Collection → External → Airtable**, connect your account and pick the `Events` table

**Image field note:** Airtable's Attachment field stores images differently than a plain URL field. Duda can handle both but the binding in Widget Builder will look different:
- URL text field → widget's `image` sub-input binds directly
- Attachment field → widget's `image` sub-input binds to the first attachment's URL (usually handled automatically, but verify in the Connected Data mapping)

If you want editors to upload images directly in Airtable rather than paste URLs, use Attachment. If images are hosted elsewhere (Cloudinary, your CDN, Unsplash) and you want full control over optimization, use URL.

**Single select for category:** if you're standardizing on specific category names ("Workshop", "Fundraiser", "Conference", etc.), convert `category` to a Single select field in Airtable. Makes data entry faster and prevents typos like "workshops" vs "Workshop". The widget renders the text as-is either way.

---

## Setup Option 3 — Internal Duda Collection

If the client doesn't need external data management:

1. In Duda: **Content → Collections → New Collection → "Events"**
2. Manually add each column with the correct type per the Column Reference table
3. Click **Import from CSV** and upload `events-collection-template.csv`
4. Duda maps columns by name automatically

Internal collections are simpler (no external connection to manage, no sync delay) but require editors to update data inside Duda's UI. Best for clients who won't be making frequent event changes.

---

## Setup Option 4 — External Database (Enterprise only)

Duda Enterprise plans support connecting to external databases via REST API. The widget doesn't care about the source — as long as the returned JSON has the nine field names matching the widget's sub-inputs, it'll render correctly.

If a client is on Enterprise and wants to drive events from an existing system (CRM, event management platform, etc.), the CSV serves as the schema reference for what fields the API response needs to include.

---

## Sync Timing

- **Internal collections:** instant — changes appear on republish
- **Google Sheets:** cached 1–2 hours. Editors can trigger a manual re-sync in Duda's collection settings
- **Airtable:** cached 1–2 hours, same as Sheets
- **External DB:** depends on the API's cache headers

For clients running time-sensitive events (e.g., a same-day event gets canceled), make sure they know about the sync lag and how to force a re-sync. Or use internal collections where they have direct control.

---

## Extending the Schema

If a client needs additional fields — `speaker_name`, `ticket_price`, `registration_deadline`, etc. — add the columns to the CSV/collection AND add matching sub-inputs to the widget's `List(events)` in Widget Builder. Both names must match exactly.

For detail-page-only fields (long description, gallery, agenda) you don't need to expose them in the widget at all — they're just columns in the collection that the dynamic detail page reads directly. Leave them out of the widget's List to keep the card markup clean.

---

## Testing Your Setup

After connecting the collection to the widget:

1. Confirm 9 events render (or 6, if `maxEvents` is at default)
2. Confirm at least one event shows as featured (hero treatment at wide container widths)
3. Confirm events without images fall back to just the date chip + text (no broken image icons)
4. Confirm events with external `url` link to the URL; events without `url` render the title as plain text
5. Confirm dates format correctly per the `dateFormat` Design Editor setting
6. Confirm past events are filtered out when `showPast` is off (the sample CSV has no past events — edit one row to 2024 to test this)

If any of the above fails, check the column name mapping in the Connected Data panel first. 90% of CSV → widget issues are name mismatches.
