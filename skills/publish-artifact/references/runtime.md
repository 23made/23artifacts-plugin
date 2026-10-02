---
title: Runtime environment
summary: What works inside the artifact sandbox and what doesn't — no cookies, no storage, no wss:. Read before writing JS that persists or connects.
router: What the sandbox allows: no cookies, storage or wss:.
---

# Runtime environment

Artifacts serve on an isolated origin under a sandbox CSP (opaque origin).

**Works:** external **https:** scripts, stylesheets, fonts, images, and fetch/XHR — CDN libraries and Google Fonts are fine. Web Workers work when constructed from `blob:` URLs (file-based `new Worker("w.js")` won't load in the sandbox).

**Does not work:** cookies, `localStorage`/`IndexedDB`, credentialed requests, and WebSockets (`wss:`).

Single-file HTML with everything inline is still the most robust shape.

For state that must survive a reload, or be shared between viewers, use the room — see `rooms.md`. That is the supported way to persist anything, and it needs no backend code.
