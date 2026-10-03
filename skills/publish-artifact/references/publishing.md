---
title: Publishing
summary: save_artifact — whole content or changes on a base, live, staged or into the draft; keys, workspaces, versions, copies, notes and the files a save leaves out. Read before your first save.
router: save_artifact: files or changes, live, staged or draft.
---

# Publishing: one save

Every change to what an artifact holds is one call, `save_artifact`. It makes a new version — live (the default) or staged — or writes the artifact's draft. A version never changes once saved and nothing deletes one; the address serves exactly one version, or none.

## A new artifact

1. Gather every file into `files`. Each entry is `{ path, content, encoding }`:
   - `path` — the file's path within the site, e.g. `index.html`, `styles.css`, `img/logo.svg`. No leading slash, at most 1,024 characters.
   - Text files (HTML, CSS, JS, SVG, JSON, markdown): `encoding: "utf8"` (the default).
   - Binary files (PNG, JPEG, fonts, …): base64 of the bytes, `encoding: "base64"`.
   - The server is remote and cannot read your sandbox or the person's disk: every file arrives inline, staged (`{ path, upload }`, see `large-sites`), copied from material (`{ path, from }`, see `library`), or as a zip (`{ archive: "<base64>" }`, expanded on arrival).
   - A page needs an entry page: `index.html`, or `entrypoint` naming another of its paths.
2. Call `save_artifact { name, files }`. Also, only when a save makes an artifact:
   - `key` — a name for tools to find it by (`docs-site`): one to sixty-three lowercase letters, digits and hyphens, unique in its workspace, and part of no address, link preview, title or share link. Left out, the artifact has none — nothing is derived from `name`. Give one whenever you will save it again (see *Saving by key*); set, rename or clear it later with `update_artifact_settings { artifact, key }` (`null` clears).
   - `workspace` — a handle, to make it in a workspace the person belongs to; their own when left out.
   - `kind` — `page`, `deck`, `markdown`, `file`, or a material kind (`image`, `video`, `sound`, `font`, `theme`). Left out, it is recognised from the content: a `document` is a deck, files with an entry page a page, one markdown file markdown, any other single file a file. Material is made only by naming its kind (see `library`). The kind is fixed at birth: a later save whose content does not fit it is refused naming the kind.
   - `description` (one line, at most 280 characters), `tags`, `access` and `preview` (see below).
3. Give the person the `address` from the answer: `<id>.23a.so`, fixed from the moment the artifact exists — its only address. The answer also carries `id` (the identifier), `key` when it has one, `kind`, `state`, `version` (`number`, its own pinned `address`, `live`, `staged`), `files` (per path its `bytes` and `sha256` — compare them with your own files instead of fetching them back), `whoCanOpen` (every way in, as one sentence) and `dashboardUrl`. `next` says what comes next: the preview image being made, a staged version one `set_artifact_state` away, the workspace's default theme when the save did not use it.

## Saving by key

`save_artifact { key, files }` names the artifact by its key in the workspace the save acts in (`workspace`, else a credential's own, else the person's own). When no artifact there holds that key, the save makes one carrying it; when one does, the save is that artifact's next version. So an agent, a CI job or a command line that always passes the same key makes its artifact once and versions it on every run after — no need to look it up first, and no save by key is ever refused as already taken. On a save that versions, `name`, `description` and `tags` are optional; any sent are left as stored and the answer's `next` says so (change them with `update_artifact_settings`), and a `kind` the artifact is not is refused. A key held by a deleted artifact is refused until it is restored (`set_artifact_state`), and is freed when the artifact is purged. A key names nothing outside its workspace.

## A new version of an artifact

Pass `artifact` — its address, its identifier or its key, whichever you hold — and the content, in one of three forms:

- **The whole of it:** `files` (or `document`, for a deck or a theme). The version holds exactly what you send. The answer's `leftOut` lists every path the version it replaces had and this one does not (the first 50, then `more`), so nothing is ever dropped without saying so; leaving out the entry page is refused unless you name a new `entrypoint`.
- **Changes on a base:** `base` — a version number, `"live"` or `"draft"` — with `changes`, applied on the server; every file you do not name is carried over by digest, moving no bytes. A change is `{ path, content, encoding }` (written whole), `{ path, upload }`, `{ path, from }`, `{ path, remove: true }`, or `{ path, replace: { old, new, all? } }` — a text edit inside the file, where `old` must occur exactly once unless `all: true`. At most 2,000 changes. This is the way to change a few files of a big site: it cannot drop the rest.
- **A base alone:** `base` and nothing else saves that version's content again as the newest — the way to bring an earlier version back as a new one, or to make the draft a version.

A base older than the newest version is accepted, and the answer's `base.newer` names the versions the save does not include. A save identical to the live version makes no version and says so (`noop: true`); identical to the draft, it writes nothing.

## Live, staged or the draft

`as` decides how a save lands:

- `"live"` (the default) — the new version is what the address serves, at once.
- `"staged"` — the version is saved at its own pinned address and the address keeps serving what it served; `set_artifact_state { artifact, state: "live", version }` makes it live later. A first save staged leaves the artifact **offline**: nothing is live, and its address says so.
- `"draft"` — the artifact's draft: one per artifact, served at no address and read only by people who can edit it. Fill it whole, or with `changes` on `base: "draft"` (or on a version, which puts the draft back to that version first). Every save into the draft names the draft's `revision` it read — the answer gives the new one — and a stale revision is refused as a conflict carrying the current one, so two writers never overwrite each other unseen. `save_artifact { artifact, base: "draft" }` then makes the draft a version, live or staged. `get_artifact_files { artifact, version: "draft" }` reads it.

