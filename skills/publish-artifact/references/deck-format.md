---
title: Deck format
summary: The bento/slides document — every element type, the morph recipe, charts-lite rules, fx, layouts, fonts and column math. Read before authoring a deck's JSON so the first draft renders.
router: The deck document: elements, charts, layouts, fonts.
---

# Deck format

The `decks` guide covers the workflow — `save_artifact` with a `document` or a
`patch`, the draft and its `revision`, going live. **This** is the document
itself: what goes in the `document` you save, field by field.

Read it before you write slides. The format's value is motion, charts, tables
and structure; a correct-but-static result — bullets on slides — wastes it and
is the single most common failure. Worse, a few fields fail *silently*: a
mistyped property is ignored, a chart with the wrong data shape renders as
empty bars, a font the document doesn't carry falls back without a word. None
of that is visible in the JSON. This guide is how you get it right the first
time — which matters more here than anywhere, because **over the connector you
are building blind**: there is no browser rendering the slide back to you.

## Start from a valid document, then change it

A new deck starts from a whole document you save — begin from the **minimal
valid document** below, with `size`, `theme` and one slide, and add to it. Once
the deck exists, `get_artifact_files { artifact, version: "draft" }` gives you
its document plus its `revision`. Build on it: change what you mean to, and
save the change — a `patch`, or the whole `document` — with the `revision` you
read.

The server escaping, the `#bento-doc` block, the shell — none of that is yours
to touch. You send JSON; the platform stores it, renders the player, and serves
it. Never put a literal `</script>` worry into your head — that's the server's
job, not yours.

## Two backstops, and their limits

You can't see the render, so lean on what you can:

- **A save checks the document strictly.** A missing required field, a bad
  element `type`, `size`/`theme` absent, or a field the document, a slide or an
  element of its type does not have — it refuses the save and names what is
  wrong and the nearest field that exists, rather than saving something that
  won't open or silently drops your styling. But it does **not** read inside an
  element's own options (a chart's `option`, a table's `style`, where unknown
  keys are ignored) and it cannot catch a chart whose data is the wrong shape.
  The field names below are exact for exactly this reason.
- **Whoever opens `<url>/edit` gets the full editor** — `window.bento.validate()`
  (reports overflow, dead effects, broken refs, un-embedded fonts), a
  *Fit height to text* button, and their own eyes. A human catching a typo is
  the pitch of decks. But don't rely on it to finish your work: build to the
  guardrails so the deck is right before anyone opens it.

## Make a GREAT deck — map material to the feature built for it

| When the material is… | Reach for | Why |
|---|---|---|
| numbers to **compare** (trend, magnitude, share) | a **chart** | bars/lines read instantly |
| **named things placed against two axes** (a 2×2, a positioning matrix) | a **scatter** chart with `{name, value:[x,y]}` items | each point labels itself and names itself on hover, and the placement stays editable |
| a **spec / pricing / feature grid** (rows × columns) | a **table** | structured cells beat 20 text boxes; it styles cohesively |
| the **same thing changing** across consecutive slides | **morph**: same element `id` on both + `transition:"morph"` on the later | the shared elements glide — the signature move, almost always missed |
| a point to **drill into** | a **state slide** (`stateOf` + an element `link`) | keeps the linear story clean; detail one click away |
| a **hero / full-slide image** | full-bleed image + scrim rect + text, with **ken-burns** | a static photo feels dead; a slow drift feels intentional |
| a **headline number** | big text + `fx:{countUp:true}` | the count-up earns the attention |
| a **sequence / flow / timeline** | a line or `path` with a `dash-march` loop, or morph a highlight through the steps | motion carries the eye along |
| **repeated chrome / a logo** | keep its `id` stable across slides | it morphs in place instead of popping in each time |
| a **demo clip / soundbite** | a **media** element (embed short, link long) | a live clip beats a screenshot of one |

## Copy-paste recipes

