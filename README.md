# Étude — website project (monochrome build)

Static site, 5 pages, no build step. Open `index.html` in a browser.
**All files must sit in the same folder** — navigation between pages uses
relative links (`models.html`, etc.).

---

## Pages

| File | What's on it |
|---|---|
| `index.html` | Homepage. Hero + two scroll-reveal sections. |
| `resources.html` | Docs layout: left sidebar (search + section nav), content area, custom dot-scroller. |
| `models.html` | Model catalogue: the four supported models as full-width rows in a vertical list. |
| `community.html` | Interactive network graphic + social buttons. |
| `download.html` | **Header only — no content yet.** Every "Download" button links here. |

---

## Brand / design tokens

Two themes, both driven by the same token names. Light is the default and
the one that's been designed against; dark is set with `data-theme="dark"`
on `<html>`.

The palette is neutral throughout, with exactly one hue on the site: `--net`,
the colour the community network turns as the cursor reaches it. Everything runs off CSS
custom properties in a `:root` block at the top of each page's `<style>` —
change them there and the page follows.

```
--ink        #0b0b0d   headlines, primary text
--ink-2      #6e6e73   secondary text
--ink-3      #a1a1a6   tertiary, placeholders
--line       #e0e0e3   hairlines
--surface    #f5f5f7   raised panels (resources sidebar)
--bg         #ffffff
--net        74,144,217   community network accent (R,G,B triplet)

--ink-glass-fg   text sitting on the inked pane
--topbar-bg      \
--menu-bg         |  values the pages used to hardcode; they're tokens
--pane            |  now so the dark theme can reach them without every
--pane-hover      |  rule being written twice
--sheen           |
--sheen-soft      |
--lip             |
--dot            /   resources scroll track
```

### Themes

`:root` holds light; `:root[data-theme="dark"]` overrides it. Because
everything reads from tokens, most of the site inverts for free — a rule
only needs its own dark entry if it hardcoded a light value, and the ones
that do are listed in that block (the Download pill's shading, mainly,
whose white highlights have nothing to do on a white pane).

The toggle is a glass circle, bottom-right, on every page. The choice goes
to `localStorage` under `etude-theme`, and an inline script in `<head>`
applies it **before first paint** so a dark-mode visitor never gets a white
flash. With nothing stored it follows `prefers-color-scheme`. All storage
access is in `try/catch`, so it degrades to session-only rather than
throwing where storage is blocked.

Flipping the theme fires `etude:theme` on `document`. The community canvas
listens for it and re-reads `--net` and `--dot`, since canvas colours can't
come from CSS on their own.

- **Name:** Étude (accented É everywhere, including the topbar logo)
- **Fonts:** unchanged from the colour build —
  - Logo — Satoshi (Fontshare)
  - Topbar nav + Download — Fira Sans (Google Fonts)
  - Homepage hero + resources body — SF Pro via `-apple-system`
  - Model cards — Cabinet Grotesk (Fontshare)
  - Models page body — Inter

### Header (identical on all 5 pages)
- 70px, `position: sticky; top: 0` — content scrolls underneath so the
  frosted blur has something to act on
- `rgba(255,255,255,0.72)` + `backdrop-filter: saturate(180%) blur(20px)`,
  hairline `--line` bottom border. On community the bar inverts: a
  translucent black bar over a black page is invisible, so it lifts to
  `rgba(255,255,255,0.055)` and lets the blurred dots read through it
- Black logo; `--ink-2` nav links ink to `rgb(var(--net))` blue on hover
  (active ones too, via an explicit `.nav-link.active:hover` — the active
  rule follows in source order and would out-cascade a plain hover) and hold
  `--ink` when active. Dropdown items take the same blue on hover
- Sliding indicator is a 2px `--ink` bar, follows hover, snaps back to the
  active page
- "Resources" is a dropdown → Docs, Blog (Blog href is still `#`)
- Download is the glassiest object on the site — see below

**Gotcha:** `body` no longer carries `height: 100%` on index / models /
community / download. A 100%-capped body caps the sticky containing block
too, and the header scrolls away. `resources.html` keeps it, because
`.page-body` sizes off it and its header never scrolls anyway.

---

## Liquid-glass system — smoked glass

The colour build tinted the pane blue because white-on-white doesn't read.
Same problem, opposite answer: the pane is a faint **graphite** wash with a
hairline dark rim and a bright white inner lip along the top edge.

