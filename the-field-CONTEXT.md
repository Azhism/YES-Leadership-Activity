# The Field — a leadership journey · Build context

A single self-contained HTML file (`the-field.html`) that runs a story-driven
project-leadership simulation for live YES Alumni Pakistan workshops. Teams open one
shared link, pick a codename, drop into a hidden-theme story, plan a project (budget,
vendor, roles), lead it through field problems, and get a debrief.

- **Stack:** vanilla HTML + CSS + JS + inline SVG. No framework, no build step.
- **Only external dependency:** Google Fonts (`Fraunces`, `Figtree`) via `<link>`. Everything else is inline.
- **State:** in memory only (no `localStorage`/`sessionStorage`), so it survives hosting and refuses nothing when embedded.
- **Published as one artifact**, updated in place (same URL across versions).

### Recent changes
- **Three worlds now** — Chak Sohni was removed (world data + story art); `charlie` is retired and falls through to the default, Banaras Town.
- **Field problems set per world** — the tyre puncture is the shared `Field · One`; the rest are world-specific. See "Field problems by world".
- **No solution reveal** — after the team writes, Continue goes straight to the next problem. The field no longer moves the meters (`chooseCard` just saves the note and advances).
- **Field meters hidden** — the meter bar shows only in the debrief; the standing is driven entirely by the plan.
- **Cinematic tyre puncture** — `Field · One` is rebuilt as two animated scenes (the drive → the flat tyre). See Part B.

---

## Global flow (5 stages)

`Scene 0 codename` → `Story intro (4 slides, word-by-word reveal)` → `Part A: the plan`
→ `Part B: the field` → `Debrief`.

Part A sub-steps: **A1** need (two text boxes) → **A2** bucket of sacks (budget) →
**A3** vendor → **A4** team roles → project **kickoff** (bus animation) → **A5** ready summary.

The team types in A1 (the need) and in each field problem (their answer). Everything else is tap/click.

---

## Codename routing (`resolveWorld`)

Input is lowercased, then mapped to a world:

| Typed codename | World | Theme |
|---|---|---|
| `bravo` / `bastinoor` | `bastinoor` | Children's health & hygiene |
| `alpha` / `hunarabad` | `hunarabad` | Women's craft, earning fairly (market access) |
| `delta` / `banarastown` | `banarastown` | One school, one computer, no teacher |
| anything else (incl. retired `charlie`) | `banarastown` | Banaras Town (the default) |

A known codename greets by its town name; an unknown one (including `charlie`, now retired) greets by the typed name and loads **Banaras Town** (the default world). The codename→town map is internal and never shown to players.

---

## Data model: the `WORLDS` object

All content lives in `const WORLDS = { bastinoor:{…}, hunarabad:{…}, banarastown:{…} }`.
Adding or editing a world is a **data** job (plus art). Each world has:

```
id            // string, also the key into STORY_ART
tint          // accent colour
codenames     // internal map (unused for routing now; routing is in resolveWorld)
scenes[]      // 4 story slides: { key, bg, lines:[string], (last also) taskCard, button }
plan:
  a1: { q1, q2 }                        // the two need prompts
  a2: { total:20000, defaultSelected:[], missing:{}, sacks:[ {key,label,cost,essential?,hint,x,y,r} ] }
  a3: { title, intro, vendors:[ {key:'A'|'B'|'C', name, desc, price, flag:'cheap'|'fair'|'premium', startEff:{meter:+n} } ] }
  a4: { roles:[ {key:'leader'|'liaison'|'coord', label} ] }
story:   { msPerWord:110 }              // story-intro reveal pace (clamped 40–300)
kickoff: { title, line, button, showReplay:false }
arriveBg // BG key used behind the debrief
cards[]  // the field problems (see Part B)
fieldStart // how the meters START from the plan (see Meters)
meters[] // { key, label } ×3
traits[] // retained in data but NOT shown (no trait scoring since field is text-box)
debrief: { succeeded:{label,title,body}, partial:{…}, failed:{…} }
```

