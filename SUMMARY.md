# 🗺️ IIT Madras Navigation PWA - Project Summary & Architectural Log

An offline-first, high-performance, mobile-optimized Progressive Web Application (PWA) designed for seamless campus navigation within IIT Madras. This project is built using a strict **Two-File Architecture** (`index.html` and `sw.js`), running entirely client-side without any external backend requirements.

---

## 🏗️ Core Architecture & Tech Stack

### 1. Two-File PWA Architecture
*   **`index.html`:** The entire frontend application shell, containing:
    *   Responsive, Google Maps inspired modern UI/UX styles.
    *   **Dynamic Data Engine:** Direct client-side integration with the **Overpass API** to fetch the latest campus buildings, paths, and boundaries.
    *   **Offline Cache:** Utilizes **IndexedDB** (v6) to store the processed routing graph and POI data locally for instant offline loading and periodic 3-month background updates.
    *   Leaflet.js map layer initialization with strict boundary enforcement and high-zoom level support (up to 22).
    *   High-accuracy client-side pathfinding engine (Custom Dijkstra) with "Virtual Building Nodes" for multi-entrance buildings.
    *   Live GPS tracking with geofencing and movement-based heading calculation.
*   **`sw.js` (Service Worker):** The offline engine:
    *   Handles aggressive caching of local assets and CDNs.
    *   Performs automated, rate-limited batch caching of map tiles.
    *   **Tile-Sync:** Automatically refreshes the map tile cache when the routing data is updated from Overpass.

---

## 📦 What Was Done & How

### Phase 1: Dynamic Data & Graph Processing
*   **Overpass Fetching:** Replaced massive hardcoded datasets with a dynamic fetcher. The app queries Overpass for `building`, `highway`, `amenity`, and `leisure` tags within the IIT Madras area.
*   **Client-Side Graph Formulation:** The browser now processes raw OSM GeoJSON on-the-fly to build a routable graph. 
*   **Complex Building Topology:** Implemented "Virtual Nodes" for buildings. Every building center is connected to all its detected entrances with 0-weight edges, allowing the routing engine to automatically find the most optimal entrance door based on the user's location.

### Phase 2: Map Rendering & Strict Geofencing
*   **Leaflet.js Integration:** Configured with `maxBoundsViscosity = 1.0` to strictly lock the user inside the campus boundary.
*   **Dynamic Boundary Extraction:** The app extracts the official campus polygon from OSM data to define strict geofencing.
*   **GPS Validation:** If a user's GPS position is detected outside the campus boundary, the app shows an "Invalid Location" notification and refuses to start navigation to ensure campus-exclusive reliability.

### Phase 3: Robust Search & POI Discovery
*   **Fuzzy Alphanumeric Search:** The search engine ignores punctuation, spaces, and casing. Searching for "ICSR" correctly finds "IC&SR Building".
*   **Alias Support:** Extracts `short_name`, `alt_name`, and `name:en` tags. The system correctly identifies common acronyms like **CLT** for Central Lecture Theatre.
*   **Formatted UI:** Automatically cleans raw OSM tags (e.g., "student_accommodation" -> "Student Accommodation") and replaces generic "yes" labels with "Building".

### Phase 4: User-Friendly Turn-by-Turn Navigation
*   **Compass Field-of-View:** Replaced simple arrows with a modern, semi-transparent **blue FOV cone** on the user dot, indicating the physical direction the user is facing.
*   **Turn Smoothing:** The engine now ignores path curves smaller than 25 degrees, only prompting for structural turns to reduce instruction "noise".
*   **Wrong-Way Detection:** Compares movement bearing with the planned route. If the user walks in the opposite direction, the HUD explicitly prompts them to **"Turn around"**.
*   **Aggressive Rerouting:** Reduced the off-route threshold to **12 meters**. If a turn is missed, the system recalculates almost instantly to get the user back on track.

### Phase 5: Routing Integrity & Network-First Caching
*   **Building Shortcut Elimination:** Refactored the graph logic so buildings only attach to the nearest road using actual distance as a weight penalty, completely stopping Dijkstra from using buildings as zero-cost shortcuts.
*   **Alternative Route Optimization:** Re-engineered the alternate routing mechanic to dynamically select the nearest future non-building node for blocking, completely avoiding staleness and off-by-one bugs. Recalculation now purges the entire path and re-routes from the exact GPS location.
*   **Network-First Auto Updates:** Overhauled the Service Worker to apply a strict Network-First strategy to all local app shell files (`.html`, `.js`, `.css`), enabling instant over-the-air updates from GitHub Pages for online users while retaining fallback offline support.