```
--glass-fill        rgba(11,11,13,0.05)
--glass-fill-hover  rgba(11,11,13,0.08)
--glass-rim         rgba(11,11,13,0.10)
--glass-rim-hover   rgba(11,11,13,0.17)
--glass-blur        blur(18px) saturate(140%)
--glass-lip         inset 0 1px 0 rgba(255,255,255,0.95)
--glass-lift        ambient drop shadow
--glass-lift-hover  ...deeper on hover
```

A few controls have to carry real weight, so they use the same construction
at full density (`--ink-glass`, `rgba(11,11,13,0.90)`) with white text: the
homepage **Download Now**, the active models filter tab, and the model
cards' **Download**.

Applied to: header Download, hero CTA, community social buttons, resources
search field + scroll thumb + sidebar hover states, models filter tabs +
carousel arrows + card buttons.

### The header Download

An **inked** pane (`--ink-glass`) with white text. On a white page glass has
nothing to refract, so the effect comes from the edges. Four layers:

1. A lens gradient across the face — light gathers at the top and falls
   away into the tint at the bottom
2. A lit top lip and a shadowed underside, so the pill has thickness
3. `::after` — a masked rim running bright at the top-left and fading out
   on the far side, like the curve of a real lozenge
4. `::before` — a specular streak parked off the left edge that sweeps
   across on hover (hidden under `prefers-reduced-motion`)

Every highlight in it sits far lower than the pale version's did: white on
near-black carries much further than white on a 5% graphite wash. Community
keeps the pale treatment, since a black pill on a black page has nothing to
sit against.

On hover the pane flows to `rgba(var(--net), 0.95)` blue with white text and
a `--net`-tinted glow — the header pill and the hero **Download Now** both.
`color` is in the transition list because in dark mode the resting pill is
white with dark text, and the text has to fade to white as the pane goes
blue rather than snapping.

Interaction pattern elsewhere is unchanged: hover lifts/brightens, press
squishes to ~0.95 and springs back via `cubic-bezier(0.34, 1.7, 0.5, 1)`.

The community social pills use the same sweeping sheen, but tuned much
softer (24% peak over a wide band, vs 95% over a narrow one on the header
Download). The high value exists on the light pages because a faint sheen
can't fight a pale pane; on black it only needs to suggest.

---

## Notable interactions

- **index** — sections blur/fade in *once* via IntersectionObserver, then
  stay clear (no scroll-linked blur; all damping/snapping was removed).
  Hero is a headline + subline pair: "Meet Étude." at
  `clamp(38px, 5vw, 72px)`, then `.headline-sub` — "LLM on your MacBook:
  run by you, stays with you" — at `clamp(19px, 2.1vw, 30px)`, weight 400,
  `--ink-2`. The long line has to be a subline: at headline size it
  out-measures "Meet Étude." three to one and wraps on anything under
  ~1450px. Under 900px the note ring has no clear band left and retires
  entirely (`display: none` on `.note-field`) — the word still inks blue. The two scroll sections centre their text/image
  pair as a unit with a `clamp()`ed gap rather than pushing them to the
  band's edges, so the copy reads nearer the middle of the screen.
  Placeholder collage cards are greyscale gradients.

  Hovering "Étude" in the headline — or the **Download Now** button — inks
  the word `--net` blue and scatters twelve music notes out around the hero,
  which retract on leave. The two triggers share one note field, so hiding
  waits 130ms: travelling between the word and the button would otherwise
  start a retract in the gap and pop straight back out. The notes
  live in a `.note-field` layer over `.hero-content`, not inside the `<h1>`,
  so they can travel outside the text box without touching its layout. Each
  carries its own `--dx/--dy/--o/--s/--r/--w` and a stagger; leaving unwinds
  the stagger in reverse so they retract in the order they appeared. Travel
  is on the wrapper and the idle bob is on an inner span, because a
  `transition` and an `animation` can't both drive `transform`. `--spread`
  pulls the whole scatter inward at five breakpoints, since the outermost
  notes sit ~640px either side of centre.

  Three things worth not undoing:

  1. **Out and back use different curves.** `.notes-on .note` carries the
     spring; `.note` carries a short accelerating ease with no bounce. One
     spring for both means the notes overshoot *through* the centre on the
     way home, which is what made leaving feel clunky.
  2. **The idle bob runs unconditionally**, not under `.notes-on`. Gated on
     the class it snapped to zero the instant the class dropped, yanking
     every note sideways mid-retract.
  3. **Positions ring the text rather than crossing it.** Everything sits
     above the headline or below the subtitle, with two out past the ends of
     the long second line. `overlap.js` in the working set measures note
     rects against the two headline lines, the CTA and the subtitle if you
     move them.

  Full retract is ~540ms, down from ~970ms.
