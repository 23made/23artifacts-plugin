---
title: Large sites
summary: Files too big to inline — stage them over this connection (upload_file, start → part → complete, no credential), hand the person a place to drop them, or save from a shell through a short-lived upload address. Read when a save won't fit in a tool call.
router: Files over 25 MB: staging in parts, or a shell upload.
---

# Large sites (too big to inline)

Inline, a save carries at most 25 MB a file, 50 MB in all and 2,000 files.
Beyond that, upload the big files first and name each in the save as
`{ "path": "video.mp4", "upload": "<uploadId>" }` — in `files`, or as a change
in `changes` — and a save with uploaded files may total 5 GB (4 GB a file).
Material has its own ceilings (see `library`). Three ways to upload; pick by
where the bytes are.

## Bytes in your context, or no shell — upload over this connection

The connection you already hold is the only authorization: nothing is minted
and no secret exists at any point.

1. `upload_file` `{ op: "start", filename, size, sha256 }` → `{ upload, maxChunkBytes }`.
   Declare `sha256` — the server then refuses corrupted bytes at `complete`
   instead of hosting them.
2. `upload_file` `{ op: "part", upload, n, data }` — base64 parts numbered
   from 1, in any order; send a part again to overwrite it (that is how to
   resume after a lost call). At most 8 MB decoded each (2–6 MB is the sweet
   spot), at most 1,000 parts.
3. `upload_file` `{ op: "complete", upload }` — verifies every part, the
   declared size and the declared digest, and answers the upload's id and the
   server-computed `sha256`.
4. Save, naming the upload as `{ "path": "…", "upload": "<upload>" }`. An
   upload is used by one save.

An upload not completed expires after 24 hours, and must happen in the
workspace the save lands in (`workspace`, a handle, as on every other tool).

## Bytes on the person's own device — offer a place to drop them

`upload_file` `{ op: "offer", for: "save" | "library" }` uploads nothing
itself. Where the host shows interfaces, it opens a hand-over: the person picks
or drops files, the surface uploads each with the three steps above over this
same connection, and then tells you each file's name and upload id — save them
as `{ "path": "…", "upload": "<upload>" }`, or, for the library, as one piece
of material each with `save_artifact { kind, name, files: [{ path, upload }] }`.
The bytes never pass through the conversation. The answer carries the limits:
4 GB a file for a save, each material kind's own ceiling for the library.
Where the host shows none, nothing opens: a file on the person's machine then
goes through an upload address (below).

## Files on disk and a shell at hand — an upload address

Chunking a big tree through tool calls is slow; a shell POSTs it in one go:

1. `create_credential { kind: "upload", artifact: "<its address, identifier or key>" }` → an **upload address**, a capability URL scoped to that one artifact — a key nobody in the workspace holds yet makes a new artifact with that key on its first save — expiring in minutes (`minutes`, default 30) and good for one save unless you ask for more (`saves`, at most 50). The URL is the whole credential: nothing to store, and losing it risks at most its saves to one artifact for a few minutes.
2. Build the same JSON `save_artifact` takes, less `artifact` (the address carries it), reading the files from disk, and POST it to the answer's `url`:
   `curl -X POST "$UPLOAD_URL" -H "Content-Type: application/json" -d @save.json`
   — `/api/v1/artifacts/<artifact>/versions?session=…` for an artifact that exists, `/api/v1/artifacts?session=…` for a new one, whose body carries `name` and the address's `key` (the key may be left out; it is the address's). `changes` on a `base` work from a shell too, so a few changed files of a big site need not travel again.
3. The same ceilings and the same answer as the tool, left-out paths included.
4. Bigger single files (video and the like, up to 4 GB each): stage them with the resumable upload routes, the address's `?session=…` on every call — `POST /api/v1/uploads` `{ "filename", "size" }` → `{ uploadId, partSize }` (16 MB parts), `PUT /api/v1/uploads/<uploadId>/parts/<n>` per part (put a part again to resume), `POST /api/v1/uploads/<uploadId>/complete` — then name them in the save as `{ "path": "video.mp4", "upload": "<uploadId>" }`. The answer's `uploadsUrl` is that staging address.

A save that is refused spends nothing, so fix it and send it again. `revoke_credential` ends an address early. Many pieces of material at once go to the library's own upload address (`{ kind: "upload", library: true, batches }`) and `POST /api/v1/library` — see `library`.

## If the upload address is denied

Some agent harnesses refuse credential-minting tools — even bounded ones. Do not stall — degrade in order:

1. **Shell not essential?** `upload_file` (above) mints nothing — there is no credential call for a harness to refuse. Any size, straight over this connection; slower per byte, but it always works.
2. **Fits inline?** At most 50 MB in all and 25 MB a file saves fine through `save_artifact` (base64 for binaries). Trim what the served site does not need — source maps, raw exports, unoptimized originals — before giving up on inline.

A save from a shell needs nothing of the person's: the upload address is its whole credential, and uploading over this connection needs none.

## Standing credentials (CI, cron)

For automation that saves on its own schedule, `create_credential { kind: "credential" }` makes a long-lived credential. The secret never appears in the conversation: the answer carries a sign-in-gated link that reveals it once to its owner — have the person open it and store the secret where the automation runs. It then saves with `POST /api/v1/artifacts/<artifact>/versions` and a bearer header. By default a credential is publish-only — saving, uploading, adding material, and listing what exists by name and state; pass `scopes` when making it to add `read` (reads, including saved file content and drafts) and `manage` (settings, state, deletion, access, share links, comments — the whole management surface, mirrored at `/api/v1`; reference at `https://23artifacts.com/docs/api`). A publish-only credential — standing or an upload address — makes new artifacts with any `access` and `preview`, but on an existing one those two must match what is stored (or be left out, which keeps them); changing them is `manage`, and the save is refused with a 403 that says so.

`get_workspace` lists them and `revoke_credential` ends one — upload addresses and credentials alike.
