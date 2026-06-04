# Drop It — Changelog

All notable changes from this working session.

---

## 2026-06-03 — Custom lists

Replaced the single saved list with full custom lists (Google-Maps-style organization, but with fewer taps). Private and per-device for now.

### Added

**Saving**
- **One-tap save.** Tapping the bookmark on a place saves it instantly to a default list (**Favorites**) — no picker, no decision. A toast confirms ("Saved to Favorites · Add to list") with an optional shortcut to file it.
- **Add-to-list sheet.** Tapping the already-saved bookmark (or the toast shortcut) opens a sheet to toggle the place into any of your lists with checkmarks. Unchecking it everywhere un-saves it.

**Lists**
- **Create / edit lists** with a name, an optional icon, and a description — all private to you. Reachable from the add-to-list sheet ("+ New list") or the profile.
- **Your lists on the profile.** The "You" tab shows every list as a card with its icon, place count, and privacy.
- **List detail view.** Tap a list to see its places (with photo, rating, and area); tap a place to open the full detail sheet (recenters the map). Remove a place with the ✕. Rename via the pencil; delete via a two-tap confirm (Favorites can't be deleted).

**Icons (no emoji)**
- Lists use a small set of **minimal line icons** matching the app's style — not emoji. The picker is optional (**None** is the default), and its label updates to the selected category as you tap.
- The set is built around how people actually tag places: **Want to go, Loved, Date night, Restaurants, Brunch, Sweets, Coffee, Drinks, Live music, Events, Outdoors, Trips**. (Gyms/shopping excluded; "Nightlife" merged into "Drinks".)
- A list with no icon shows a clean tinted glyph (a heart for Favorites, a list mark otherwise), each in its own accent color. Prefer an emoji? Just type it into the list's name — it's free text.

**Map**
- **"♥ Saved" filter chip** on the map shows just your saved pins; a chip per non-empty custom list lets you filter to one list. Tap a pin → full detail, without leaving the map.

### Changed
- Old `dropit_saved` data is migrated into the **Favorites** list automatically (nothing lost).
- Storage now uses `dropit_lists` (lists + membership) and `dropit_places` (place info), still per-device via `localStorage`.

---

## 2026-06-03

UX polish on the map and sheet, a first personalized recommendation system, working Save and Directions, city/neighborhood search, and a Profile/Settings split.

### Added

**Map & markers**
- **Selected-pin emphasis.** Tapping a place scales its pin up with a white halo and dims/desaturates the others; clears on close or when another pin is tapped.

**Bottom sheet**
- **Half-height "peek" detent.** Places open to ~52% of the screen with the map still visible and tappable above; drag/flick up for full, down to dismiss. Friend cards open full.
- **Color-coded open/closed status line** in the sheet header (green = open, red = closed, amber = event-style hours).

**Scrolling**
- **Scroll-snap + right-edge fades** on the photo, video, and trending rails.

**Search**
- **City & neighborhood search.** The search box now returns a "Cities & neighborhoods" group (geocoded live via komoot's Photon, debounced, filtered to `osm_tag=place`) alongside "Places." Picking a location pans/zooms the map there (fits the bounding box when available; sensible per-type zoom otherwise). Biased toward the current map center but works worldwide.

**Saved list**
- **Save is a real toggle.** The bookmark fills when saved, un-saves on tap, and persists per-device. Saved places appear in the profile and tap through to recenter/open. Still logs a `save` signal to Supabase for recommendations.

**Directions**
- **Maps-app chooser** — Apple Maps (iOS), Google Maps, Waze. The three ETA tiles also open directions, each in its travel mode.
- **Custom-scheme handoff** (`maps://`, `comgooglemaps://`, `waze://`) on iOS so the maps app opens over the app **without** a new tab and **without** navigating the app's own tab away. Non-iOS falls back to `https://` map links.
- **Remember my choice** toggle in the chooser sets a default and skips the chooser next time; also controllable in Settings.

**Recommendations**
- **Interaction logging** to Supabase, weighted by intent: open `1`, video `2`, expand-to-full `3`, save `5`, directions `5`, post `6`, quick-close `skip` `-2`.
- **Anonymous per-device id** in `localStorage`.
- **"For you" rail** in Discover from the `recommend_places` RPC; personalizes by cuisine + neighborhood, falls back to popular-nearby for new users, hides itself if the backend isn't set up.
- **`dropit_setup.sql`** — `interactions` table, prototype RLS, and the `recommend_places(client, bbox, limit)` function.

**Profile & Settings**
- **You is now a profile** — avatar, **editable name** (persisted; avatar initials follow it), a live "Sharing: <precision> · N saved" summary, and Saved places.
- **New Settings screen** (gear in the profile header): visibility precision, Ghost mode, **Directions app** (Always ask / Apple / Google / Waze), and Map theme.

### Changed
- The bottom sheet moved from a single open state to a three-state machine (peek / full / closed); the scrim only appears at full so peek keeps the map interactive.
- Directions no longer use `window.open(_blank)` (which stranded an `about:blank` tab in sandboxed/embedded contexts); see the custom-scheme handoff above.
- Privacy/Ghost/Theme controls moved out of the You tab into Settings.

### Notes & decisions
- **Ranking pipeline.** Fetch gates by `pop` (top ~300 in view); a richer `tscore` (stars, rating count, live, busy, posts, pop) picks the "hot" top-8; `render()` draws a zoom-scaled budget sorted HOT -> pop -> tscore. Known mismatch: the gate uses `pop` while prioritization uses `tscore`, so a great low-`pop` spot can be filtered out before quality is considered.
- **Recommendations** use a tag-based taste model (cuisine + neighborhood in plain SQL) for v1 so it works from the first tap with no new infrastructure; embeddings are a later upgrade.
- **All client settings** (theme, client id, saved list, directions default, name) persist per-device in `localStorage`.
- **Directions caveat:** they only launch a real maps app on the hosted site or a normal browser tab, not inside a sandboxed preview.

### Discussed / planned (not yet built)
- **Semantic recommendations** via embeddings in pgvector.
- **Re-rank the map pins** by personal taste, not just the Discover rail.
- **Real accounts** so data is per-user instead of per-device, with tightened RLS; optional Supabase `saves` table for cross-device saved lists.
- **Fix the ranking gate** to lead with quality/liveness instead of `pop`; decide how unrated/new spots surface.
- **Data sources** identified: Yelp Places API, Google Places (+ Places Insights for foot traffic), Ticketmaster / Eventbrite / SeatGeek / DICE / Bandsintown / Meetup for events, PredictHQ for event demand, Placer.ai for foot traffic.

### Files
- `index.html` — the app (all of the above).
- `dropit_setup.sql` — run once in the Supabase SQL editor for recommendations.