**Morph a title + accent bar between two slides** — identical ids, `transition:"morph"` on the second:
```json
// slide 1
{ "id":"s1","transition":"none","elements":[
  { "id":"headline","type":"text","x":96,"y":140,"w":900,"h":200,"html":"Big claim.","fontSize":120,"fontWeight":900,"color":"#111","align":"left","valign":"top","lineHeight":1,"rotation":0,"opacity":1 },
  { "id":"bar","type":"shape","shape":"rect","x":96,"y":380,"w":320,"h":16,"fill":"#aa412a","stroke":"none","strokeWidth":0,"radius":0,"rotation":0,"opacity":1 } ] }
// slide 2 — same ids, new frames → they animate between them
{ "id":"s2","transition":"morph","elements":[
  { "id":"headline","type":"text","x":96,"y":84,"w":500,"h":80,"html":"Big claim.","fontSize":40,"fontWeight":900,"color":"#888","align":"left","valign":"top","lineHeight":1,"rotation":0,"opacity":1 },
  { "id":"bar","type":"shape","shape":"rect","x":96,"y":170,"w":16,"h":450,"fill":"#aa412a","stroke":"none","strokeWidth":0,"radius":0,"rotation":0,"opacity":1 } ] }
```

**A bar chart** — bar/line data is PLAIN NUMBERS (see chart rules below):
```json
{ "id":"c1","type":"chart","x":96,"y":260,"w":1088,"h":380,"rotation":0,"opacity":1,"preset":"bar","option":{
  "xAxis":{"type":"category","data":["2022","2023","2024","2025"]},
  "yAxis":{"type":"value"},
  "series":[{"type":"bar","data":[420,780,1300,2450],"itemStyle":{"color":"#141310"},"barWidth":90}],
  "tooltip":{"trigger":"item","formatter":"{b}: {c}"} },
  "fx":{"enter":"fade-up"} }
```

**A comparison table** — a real HTML table; cells take the same inline-html subset as text:
```json
{ "id":"tbl1","type":"table","x":240,"y":220,"w":800,"h":260,"rotation":0,"opacity":1,
  "header":true,
  "columns":[{"w":1.4},{"w":1},{"w":1}],
  "rows":[
    { "cells":[{"html":"Plan"},{"html":"Price","align":"right"},{"html":"Seats","align":"right"}] },
    { "cells":[{"html":"Team"},{"html":"$29"},{"html":"5"}] },
    { "cells":[{"html":"Business"},{"html":"$79"},{"html":"25"}] } ],
  "style":{"headerBg":"#1E2A3A","headerColor":"#fff","zebra":"rgba(30,42,58,0.05)",
    "borderColor":"rgba(30,42,58,0.14)","borderWidth":1,"cellPadX":16,"cellPadY":11,
    "fontSize":18,"color":"#1E2A3A","radius":10} }
```

**A state slide reached by clicking a node** — the clickable element is on the parent; the state lives adjacent:
```json
// on the parent slide, an element the viewer clicks:
{ "id":"node-ingest","type":"shape","shape":"ellipse","x":330,"y":180,"w":74,"h":74,"fill":"#0B0E1E","stroke":"#7A5CFF","strokeWidth":2,"radius":0,"rotation":0,"opacity":1,"link":"state-ingest" }
// a hidden state slide (arrow keys skip it; ← returns to the parent):
{ "id":"state-ingest","stateOf":"parent-slide-id","transition":"morph","name":"INGEST","elements":[ /* … */
  { "id":"dismiss","type":"shape","shape":"rect","x":0,"y":0,"w":1280,"h":720,"fill":"rgba(0,0,0,0)","stroke":"none","strokeWidth":0,"radius":0,"rotation":0,"opacity":1,"link":"parent-slide-id" } ] }
```

**Full-bleed hero image with ken-burns + scrim + text:**
```json
{ "id":"photo","type":"image","x":0,"y":0,"w":1280,"h":720,"src":"asset:hero","fit":"cover","radius":0,"rotation":0,"opacity":1,"fx":{"ambient":"kenburns","ken":{"dir":"drift","scale":1.09,"duration":22}} },
{ "id":"scrim","type":"shape","shape":"rect","x":0,"y":0,"w":1280,"h":720,"fill":"rgba(10,14,26,0.55)","stroke":"none","strokeWidth":0,"radius":0,"rotation":0,"opacity":1 },
{ "id":"htitle","type":"text","x":96,"y":460,"w":1000,"h":180,"html":"On top of the photo.","fontSize":76,"fontWeight":800,"color":"#fff","align":"left","valign":"top","lineHeight":1.05,"rotation":0,"opacity":1,"fx":{"enter":"fade-up"} }
```
Reference the image with `"asset:hero"` and put the material in `doc.assets`
under `hero` — `"hero": "asset:dashboard-dark"`, the material's slug or
identifier; see the `library` guide for finding it. The key becomes real bytes
when a version is saved, so the saved file stays whole.

## Minimal valid document

`size` and `theme` (including `fontFamily`) are **required** — the deck will not
open without them — and elements should carry the full field set shown.