- **resources** — sidebar links behave like the models filter tabs: glass
  pane on hover, squish-and-spring on press, and the selected one holds the
  same dark pill. Selection is set on click *and* kept in sync by a small
  scrollspy on `.content-area` — without the scroll half the pill would sit
  on whatever you last clicked long after you'd read past it. Clicks lock
  the spy for 700ms so it doesn't fight the smooth scroll. (The
  `.sidebar-subitem.active` rule existed before this but nothing ever
  applied the class.)

  Sidebar search filters the nav live; the dot-scroller lens is draggable and
  squashes when it hits either end.

  The lens no longer maps raw scroll progress onto the track. It's driven by
  a list of (scroll position, lens position) pairs, one per visible sidebar
  row, so it comes to rest beside the row you're actually reading and glides
  between rows in between. Anchors use the position a sidebar click lands on
  — the heading at the top of the scrollport — so clicking a row puts the
  lens next to it within a pixel. They're clamped to the reachable scroll
  range, because the last few headings sit past where the content can scroll
  to and the lens would otherwise stall short of them. Dragging inverts the
  same mapping, and the list is rebuilt on resize and whenever the search
  filter hides rows (`etude:sidebar-filtered`). The thumb is a lens: dots
  blur and smear as it passes over them but stay legible. It carries two
  blurs — 2.6px on the element itself, plus a heavier 5px on `::before`
  masked to a ring, so distortion is strongest at the rim (where real glass
  bends most) and eases toward the centre. Both step up on hover.

  Two things to know if you retune it. The shared `--glass-blur` is 18px,
  which is wider than the spacing between dots and flattens the track into
  grey — anything past ~3px does the same. And do **not** reach for
  `contrast()` or `brightness()` to rescue the dots: the backdrop is
  near-white and the dots are light grey, so both sit on the same side of
  the pivot and any boost washes them out further. The dots were darkened at
  source instead (`rgba(11,11,13,0.34)`, was `0.28`) so they survive the blur.
- **community** — full-page canvas: grid of dots, lines fan out to the
  cursor; effect dims over the headline so text stays readable, and the
  cursor blooms into a glowing orb there. The grid is neutral at rest
  (`--dot`); the lines, the dots the cursor reaches, and the bloom all run
  in `--net` blue. Both colours are read from CSS at runtime and
  re-read on `etude:theme`.
  Headline letters nudge away from the pointer. The block sits at 55% of the
  offset that would centre it (`margin-left: clamp(0px, calc((100vw - 1120px)
  * 0.275), 240px)`) — nudged in from the margin without becoming a centred
  layout, tracking the viewport and collapsing to 0 below 1120px. How much
  web survives over
  the text is one constant, `DAMP_MIN` (currently `0.44`) — lower it to
  diffuse harder, raise it to keep more of the web. The orb's strength is
  normalised against that floor rather than tied to `1 - damp`, so changing
  `DAMP_MIN` no longer dims the orb as a side effect. Canvas alphas were retuned,
  not just recoloured: white-on-black at a given alpha reads far dimmer than
  black-on-white at the same value.

  The connecting lines are deliberately diffuse. Two things do that work:
  each line is stroked with its own gradient running bright at the cursor to
  nearly nothing at the dot, so it reads as haze rather than a drawn spoke;
  and the distance falloff is `pow(t, 1.15)` rather than the old clamped
  `t * 1.45`, which sat at full strength across the inner third of the reach
  and made the web look like rays. Peak alpha `0.40`, width `0.55–1.0px`.
- **models** — four models, four rows (Llama 3.1, DeepSeek-V2, Qwen2-VL,
  Mistral 7B — one per former category, named in the meta line). The
  horizontal carousel, its arrows, the snap scrolling and the category
  filter tabs are gone; the page scrolls vertically like any other. Rows
  fade up with a 70ms stagger on load. Card art still distinguishes the
  categories by *value*: general charcoal, code mid-grey, vision light
  silver, compact near-black. On hover a row's `border-color` must go
  *darker* than the resting `--line`, not lighter — a white card on a white
  page has nothing but its outline holding the edge.