### Part A details
- **Bucket (A2):** one PKR 20,000 bucket; each sack is a fixed cost added once. `essential:true` flags the two items the field quietly checks (shown only in the debrief's "left out of the bucket" line, never labelled essential in the UI). `x,y,r` position sacks in the pile. `defaultSelected` pre-loads sacks (currently `[]` in all three worlds). Component: `mountBucket(root, CONFIG, initial)`; it emits `bucket:change|back|next`.
- **Vendors (A3):** three illustrated characters (keys A/B/C, art in `VENDOR_ART`). `flag` drives field effects (`vendor:cheap|fair|premium`); `startEff` nudges starting meters; `price` is a grade label shown when there is no numeric `cost`.
- **Roles (A4):** three roles; a role counts as "taken" only if a name is typed. `liaison` and `coord` are the two the field keys on.

---

## Meters & starting values (`fieldStart`, handled in `startField`)

Meters are computed from the **plan** at the start of the field and do **not** move during it (field problems only collect written answers, see Part B). The meter bar is **hidden through the field** and appears only in the debrief. Starting values per world:

- **Money worlds** (`hunarabad`, `banarastown`): meters **Budget / Community Trust / Team Morale**. `fieldStart:{ trust:50, morale:60, reserveKey:'budget', reserveBase:35, reserveScale:50 }` → Budget starts at `clamp(reserveBase + (reserve/total)*reserveScale)` (≈35 when fully spent, ≈85 when little spent).
- **Health world** (`bastinoor`): meters **Health Impact / Community Trust / Team Morale**. `fieldStart:{ impact:50, trust:50, morale:60, spendKey:'impact', spendBonus:8 }` → Impact gets a small bonus for spending more of the budget.
- Vendor `startEff` is applied on top for all worlds.

All three worlds use `fieldStart`; the old hard-coded `else` branch in `startField` is now dead code (Chak Sohni, its only user, was removed).

---

## Part B: the field

**Three field problems per world.** The **tyre puncture is the shared opener (`Field · One`) in all three worlds**; the other two are world-specific. After each problem the team writes an answer and taps **Continue**, and that is it: there is **no solution/consequence reveal** (it was removed). `chooseCard()` just saves the typed note and advances to the next problem (or the debrief), and the field does **not** move the meters.

Two card renderers, dispatched in `renderCard` (`cinematic` is checked first, then `beats`):

- **Cinematic (`card.cinematic === true`) — the tyre puncture only.** `renderPunctureCard` plays two full-bleed animated scenes: **Scene 1 (the drive)** with the "Set off" CTA, then **Scene 2 (the flat tyre)**, a CSS-timed sequence (road decelerates to a stop, "pssshhh" + air puffs, rear wheel squashes flat, van tilts with a jolt, scene shakes, dust skids, red "!" badge pops and pulses, clock appears and ticks) with a **Replay** button and the "Check the damage" CTA. Narration fades in line by line (split from `beat1` / `beat2`); a 12-segment progress strip (`CIN_TOTAL_SCENES`) sits in the label row. Both CTAs lead into the existing text box (`question()`). Everything is scoped under the `cin-` prefix; the illustration keeps its daylight palette in both themes on purpose. Reduced motion collapses it to the end state.
- **Beat card (`card.beats === true`) — every other problem.** `renderBeatCard` — a focal **vignette** + **Beat 1** (pop-in) → tap/Next → **Beat 2** → a **slide-up sheet** with the question + the **text box** + Continue.

### `resolve` arrays (currently inert)
Each card still carries a declarative `resolve` array (`when` → `eff` meter deltas + a `say` sentence), kept as data. But since the solution reveal was removed and the field no longer moves meters, `resolveField` is **not called** and these arrays are not applied. Re-enabling them means calling `resolveField`/`renderConsequence` from `chooseCard` again and un-hiding the meter bar. `when` grammar (array = all must pass): `have:ITEMKEY` · `miss:ITEMKEY` · `role:ROLEKEY` · `norole:ROLEKEY` · `vendor:FLAG` · `always`.

### Field problems by world
Each world has **3** problems, `Field · One` first. The tyre puncture is shared (cinematic); the rest are world-specific.

**Hunarabad** — women's craft, earning fairly
1. **A tyre puncture** *(cinematic)* — a breakdown on the way, late before you start.
2. **The language wall** — the alumni who can pitch speaks Urdu; the women here speak only the local tongue.
3. **Forgot to document it** — a great session, but no photos and no attendance, and the report needs proof.

**Basti Noor** — children's health & hygiene
1. **A tyre puncture** *(cinematic)*
2. **A surprise headcount** — more children turn up than you brought kits for.
3. **No water for the soap demo** — the handwashing lesson, and the tap runs dry.

**Banaras Town** — one school, one computer, no teacher
1. **A tyre puncture** *(cinematic)*
2. **The projector will not load** — the whole session was on a presentation, no internet, and the old machine stalls.
3. **The students already know it** — they are sharper than expected and sit ahead of your plan.

---

## Debrief (`goDebrief`)
Outcome from the three meters' average. Since the field no longer moves them, the standing is driven entirely by the **plan** (bucket spend + vendor `startEff`): `failed` (<35, or any meter ≤0), `succeeded` (≥60 and none <20), else `partial`. Tier `label`/`title`/`body` come from the world's `debrief`. The screen shows the verdict, final meters, the "back to your plan" recap (the essentials left out of the bucket + the A1 answers), and "what you wrote in the field" (the per-problem notes). No trait bars.

---

## Art registries
- **`BG`** — full field/arrival backgrounds, `viewBox 0 0 400 700`, `preserveAspectRatio` slice. Keys referenced by `card.bg` and `world.arriveBg`. Helper `sky(a,b)` makes the gradient.
- **`STORY_ART`** — keyed by world `id`; 4 full scenes each, `viewBox 0 0 1280 720`. Used by the story-intro engine (indexed by slide position).
- **`VIGNETTE`** — focal field illustrations, `viewBox 0 0 320 200`, shown inside the card frame. Motion via CSS classes (`vg-dust`, `vg-bird`, `vg-float`, `vg-spin`, `vg-pop`, `vg-drop`, `vg-bounce`), all disabled under `prefers-reduced-motion`.
- **`CIN_ART`** — the two cinematic tyre-puncture scenes (`drive`, `flat`), `viewBox 0 0 480 220`. Used by `renderPunctureCard`; its motion and chrome styles are injected once (keyframes/classes all `cin-`/`cin*` prefixed to avoid collisions), and collapse to the end state under `prefers-reduced-motion`.

---

## Design rules (keep these)
- Palette tokens live on `:root` (warm paper, dusk-blue ink `--ink`, marigold `--gold`, jade, per-world plum/teal/green/indigo tints). Light/dark via CSS variables.
- Fonts: `Fraunces` (serif) for story/headings, `Figtree` (sans) for UI.
- **Reader-facing text:** no `—` (em dash); use commas or colons. Use curly quotes.
- Respect `prefers-reduced-motion`: no ambient/looping motion, no word-by-word typing (show instantly).
- Real `<button>` elements, visible focus, works at phone width and in dark mode, no console errors.

---

## Extending it
- **New field problem:** add a beat card to a world's `cards[]` (`beats:true`, `vignette`, `beat1`, `beat2`, `question`) and add the `VIGNETTE` entry. Each world is intentionally **3** problems, `Field · One` (the cinematic tyre puncture) first. The cinematic treatment (`cinematic:true` + `renderPunctureCard`) is puncture-only; `resolve` can be added but is currently inert.
- **New world:** add a `WORLDS` entry (data), add its `STORY_ART[id]` scenes and any `BG`/vignettes, and add a route in `resolveWorld`.
- **Tune difficulty:** the meter numbers live entirely in each card's `resolve` and in `fieldStart`.

## Testing
Playwright over `file://`, Chromium at `/opt/pw-browsers/chromium` (`NODE_PATH=$(npm root -g)`). A run drives codename → story → Part A → field → debrief and asserts the outcome and zero `pageerror`s (ignore the Google-Fonts tunnel error when offline).

## Notes / open items
- With field effects off, the outcome is entirely plan-driven (bucket spend + vendor). Weak builds tend to land on **Partial** rather than Struggled unless most essentials and roles are skipped (Budget stays high on low spend). Tune `fieldStart` for harsher fails, or re-enable `resolve` to let the field swing meters again.
- The field is reflective now: problems collect written answers for the debrief but no longer judge or score them in the moment.
- Art is a solid first pass per world tint; the story scenes and vignettes are the easiest things to refine. The cinematic puncture scenes (`CIN_ART`) match the "scenes" reference build.
