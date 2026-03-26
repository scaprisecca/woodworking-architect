# Performance Optimization Plan for Logseq

## Context

This is a Logseq-based knowledge management app (ClojureScript + React 18 via Rum + Shadow CLJS). The app loads 20 script tags in `index.html`, ships a single monolithic main.js bundle (only code-editor is split), eagerly initializes heavy libraries like emoji-mart (1.4MB), and has no service worker. The goal is to reduce initial startup time and input-to-action latency on both mobile web and desktop without sacrificing features.

---

## Priority 1: High Impact

### 1.1 Defer Non-Critical Scripts in index.html

**Problem**: `resources/index.html` loads 12+ library scripts (`highlight.min.js`, `interact.min.js`, `marked.min.js`, PDF.js (2 files), PhotoSwipe (2 files), `html2canvas.min.js`, `eventemitter3`, `prop-types.min.js`) that are not needed for initial render. Even with `defer`, browsers must download, parse, and execute all before the app is interactive.

**Solution**: Remove non-critical script tags and load them on-demand when first needed.

**Scripts to defer until use**:
- `highlight.min.js` → load when a code block is first rendered
- `interact.min.js` → load when drag interaction starts
- `marked.min.js` → load when markdown preview is needed
- `pdf.mjs` + `pdf_viewer3.mjs` → load when a PDF is opened
- `photoswipe.umd.min.js` + `photoswipe-lightbox.umd.min.js` → load on image lightbox trigger
- `html2canvas.min.js` → load on screenshot/export
- `prop-types.min.js` → **remove entirely** (dev-only, should not be in production)
- `eventemitter3.umd.min.js` → load alongside PhotoSwipe/plugins that need it

**Scripts to keep** (critical path): `magic_portal.js`, `react.production.min.js`, `react-dom.production.min.js`, `ui.js`, `main.js`, `lsplugin.core.js`

**Implementation**: Create a dynamic script loader utility and add check-and-load guards in extension files that consume these globals (e.g., `frontend.extensions.highlight`, `frontend.extensions.pdf.*`, `frontend.extensions.lightbox`).

**Files to modify**:
- `resources/index.html` — remove 8-10 script tags
- Create `src/main/frontend/util/script_loader.cljs` — dynamic script injection utility
- `src/main/frontend/extensions/highlight.cljs` — lazy load highlight.js
- `src/main/frontend/extensions/pdf/` — lazy load PDF.js
- Other extension entry points that rely on removed globals

**Expected impact**: ~300-500ms faster on mobile, ~100-200ms on desktop (fewer scripts to download/parse before interactive).

---

### 1.2 Add Shadow CLJS Module Splitting for Heavy Features

**Problem**: `shadow-cljs.edn` only splits `:code-editor` from `:main`. Everything else — graph visualization (pixi.js 6.2, d3-force, graphology), PDF viewer, settings, import/export, all 462 components — ships in one `main.js` bundle.

**Solution**: Add Shadow CLJS modules for route-gated and rarely-used features using `shadow.lazy` (already proven in `lazy_editor.cljs`).

**New modules to add**:
```
:graph-view    — frontend.extensions.graph, frontend.extensions.graph.pixi (pixi.js + d3-force + graphology)
:pdf-viewer    — frontend.extensions.pdf.* (pdfjs-dist)
:settings      — frontend.components.settings, frontend.components.plugins*
:import-export — frontend.components.imports, frontend.components.export (html2canvas)
```

**Files to modify**:
- `shadow-cljs.edn` — add module entries
- `src/main/frontend/routes.cljs` — wrap route handlers with `shadow.lazy/loadable`
- Component entry points for each new module — add lazy loading wrappers

**Expected impact**: 15-25% reduction in initial `main.js` parse time.

---

### 1.3 Lazy-Load emoji-mart Data

**Problem**: `ui.cljs:52` eagerly initializes emoji-mart with `@emoji-mart/data` (~1.4MB JSON) at module load time. This runs during app startup even though the emoji picker is only used when a user actively inserts an emoji.

**Solution**: Defer initialization to first emoji picker open.

**File to modify**: `src/main/frontend/ui.cljs` lines 3, 5, 52

**Expected impact**: ~50-100ms saved in initial JS evaluation; 1.4MB less data parsed at startup.

---

### 1.4 Optimize Tailwind CSS Build

**Problem**: `tailwind.config.js` safelist generates classes for every color/shade combination via a massive regex across 24 Radix color scales. `tailwind.all.css` uses glob imports pulling in 30+ component CSS files unconditionally (including PDF viewer, CodeMirror, KaTeX styles).

**Solution**:
1. Audit and trim the safelist — determine which dynamic color/shade combos are actually used at runtime
2. Move vendor CSS (PDF viewer, CodeMirror, KaTeX, PhotoSwipe) out of the main bundle and load alongside their corresponding JS modules
3. Audit which Radix color scales are actually used — likely only 8-10 of the 24

**Files to modify**:
- `tailwind.config.js` — reduce safelist and unused color scales
- `tailwind.all.css` — remove vendor CSS imports for lazily-loaded features

**Expected impact**: 20-40% reduction in CSS file size; faster CSS parse on mobile.

---

### 1.5 Add `rum/static` to High-Frequency Leaf Components

**Problem**: `block.cljs` has 53 component definitions (178KB file), many without `rum/static`. Without it, parent re-renders cascade to all children even when props haven't changed. Similarly, `property/value.cljs` has 25 components with only 3 using `rum/static`.