Save live unless the person asked for a step before it goes live. Versions are immutable and going back is one `set_artifact_state` call, so saving live is already safe; a staged version sits unseen until someone makes it live. Someone who can edit but may not make changes live has a live save staged, and the answer says so and who may make it live.

## Copies and remixes

`base: "<other artifact>@<version>"` (or `@live`), with no `artifact`, makes a **new** artifact from that version: pass `changes` (or a deck's `patch`) to remix it — which needs only that you can open it — or nothing, to duplicate it, which is its owner's. The new artifact records its source, and the source's source, and every read of it names them. `name` and `key` name the copy as they name any new artifact; a key the workspace already holds is refused for a copy.

## What else a save takes

- `note` — why this version exists, in your words, at most 2,000 characters; kept with the version and shown with it. A save into the draft takes none.
- `work` — the work item this save answers (see `comments`). Any save you make while you hold work on the artifact is that work's save, named or not: where the artifact's agent policy needs approval, it is staged whatever `as` says, and waits for a person to make it live.
- `theme` — a theme to copy in as `theme.css` (link it from your HTML): its address, identifier or key (the workspace's own first, then ours), pinned with `@<version>` if you like; `"none"` says you chose none. A `theme.css` you send yourself wins. See `themes`.
- `keepPhotoMetadata: true` — keeps every photograph in this save (JPEG, PNG, WebP, GIF, AVIF, HEIC; staged ones too) exactly as sent. Without it, each is kept without where it was taken — GPS and place names — and without the camera's and lens's serials and the owner's name; orientation, date and camera model stay, and the pictures are never re-encoded. Pass it only when the person wants the location published; each file in the answer says `photoMetadata: "removed"` or `"kept"`. A file copied with `from` is not read again.
- `validate: true` — runs every check and answers as the save would, in the same words, saving nothing.

## Settings as a file: `artifact.json`

`artifact.json` at the root of a save's files — sent inline, staged, inside a zip, or written by a change — is read as the artifact's settings and kept nowhere: no version or draft holds it and no address serves it. Any other path is an ordinary file. It is one JSON object, at most 64 KB, every key optional:

- `name`, `description` (one line; `null` clears it), `tags` (the whole list) — on a new artifact the save's own fields; on an existing one, the change `update_artifact_settings` makes.
- `library` (`true`/`false`), `room` (`on`/`off`), `crate` (`shown`/`hidden`), `agentPolicy` (`workspace`, `approval`, `live`) — set as `update_artifact_settings` sets them; on a new artifact, once it is made.
- `access` and `preview`, in the shapes the save takes — a new artifact's own; on an existing one they must equal what is stored, and a different value is refused naming `update_access`, so a file never changes who can open it.

A key left out leaves its setting as it is. A key equal to what is stored changes nothing and needs nothing, so a file read back and saved again changes nothing. A key that changes something needs what `update_artifact_settings` needs — Edit for the words, the tags and the library, the owner for the room, the crate and the agent policy — and, from a credential, its `manage` scope on an existing artifact. On a new artifact a field passed both in the save and in the file must agree. The file is applied after the version is written or found unchanged, so a file sent with unchanged content changes the settings and makes no version (`noop: true`). The answer's `settings` lists each change in a sentence, empty when there was none; with `validate: true` it names the keys the file would change, and nothing changes.

Refused whole, writing nothing: the file in a save `as: "draft"` (settings are the artifact's, never the draft's); a file that is not one JSON object; a key outside the list above, named, with the keys it takes — theme, domain and provenance are not among them yet; a value its setting refuses, in that setting's words; a change the saver may not make. A backup's artifact folder holds the file in this shape, so that folder saved back as it stands changes nothing.

## Things refused by name

- An existing artifact's `name`, `description`, `tags`, `kind`, `key` and `workspace` are not a save's fields to change — a save naming it by `artifact` with any of them is refused (one reaching it by `key` keeps them as stored, above): settings change with `update_artifact_settings` or an `artifact.json` (above), access and the link preview with `update_access`. A save may pass the `access` and `preview` already stored; a different one is refused.
- An unknown field in a document is refused, naming the nearest field that exists.

## Description and tags

Pass `description` so the artifact can be found later: `list_artifacts` matches it, and it shows under the name. Pass `tags` (lowercase, e.g. `["client-acme", "2026"]`) when some clearly apply, reusing the workspace's own vocabulary — `list_artifacts`'s `facets.tag` lists every tag in use — rather than inventing one-offs. A new artifact without tags gets up to three of the workspace's existing tags whose exact token appears in its name. Change either later with `update_artifact_settings`.

## Access, and what a link preview reveals

`access` says who can open a new artifact, as entries with roles — `[{kind: "anyone", role: "viewer"}]` for public, `[{kind: "address", value: "dana@acme.com", role: "commenter"}]`, `[{kind: "workspace", role: "editor"}]`. Left out, the workspace's default applies — **private** unless one was set. `preview` (`{level: "generic"|"title"|"custom"|"full", title?, description?, includeScreenshot?}`) says what an unfurl of a gated artifact reveals; `generic`, the platform card, by default. Both belong to the save that makes the artifact; afterwards they change with `update_access`. See `access`.

## The sandbox

Artifacts serve on an isolated origin under a sandbox policy: external https: scripts, styles, fonts, images and fetch work; cookies, `localStorage`/`IndexedDB`, credentialed requests and `wss:` do not. For state that survives a reload or is shared between viewers, use the room. See `runtime` and `rooms`.

Multi-page static exports work: `/route` serves `route/index.html` (or `route.html`).
