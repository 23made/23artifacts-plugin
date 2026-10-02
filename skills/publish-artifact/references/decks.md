---
title: Decks
summary: Real presentation decks people can edit directly, present and export — not an HTML page that looks like one. Saving, reading and changing a deck's document, its draft and its revision. Read before building any slide deck.
router: Slide decks people edit and export: saving, reading, changing.
---

# Decks

A deck is an artifact whose content is a **bento/slides document** — structured
JSON, not hand-placed HTML. That matters for one reason above all: the person
you make it for can open it in a full slide editor and fix the typo themselves,
without coming back to a model. It is saved, read and changed with the same
tools as every artifact:

- `save_artifact { name, kind: "deck", document }` — a new deck from a whole
  document (`kind` may be left out: a document is a deck). `slug`, `workspace`,
  `tags`, `description`, `access` and `preview` as for any new artifact; the
  answer says who can open it. `get_guide('deck-format')` has the document's
  every field and a minimal valid one to start from.
- `get_artifact_files { artifact, version: "draft" }` — the working document
  (the deck's draft) and its `revision`; `version: "live"` or a number reads
  the document of a saved version. A document past one answer's size comes a
  page of slides at a time, `nextCursor` continuing; change such a deck by
  `patch`, never by a whole document rebuilt from a page.
- `save_artifact { artifact, base, patch | document, revision, as }` — change
  it: `document` replaces it whole, `patch` on a `base` changes a piece at a
  time.

## The draft, and going live

A deck's working document is its **draft**: read by people who can edit it,
served at no address. Where a save lands is `as`:

- `as: "draft"` — the change goes into the draft and the address keeps serving
  what it served. Keep working this way — slide by slide, patch after patch —
  until the deck is ready.
- `as: "live"` (the default) or `"staged"` — the change becomes a new version,
  live or at its own pinned address.
- `save_artifact { artifact, base: "draft" }` with nothing else makes the draft
  as it stands the next version.

**Always pass `revision`.** Read with `get_artifact_files { version: "draft" }`,
change what you mean to, and send the change with the `revision` you read. If
someone edited in between, the save is refused as a conflict carrying the
current revision — re-read and apply your change again — instead of silently
throwing away their work. A save into the draft needs it once a draft exists.

**Editing at `/edit` saves into the draft**, so the person can rearrange slides
all afternoon without an audience watching; the editor's **Publish** makes the
draft the next version and puts it live. `hasDraft` on a read says there are
changes the audience has not been given.

## Building a deck a piece at a time

A whole document has a ceiling: it has to be written out in one call, so
anything past a few tens of KB — ten inline `svg` figures, a big table, a deck
you are still adding to — does not fit. `patch` is the same save without that
ceiling:

```json
{ "artifact": "my-deck", "base": "draft", "revision": 7, "as": "draft", "patch": [
  { "op": "put-slide", "slide": { "id": "s-costs", "elements": [] }, "after": "s-title" },
  { "op": "put-asset", "key": "brand-600", "value": "asset:inter-600" },
  { "op": "merge-doc", "value": { "title": "Q3 review" } }
] }
```

- `put-slide {slide, after?}` — `after` is a slide id, `null` for the front,
  omitted to append. A slide id that already exists is **replaced**, and giving
  it a position **moves** it.
- `remove-slide {id}`, `put-asset {key, value}`, `remove-asset {key}`.
- `merge-doc {value}` — everything else at the top of the document: `title`,
  `theme`, `fonts`, `meta`, `size`. It refuses `slides` and `assets`, which have
  their own ops, because a shallow merge would replace the whole array.

The ops apply **all-or-nothing**, in order, at most 200 in one patch, and the
result is checked exactly as a whole document is — so a patch naming a slide
that is not there changes nothing and says which op was wrong. Removing
something already gone is an error, not a quiet no-op. Every save returns the
new `revision`, so a run of patches chains without re-reading: build a deck
slide by slide by appending each one and passing the revision you just got
back.

## Make a GREAT deck, not just a correct one

The format's value is motion, charts and structure. A correct-but-static result
— bullets on slides — wastes it, and is the single most common failure. Map the
material to the feature built for it:

| When the material is… | Reach for | Why |
|---|---|---|
| numbers to **compare** | a `chart` element | bars and lines read instantly |
| a **spec / pricing / feature grid** | a `table` element | structured cells beat 20 text boxes, and they style cohesively |
| the **same thing changing** across slides | **morph**: the same element `id` on both, `transition: "morph"` on the later | the shared elements glide; this is the signature move and it's almost always missed |
| a point to **drill into** | a state slide (`stateOf` + an element `link`) | keeps the linear story clean, detail one click away |
| a **hero image** | full-bleed image + scrim + text | a still photo feels dead; let it drift |
| a **headline number** | big text with `fx: {countUp: true}` | the count-up earns the attention |
| **repeated chrome or a logo** | keep its `id` stable across slides | it morphs in place instead of popping in each time |

### Element field names are exact

- **A missing required field, a bad `type`, or a field the document, a slide or
  an element of its type does not have** is refused, naming the nearest field
  that exists — so a misspelled `fontsize` comes back as "did you mean
  `fontSize`" instead of a deck that silently ignores it.
- **Inside an element's own options** — a chart's `option`, a table's `style` —
  unknown keys are still ignored with no error, and over the connector you
  cannot see the render. So those names have to be right the first time.

The essentials:

- **text** — content is `html` (inline markup), size is `fontSize`.
  NOT `text`/`size`. Also `fontFamily`, `fontWeight`, `color`, `align`,
  `valign`, `lineHeight`, `rotation`, `opacity`.
- **table** — `columns: [{w: 1}, …]` (one per column), `rows: [{cells: [{html}]}]`,
  `header`, and `style`.
- **chart** — `option`, a pure-JSON ECharts option. **Bar/line series data must
  be plain numbers** — an object like `{value, itemStyle}` coerces to 0 and
  renders an empty chart, silently. Charts pick up the deck's palette; set
  `option.color` only to override.
- **image** — `src` (a data: URI, or `asset:<key>` into `doc.assets`).

**For the full reference — every element type (`shape`, `media`, `html`, `svg`
too), the morph recipe, the rest of the charts-lite rules, `fx`, layouts, fonts
and the column-math table — read `get_guide('deck-format')` before authoring
slides.** It is the difference between a deck that renders and one that saves
wrong in a way you cannot see from here.

Element ids are load-bearing. Morph depends on them, and so do comments,
which anchor to a slide and element id and follow them through edits. Keep them
stable; do not regenerate them on every save.

## On brand

A deck carries its look in its own document: the `theme` block (palette and
type) and, for real slide templates, `layouts`. With a theme to follow, read it
with `get_artifact_files { artifact: "<theme>" }` and put its tokens in the
deck's `theme` block and its deck layouts in `layouts`; its `guidance` — and
`surfaces.deck` — say how the look is meant to be applied, which is the part
that makes the result look designed rather than merely coloured. See `themes`.

## Real images

Reference material from a deck's `assets` map with an `asset:` key naming it
by slug or identifier, pinned with `@<version>` if you like:

```json
"assets": { "hero": "asset:dashboard-dark" }
```

The key is resolved when a version is saved — the saved file carries the real
bytes, so it stays whole forever — while the draft keeps the short key and
stays small enough to save on every edit. Changing the material never changes
a saved version; save again to pick up new bytes. The workspace's material and
ours resolve; a key in the old `library/name` form is refused, naming the
material that replaced it.

**Real fonts ride the same rail.** A font is material like any other (see
`library`), referenced from `assets` and named in `fonts`:

```json
"assets": { "brand-600": "asset:inter-600" },
"fonts":  [{ "family": "Inter", "asset": "brand-600", "weight": 600 }]
```

Add it once and every deck you make can have it. Inlining the woff2 as a
data URI works too, but a 278 KB one then travels in every save — see
`deck-format` under fonts.

See `library` for finding material. Look before you invent a placeholder.

## Embedding an artifact in a slide

A slide can hold a live 23artifacts artifact: an `html` element with `src` set
to the artifact's URL. It **renders either way** — what `"interactive": true`
buys is script execution. Without it the iframe gets an empty sandbox: the
artifact still draws, nothing in it runs. So set it for anything that has to do
something — a poll, a widget, a chart that animates on arrival — and leave it
off for a figure that is only there to be looked at. Because it stays on its own
origin, **the embedded artifact's room, live updates and presence all keep
working inside the deck** — a live poll or collaborative widget on a slide
works. Set `poster` so the slide still prints.

An interactive embed **holds the keyboard while it has focus**. Click into a
live widget and the deck's own shortcuts — arrows, `?`, `f` — go to the artifact
instead, because keystrokes inside a cross-origin document cannot reach the deck
that frames it. Clicking any deck surface outside the embed hands them back, so
**leave a margin around an embed a viewer is meant to click**: one sized to the
whole slide leaves nowhere to click, and the deck stays unnavigable by keyboard
until the page is reloaded. A non-interactive embed is pointer-inert and never
takes focus, so a static figure can safely go full-bleed.

Staying on its own origin also means the browser runs it in a **separate
process**, and hands that process a picture to draw with only for a tab it is
actually painting. A tab that never becomes visible — which is what a browser
driven by an agent leaves every tab in — has painted nothing yet when its first
frame is captured, so **screenshotting a freshly loaded deck can show the
opening slide's embed as blank** while a person opening the same URL sees it
immediately. Only the slide the deck opens on is affected; slides you navigate
to have painted by the time you get there. If you are checking your own work
this way, bring the tab to the front, or capture a second time, before
believing a figure is missing — and if the screenshot IS the deliverable,
inline the artifact's markup instead of linking it.

## Comments on a deck

Deck comments are the comments every artifact has, with an anchor that
points into the DOCUMENT rather than at rendered DOM:

```json
{ "type": "doc-node", "slide": "s3", "element": "t1" }
```

Omit `element` to comment on the slide as a whole. Because bento keeps element
ids stable — morph depends on it — these anchors **follow the deck through
edits**, and deck comments are listed across versions rather than pinned to the
one they were written on. (DOM anchors on ordinary artifacts still pin, because
a new version of an ordinary artifact really does replace it.)

Pass `parent` to reply to a comment instead of starting a new thread. Replies
are one level deep; replying to a reply joins its thread.

## Who can edit

The public URL serves a **player** build — no editor code in it at all.
The editor lives at `<url>/edit` and needs Edit, which the owner always has,
an Editor entry gives, and a share link can confer (`access`). So handing
someone an edit link is a real grant, not an honour-system flag.

## Two people editing at once

Co-editing is simply on for anyone at `/edit` — no session to start, no link to
exchange. Two editors' changes merge live, including character-level merging
inside a text box, because bento's CRDT does the work; the server only relays
frames between editors of the same deck and writes through the draft's
revision, so a conflict with a save from anywhere else is refused rather than
overwritten.