### Phase 6: Stability Hotfixes
*   **SW Immediate Activation:** Added `self.skipWaiting()` to the SW install event so updated service workers activate immediately on next load, without requiring the user to close all tabs.
*   **TypeError: `nid.startsWith` Fix:** OSM node IDs from the Overpass API are raw numbers, not strings. The building graph connection logic now coerces all node IDs with `String(nid)` before calling `.startsWith('b_')`, eliminating a hard crash during graph processing.
*   **Graceful 429 Error Handling:** When the Overpass API rate-limits the client and the local cache is empty, the loading screen now shows a clear explanation and a **"↺ Retry"** button instead of silently failing with a cryptic error.
*   **SW `waitUntil` InvalidStateError Fix:** Moved `event.waitUntil()` to be called synchronously before the async fetch chain in the Stale-While-Revalidate handler, preventing the SW from throwing an `InvalidStateError` when attempting to extend a fetch event that had already settled.

### Phase 7: Search Quality & Data Integrity
*   **Campus-Only Geofencing:** Added a ray-casting point-in-polygon function. Every building extracted from OSM is checked against the official campus boundary polygon before being added to the search index. Buildings in the bounding box but outside the actual campus are silently discarded.
*   **Clean POI Type Labels:** Replaced raw OSM tag values (`yes`, `student_accommodation`, `fast_food`, etc.) with a curated `TYPE_LABELS` lookup table that produces human-readable labels (`Building`, `Student Accommodation`, `Food Stall`). Any unlabelled tag is auto-capitalised and de-underscored as a fallback.
*   **IndexedDB v7:** Bumped the cache version to force a clean graph rebuild incorporating the geofenced buildings and corrected type labels.

---

## ⚡ Service Worker precaching & Offline Strategy
1.  **Tile Bounding Box Math:** Identifies tiles required across **Zoom levels 13 to 19**.
2.  **Rate-Limited Batch Caching:** Fetches tiles in small batches (15 tiles) with throttle delays to comply with OSM usage policies.
3.  **Tile Cache Invalidation:** When fresh Overpass data is fetched (every 3 months), the client triggers a `REFRESH_TILES` message to the Service Worker to synchronize the visual map with any new road/building changes.

---

## 🛠️ Major Debugging & Refactoring Milestones

### 1. Overpass GET vs POST (404/504 Fix)
*   **Issue:** Long queries caused 404 errors due to URL length limits, and large area requests timed out.
*   **Resolution:** Switched to **POST** requests with a **900s timeout**, ensuring reliable data retrieval for the massive IITM campus area.

### 2. Boundary Segment Merging
*   **Issue:** The campus boundary failed to render because relations consisted of multiple disjointed ways.
*   **Resolution:** Implemented a robust segment merger that combines all "outer" ways into a single continuous polygon for Leaflet geofencing.

---

## 📈 Current Project State & Verification Status
*   **Offline Capability:** 100% Functional.
*   **Navigation Stability:** High (includes segment-distance math, path smoothing, and wrong-way detection).
*   **Search Robustness:** Excellent (supports abbreviations and fuzzy matching).
*   **UI/UX Aesthetic:** Modern, high-performance, Google Maps inspired.
*   **Service Worker:** Robust — immediate activation, Network-First shell updates, graceful error recovery.
*   **Direction Arrow:** Stable — angle-unwrapped rAF easing with low-pass sensor filtering; no more oscillation.

---

### Phase 8: Direction Arrow Stabilisation

*   **Root Cause 1 — CSS `transition` wrap-around spin:** The arrow had `transition: transform 0.25s ease-out`. When the raw compass angle jumped near the 0°/360° boundary (e.g., `350° → 10°`), the browser interpolated **backward** through 340° instead of the short 20° path, causing a near-full-circle spin. **Fix:** Removed the CSS transition entirely.
*   **Root Cause 2 — Noisy sensor data:** `deviceorientation` events fire with significant jitter on Android devices. Without filtering, every erratic spike was applied instantly, causing rapid oscillation. **Fix:** Applied an **exponential low-pass filter** (α = 0.15) using shortest-path angle blending (`_angleDiff`) so noisy spikes are dampened before reaching the arrow.
*   **`requestAnimationFrame` easing loop:** Replaced the CSS transition with a JS-driven `_animateArrow()` rAF loop that:
    *   Computes the **shortest angular delta** (always ≤ 180°) via `_angleDiff()`, preventing long-way-around spins regardless of the raw angle.
    *   Tracks a **cumulative unwrapped `_displayAngle`** so the arrow never resets across the 0°/360° wrap boundary.
    *   Eases at 15% per frame (~200 ms settle at 60 fps) for smooth visuals, and **cancels itself** once within 0.5° of the target to avoid wasting CPU when stationary.

