# TV World — Project State

> **Next chat name: `TV World ver 4`**
> Claude: your FIRST reply in a new chat must start with "Rename this chat to: **TV World ver 4**" (you can't rename chats yourself). In your first commit this session, bump the number here by 1.

**State version:** 3 (moved into the repo 2026-10-22 by the conductor chat "Life Plan Roadmap ver 3")
**Where this lives:** `PROJECT_STATE.md` in the repo. New chats `git pull` and read it; no uploads needed.

## ⚙ Session Rules (every chat that works on this app follows these)

**1. Pick the cheapest model that can do the job.** At the start of every session Claude names the model it recommends and the estimated size (below). If the chat is on a pricier model than needed, Claude says so before starting.

| Model | Use for |
|---|---|
| **Haiku** | State-file edits, text or label changes, small data packs, lookups, "where is X" questions |
| **Sonnet** (default) | Features, bug fixes, refactors, data processing (e.g. novel volumes, PullList), most app work |
| **Opus** | Only for new-app builds, cross-app architecture, or a bug Sonnet has failed on twice. Claude says why before using it |

**2. Every push updates this file in the same commit.** No exceptions, including PR branches and one-line fixes.
- `git pull` before starting, so you build on the latest state.
- Add a dated line to the change log (what changed and why, in plain words) and update any section the change made wrong.
- If a session ends without a push, nothing changes here.

**3. Size the work first, and plan the pause points.**
- Before starting, Claude lists the steps and sizes the job: **S** (under ~20k tokens), **M** (~20–60k), **L** (over ~60k).
- **M and L jobs are split into checkpoints.** Each checkpoint ends with a commit and a state update, so a cut-off session loses nothing.
- **Pause** when usage is around half the window, when the chat has grown long (heavy file reads, ~30+ turns), or before a step that won't finish in what's left. Claude can't see the remaining credit directly, so on L jobs it asks Marvin to check usage at each checkpoint.
- **On pause,** Claude writes a `▶ RESUME HERE` block at the top of this file: what's done, the exact next step, branch and files, and the first command to run. It also gives Marvin a one-line restart prompt ("Continue <App> ver X from RESUME HERE"). The next session starts from that block and deletes it once the step is done.

**4. Chat naming.** Use the **Next chat name** line at the top of this file; tell Marvin in the first reply, then bump it.

---

## Latest changes (Oct 20–22, 2026, conductor chat)
- **Monthly refresh (PR #1, merged 20 Oct):** Settings → **Refresh All from TMDB** updates every title in one go. `upcoming[]` gets a `due-date` reminder 30 days after the last run.
- **Movie release dates:** unreleased watchlisted movies add `release` items to `upcoming[]` ("in SG cinemas" / "digital release").
- **New icon** from the ecosystem icon set (21 Oct); `sw.js` cache now `tvworld-v4`.
- **Manifest id fixed (22 Oct):** `"id": "/TVWorld/"`. The old `"./"` resolved to the site root, so Android treated every marvyeo.github.io app as one.
- **Life Plan** dashboard reads `tvworld-data.json`. TV World is on its own Google Cloud project, so Life Plan needs a one-time **Pick missing files** grant.

## Session (Oct 13, 2026) — installable app + multi-device sync
- **PWA:** `manifest.json`, `sw.js`, `icons/` now sit beside `index.html` in the GitHub Pages repo. Install from Chrome/Edge (PC), Chrome (Android), Safari → Add to Home Screen (iPhone). Settings → *Install on this device* shows the right steps / an Install button.
- **Multi-device sync:** saves check Drive's file `version` first and three-way merge another device's changes (see *Multi-device Sync* below). Pulls changes on returning to the app, every 5 min, and when back online. Settings → ↻ Sync now.
- **Expired sign-in:** a Reconnect bar appears instead of silently failing; reconnecting keeps the session and saves anything pending.
- **Redirect sign-in fallback** ("Sign-in window not opening? Try another way" on the sign-in screen) for devices where the Google pop-up fails. Needs the app's URL added as an Authorized redirect URI on the OAuth client.

## Previous Session (Oct 11, 2026)
- **Streaming accuracy fixed:** providers now matched by TMDB `provider_id` with a normalised-name fallback (`"Disney+"` = `"Disney Plus"`, `MAX` = `HBO Max`, `Apple TV+` = `Apple TV`). Before this, 287 Disney titles never matched the saved `Disney+` subscription.
- **Streaming vs buy split:** `fetchProviders` no longer merges buy stores (Apple TV Store, Google Play Movies) into `streaming_platforms`; SG data is stored as `s.avail = { s, b, r }`
- **Settings → My Streaming Platforms** now lists TMDB's own Singapore provider list (cached 7 days in localStorage `tvworld-prov-sg`), with a count of your titles on each
- **🛒 Need to buy / 💾 Owned** — per season for TV (TMDB season-level providers), whole-title for movies and anime; card badges, season-header chips, detail summary
- **🛒 To Buy** sidebar page — Need to buy (grouped Watching / Paused / Plan to Watch) and Owned tabs; see *Availability & To Buy* below

## Previous Session (Sept 24–27, 2026)
- **Discover pages** (now TV / Movies / Anime / **J-Drama**): three signals per page — stars, TMDB recommendations, and (J-Drama) genre taste — see *Discover Pages* below
- **J-Drama Discover page** + Japanese live-action moved out of the TV and Movies pages; new `origin_country` / `original_language` fields on library entries; 🌏 Detect origins in Settings
- **Follow a star** — add an actor by name to any Discover page's tracked pool without owning any of their titles
- **🇯🇵 Japanese only** filter on All Series
- Discover pages renamed: sidebar Discover → TV · Movies · Anime · J-Drama; page titles "Discover …" (they're no longer only "from your stars")
- **Missing `type` bug fixed:** JSON-ingested entries had no `type`; backfilled to `'tv'` on load, ingest now sets it (was hiding e.g. *The Rookie* from Discover)
- **Auto status:** Watching ↔ Plan to Watch follows watch progress for TV/anime (one-time cleanup on load + on episode ticks)
- **All Series:** Hide completed toggle; scroll position kept when returning from a title (all views); floating ↑ Top button
- **Movies sidebar tab removed** (redundant with type pills)
- **Anime** Discover page added (voice-actor overlap across anime; series + films)
- **📅 Calendar** — new episodes + movie releases from the library, Jun 2025 → 18 months ahead; List / Month views (see *Calendar* below)
- **All Series hide chips:** Hide Plan to Watch / Completed / Dropped (replaces the single Hide completed checkbox)
- **Mobile bottom nav:** Plan → 📅 Calendar (Plan to Watch is in the ☰ drawer)
- **🔗 Match missing TMDB IDs** (Settings → TMDB) — bulk-matches entries without a `tmdb_id`, pulls their cast, and asks you to confirm uncertain ones
- Stars view no longer stays visible behind the detail view
- **Mobile layout rebuilt** (was missing from this copy) — see *Mobile* below

---

## What TV World Is

A single-file HTML app (`index.html`) that tracks TV shows, anime, and movies. Part of Marvin's Life Plan ecosystem of apps connected via Google Drive.

**Hosted:** GitHub Pages — repo files: `index.html`, `manifest.json`, `sw.js`, `icons/` (icon-192, icon-512, icon-maskable-512, apple-touch-icon)  
**Installable:** PWA on PC / Android / iPhone  
**Data:** `tvworld-data.json` on Google Drive  
**Auth:** Google OAuth 2.0 (drive.file scope)  
**Google Cloud Project:** Separate from ComicsTracker  
**Client ID:** `678456232174-ilpocmjaijsjsi4fr1or9r4o0to8vh9h.apps.googleusercontent.com`

---

## Tech Stack

- Vanilla JS, Tailwind CDN, no build step
- Google Identity Services (GIS) for OAuth
- TMDB API (v3, user's own key stored in app settings) for TV/movie metadata, providers, cast
- AniList GraphQL API (no key needed) for anime metadata and sequel chain resolution
- Google Drive as data layer

---

## Data Structure (`tvworld-data.json`)

```json
{
  "settings": {
    "tmdb_api_key": "",
    "subscribed_platforms": [],
    "subscribed_provider_ids": []
  },
  "series": [
    {
      "id": "uuid",
      "type": "tv | anime | movie",
      "title": "",
      "status": "Watching | Plan to Watch | Paused | Completed | Dropped",
      "genres": [],
      "synopsis": "",
      "poster_url": "",
      "tmdb_id": "string or null",
      "anilist_id": "string or null",
      "origin_country": ["JP"],
      "original_language": "ja",
      "year": "string or null",
      "rating": null,
      "streaming_platforms": [],
      "avail": { "at": 0, "seasonsAt": 0, "s": [[8, "Netflix"]], "b": [[2, "Apple TV Store"]], "r": [] },
      "owned": true,
      "cast": [{ "id": 123, "name": "", "character": "", "profile_path": "" }],
      "seasons": [
        {
          "season_number": 1,
          "avail": { "at": 0, "s": [[8, "Netflix"]] },
          "owned": true,
          "episodes": [
            {
              "episode_number": 1,
              "title": "Episode 1",
              "synopsis": null,
              "air_date": "YYYY-MM-DD or null",
              "watched": false
            }
          ]
        }
      ],
      "total_seasons": 1
    }
  ],
  "upcoming": [
    {
      "label": "Show S01E05 — airs 2026-09-08",
      "date": "2026-09-08",
      "type": "episode",
      "app": "tvworld",
      "priority": "normal",
      "url": "https://..."
    }
  ]
}
```

---

## Architecture Decisions

### Anime Data Pipeline
- **Search:** AniList GraphQL (reliable, no key needed)
- **Episode structure:** AniList sequel chain (`fetchAnilistChain`) — walks SEQUEL relations to build multi-season structure
- **Episode titles + air dates:** TMDB refresh (↻ TMDB button) — distributes TMDB episodes across AniList seasons sequentially when TMDB has fewer seasons
- **Providers + cast:** TMDB (`/watch/providers` + type-appropriate credits endpoint)
- **NO Jikan/MAL:** Jikan has been down since Aug 28 2026. Dropped entirely. MAL ID field removed.
- `anilist_id` stored on anime entries (root entry of the chain)
- `streamingEpisodes` from AniList intentionally NOT used — it returns cross-season data incorrectly

### Why AniList for Anime Structure
TMDB lumps anime cours into single seasons (e.g. Re:ZERO = 1 TMDB season). AniList has each cour as a separate Media entry linked via SEQUEL relations. The `fetchAnilistChain` function walks these to build proper multi-season structures.

### TMDB Refresh for Anime (key logic)
When TMDB has fewer seasons than AniList-built seasons:
- Flattens all TMDB episodes into a sequential list
- Distributes across AniList seasons in order
- Preserves `watched` state
- Preserves `anilist_id` and `type: 'anime'` — does NOT overwrite with TMDB metadata

### TV/Movies
- Search and metadata from TMDB directly
- No AniList involvement

### TMDB Search — TV vs Movie (critical)
TMDB movie IDs and TV IDs are **independent namespaces** — the same number refers to completely different titles across the two. The search UI has a **📺 TV / 🎬 Movie sub-toggle** (appears under the TMDB source pill, hidden when AniList is selected) that controls which endpoint is used:

| Toggle | Search endpoint | selectTMDB path |
|---|---|---|
| 📺 TV (default) | `/search/tv` | `/tv/{id}` + all seasons |
| 🎬 Movie | `/search/movie` | `/movie/{id}` metadata only, no seasons |

- `tmdbSearchType` global (`'tv'` default) drives this
- Resets to `'tv'` whenever switching back to TMDB source
- Previously: search always used `/search/tv` regardless — movies searched as TV would match a TV show at the same numeric ID and import that show's episodes instead. Only 2 entries in Marvin's library were affected by this: *Out of Practice* (= My Super Ex-Girlfriend) and *Joan of Arcadia* — both need to be deleted and re-added using the 🎬 Movie toggle

### Cast + Providers Endpoint Logic (critical — movie vs TV ID spaces are separate)
`fetchProviders` branches on `s.type`:

| Entry type | Providers endpoint | Cast endpoint | Cast response shape |
|---|---|---|---|
| `movie` | `/movie/{id}/watch/providers` | `/movie/{id}/credits` | flat `cast[]` sorted by billing order |
| `tv` / `anime` | `/tv/{id}/watch/providers` | `/tv/{id}/aggregate_credits` | `cast[]` with `roles[]` + `total_episode_count`; sorted by episode count; top role picked per member |

- All pre-fix movie entries had wrong cast/providers (336 of 604 movies had cast data, all wrong) — fix by running Settings → **↻ Refresh Cast (All)**
- TV entries are unaffected by the cast bug — `/tv/aggregate_credits` was always correct for them

### ↻ TMDB Refresh — type-aware
`refreshFromTMDB` branches on `s.type === 'movie'`:
- **Movie:** fetches `/movie/{id}` metadata only (title, synopsis, poster, genres, year) — no seasons, no episodes. Then calls `fetchProviders`.
- **TV / Anime:** fetches `/tv/{id}` + all seasons, merges episodes preserving `watched` state, then calls `fetchProviders`. Anime uses the flatten-and-distribute logic when TMDB has fewer seasons than AniList.

### Stars Page — Filmography Add Flow (persistence)
When adding a title from an actor's filmography:
- Person modal stays open underneath (not closed)
- Add modal is raised to `z-index: 200` to layer above it
- `filmographyReturnState` captures scroll position + active tab before opening add flow
- On save OR cancel/close of add modal: z-index reset, scroll restored via `requestAnimationFrame`, tab re-rendered so newly added titles show "✓ In Library" badge immediately
- Handled in both the save path and `closeModal('add-modal')`

### Discover Pages (TV / Movies / Anime / J-Drama)
Sidebar → Discover (four pages; also a 📺 / 🎬 / ⛩ / 🗾 switch under the page title). Each page ranks titles you haven't tracked by up to three signals, shown on every card and in the detail modal:
- **★ Stars** — people in the top-20 cast of your library titles, and everything else they've been in (`DISC_CAST_LIMIT = 20`, from `s.cast`). Badge `★ N`.
- **👍 Recommendations** — TMDB `/{tv|movie}/{id}/recommendations` (page 1) for each non-Dropped library title in that page's pool. "Recommended for <your title>" chips in the modal; badge `👍 N`. Cached 30 days per title.
- **◆ Genre taste (J-Drama only)** — `discGenreTop()` weights genres across your Japanese library titles (Completed ×1.5, Watching ×1.3, Plan/Paused ×0.8, Dropped ignored; × (0.5 + rating/5) when rated), takes the top 3 (leading with "Drama" only if there's nothing else), then queries `/discover/tv?with_origin_country=JP` and `/discover/movie?with_original_language=ja` (genre OR, vote-count floors, popularity sort, 3 pages each). Genre names are mapped to TV/movie IDs through the canon tables `DISC_GENRE_TV` / `DISC_GENRE_MOVIE` (TV has combined "Action & Adventure", "Sci-Fi & Fantasy", "War & Politics"; movies don't). Refreshed weekly or when the profile changes. Badge `◆`, "Matches your Mystery taste". Not built for the other pages (Western TV/movies would return the whole of TMDB).

**Pools (who counts as a tracked star / which titles seed recommendations):**
- *live* (TV + Movies pages) = library TV/movie entries that are **not** Japanese live-action and not Japanese animation misfiled as TV/Movie
- *jdrama* = library TV/movie entries where `isJapaneseLive(s)` — `origin_country` includes JP **or** `original_language === 'ja'`, and genres don't include Animation
- *anime* = library entries of type `anime`
Entries without recorded origin count as *live* until detected. `discEntryInPool`, `discBuildPool(kind)`, `discPoolKind(media)`. Prune keeps anyone in any pool, plus every followed star (below) regardless of whether their cast overlaps the library.

**Follow a star (`data.settings.followed_stars: [{ id, name, profile_path, kind }]`, synced via Drive):** a "+ Follow a star" row sits under the scan panel on every Discover page, showing removable chips for stars followed under that page's pool (`live` / `anime` / `jdrama`) plus a button that opens a TMDB `/search/person` modal (`openFollowModal`, `searchFollowStar`, `followStar`, `unfollowStar`). `discBuildPool` seeds the pool with followed stars first (`minOrder: -1`, `followed: true`) before folding in cast, so they rank ahead and are never bumped by a library appearance's billing order — only their `knownFrom` list grows if they do overlap. A followed star with no library titles shows "⭐ You follow this star" in the modal and "(followed)" on the card instead of a from-list. They still need a normal Scan/Resume to pull their filmography like any other tracked star; unfollowing doesn't clear their cached filmography (prune drops it once nothing needs it).

**Origin detection:** library entries gain `origin_country[]` + `original_language` (saved in tvworld-data.json). Set by: the add flow (`selectTMDB` → `pendingOrigin` → `saveSeries`), ↻ TMDB (`refreshFromTMDB`), 🌏 Detect origins in Settings (`detectOriginsAll`), and automatically as step 1 of any TV / Movies / J-Drama scan (`discDetectOrigin`; one `/tv|movie/{id}` call each). `Array.isArray(s.origin_country)` = "checked" (empty array = checked, unknown). Movies fall back to `production_countries` when `origin_country` is empty.

**Scan (`toggleDiscoverScan`, per page):** 1) origin check → 2) recommendations → 3) genre browse (J-Drama) → 4) filmographies (`/person/{id}/combined_credits`, top-billed first). 6 parallel lanes, retries on 429/5xx, stops on 401, Pause/Resume, incremental. `discPlan(media)` computes what's outstanding (also drives the panel text). "Rescan all" clears filmographies + recommendations (not origins).

