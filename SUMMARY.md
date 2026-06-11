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

