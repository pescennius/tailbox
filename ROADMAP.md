# Project Roadmap: Tailbox

This document outlines the step-by-step plan to build **Tailbox**, a static SPA for browsing Taildrive (WebDAV) storage.

## Phase 1: Foundation & Setup
- [ ] **Initialize Project**
    - [ ] Create a new Vite project (Vanilla JS).
    - [ ] Configure `vite.config.js` for a static build.
    - [ ] Initialize Git repository (if not already done).
    - [ ] Install `arrow-js` (Frontend Framework).
    - [ ] Create `Makefile` with entrypoints (`install`, `dev`, `build`, `test`).
- [ ] **UI Shell & Styling**
    - [ ] Install `picocss` (CSS Framework).
    - [ ] Create directory structure (`src/components`, `src/services`, `src/utils`, `src/assets`).
    - [ ] Create `index.html` with PicoCSS theme.
    - [ ] Create a basic layout (Sidebar + Main Content) using Arrow.js templates.
    - [ ] Implement a "Hello World" Arrow.js component to verify setup.

## Phase 2: CheerpX Setup & Connectivity Verification
- [ ] **CheerpX Environment & Engine**
    - [ ] Research CheerpX integration for vanilla JS.
    - [ ] Create a `Dockerfile` for the CheerpX runtime environment (Alpine Linux + `curl` + `xmlstarlet` or similar for parsing WebDAV XML).
    - [ ] Document instructions for building the `ext2` image from the Dockerfile.
    - [ ] *Self-Correction*: If Docker is not available, identify a pre-built CheerpX image (e.g., standard Alpine) that can be used.
    - [ ] Implement `CheerpxService`: Initialize CheerpX engine with the image.
    - [ ] Implement `runCommand` function: Execute shell commands inside CheerpX.
- [ ] **Authentication & Connectivity Check**
    - [ ] Create `LoginComponent`: Input for Tailscale Auth Key (store in `localStorage` or session).
    - [ ] **Implement Connectivity Check**: Use `CheerpxService` to run `curl -I http://100.100.100.100:8080` (or similar) to verify the WebDAV server is reachable via the CheerpX environment. *This replaces the browser-native ping.*
- [ ] **WebDAV Client Implementation**
    - [ ] Implement `WebDavService.list(path)`:
        - [ ] Construct `curl` command to `PROPFIND` the WebDAV URL.
        - [ ] Execute via CheerpX.
        - [ ] Parse XML output into a JSON object (File list).

## Phase 3: Data Layer (SQLite)
- [ ] **SQLite Setup**
    - [ ] Install `sqlite3` or `sql.js` (likely `sql.js-httpvfs` or `wa-sqlite`). *Research: Select best SQLite WASM option.*
    - [ ] Initialize SQLite WASM database.
    - [ ] Create schema: `CREATE TABLE files (path TEXT PRIMARY KEY, name TEXT, size INTEGER, type TEXT, last_modified INTEGER, is_favorite BOOLEAN, tags TEXT)`.
    - [ ] Create FTS table: `CREATE VIRTUAL TABLE files_fts USING fts5(name, path)`.
- [ ] **Indexing Engine**
    - [ ] Implement `IndexerService`:
        - [ ] Call `WebDavService.list("/")` (recursive or iterative).
        - [ ] Insert/Update records in SQLite.
        - [ ] Handle deletions (compare snapshot or mark missing).
    - [ ] Implement "Loading Spinner" UI during indexing.
    - [ ] Optimize for speed (bulk inserts).
- [ ] **Persistence**
    - [ ] Implement mechanism to persist SQLite DB to `localStorage` (or IndexedDB) so it survives reloads.
    - [ ] Load cached DB on startup.

## Phase 4: UI Development
- [ ] **Sidebar Navigation**
    - [ ] Render folder tree from SQLite data.
    - [ ] Implement navigation logic (update current path state).
- [ ] **File Browser (Main Area)**
    - [ ] Create `FileGrid` and `FileList` components.
    - [ ] Render files from SQLite based on current path.
    - [ ] Implement Sorting (Name, Size, Date).
    - [ ] Add "Sort By" controls.
- [ ] **File Details Panel**
    - [ ] Create `FileDetails` component (Slide-over or Modal).
    - [ ] Show metadata (Size, Date, Path).
    - [ ] Fetch live metadata via CheerpX (if detailed props needed beyond listing).

## Phase 5: File Interactions & Previews
- [ ] **Download & Basic Previews**
    - [ ] Implement `FileService.getDownloadUrl(path)`: Return `http://100.100.100.100:8080/...`.
    - [ ] Implement "Download" button (Direct `<a>` link).
    - [ ] Implement "Preview" logic:
        - [ ] **PDF**: Embed using `<iframe>` or PDF.js.
        - [ ] **Video (Native)**: `<video>` tag for MP4/WebM.
        - [ ] **Images**: `<img>` tag.
- [ ] **Advanced Previews (Video Conversion)**
    - [ ] Install `ffmpeg.wasm`.
    - [ ] Integrate `ffmpeg.wasm`.
    - [ ] Add "Convert to MP4" button for non-native formats.
    - [ ] Implement conversion progress UI.
- [ ] **DOCX Preview**
    - [ ] Research/Install `docx-preview` or similar library.
    - [ ] Implement DOCX rendering.

## Phase 6: Metadata & Organization
- [ ] **Favorites & Tags**
    - [ ] Add "Star/Favorite" toggle on files.
    - [ ] Update `is_favorite` in SQLite.
    - [ ] Add "Add Tag" UI.
    - [ ] Update `tags` in SQLite.
- [ ] **Search**
    - [ ] Implement Search Bar.
    - [ ] Query SQLite FTS table.
    - [ ] Display search results.
- [ ] **File Management**
    - [ ] **Delete**:
        - [ ] Implement `WebDavService.delete(path)` via CheerpX (`curl -X DELETE`).
        - [ ] Remove from SQLite.
    - [ ] **Upload**:
        - [ ] Implement "Upload" button.
        - [ ] Use `curl -T` via CheerpX (or direct `fetch` if CORS allows - test this).
        - [ ] Add to SQLite.

## Phase 7: Polish & Optimization
- [ ] **Settings & Tools**
    - [ ] "Download Index": Export SQLite DB file.
    - [ ] "Reindex": Trigger full re-scan.
    - [ ] "Change WebDAV URL": Configurable setting.
- [ ] **Responsiveness & Accessibility**
    - [ ] Verify Mobile layout (Hamburger menu for sidebar).
    - [ ] Verify Keyboard navigation.
    - [ ] Ensure ARIA labels.
- [ ] **Performance**
    - [ ] Audit bundle size.
    - [ ] Optimize SQLite queries.
- [ ] **Final Verification**
    - [ ] Test full flow: Login -> Index -> Browse -> Preview -> Download.
    - [ ] Verify "Offline" mode (browsing cached index).

## Research & Questions
- [ ] **CheerpX Image**: Can we use a public Alpine image, or *must* we build one? (Plan: Try public first).
- [ ] **CORS**: Verify if `100.100.100.100` allows direct GETs from browser. (Plan: Test early).
- [ ] **SQLite Persistence**: `sql.js` vs `wa-sqlite` + IndexedDB. (Plan: Use `wa-sqlite` with IDB backend for persistence).