---

### Phase 9: Install Banner UI/UX Fix

*   **Root Cause — Bottom collision:** The PWA install banner was anchored at `bottom: calc(var(--safe-bottom) + 88px)` with `z-index: 3000`, causing it to visually overlap the search bar, search results dropdown, and the route card bottom sheet simultaneously. There was no awareness of `search-active` or `routing-active` states.
*   **Fix — Top-anchored slide-in:** Repositioned the banner to the top of the screen (`top: calc(var(--safe-top) + 8px)`), where it slides down from above using a `transform + opacity` animation. This placement is completely independent of all bottom-anchored UI elements.
*   **State-aware auto-hide:** Added CSS rules for `body.search-active #install-banner` and `body.routing-active #install-banner` that immediately hide the banner (opacity 0, pointer-events none) whenever the search keyboard is active or the route card is open, preventing any visual interference.
*   **Z-index cleanup:** Reduced from `3000` → `1200`, correctly placing the banner above the map and below the search overlay/route card in the stacking context.
*   **Visual polish:** Reduced padding and icon size for a more compact, non-intrusive pill style appropriate to a top notification.

---

### Phase 10: Install Button & Guide Modal Overhaul

*   **Always-Visible Install Button:** The `#install-btn-subtle` download icon in the search bar was previously hidden (`display:none`) and only appeared if the browser fired a `beforeinstallprompt` event. This event is suppressed once the app is installed or after prior dismissal, so the button effectively disappeared permanently. **Fix:** Removed the `display:none` default; the button is always visible and only hides itself when running in standalone (installed) mode.
*   **Permanent Dismiss Bug:** Clicking ✕ on the install banner previously wrote `navv_install_dismissed` to `localStorage`, blocking both the banner and the button from ever appearing again, even across sessions. **Fix:** Removed the permanent-dismiss key entirely. The banner now only hides temporarily and can re-appear on the next session if the browser fires `beforeinstallprompt` again.
*   **Install Guide Modal:** When the native browser install prompt is unavailable (already installed, Firefox, or after Chrome's internal suppression), tapping the install button previously showed only a one-line toast — insufficient for users who didn't know the browser-native flow. **Fix:** Added a full bottom-sheet modal (`#install-modal-overlay`) with animated slide-up, glassmorphism styling, and **platform-aware step-by-step instructions**:
    *   **iOS (Safari):** Tap Share → Scroll & tap "Add to Home Screen" → Tap "Add"
    *   **Android (Chrome):** Open ⋮ menu → "Add to Home screen" → Confirm
    *   **Generic:** Fallback steps for other browsers
*   **`appinstalled` Listener:** Added a `window.addEventListener('appinstalled', ...)` handler that hides the button and banner and shows a confirmation toast once the app is successfully installed, keeping the UI in sync with install state.

---

### Phase 11: Install Prompt Race-Fix, Compass Auto-Start & CSS Variable Bug

#### Fix 1 — `beforeinstallprompt` Race Condition (Android install button showed guide instead of native prompt)
*   **Root Cause:** The `beforeinstallprompt` event fires very early in page load — often before `DOMContentLoaded` and always before `initMap()` is called (which waits for Overpass API data). The listener was inside `initMap()`, so it consistently missed the event, leaving `deferredPrompt = null`. The install button always fell through to the instructions modal instead of triggering Chrome's native install dialog.
*   **Fix:** Moved `beforeinstallprompt` and `appinstalled` listeners to the **very top of the `<script>` block** at global scope. A dedicated `setupInstallPrompt()` wired to `DOMContentLoaded` handles all UI bindings. A `window._installBannerReady` + `window._showInstallBanner()` bridge handles the case where the event fires before or after setup completes.

#### Fix 2 — Compass Arrow Not Shown Until GPS Button Tap (Android)
*   **Root Cause:** `deviceorientation` listeners were only attached inside `gpsBtn.onclick`. On Android, no permission is required — the sensor fires automatically — but since listeners weren't registered until the button was tapped, the direction arrow was invisible on first load.
*   **Fix:** For Android/Desktop (no `requestPermission` API), listeners are now registered **immediately inside `setupGeolocation()`**. iOS path (which requires a user gesture for `requestPermission()`) remains inside `gpsBtn.onclick`.

#### Fix 3 — `var(--accent)` Undefined CSS Variable on Retry Button
*   **Root Cause:** The Overpass-failure error screen's "↺ Retry" button used `var(--accent)`, which was never declared in `:root`. The button rendered with no background colour.
*   **Fix:** Replaced `var(--accent)` with the correctly defined `var(--primary)` (`#4F46E5`).