```json
{
  "format": "bento/slides", "version": 1, "title": "My deck",
  "size": { "width": 1280, "height": 720 },
  "theme": { "background": "#0b0f19", "color": "#f8fafc",
             "accent": "#aa412a", "fontFamily": "system-ui, sans-serif" },
  "slides": [
    { "id": "s1", "background": "#0b0f19", "transition": "none",
      "notes": "speaker notes here",
      "elements": [
        { "id": "t1", "type": "text", "x": 96, "y": 260, "w": 1088, "h": 160,
          "rotation": 0, "opacity": 1,
          "html": "Hello.",
          "fontSize": 88, "fontFamily": "system-ui, sans-serif",
          "fontWeight": 800, "color": "#f8fafc",
          "align": "left", "valign": "top", "lineHeight": 1.1 }
      ] }
  ]
}
```

## Element types (all share `id, x, y, w, h, rotation, opacity`)

- **text** — `html` (inline `<b> <i> <br>` ok), `fontSize`, `fontFamily`,
  `fontWeight`, `color`, `align` (`left|center|right`), `valign`, `lineHeight`,
  optional `letterSpacing`. Content is `html`, size is `fontSize` — **not**
  `text`/`size`.
- **shape** — `shape` = `rect|ellipse|triangle|arrow|line|path`, `fill`,
  `stroke`, `strokeWidth`, `radius` (rect corner). Optional `fillGradient`
  `{angle, stops:[{at:0..1, color}]}` (CSS-convention angle). Lines take their
  colour from `fill` and draw horizontally across the box (rotate for vertical);
  `strokeStyle: solid|dashed|dotted`; tips `lineStart`/`lineEnd` =
  `arrow|dot|bar`. A `path` is a free vector: `d` (SVG path data) + `pathBox`
  `[x,y,w,h]` authoring viewBox stretched into the element box; for a **curved
  line** set `fill:"transparent"` + a `stroke` + `strokeWidth`. A **connector**
  is a `line`/`path` with `from`/`to: {el, side}` — its ends follow those
  elements and re-route when they move (side `"auto"` picks the nearest border).
- **image** — `src` = data URI or `"asset:<key>"` into `doc.assets`;
  `fit: cover|contain|fill`, `radius`.
- **chart** — `preset: bar|line|pie|scatter`, `option` = ECharts-SHAPED pure
  JSON. See the chart rules below — this is the element most likely to render
  wrong.
- **table** — `columns` (array of `{w}` fractional weights, one per column),
  `rows` (array of `{cells:[{html, align?, color?, bg?, bold?}]}`), `header`
  (bool — row 0 is the header), and a `style` object (`headerBg`, `headerColor`,
  `zebra?`, `borderColor`, `borderWidth`, `cellPadX`, `cellPadY`, `fontSize`,
  `color`, `radius`). A real HTML table. For grids — **not** numeric trends
  (use a chart).
- **media** — `kind: video|audio`, `src` = data URI (embedded, travels in the
  file), external URL/relative path (referenced, keeps the file small, needs the
  network at play time), or `"asset:<key>"`. Video also takes `poster`, `fit`,
  `radius`. Flags: `controls`, `autoplay`, `loop`, `muted`. **Autoplay fires
  only in present mode**, and browsers require `muted:true` for a video to
  autoplay. **Embed only SHORT clips** — a big data URI bloats the file; host
  large media and reference its URL.
- **html** — a live artifact embedded in a sandboxed iframe. `markup` (inline
  HTML) or `src` (a 23artifacts artifact URL — the deck stays small and always
  shows its live version). It renders either way; **set
  `"interactive": true`** for an artifact that has to RUN — without it the
  iframe gets an empty sandbox, so the page draws and its scripts don't. Give it
  a `poster` so the slide still prints. A cross-origin 23artifacts artifact keeps its own room,
  live updates and presence working *inside* the slide.
- **svg** — `asset` or `markup` for static artwork. Prefer composing
  rects/texts/paths — those stay editable and can morph.

## Charts render silently wrong unless you follow these

The engine is **charts-lite** — it reads the ECharts option *shape* and ignores
every key it doesn't implement, with no warning. The two failures that bite:

- **Bar/line series data must be plain numbers.** `[420, 780, 1300]`, not
  `[{value:420, itemStyle:{…}}]` — an object item coerces to **0**, so your
  bars render flat and empty. **Pie** takes `{name, value}` items, and
  **scatter** takes `[x, y]` pairs or `{name, value:[x, y]}`. Colour by series
  (`series[].itemStyle.color`), not per item.