### Text editor (index only)

**The editor is a workbench tool, not part of the site.** It only wires up
when the page is opened from disk (`file:`) or a local server (localhost /
127.x), or when `?edit` is added to the URL as an escape hatch; anywhere
else — github.io included — the pencil, grip and toolbar are removed from
the DOM before anything runs. So the deployed site ships the editor code
but never shows it, and `https://…/index.html?edit` turns it on in a pinch.

The pencil above the theme toggle turns on edit mode. Seven text blocks are
marked with `data-edit` ids — the hero title, hero subline, button caption,
and both scroll sections' titles and subtitles. Click one: it becomes
`contenteditable` in place, drags by the blue grip above it, and the glass
toolbar at the bottom sets size, colour (four token swatches + a custom
picker), position (arrow nudges, Shift for 10px), and a Style group
(bold / italic / underline / clear).

Colour and Style are selection-aware: highlight a run of text inside the
box and they act on those words only (the toolbar label shows
"(selection)"); with nothing highlighted they act on the whole box. Word
styling rides on `document.execCommand` with `styleWithCSS`, with three
supporting tricks: toolbar `mousedown` is prevented so clicking a control
doesn't collapse the highlight; the box flips from `plaintext-only` to
rich `contenteditable` just for the instant a command runs (plaintext-only
silently disables every formatting command); and token colours like
`var(--ink)` — which execCommand rejects — go on via a sentinel colour
that's swapped for the token afterwards, so word colours stay theme-aware.
Word spans persist through the same saved-innerHTML path as text edits,
and "Clear" (`removeFormat`) strips them from a selection. Links don't navigate
while edit mode is on, and the hero note burst is suppressed.

Edits live in `localStorage` under `etude-edits`, **this browser only** —
visitors to the deployed site never see them. The workflow is: tweak, hit
**Copy changes** (the full edit set as JSON goes on the clipboard), paste it
back to have the changes baked into the file. "Reset this" / "Reset all"
restore the originals, which are captured at load before saved edits apply.

Two rules the implementation depends on:

1. The toolbar's visibility follows **edit mode**, not the selection —
   otherwise the global actions (Copy / Reset all / Done) go unreachable the
   moment nothing is selected, and Reset All strands you in edit mode.
2. The hero note triggers are **delegated** (`mouseover`/`mouseout` on
   `document` matching `#heroSpark, .cta-button`), not bound to the nodes:
   resetting an edited headline rebuilds its innerHTML, which destroys and
   recreates the `#heroSpark` span, and direct listeners die with the old
   node.

Setting a size overrides the responsive `clamp()` with a fixed px value, and
a custom hex colour ignores the theme (the swatches use tokens and follow
it). Retyping the whole first line removes the `#heroSpark` span, and the
note burst with it, until reset.

### No annotations

The hand-drawn SVG marks are all gone — the double underline under "Étude",
the circle around "fastest LLM inference", the wave under "Mac", and the
wave under "Unmatched latency and throughput". The whole `.hl` family
(`.hl`, `.hl-wave`, `.hl-text`, `.hl-link`, `.hl path`) is removed; the hero
headline is now plain text with no link in it.

---

## Design-tweaks panel — removed

The floating gear panel is not in this build. It injected brand blue on
every page and was reported broken; on inspection the script ran fine, but
`buildCSS()` used one selector list assembled from all five pages, so on any
page other than models the tint / radius / shadow sliders matched nothing.
Its always-on `filter:` rule was also flattening the `backdrop-filter` glass
on `.topbar` and `.cta-button`, since a filtered ancestor becomes a new
backdrop root.

It's still in the colour build if you want it back.

---

## Archived / parked

- `archive_glittery/glittery-dot-orb.html` — "glittery design": rotating
  ~1100-dot sphere that parts around the cursor. Removed from community but
  kept for reuse. **Still styled for the colour build.**
- `archive_community/stay-connected-section.html` — earlier "Stay Connected"
  heading + email signup + social icons. **Still styled for the colour
  build.**

## Open items

- `download.html` has no content
- Blog link, and the social buttons' hrefs (GitHub/Discord/X), are `#`
- Email button is a `mailto:` placeholder
- Resources page content is all lorem-style placeholder
- Model card links/descriptions are placeholder
- Layout below ~1000px is inherited from the colour build and not tuned —
  the homepage scroll sections overflow horizontally at phone widths
