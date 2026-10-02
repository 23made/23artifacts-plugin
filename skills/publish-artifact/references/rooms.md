---
title: Rooms — shared state, live updates and presence
summary: Shared state between viewers with no backend — votes, presence, live cursors, collaborative editing. Read when more than one person interacts with the same page.
router: Shared state between viewers, with no backend.
---

# Shared state: the room (opt-in)

Every artifact can have ONE shared JSON blob — its "room" — that the artifact's own JavaScript reads and writes on its own origin. Viewers collaborate through it (votes, presence, shared form/game state) with no backend code. **Off by default**; turn it on with `update_artifact_settings {artifact, room: "on"}`.

- `GET /__room` → `{ schema, version, updatedAt, data, viewer }`. On gated artifacts `viewer.email` is the platform-verified viewer identity — use it instead of writing any auth code; it's `null` on public artifacts.
- `PUT /__room` body `{ schema?, ifVersion, data }` → `200 { version, updatedAt }`, or `409` with the current `{ schema, version, updatedAt, data }` — merge and retry. The first write uses `ifVersion: 0`.
- The room belongs to the ARTIFACT, not a version: a new version inherits the data. Always set `schema` (e.g. `"myapp/v1"`) and have new versions check it before parsing old shapes.
- Inside a share embed (`/__share/<token>/…`), reach the room at the **relative** path `__room` so the request stays under the token prefix.
- Limits: 256 KB per room, ~30 writes/min per viewer. Anyone who can view the artifact can write its room — for sensitive state, encrypt client-side with a key kept in the URL fragment (the server then holds only ciphertext).
- Owner tools: `get_room` (inspect/export what viewers wrote); `update_artifact_settings` with `room: "wipe"` (empty it) or `room: "off"` (the kill switch, keeping what it holds).

## Live updates & presence (no polling needed)

When the room is enabled, two more endpoints make it realtime:

- `GET /__room/events` — Server-Sent Events (`new EventSource('__room/events')`, relative path). First event `hello`: `{schema, version, updatedAt, data, presence, viewer}` — the durable state plus who's here, so no initial GET is needed. Then: `version` events — a CAS write landed. When the blob is small (and the audience modest) the event inlines `schema` + `data`: if `data` is present, apply it directly and skip the follow-up GET; otherwise GET `__room` (feature-detect on `data` — inlining is best-effort: large blobs, crowded rooms, and older servers all send only `{version, updatedAt}`). A wipe (`update_artifact_settings` with `room: "wipe"`) arrives as `{version: 0, data: null}` — reset local state when you see it. Also `presence` diff events (`{u: {id: data, …}, d: [ids]}`) and comment heartbeats. The server ends each stream after a few minutes — a clean close is normal, and `EventSource` reconnects automatically (re-checking access).
- `POST /__room/presence` body `{id, data}` — EPHEMERAL, never stored: cursors, selections, "who's here". Send `data: null` to leave. Entries expire ~12 s after the last post (re-post every ~5 s while visible). The allowance (~10/s) is generous for one person but metered per IP, so viewers behind the same NAT — an office, a classroom — share it; stay well below the ceiling. The first viewer to use an `id` owns it for its lifetime; one viewer can hold at most 8 ids at once, so use one id per viewer, not per widget instance.

## Cursor recipe (section-anchored, whole-page)

Don't send viewport or document pixels — layouts reflow and viewers on a 390 px phone and a 1440 px desktop will never agree on them. Instead mark each major page region with a stable `data-cursor-anchor` id (hero, main panel, sidebar, footer…), track `pointermove` over the whole page, and send `{s: anchorId, x, y}` where x/y are per-mille *within that region's box* (`e.target.closest('[data-cursor-anchor]')` + `getBoundingClientRect`). Render a remote cursor by resolving its anchor element and absolutely positioning the chip at the per-mille offset in document flow (cache anchor boxes; invalidate on resize/reflow, not scroll) — cursors land in the *right region* on every viewport. For surfaces where positions must be exact across screens (a canvas, a grid, a map), give that element its own innermost anchor — per-mille within its box *is* its coordinate space. Sample locally at ~30 Hz, POST at ~10 Hz (only when the anchor or position changed), and glide remote chips to each new position over ~100 ms — smooth and effectively live.

## Wet-ops pattern

Piggyback your *unconfirmed* operations on the presence posts you already send (e.g. `data.pp = [[x, y, color], …]`) and render other viewers' pending ops as an ephemeral overlay keyed by presence id — instant visible effect over presence, durable settlement over CAS. Promote overlay entries when a version event confirms them into the durable state; drop them when their presence entry leaves. Cap piggybacked ops well under the 1 KB presence limit (a few dozen entries).

ALWAYS keep a polling fallback: if `EventSource` errors or the path 404s (older deployment, room disabled), fall back to polling `__room` every few seconds. Treat presence/room data as untrusted input — render with `textContent`, never `innerHTML`.

## A robust write helper

```js
async function writeRoom(mutate, tries = 4) {
  let cur = await (await fetch('/__room')).json();
  for (let i = 0; i < tries; i++) {
    const res = await fetch('/__room', {
      method: 'PUT',
      headers: { 'content-type': 'application/json' },
      body: JSON.stringify({ schema: 'myapp/v1', ifVersion: cur.version, data: mutate(cur.data) }),
    });
    if (res.status !== 409) return res.json();
    cur = await res.json(); // lost the race — 409 carries current state to merge
  }
  throw new Error('room contention');
}
```