- **`label` on a bar/line series does nothing** — value labels above bars are
  read for **pie and scatter** only. To show numbers on a cartesian chart, put
  them in a table beside it or in text elements.

Template formatters only (`{b}` `{c}` `{d}`), never functions — the JSON must
be pure. What charts-lite honours:
- top level — `color`, `series`, `xAxis`, `yAxis`, `legend`, `grid`, `tooltip`,
  `textStyle`, `dataZoom`
- any series — `type`, `name`, `data`, `yAxisIndex`, `itemStyle.color`
- bar — `itemStyle.borderRadius`; line — `smooth`, `symbol`, `symbolSize`,
  `lineStyle.color`, `lineStyle.width`, `lineStyle.type`
  (`solid|dashed|dotted` — the same pattern a dashed line *shape* draws, so use
  it for a series that isn't real: a forecast, a target, a counterfactual),
  `areaStyle.color`
- pie — `radius`, `label.formatter` (or `label:false`), `itemStyle.borderColor`,
  `itemStyle.borderWidth`
- scatter — `symbolSize`, `label` (`show`, `position`
  `right|left|top|bottom`, `formatter`, `fontSize`, `fontWeight`, `color`)
- axes — `type`, `data`, `min`, `max`, `axisLabel` (`show`, `fontSize`,
  `fontWeight`, `color`, `formatter`), `axisLine` (`show`), `splitLine` (`show`)
- legend — `show`, `top`, `bottom`, `textStyle.fontSize`, `textStyle.fontWeight`

**A scatter point can carry a name, and that is usually the point.** `[[21.8,
81.9]]` is a position and nothing else: it cannot be labelled, and the best its
tooltip can say is its own coordinates. Give the item a name and it labels
itself beside the dot and names itself on hover:

```json
{ "type":"scatter","name":"Candidate venture","symbolSize":8,
  "itemStyle":{"color":"#94A3B8"},
  "label":{"position":"right","fontSize":12,"fontWeight":600,"color":"#94A3B8"},
  "data":[ {"name":"23artifacts","value":[6.4,61.6]},
           {"name":"23rides","value":[70.5,86.2]} ] }
```

That is what turns a **2×2 positioning matrix** into a real chart rather than a
picture of one — one series per band (focal vs the rest) so colour and symbol
size stay per-series, `xAxis`/`yAxis` `min`/`max` pinned to `0`/`100` so the
centre lands in the centre, and `axisLabel:{show:false}`,
`axisLine:{show:false}`, `splitLine:{show:false}` when the slide draws its own
cross and quadrant tags. A scatter's x axis is a **value** axis: give `xAxis` no
`data`, or the points won't be drawn at all.

**Dual axis** for two series on very different scales (e.g. volume + a %): make
`yAxis` an ARRAY of two `{type:"value"}` axes (give the 2nd
`axisLabel:{formatter:"{value}%"}`), point the odd series at it with
`"yAxisIndex":1`, and render that one as a `line` over the bars.

Charts inherit the deck's palette automatically; set `option.color` only to
override it.

## The rules that make decks feel designed

- **Morph = shared ids.** A slide with `"transition":"morph"` tweens any element
  whose `id` matches the previous slide — position, size, colour, gradients.
  Carry 2–4 ids through the deck and rearrange them per slide. Emit
  deterministic ids so this keeps working across edits.
- **`morphId` decouples morph identity from `id`.** The pairing key is
  `morphId || id`, so an element can keep its own `id` and set
  `"morphId":"running-head"` to morph against a differently-named element on the
  next slide. Unique **within** a slide. Plain shared `id` still works and is
  simplest when you control both slides.
- **Entrances**: `fx:{ enter:"fade-up", order:0 }` — equal `order` =
  simultaneous. On a **morph arrival** the rule is per element: one that has a
  morph partner on the previous slide is already in motion, so `fx.enter` and
  `fx.countUp` are skipped for it; one that is **new** to the slide runs both
  normally (and, absent `fx.enter`, gets an automatic fade-and-rise so nothing
  just pops in). So a headline count-up or a panel sweeping in is fine on a morph
  slide — just make sure it's new to that slide.
- **Ken-burns**: `fx:{ ambient:"kenburns", ken:{ dir:"drift|out|in",
  scale:1.08, duration:20 } }` — `drift` loops, `out`/`in` settle once on entry.
- **Loops** (`fx.loop`): `{ type:"dash-march", distance:18, duration:1.4 }`
  marches stroke dashes — it needs a `stroke` **and** a dash pattern
  (`strokeStyle:"dashed"`); on a solid stroke there is nothing to see.
  `{ type:"motion-path", path:"M0,0 C60,-40 140,40 200,0", duration:6 }` drifts
  the element along a path **relative to its resting position**. Never put an
  entrance tween on a motion-path element — they fight over the same transform.
- **Interactivity**: element `link:"<slide-id>"` jumps on click; a slide with
  `stateOf:"<parent-id>"` is a hidden variant reached only by links (arrow keys
  skip it, ← returns to parent). Give clickable things a padded transparent rect
  as the hit target, not the text itself.
- **Numbers count up** with `fx:{ countUp:true }`.
- **Speaker notes** (`notes`, per slide) are part of the document — write them.

## Layout guardrails

- Canonical canvas **1280×720** (`doc.size` can differ — read it first).
- Keep **96px** side margins → right-most content `x ≤ 1184`.
- Column arithmetic on 1280×720 inside 96px margins (content band 1088px wide),
  already done — use these rather than computing your own:

  | Split | Width | `x` positions | Gutter |
  |---|---|---|---|
  | 2 columns | 528 | 96, 656 | 32 |
  | 3 columns | 340 | 96, 470, 844 | 34 |
  | 4 columns | 254 | 96, 374, 652, 930 | 24 |
  | 60 / 40 (text + image) | 624 / 432 | 96, 752 | 32 |

  A title band `y:72 h:84` over content starting at `y:208` leaves 416px of
  content height above a 96px bottom margin.
- One accent colour; two typefaces max. `theme` sets deck defaults — the
  `themes` guide carries the workspace brand and how it's meant to be applied.
- **Fonts belong to the DOCUMENT, not the app.** A `fontFamily` naming a face
  the document doesn't carry falls back **silently** — and it will usually look
  right to *you*, because you're the one with it installed; everyone else gets
  the fallback. Either carry the woff2 or name a **full system stack** and mean
  it (`"'Fraunces', Georgia, serif"`, never a bare family name).

  Carry it **by reference**, not inline. A woff2 is material of kind `font`
  (see `library`), and a deck points at it the same way it points at an image —
  a saved version inlines the bytes, the draft stays small enough to rewrite on
  every save, and the typeface is added once for every deck you ever make:

  ```json
  "assets": { "brand-600": "asset:inter-600" },
  "fonts":  [{ "family": "Inter", "asset": "brand-600", "weight": 600 }]
  ```

  Then `"fontFamily": "Inter, system-ui, sans-serif"` renders in the real face
  for everyone. A woff2 data URI inline in `doc.assets` also works and is what
  a standalone `.bento.html` needs, but a 278 KB one has to travel in every
  save — which is how decks end up with system-font chrome around
  figures in the real brand faces, disagreeing with themselves on one slide.

## Layouts and `role`

`doc.layouts` is a supported top-level array of Slide-shaped templates the
editor offers under *Apply layout*; every deck also gets five built-ins scaled
to its `doc.size`. When generating, the part that matters is **`role`**: any
text element can carry `"role": "title" | "subtitle" | "body" | "kicker"`.
Applying a layout matches donor to target by `id` first, then by `role` + type —
so roles are what let someone restyle your deck later without re-typing it.
One key per element, and the deck feels native to the editor.

## Dynamic fields (tokens in text `html`)

Resolve at render time (the model keeps the raw token, so numbering updates
automatically): `{{page}}`, `{{pages}}` (position among non-state slides;
zero-pad with `{{page:2}}`→"06"), `{{title}}`, `{{date}}`, `{{time}}`, and the
document properties `{{author}}`, `{{company}}`, `{{subject}}`, `{{event}}` —
set those in an optional top-level `"meta": {author, company, subject, event,
keywords}`. Great for title slides and footers.

## Gotchas

- **Don't invent property names** — unknown keys are ignored, so a typo means
  your styling silently doesn't apply. The field names above are exact.
- **`docId`** is the document's identity — never regenerate it when editing;
  round-trip it through what `get_artifact_files` read.
- **Don't set `readonly` or `template`** — those are standalone-file flags.
  Hosted, view-vs-edit is decided by who holds the `edit` capability, and the
  platform strips these anyway. Going live is the save's `as`.
- **Charts and fonts fail quietly** — the two silent failures above are the ones
  that will land you an empty chart or a wrong typeface with no error. Re-read
  those sections before you trust a chart or name a font.
