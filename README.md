# 🗺️ Navv

**Navv** is a high-performance, mobile-optimized, offline-first navigation Progressive Web App (PWA) designed for campus environments. It delivers a modern, seamless navigation experience entirely within the browser, requiring no external backends or native app downloads.

---

## 🚀 Key Features

*   📶 **Offline-First Excellence:** Once the initial cache is complete, all rendering, searching, and routing logic works 100% offline.
*   🔄 **Dynamic Alternative Detours:** Encounter a blocked path? Use the "Alternative" button to instantly block the road ahead and recalculate a detour from your live location.
*   🧭 **Smart Turn-by-Turn Guidance:** Includes movement-based bearing calculation, turn smoothing (ignoring minor road curves), active "Wrong Way" detection, and dynamic route trimming that shrinks the path as you walk.
*   🔼 **Stable Direction Arrow:** Compass arrow uses angle-unwrapped `requestAnimationFrame` easing with an exponential low-pass filter to eliminate 360° oscillation caused by noisy sensor data or wrap-around jumps.
*   🎯 **Robust Multi-Node Snapping:** Intelligent pathfinding that tries multiple entry points to the routing graph, ensuring reliable navigation even when starting from off-path locations.
*   ✨ **Modern Indigo Design System:** A clean, mobile-first interface featuring standardized glassmorphism, responsive panels, and a polished indigo brand palette.
*   🔍 **Intelligent Search:** Fuzzy alphanumeric search that handles abbreviations and aliases, making POI discovery instant and reliable.
*   🛡️ **Strict Geofencing:** Keeps the experience focused and reliable with locked map viewports and GPS boundary validation.
*   📈 **Dynamic Data Sync:** Uses the Overpass API to pull the latest buildings and walkable paths directly from OpenStreetMap, with background updates every 3 months.
*   ☁️ **Instant Over-the-Air Updates:** Employs a strict Network-First caching strategy for the core app shell, ensuring that you always get the latest features pushed to GitHub Pages the moment you are online, with instant offline fallback.
*   🔁 **Resilient Error Recovery:** When the map data server is temporarily unavailable (rate-limited), the app shows a clear explanation and a one-tap **Retry** button instead of a dead screen.
*   🏛️ **Campus-Only Search:** Buildings are filtered through the exact OSM campus polygon at data-processing time. Nothing outside the boundary ever appears in search results.
*   🏷️ **Clean POI Labels:** All raw OSM tags (`yes`, `student_accommodation`, etc.) are mapped to human-readable labels like `Building`, `Student Accommodation`, `Café`, etc.
*   📲 **Non-Intrusive Install Prompt:** The PWA install banner slides in from the top of the screen and automatically hides whenever the search keyboard or route card is active, ensuring it never interferes with navigation.

---

## 🏗️ Architecture

Navv is built on a strict **Two-File Philosophy** to ensure maximum performance and maintainability:

1.  **`index.html`**: The entire application shell. It contains the UI (Modern Indigo CSS), the client-side data engine, the routing logic (Custom Dijkstra), and map initialization.
2.  **`sw.js`**: A robust Service Worker responsible for aggressive asset caching, automated rate-limited map tile pre-caching, and immediate activation via `skipWaiting()` for instant OTA updates.

---

## 🛠️ Data & Performance

*   **Caching:** Uses **IndexedDB** for high-speed local storage of processed routing graphs and POI catalogs.
*   **Tile Management:** Automatically manages and synchronizes visual map tiles with the underlying routing data.
*   **Performance:** Optimized for low-power mobile devices with efficient graph traversal and minimal DOM overhead.

---

## 📱 Getting Started (Mobile)

For the best experience on your phone:

1.  **Open:** Load the Navv URL in your mobile browser.
2.  **Install:** Select **"Add to Home Screen"** to install it as a standalone PWA.
3.  **Authorize:** Allow location access when prompted to enable live GPS tracking.
4.  **Sync:** Wait for the **"Caching Navv..."** progress bar to finish. This ensures full map functionality even in dead zones.

---

*Built with Leaflet.js, OpenStreetMap, and Overpass API.*
