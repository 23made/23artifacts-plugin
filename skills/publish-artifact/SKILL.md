---
name: publish-artifact
description: Publish what you make to 23artifacts — a page with its images, video, sound and scripts, markdown, a deck, or a single file — as a live, private-by-default address, and build with the platform's pieces — a workspace's material library and themes, shared realtime state (rooms), access control, and comment threads. Use when the user asks to publish, host, deploy or share an artifact, page, deck or site, or wants one in a named theme or collaborative.
---

# 23artifacts

Publishes the artifact you're working on at a live address and returns it — and gives you the pieces to make what you publish good: the workspace's real material (screenshots, logos, clips, sounds, typefaces), its themes, live shared state between viewers, and a place for people to comment on it.

## The connector

Everything here runs through the **23artifacts** MCP connector (`https://23artifacts.com/mcp`). If its tools aren't available, tell the user to add the connector once — they'll be walked through signing in the first time it's used.

## Read this first, in this order

**The look is yours unless a theme is named.** A workspace either leaves the design to you — the starting value, *let the agent decide* — or has a default theme, which you use unless the user asks for another look; `get_workspace` says which, and so does `list_artifacts` with `q: "kind:theme"`. When the user names a theme ("the RapidAI theme", "our house theme"), use that one. With no default and no name, design what suits the artifact; picking one of the workspace's themes is yours to choose when you have nothing better to go on.

**Real material beats a stand-in.** If the user has a screenshot, a logo or a clip of the thing you're describing, `list_artifacts` with `q: "in:library"` finds it; use it instead of inventing a placeholder.

**Then save.** The minimum is `save_artifact { name, files }`: a new artifact, live at its own address.

**When 23artifacts gets in your way, say so.** `send_feedback` tells the people who run 23artifacts — `friction`, a `bug` or an `idea`, what happened in your own words, and the `tool` and `error` when a call went wrong. Send it when the user asks ("tell 23artifacts this is confusing"), and on your own when the product got in your way: a refusal that didn't say what to do, a capability it lacked, a workaround you needed. When you send on your own, tell the user in one line, with the reference it answers. Never include an artifact's contents or anyone's personal details.

## Guides

Each of these is a file under `references/`, beside this one. Read the one that matches what you're doing — you don't need them all.

- **`references/publishing.md`** — `save_artifact`: whole content or changes on a base, live, staged or into the draft, slugs, workspaces, copies and the files a save leaves out. Read before your first save.
- **`references/decks.md`** — real presentation decks the recipient can edit and present themselves: saving, reading and changing a deck's document and its draft. Read before building any slide deck.
- **`references/deck-format.md`** — the deck document itself, field by field: every element type, the morph recipe, charts-lite rules, fx, layouts, fonts, column math. Read before authoring a deck's JSON so the first draft renders.
- **`references/themes.md`** — themes as material: tokens, a stylesheet, guidance and deck layouts, and the workspace's default theme. Read when a theme is named or set as the default.
- **`references/library.md`** — the material library: real screenshots, logos, device frames, clips, sounds, typefaces — finding, using and adding them. Read before inventing a placeholder.
- **`references/runtime.md`** — what works inside the artifact sandbox and what doesn't (no cookies, no storage, no `wss:`). Read before writing JavaScript that needs to persist or connect.
- **`references/rooms.md`** — shared state between viewers: votes, presence, live cursors, collaborative editing, with no backend. Read when more than one person interacts with the same page.
- **`references/comments.md`** — comment threads people leave on the page, recordings, the held wait, and taking work on as an agent. Read before collecting feedback or acting on it.
- **`references/access.md`** — public, private, email-gated, org-only, share links. Read when the artifact isn't meant for everyone.
- **`references/large-sites.md`** — file trees too big to inline, and video up to 4 GB: uploading in parts, or from a shell. Read when a save won't fit in a tool call.
- **`references/managing.md`** — finding, reading, versions, offline, live or deleted, settings, the log, workspaces and credentials.

Call `get_guide` with a topic name to read one, or read the resource directly if your client exposes them.
