# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the App

No build step. Open `index.html` directly in a browser, or serve it locally:

```bash
python -m http.server 8000
# then visit http://localhost:8000
```

Tailwind CSS is loaded via CDN — no npm, no bundler.

## Architecture

Two-layer separation:

- **`js/storage.js`** — data layer. A plain object literal (`BucketStorage`) that owns all LocalStorage access. Every method reads, mutates, and re-saves the full array. Key methods: `addItem`, `updateItem`, `deleteItem`, `toggleComplete`, `getStats`, `getFilteredList`.

- **`js/app.js`** — UI layer. An ES6 class (`BucketListApp`) instantiated as the global `app` on `DOMContentLoaded`. It calls `BucketStorage` for all data operations and calls `this.render()` after every mutation to fully re-render the list from scratch.

`storage.js` must be loaded before `app.js` (see script order in `index.html`).

## Key Patterns

**Rendering**: stateless full re-render on every change — no diffing. `render()` calls `BucketStorage.getFilteredList(this.currentFilter)`, then replaces `bucketListContainer.innerHTML` entirely.

**Inline event handlers**: dynamically generated list items use `onclick="app.handleToggle('${item.id}')"` — they rely on `app` being a global. New item actions must follow the same pattern.

**XSS prevention**: user-supplied text must go through `escapeHtml()` before being interpolated into HTML strings. This method creates a temporary `div`, sets `textContent`, and reads back `innerHTML`.

**Data shape**:
```js
{ id: String (Date.now()), title: String, completed: Boolean, createdAt: ISO string, completedAt: ISO string | null }
```

`id` is a timestamp string; newest items are prepended (`unshift`).
