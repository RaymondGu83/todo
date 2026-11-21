# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A framework-free, single-file to-do tracker (vanilla HTML/CSS/JS). There is no build step, package manager, linter or test suite. Not a git repository.

## Running

- Open `index.html` directly in a browser for basic use (`file://` works for everything except PWA features).
- The service worker and install prompt only work over `https://` or `http://localhost`. `index.html` registers `sw.js` only when `location.protocol` is http(s).
- No Python or Node is installed on the dev machine, so there is no local web server. Don't try `python -m http.server` or `preview_start` with a launch config. To preview, open the file in the browser pane or test via a self-contained copy.

## Architecture

- `index.html` holds everything: markup, CSS and JS (about 29KB). Keep it a single file with no dependencies.
- `manifest.webmanifest`, `sw.js` (network-first, cache `todo-v1`) and `icon-192.png`/`icon-512.png` make it an installable PWA. If you add a file the app needs offline, add it to `FILES` in `sw.js` and bump the cache name.
- `mockup.html` shows two iPhone 14 Pro Max frames (light and dark) containing iframes of `index.html?demo&scroll` and `index.html?demo&theme=dark&scroll`. It must sit next to `index.html`.

### Data and state

- Task model: `{id, item, priority: "High"|"Medium"|"Low", deadline: "YYYY-MM-DD", requestedBy, notes, done}`.
- Persistence goes through the `store` wrapper (localStorage with an in-memory fallback, since localStorage can throw, e.g. on data: URLs or in private mode). Keys: `tasks`, `theme`, `sortBy`, `groupBy`. Never call `localStorage` directly.
- `render()` rebuilds the list from `tasks` (filter, then sort, then group, with done items always below open ones). Mutations update `tasks`, save, then call `render()`, often wrapped in `withTransition` (View Transitions API).
- Delete and "clear completed" use a snapshot-based undo toast (`showToast(msg, prev)`).
- Editing uses the `<dialog id="editDialog">` sheet.

### Layout and theming

- One `<table>` is restyled into grid cards at `max-width: 640px` using `td:nth-child(n)` selectors. If you reorder or add columns, update those selectors. The overdue/today colors for the deadline cell (4th column) have separate mobile and dark overrides.
- Dark mode is a `body.dark` class. It follows `prefers-color-scheme` until the user toggles (the choice is then saved under `theme`).
- Mobile requirements already in place: `viewport-fit=cover`, safe-area insets, 16px inputs (prevents iOS zoom), 44px tap targets. Animations respect `prefers-reduced-motion`.
- `[hidden]{display:none !important}` is needed because of the table's `display:block` on mobile.

### Demo mode

`?demo` seeds 6 sample tasks (relative dates) if storage is empty; `&theme=dark` forces dark; `&scroll` scrolls to the list. This block sits just before the final `render()` and exists only for the mockup.