**Credit classification (`discClassifyCredit`, at fetch):** drops Self/Himself/Herself credits, TV News/Reality/Talk (10763/10764/10767) and pre-1990. Stored code: 0 movie · 1 TV · 2 anime series · 3 anime film · 4 J-drama series · 5 J-drama film. Anime = genre 16 + JP/KR/CN/TW origin or ja/ko/zh. J-drama = JP origin or `ja` language, no Animation genre (so live-action tokusatsu counts). Pages show: TV → 1, Movies → 0, Anime → 2/3, J-Drama → 4/5 (Series / Films pills on Anime + J-Drama). Because anime = genre 16, a title can never land on both Anime and J-Drama.

**Cache:** browser IndexedDB `tvworld-discover` v2 — stores `personCredits` (`{ v: 3, c: rows }`), `recs` (`tv:{id}` / `movie:{id}` → `{ at, r }`, plus `genre:jdrama` → `{ at, sig, r }`), `meta`. NOT Drive; per browser. Person records: anime needs v ≥ 2, TV/Movies/J-Drama need v3, so pre-J-Drama records are quietly re-fetched (that's the one-time full rescan when Japanese titles were moved out; anime results are not lost).

**Layout:** tiers — In your library (green outline, bold) → Strong matches (2+ signals, counting each star and each recommending title) → Single match. Sort: **Best match** (default: stars×1 + recs×1.5 + genre×0.5 each) / Most Stars / A→Z / Year (New) / Year (Old), applied within each tier. "Hide titles in my library" toggle; search matches title or star name; infinite scroll in 120-item chunks.

**Library matching:** TV/Movies by TMDB ID; Anime series by TV ID against anime (or TV) entries, anime films by movie ID else normalised title (best effort); J-Drama series by TV ID, films by movie ID.

**Card click:** modal with synopsis (lazy TMDB fetch), recommendation/genre chips, each star's role + which library titles they're known from; + Add to library → TMDB add flow with the right TV/Movie endpoint (Anime → AniList search pre-filled). Page re-renders in place after saving.

**Detail back-navigation:** `detailReturn` makes ← Back from a detail view return to Stars/Discover/Calendar with scroll restored.

### Missing `type` on older entries
Entries created via Ingest JSON used to be saved with no `type` field (`normalizeIngestedSeries` never set it). The library grid treated them as TV (`s.type || 'tv'`), but strict `=== 'tv'` checks elsewhere skipped them — e.g. Discover didn't mark them In Library or count their cast. Fixed three ways: `loadFromDrive` backfills `type: 'tv'` on load (persists on next save), ingest now sets `type` (from `raw.type`, else `media_type`, else `'tv'`), and Discover defaults missing type to `'tv'`. Discover's scan panel also reports TV/movie entries with no `tmdb_id` (these can't be matched).

### Auto status from watch progress (TV / Anime)
`applyAutoStatus(s)`: Watching with 0 episodes watched → Plan to Watch; Plan to Watch with ≥1 watched → Watching. Runs on `toggleEp` and `bulkMarkSeason` (detail select + badge update in place, toast), plus a one-time cleanup on load guarded by `settings.auto_status_v1`. Paused / Completed / Dropped and movies are never changed. Manual status changes via the selector are respected until episodes are next ticked.

### All Series 🇯🇵 filter
"Japanese only" pill next to the type toggles (`toggleJpFilter`, `jpOnly`), All Series only — same `isJapaneseLive` used by J-Drama. Not persisted between sessions.

### List navigation
- **Hide chips** on All Series only: Plan to Watch / Completed / Dropped (`hiddenStatuses` Set, localStorage `tvworld-hide-statuses`; the old `tvworld-hide-completed` key is carried over once).
- **Scroll persistence:** `detailReturn` now records the originating view + scroll for every view except Settings; `openDetail` resets scroll to top, `closeDetail` returns to the view and restores scroll.
- **↑ Top** floating button (`#to-top-btn`) appears once `#main` is scrolled past 600px.

### Calendar (`#calendar-view`, sidebar 📅 Calendar, mobile bottom nav)
- **Window:** 1 Jun 2025 (`CAL_START`) → today + 18 months. Month navigation is clamped to that range.
- **Sources:** (1) episode `air_date`s already in the library; (2) IndexedDB `tvworld-calendar` cache of fresh TMDB dates — `tv:{id}` → `{ at, ended, eps: [[season, ep, date, name]] }` (only seasons airing from 2025 on, plus the latest/next season), `movie:{id}` → `{ at, primary, theatrical, theatricalSG, digital, digitalCountry }` from `/movie/{id}?append_to_response=release_dates` (SG theatrical type 3/2, else primary date; digital type 4 SG, else US). **Library data is never modified** — ↻ TMDB on a show still adds new episodes to the tracker.
- **Merging:** TV — keyed by season/episode; TMDB date wins, library keeps `watched`. Anime — TMDB numbering differs from AniList seasons, so cache episodes only fill days the library has nothing for, and are labelled "New episode · name" without an SxxEyy code. Multiple episodes of one show on one day are grouped ("S02E01–E08 · 8 episodes").
- **Refresh:** automatic when the cache is > 12h old, plus ↻ Refresh dates. Shows: every 12h; shows TMDB marks Ended/Canceled with nothing aired since the window start every 30 days (so a revival is picked up within a month). Manual ↻ Refresh dates re-checks every show, including ended ones. Movies: only `year ≥ 2024` (or no year); every 12h while the release is recent/unknown, else every 30 days. Dropped titles are skipped. 5 parallel requests via `discFetchJSON`.
- **Views:** List (grouped Last week / This week / Next week / Week of…, auto-scrolls to this week; for the current month the list runs 14 days past today so month-end never cuts it off) and Month grid (chips on desktop, coloured dots on mobile; tap a day for its list). Mode persisted in localStorage `tvworld-cal-mode`. Header shows a rolling *Next 7 days* summary.
- **Filters:** 📺 TV / ⛩ Anime / 🎬 Movies pills; "Watching & Plan to Watch only". Dropped never shown.
- **Items:** tap → detail view; ← Back returns to the calendar at the same scroll. Cinema = pink, digital = amber, TV = blue, anime = purple. "✓ Watched" when every episode in the group is ticked.

### Availability & To Buy (Singapore)
- **Source:** TMDB `/{tv|movie}/{id}/watch/providers` (JustWatch data — attribution shown in the detail view and To Buy page) and, for TV, `/tv/{id}/season/{n}/watch/providers`. Region constant `AVAIL_REGION = 'SG'`.
- **Stored (Drive):** `s.avail = { at, seasonsAt?, s: streaming (flatrate+ads+free), b: buy, r: rent }` as `[provider_id, name]` pairs; `season.avail = { at, s }`; `season.owned` / `s.owned` (movies and shows with no season data). `streaming_platforms` now holds streaming names only.
- **Subscriptions:** `settings.subscribed_provider_ids` + `subscribed_platforms` (names). `subscribedMatcher()` matches by ID, else `provNorm(name)` (lowercase, `+`→`plus`, strips "with Ads"/"Kids" suffixes, aliases `appletvplus→appletv`, `max→hbomax`). Old name-only settings keep working until re-saved.
- **Season status (`availInfo`):** owned → `owned`; on a subscribed service → `stream`; not checked → `unknown`; nothing aired yet → `upcoming`; else `buy`. Per-season data is used only once TMDB has returned some for that show (otherwise title-level). Anime always uses title-level (AniList seasons ≠ TMDB seasons).
- **Need to buy (`availNeed`):** status Watching / Paused / Plan to Watch, and (seasons) aired with unwatched episodes. Partly-streaming shows appear in both places — e.g. "S1 on Netflix · S2 on Disney Plus" plus "Need S3–4". Marking owned removes it from Need to buy for good.
- **Refresh:** 3-day TTL (`AVAIL_TTL`). Opening a title refreshes it in the background (`availMaybeRefresh`; season calls only for active TV); opening To Buy refreshes stale active titles (`refreshAvailability`, 5 lanes); ↻ Check availability forces all active titles; "↻ Check now" on a detail view forces one. Add / ↻ TMDB / Refresh Cast refresh title-level only.
- **UI:** card badge `🛒 Buy S3–4` / `💾 Owned`; season header chip (tap toggles owned); detail summary with your services, other services, Buy/Rent stores, last checked; To Buy page with Need/Owned tabs, type pills, search, per-season "Mark owned" + "Mark all owned".
- **Not available from TMDB:** arrival / leaving-soon dates. Season-level SG data is patchy — a season TMDB has no data for will show as Need to buy.

### Multi-device Sync
- **State:** `driveVersion` (Drive `version` of the file as last loaded/saved), `syncBase` (snapshot: settings JSON + Map of series id → JSON), `unsavedChanges`, `saveRunning` / `saveQueued`, `tokenExpiresAt`, `appReady`.
- **Save (`saveToDrive` → `saveOnce`):** queued (a change made mid-save triggers one follow-up save). Checks `files/{id}?fields=version`; if it differs, downloads, `mergeData(local, remote, syncBase)`, re-renders (`rerenderAfterSync`) and toasts. Uploads with `fields=id,version`; `syncBase` is rebuilt from the exact body uploaded.
- **Merge rules:** per title — changed on one side wins; both changed → per field, with `seasons` merged per episode `watched` and per season `owned` (episode structure from whichever side changed it, e.g. ↻ TMDB). Same thing changed on both sides → this device wins. Added on either side kept; deleted on one side is dropped unless the other side edited it since. Settings merged per key.
- **Pull (`syncNow`):** on visibility return (>15 s hidden), every 5 min while visible, on `online`, and Settings → ↻ Sync now. Local changes → save (which merges); else replace with Drive's copy if the version moved.
- **Known gap:** a save landing between another device's version check and upload (sub-second window) can still overwrite — Drive v3 has no conditional upload.
- **Leaving with unsaved changes** triggers the browser's "leave page?" prompt.

### Sign-in
- Pop-up token flow (GIS `initTokenClient`); tokens last 1 hour (`tokenExpiresAt` = expiry − 60 s). Expiry or a 401 → `needReconnect()` shows `#reconnect-bar`; `reconnect()` requests a token with `prompt: ''`, and the GIS callback calls `onTokenRenewed()` (no reload) when `appReady`.
- **Redirect fallback:** `startRedirectSignIn()` → Google OAuth implicit flow with `redirect_uri = origin + pathname`; `consumeRedirectToken()` reads `#access_token` on load. Requires that URL under *Authorized redirect URIs* in Google Cloud → Credentials → the OAuth client. Loses unsaved in-memory changes, so it's only offered on the sign-in screen.

### PWA
- `sw.js` (`CACHE = 'tvworld-v4'`; bump on every release): same-origin network-first with cache fallback (new `index.html` uploads apply on next online open); Tailwind / Google Fonts stale-while-revalidate; Google, Drive, TMDB, AniList untouched. Registered only on https / localhost.
- `beforeinstallprompt` → Install button in Settings (`renderInstallSection`, `doInstall`); iOS / Android / desktop instructions otherwise; "✓ installed" when `display-mode: standalone`.
- The app still needs internet to sign in and load data (no offline copy of your library).

### Drive Save Error Handling (important)
- `fetch()` only throws for network failures — HTTP 4xx/5xx from Drive API do NOT throw; they return a response with `res.ok === false`
- Both `saveToDrive` and `loadFromDrive` check `res.ok` before proceeding
- 401 errors show "Session expired — please sign in again to save" toast
- Other HTTP errors show "Save failed (statusCode)" toast
- Google OAuth tokens expire after **1 hour** — the most likely cause of silent save failures in long sessions; re-signing in refreshes the token

---

## Current Features

### Library
- Unified grid: TV, Anime, Movies in one view
- Sidebar: All Series, Watching, Plan to Watch, Paused, Completed, Dropped, Stars, Calendar; Discover section (TV, Movies, Anime, J-Drama)
- All Series: 🇯🇵 Japanese only filter alongside the type toggles
- No separate Movies tab (removed — it duplicated All Series + 🎬 pill, and showed nothing when the 🎬 pill was off). Use the type pills to filter by TV / Anime / Movie
- Hide chips on All Series (Plan to Watch / Completed / Dropped); scroll position kept when returning from a title; floating ↑ Top button
- Floating type toggle pills: 📺 TV · ⛩ Anime · 🎬 Movie (multi-select, filters within status view)
- Sort bar: Added (New/Old), A→Z, Z→A, Year (New/Old)
- Continue Watching strip (TV/Anime only)
- Card badges: AL badge for AniList-sourced anime

### Adding Content
- Search tab: TMDB (TV / Movies) | AniList (Anime) source toggle
  - When TMDB is selected: 📺 TV / 🎬 Movie sub-toggle appears (defaults to TV)
  - TV search: `/search/tv` → imports seasons + episodes
  - Movie search: `/search/movie` → imports metadata only, no episodes
- AniList flow: search → select → `fetchAnilistChain` builds all seasons → `autoMatchTMDB` fills TMDB ID → save → ↻ TMDB for episode titles/dates
- Manual entry with type selector (TV/Anime/Movie)
- Ingest JSON: drag-and-drop, merges by ID or title

### Detail View
- Progress bar (TV/Anime)
- Season accordions with per-episode checkboxes
- Bulk mark season watched/unwatched
- Cast strip (from TMDB credits, type-aware)
- Status selector
- ↻ TMDB button (all types, type-aware), ↻ AniList button (anime with anilist_id)
- Recap placeholder button (Sprint 10)

### Stars Page
- Grid of all cast members across library
- Sorted by: Most Titles, Fewest, A→Z, Z→A
- Clicking opens person modal
- Person modal: Movies / TV Shows / Anime tabs
- Anime detection: genre_id 16 (Animation) + JP/KR/CN/TW origin country
- Auto-opens to Anime tab if person has more anime than TV credits
- 1990 hard cap: hides works entirely before 1990 (unless they continued into 1990+)
- Tab labels show filtered count e.g. `Anime (147)`
- Adding from filmography opens full Search flow (not a small sheet):
  - Anime → AniList search pre-filled with title
  - TV/Movie → TMDB data fetched directly
  - **Person modal persists underneath; scroll position restored on return**

### Settings
- TMDB API key
- **🌏 Detect origins** (`detectOriginsAll`) — records `origin_country` + `original_language` for every TV/movie entry with a `tmdb_id` (one TMDB call each); needed to tell Japanese live-action from everything else. Also runs automatically at the start of a Discover scan.
- **🔗 Match missing TMDB IDs** (`bulkMatchTmdb`) — for every entry without a `tmdb_id`: batch-fetches AniList titles (English / romaji / native, format, start year) for anime with an `anilist_id`, searches TMDB (`/search/tv` for TV + anime — same namespace `fetchProviders` uses — `/search/movie` for movies) with each title in turn, scores with `tmdbScore` (native Japanese vs TMDB `original_name` counts as an exact match), and applies clear matches (best ≥ 0.75 and > 0.2 ahead of the runner-up — same rule as `autoMatchTMDB`) followed by `fetchProviders`. Uncertain ones open the **Confirm TMDB matches** modal (poster picks with year + country, or Skip). AniList `format: MOVIE` anime are skipped (the anime pipeline is TV-only). Does not refresh episode titles — use ↻ TMDB per title for that.
- **↻ Refresh Cast (All)** — bulk re-fetches providers + credits for every entry with a tmdb_id using correct type-aware endpoints; 300ms throttle per entry; use to fix any stale/wrong cast data
- Subscribed streaming platforms — chips from TMDB's SG provider list (by provider ID), with library counts; drives Need to buy
- AniList Integration: Import from AniList (by username, no auth needed), Resync All Anime
- TMDB Account Import: OAuth flow, imports watchlist/rated/favourites/custom lists
- Export JSON, Force Save

### Mobile (rebuilt Sept 24, 2026 — `@media (max-width: 768px)`)
- **Top bar:** ☰ menu · TV World wordmark · ⚙️ Settings
- **Bottom nav:** All · Watching · Calendar · Discover · More (`updateBottomNav()` sets active state from `currentView`; Discover remembers TV vs Movies via `lastDiscoverView`; More opens the drawer)
- **Drawer:** the desktop sidebar slides in from the left (`openDrawer()` / `closeDrawer()`, `body.drawer-open`); closes on backdrop tap, any sidebar item, or `switchView`
- **FAB (+)** opens the add flow; the desktop "+ Add Series" toolbar button is hidden (`.desktop-only`). ↑ Top moves to bottom-left
- **Layout:** 2-column library and Discover grids, 3-column Stars and filmography grids, stacked detail hero with full-width status select, larger episode tick targets, sort bars scroll sideways (Hide completed / Hide library toggles sit at the end)
- **Modals** become bottom sheets (92dvh, safe-area padding); form rows stack
- Inputs use 16px text to stop iOS zoom-on-focus; `viewport-fit=cover` + safe-area insets for notched phones; toast sits above the bottom nav
- Discover page has a 📺 TV / 🎬 Movies switch under the title (all screen sizes)

### Upcoming[] Envelope
- Computed on every save
- Next unwatched episode for all Watching TV/Anime where air_date is in the future
- Schema compatible with Life Plan Dashboard (Sprint 7)

---

## Known Limitations / Future Work

- **J-Drama coverage:** TMDB cast lists and recommendations are patchier for Japanese live-action, and titles with no cast contribute no stars. Recommendations and genre browse carry the page when the library has few J-dramas. If the library has no Japanese live-action at all there is nothing to build a taste profile from.
- **Mis-tagged anime:** a Japanese anime TMDB doesn't tag as Animation would appear on J-Drama. No per-card Hide button yet.
- **Ideas not built:** a per-card Hide button for mis-tagged anime on J-Drama; K-drama / other-region pages (the origin fields make this a small addition); genre browse for the TV / Movies / Anime pages; persisting the 🇯🇵 filter between sessions.

- **Episode titles:** AniList's `streamingEpisodes` field is broken/cross-season — titles come from TMDB refresh only. Consequence: newly added anime show "Episode 1, Episode 2…" until you hit ↻ TMDB.
- **AniList import:** Imports with `anilist_id` but no `tmdb_id`. Run 🔗 Match missing TMDB IDs in Settings to fill IDs + cast in bulk; episode titles still need ↻ TMDB per entry.
- **Anime films** (AniList `format: MOVIE`) aren't TMDB-matched in bulk — the anime pipeline uses TV endpoints. Set manually if needed.
- **Recap button:** Greyed out, "Coming Soon" — placeholder for Sprint 10.
- **Life Plan Dashboard:** live. Reads this app's `upcoming[]` (needs the one-time Pick missing files grant).
- **Movies:** No episode tracking. Status options: Plan to Watch / Completed / Dropped only.
- **Corrupted entries to fix manually:** *Out of Practice* (= My Super Ex-Girlfriend, tmdb_id:4474) and *Joan of Arcadia* (tmdb_id:1593) — both were added via TV search and contain TV episode data. Delete and re-add using 🎬 Movie toggle.

---

## Key Functions Reference

| Function | Purpose |
|---|---|
| `fetchAnilistChain(id)` | Walks AniList SEQUEL chain from root entry, returns ordered array of Media objects |
| `selectAnilistAnime(id)` | Full AniList add flow: chain → seasons → form fill → autoMatchTMDB |
| `autoMatchTMDB(title, year)` | Searches TMDB by title+year, auto-fills or shows picker if ambiguous |
| `searchTMDB()` | Searches `/search/tv` or `/search/movie` depending on `tmdbSearchType`; results call `selectTMDB` with correct mediaType |
| `setTmdbSearchType(type)` | Switches TV/Movie sub-toggle; clears results |
| `selectTMDB(tmdbId, mediaType)` | TV: fetches series + all seasons, fills manual form. Movie: fetches `/movie/{id}` metadata only, sets type to movie |
| `refreshFromTMDB()` | Type-aware: movie → `/movie/{id}` metadata + fetchProviders only; tv/anime → `/tv/{id}` + seasons merge + fetchProviders |
| `refreshFromAnilist()` | Re-walks sequel chain, rebuilds all seasons, preserves watched state |
| `bulkResyncAnilist()` | Runs syncEntryFromAnilist on all anime with anilist_id |
| `syncEntryFromAnilist(s)` | Re-walks chain for one entry, rebuilds seasons preserving watched state |
| `fetchProviders(seriesId, tmdbId)` | Type-aware: movie → `/movie/` endpoints + flat credits; tv/anime → `/tv/` endpoints + aggregate_credits. Writes streaming_platforms + cast. |
| `bulkRefreshCast()` | Re-runs fetchProviders for all entries with a tmdb_id; 300ms throttle; status in Settings |
| `saveToDrive()` / `saveOnce()` | Queued save: version check → merge if Drive changed → upload; Reconnect bar on 401 / expired token |
| `loadFromDrive()` | Loads data + version from Drive on sign-in, sets `syncBase` |
| `syncNow(manual)` | Pull other devices' changes (or save + merge if there are local ones) |
| `mergeData(local, remote, base)` / `mergeSeries` / `mergeSeasons` / `pick3` | Three-way merge |
| `rerenderAfterSync()` | Redraws the current view after a merge/pull, keeping scroll |
| `needReconnect()` / `reconnect()` / `onTokenRenewed()` | Expired sign-in handling |
| `startRedirectSignIn()` / `consumeRedirectToken()` | Redirect sign-in fallback |
| `renderInstallSection()` / `doInstall()` | Settings → Install on this device |
| `buildUpcoming()` | Computes upcoming[] envelope, called on every saveToDrive() |
| `renderStarsGrid()` | Aggregates cast across library, renders Stars page with sort |
| `openPersonModal(id)` | Fetches TMDB person credits, splits into Movies/TV/Anime tabs |
| `toggleDiscoverScan()` | Starts / pauses / resumes the Discover scan (origin → recommendations → genre → filmographies), writes to IndexedDB, then prunes |
| `discPlan(media)` | What's outstanding for a page (origins, stars, recommendation lists, genre browse) |
| `isJapaneseLive(s)` / `isJapaneseAnimation(s)` | Library-entry origin classification used to split the pools |
| `originFromTmdb(json)` | Extracts `{ origin_country, original_language }` from a TMDB detail response |
| `detectOriginsAll()` | Settings → 🌏 Detect origins |
| `discGenreTop()` / `discFetchGenre()` | J-Drama taste profile and TMDB Discover query |
| `discAggregate(media)` | Builds the title → tracked-stars map for `'tv'`, `'movie'` or `'anime'` from the cache |
| `discClassifyCredit(c)` | Classifies a TMDB credit (codes 0–5: movie / TV / anime series / anime film / J-drama series / J-drama film), or drops it |
| `renderDiscover(keepPosition)` | Tiers, sorts, filters and renders the Discover grid |
| `applyAutoStatus(s)` | Watching ↔ Plan to Watch from watch progress (TV/anime only); returns true if changed |
| `setHideCompleted(on)` | All Series hide-completed toggle; persists to localStorage |
| `bulkMatchTmdb()` | Bulk TMDB ID matching for entries missing `tmdb_id`; review modal for uncertain matches |
| `tmdbScore(results, titles, year)` | Scores TMDB search results against several titles (Unicode-aware normalisation) |
| `openCalendarView()` / `renderCalendar(keepScroll)` | Calendar entry + render (List or Month) |
| `calBuildEvents()` | Merges library episode dates + TMDB cache into dated events, grouped per show per day |
| `calRefresh(force)` | Fetches fresh TV season / movie release dates into the calendar cache |
| `toggleHideStatus(status)` | All Series hide chips |
| `toggleJpFilter()` | All Series 🇯🇵 Japanese-only filter |
| `followStar(person, kind)` / `unfollowStar(id, kind)` / `discFollowed(kind)` | Follow-a-star add/remove and lookup |
| `openFollowModal()` / `searchFollowStar()` | Follow-a-star search UI (TMDB `/search/person`) |
| `availInfo(s)` / `availNeed(info)` / `availOwned(s, info)` | Per-season / whole-title availability status, need-to-buy and owned lists |
| `fetchAvailability(s, key, withSeasons)` | SG providers (title + optional per-season) into `s.avail` / `season.avail` |
| `refreshAvailability(force)` / `availMaybeRefresh(s)` / `checkAvailabilityOne(id)` | Bulk (To Buy), background (detail open), and forced single refresh |
| `subscribedMatcher()` / `provNorm(name)` | Provider matching by ID or normalised name |
| `toggleSeasonOwned(id, si)` / `toggleTitleOwned(id)` / `markSeasonsOwned(id, sis)` | Owned toggles |
| `openBuyView()` / `renderBuyView(keepScroll)` | 🛒 To Buy page |
| `loadProviderCatalog()` / `buildPlatformGrid()` / `savePlatforms()` | Settings platform chips from TMDB SG provider list |
| `addFromFilmographyFull(item, tab)` | Saves `filmographyReturnState`, raises add-modal z-index, opens full Search flow; person modal stays open underneath |

---

## Global State Variables (notable)

| Variable | Purpose |
|---|---|
| `tmdbSearchType` | `'tv'` (default) or `'movie'` — controls which TMDB search endpoint is used; reset to `'tv'` when switching back to TMDB source |
| `filmographyReturnState` | `{ scrollTop, tab }` — set by `addFromFilmographyFull`, cleared on save or cancel; drives person modal scroll restoration |
| `personFilmData` | `{ movies, tv, anime }` — loaded by `openPersonModal`, consumed by `renderFilmTab` |
| `currentFilmTab` | Active tab in person modal: `'movies' \| 'tv' \| 'anime'` |
| `editingSeriesId` | Non-null when add modal is in edit mode |
| `detailReturn` | `{ view, scroll }` — where ← Back from a detail view returns to; set by `openDetail`, used by `closeDetail` |
| `hiddenStatuses` | Set of statuses hidden on All Series (localStorage `tvworld-hide-statuses`) |
| `calMonth` / `calMode` / `calTypes` / `calActiveOnly` | Calendar state: displayed month, `'list' \| 'month'`, type filter, active-only toggle |
| `discMedia` / `discSort` / `discHideTracked` / `discFormats` | Discover page state: `'tv' \| 'movie' \| 'anime' \| 'jdrama'`, sort key (default `'match'`), hide-library toggle, Series/Films pills for Anime + J-Drama |
| `pendingOrigin` | `{ tmdb_id, origin_country, original_language }` captured by `selectTMDB`, applied by `saveSeries` for a new entry |
| `buyTab` / `buyTypes` | To Buy page: `'need' \| 'owned'`, type filter |
| `accessToken` / `tokenExpiresAt` | Google OAuth token and its expiry (1 hour); Reconnect renews without reloading |
| `driveVersion` / `syncBase` / `unsavedChanges` / `appReady` | Multi-device sync state |

---

## Monthly Refresh (replaces the old sync workflow)

The `tvworld-sync.md` / `tvworld-updates-YYYY-MM.json` routine was retired on 2026-10-20. Use Settings → **Refresh All from TMDB** once a month; the Life Plan dashboard shows a reminder when it's due.

For anime: ↻ AniList resync handles metadata; ↻ TMDB handles episode titles/dates.

---

## Re:ZERO as Test Case

Complex structure: 5 AniList entries (S1, S2P1, S2P2, S3, S4).
- AniList chain correctly builds 5 seasons
- TMDB has 1 season with all episodes — refresh distributes them across 5 seasons
- Working correctly as of latest build

---

## Change log
- **Up to Oct 13, 2026:** see the session notes at the top (kept as a downloadable file before this date).
- **ver 3 (2026-10-22):** moved into the repo as `PROJECT_STATE.md`; added Session Rules; recorded the Oct 20–22 changes (monthly refresh, movie dates, icon, manifest id).