**Solution**: Audit and add `rum/static` to leaf/presentational components in the block rendering tree.

**Priority files**:
- `src/main/frontend/components/block.cljs` — 53 components, audit each
- `src/main/frontend/components/property/value.cljs` — 25 components
- `src/main/frontend/components/property.cljs`
- `src/main/frontend/components/icon.cljs` — 12 definitions, only 4 have `rum/static`

**Expected impact**: 15-30% reduction in unnecessary re-renders during editing and scrolling.

---

## Priority 2: Medium Impact

### 2.1 Font Preloading

**Problem**: No `<link rel="preload">` for fonts in `index.html`. Inter font and Tabler icons webfont trigger layout shifts (FOIT).

**Solution**: Add preload hints and ensure `font-display: swap`.

**File**: `resources/index.html`

**Expected impact**: Eliminates FOIT, ~100-200ms perceived improvement.

---

### 2.2 Virtualize Block Lists More Aggressively

**Problem**: `block-list` in `block.cljs` only virtualizes under specific conditions (reference blocks, not sidebar). Journal pages with hundreds of blocks render all DOM nodes.

**Solution**: Extend virtualization to journal page block lists (>50 blocks), right sidebar reference panels, and custom query results.

**Files to modify**:
- `src/main/frontend/components/block.cljs` — expand virtualization conditions
- `src/main/frontend/components/journal.cljs`
- `src/main/frontend/components/right_sidebar.cljs`

**Expected impact**: Significant DOM reduction for large pages, reducing layout/paint time.

---

### 2.3 Reduce Global State Subscription Overhead

**Problem**: `state.cljs` uses a single giant atom. `rum/reactive` components that call `state/sub` subscribe broadly. `db/react.cljs` `refresh-affected-queries!` iterates ALL cached queries on every DB transaction.

**Solution**:
1. Migrate high-frequency subscriptions to flow-based pattern (already used in `sub-editing?`)
2. Add entity-to-query-key index in `db/react.cljs` so only truly affected queries re-execute

**Files to modify**:
- `src/main/frontend/db/react.cljs`
- `src/main/frontend/state.cljs`

**Expected impact**: Reduces wasted CPU during transactions, especially for large graphs.

---

### 2.4 Optimize Worker Communication

**Problem**: DB worker queries via Comlink require serializing ClojureScript data each way. `db/async.cljs` fires many individual queries per page load.

**Solution**: Batch related queries into single worker round-trips for initial page loads (e.g., batch `<get-block` calls).

**Files to modify**:
- `src/main/frontend/db/async.cljs` — add batch query API
- `src/main/frontend/persist_db/browser.cljs`

**Expected impact**: 10-20% reduction in data layer latency during page navigation.

---

### 2.5 Debounce State Watchers During Editing

**Problem**: `persist_db/browser.cljs` `sync-ui-state!` watches the entire `state/state` atom and fires on every change, serializing via Transit. During editing, state changes many times per keystroke.

**Solution**: Add 100ms debounce to the watcher. Undo/redo doesn't need keystroke-level granularity.

**File**: `src/main/frontend/persist_db/browser.cljs`

**Expected impact**: Reduces Transit serialization overhead during editing, improving keystroke latency.

---

## Priority 3: Lower Impact / Longer Term

### 3.1 Add a Service Worker for Static Asset Caching

**Problem**: No service worker. Every page load re-downloads all static assets. SQLite WASM is particularly large and stable.

**Solution**: Add cache-first service worker for immutable assets (JS, CSS, fonts, WASM).

**Files**: Create `resources/js/sw.js`, modify `resources/index.html` to register it.

**Expected impact**: Near-instant subsequent loads on web. Critical for mobile web.

---

### 3.2 Split block.cljs Monolith

**Problem**: `block.cljs` is 4143 lines with 53 components and 90 requires. Every change recompiles the entire file. It hinders code splitting potential.

**Solution**: Extract into sub-namespaces: `block.inline`, `block.properties`, `block.references`, `block.dnd`. Keep `block` as orchestrator.

**Expected impact**: Better code splitting potential, faster dev recompilation, clearer `rum/static` audit scope.

---

## Verification Plan

1. **Startup time**: Use Chrome DevTools Performance tab → record page load → compare "Time to Interactive" and "Total Blocking Time" before/after
2. **Bundle size**: Run production build and compare `main.js` size with `ls -la static/js/`
3. **CSS size**: Compare `static/css/style.css` before/after safelist trimming
4. **Rendering**: Use React DevTools Profiler to verify fewer re-renders after `rum/static` additions
5. **Input latency**: Type in editor, measure keystroke-to-render via Performance tab
6. **Regression**: Run existing test suite (`yarn test`, Playwright E2E) to verify no feature breakage

---

## Implementation Order

1. **Phase A** (config-only, low risk): 1.1 (defer scripts) + 1.3 (lazy emoji) + 1.4 (CSS trim) + 2.1 (font preload) + remove prop-types
2. **Phase B** (component changes): 1.5 (rum/static audit) + 2.5 (debounce watchers)
3. **Phase C** (build system): 1.2 (Shadow CLJS module splitting)
4. **Phase D** (architecture): 2.2 (virtualization) + 2.3 (state subscriptions) + 2.4 (worker batching)
5. **Phase E** (long-term): 3.1 (service worker) + 3.2 (block.cljs split)
