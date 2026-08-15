# Design

The site is one hand-maintained `index.html`. There is no build step, no
framework and no CSS file: all styles live in a single `<style>` block, all
scripts in `<script>` blocks at the end of `<body>`. Anything added has to be
inlinable, which rules out CDN stylesheets, sprite sheets and external images.
The only outbound requests are the two Google Fonts links.

Every value below is a token in `:root`. Prefer the token over a literal, and
if you need a value that does not exist yet, add a token rather than a one-off
hex code.

## Fonts

Two families, loaded in one request:

```html
<link href="https://fonts.googleapis.com/css2?family=Newsreader:opsz,wght@6..72,600&family=Manrope:wght@400;500;600&display=swap" rel="stylesheet">
```

| Token | Value | Used for |
| --- | --- | --- |
| `--serif` | Newsreader 600, falling back to Georgia | `h1`, `h2`, `h3`, `h4` and display type only |
| `--sans` | Manrope 400/500/600 | everything else: body, leads, labels, buttons, tables |
| `--mono` | Manrope (an alias, not a monospace) | labels and figures that want the sans at a small size |
| `--code` | `ui-monospace, "SF Mono", Menlo` | literal code in the task spec block, nowhere else |

There is no monospace in the interface. `--mono` is a legacy alias that still
resolves to Manrope, so a rule using it renders identically to `--sans`. Do not
reintroduce a monospace UI font: the spec block at `.cb__spec` is the single
place code-like text is genuinely code.

Headings are always Newsreader 600 with `letter-spacing:-.015em` and
`text-wrap:balance`. Body copy is Manrope 400 at `line-height:1.6`.

**Never set `text-transform:uppercase`.** The site had uppercase eyebrows and
they were removed deliberately. `.eyebrow` explicitly restates
`text-transform:none` so it cannot creep back through inheritance.

## Type scale

Every size is a token. Headings are fluid, body copy is fixed.

| Token | Value |
| --- | --- |
| `--fs-display` | `clamp(2.6rem, 1.9rem + 3.6vw, 4.7rem)` |
| `--fs-h2` | `clamp(1.75rem, 1.35rem + 1.7vw, 2.55rem)` |
| `--fs-h3` | `clamp(1.15rem, 1.05rem + .5vw, 1.4rem)` |
| `--fs-lead` | `clamp(1.1rem, 1.02rem + .4vw, 1.3rem)` |
| `--fs-body` | `1.0625rem` |
| `--fs-body-sm` | `1rem` |
| `--fs-small` | `.9rem` |
| `--fs-data` | `.8rem` |
| `--fs-label` | `.75rem` |
| `--fs-label-sm` | `.65rem` |

Line heights: `h1` 1.06, `h2` 1.14, `h3` 1.3, `.lead` 1.5, body 1.6.

## Colour

The palette is warm off-white with a single teal accent. There is no secondary
accent and no semantic red, amber or green anywhere in the interface.

| Token | Value | Role |
| --- | --- | --- |
| `--bg` | `#FCFAF6` | page ground, a warm vellum |
| `--surface` | `#FEFEFB` | cards sitting on the ground |
| `--surface-2` | `#F7F2E5` | recessed panels, the status bar, the closing call |
| `--line` | `#C5C0B4` | ordinary hairlines and section rules |
| `--line-strong` | `#898477` | grid separators and emphasised borders |
| `--ink` | `#231F19` | body text and headings |
| `--ink-muted` | `#656057` | leads, captions, secondary copy |
| `--ink-faint` | `#716C63` | the quietest supporting text |
| `--accent` | `#005F73` | links, primary buttons, focus rings |
| `--accent-strong` | `#003A46` | hover state for anything accent-coloured |
| `--accent-contrast` | `#F3F9FA` | text on an accent fill |

Selection is `color-mix(in srgb, var(--accent) 24%, transparent)`.

### The hero has its own palette

The hero is the one place the page speaks rather than lists, so it gets a
separate ground and its own foreground set. These tokens exist only for it and
should not be borrowed elsewhere.

| Token | Value |
| --- | --- |
| `--hero-grad` | `linear-gradient(158deg, #17475A 0%, #0E2A33 52%, #0A1F27 100%)` |
| `--hero-bg` | `#0E2A33` (the flat fallback under the gradient) |
| `--hero-ink` | `#F3EFE6` |
| `--hero-lead` | `#9AB8BF` |
| `--hero-link` | `#7FD3E8` |
| `--hero-btn` | `#33ABC4` |
| `--hero-btn-ink` | `#0E1B1F` |
| `--hero-rule` | `#6C9BA8` |

The gradient is lit from the upper left. Every stop was checked against all
four things that sit on it; the lightest stop is the binding one, at 4.79:1 on
the lead. If you change any stop, re-check all four, and measure the rendered
pixels rather than the declared stops, because `background-blend-mode` and
overlapping fills change what the text actually sits on.

## Light only

The site ships light. There is no dark theme, no toggle and no
`prefers-color-scheme` block; they were removed in `93b7847`.

`:root` declares `color-scheme:light`. That line is load-bearing: without it a
dark OS preference repaints form controls, scrollbars and the canvas even
though every rule is gone. Keep it.

If dark mode is ever reintroduced, note the trap that caused a real bug: a bare
`:root` inside `@media (prefers-color-scheme:dark)` outranks `:root`, so any
light block that does not restate *every* token will leak dark values.

## Layout and spacing

| Token | Value | Role |
| --- | --- | --- |
| `--maxw` | `1080px` | content column, applied by `.wrap` |
| `--gutter` | `clamp(18px, 4vw, 40px)` | horizontal page padding |
| `--measure` | `31em` | reading measure, roughly 70 characters |
| `--space-fig` | `clamp(1.4rem, 2.6vw, 2.1rem)` | space around figures |
| `--col-gutter` | `2.5rem` | gap between text columns |
| `--col-break` | `25rem` | column break threshold |

`.wrap` centres content at `--maxw` with `--gutter` either side. `.band` is the
section unit: `padding-block: clamp(52px, 6.2vw, 96px)` with a `1px solid
var(--line)` top rule. The status bar and `#problem` suppress the following
band's top border so rules never double up.

Breakpoints are ad hoc rather than a fixed set, chosen per component where the
layout actually breaks. The common ones are 660px, 760px and 820px.
`scroll-padding-top` is `72px` to clear the sticky header.

### The measure trap

`--measure` is in `em`, so it resolves against the *parent's* font size, not the
child's. Putting `max-width: var(--measure)` on a wrapper silently clips any
child with a larger font size. This caused two real bugs, a heading stuck at
five lines and a lead stuck at three. That is why `.prose` is `max-width:none`
and the measure is applied to `.prose > p` instead.

Apply the measure to paragraphs, never to a container that holds headings.

## Corners

Radii are small and deliberate. Do not introduce new values.

| Radius | Used for |
| --- | --- |
| `6px` | buttons, cards, the task-spec block, the results table, panels |
| `5px` | figures and artwork frames nested inside a 6px parent |
| `4px` | recessed panels and the dashed placeholder |
| `2px` | the focus ring |
| `999px` | pills, currently the replay and resample controls |
| `50%` | dots, currently the brand mark and the stage markers |

The 6px / 5px pairing is intentional: a frame nested inside a rounded parent
takes one step less so the curves nest rather than fight.

## Borders and surfaces

Hairlines are `1px solid var(--line)`. Emphasised borders use
`var(--line-strong)`. Grids that need visible separators draw them as a `1px`
gap over a `--line-strong` background rather than as per-cell borders, which is
how `.statusbar .wrap` works.

There are no drop shadows anywhere. Elevation is expressed with surface colour
and a hairline, not with shadow.

## Buttons

```css
.btn{
  font-family:var(--sans); font-weight:600; font-size:.9rem;
  border:1px solid transparent; border-radius:6px;
  padding:.6rem 1.05rem;
  display:inline-flex; align-items:center; gap:.5rem; white-space:nowrap;
  transition:background-color .18s, border-color .18s, color .18s, transform .12s;
}
```

| Variant | Rest | Hover |
| --- | --- | --- |
| `.btn-primary` | `--accent` fill, `--accent-contrast` text | fill and border go to `--accent-strong` |
| `.btn-ghost` | transparent, `--ink` text, `--ink-muted` border | border and text go to `--accent` |

| Size | Padding | Font size |
| --- | --- | --- |
| `.btn-sm` | `.45rem .85rem` | `.82rem` |
| default | `.6rem 1.05rem` | `.9rem` |
| `.btn-lg` | `.8rem 1.4rem` | `1rem` |

Every button carries a `1px` border, including the primary, so swapping variants
never changes the box size. Active state is `translateY(1px)`, shared with the
other pressable controls as `translateY(1px) scale(.99)`.

## Links and focus

Links are `--accent` with **no underline by default**. Underlines are added only
where a link sits inside running prose: `.lead a`, `.faq details p a` and
`.worked a`, at `text-underline-offset:.18em` and
`text-decoration-thickness:.06em`. Hover goes to `--accent-strong`.

Focus is a global `:focus-visible` of `2px solid var(--accent)` at
`outline-offset:2px` with a `2px` radius. Two deliberate exceptions: primary
buttons switch the outline to `--ink` so the ring is visible against the accent
fill, and stage cards use a negative offset so the ring sits inside the card
edge.

Never remove a focus ring without replacing it with something equally visible.

## Motion

One easing token, `--ease: cubic-bezier(.22, 1, .36, 1)`, and one duration for
interface transitions, `.18s`. Transforms use `.12s`. Longer timings belong to
the arm animation only.

Every animation must have a `prefers-reduced-motion: reduce` branch. There are
eight such blocks; match the pattern when you add a ninth. The `.reveal` class
is the scroll-in mechanism and its base state is `opacity:1`, so a JavaScript
failure can never leave the page invisible.

## Print

There is a real print stylesheet and it is worth keeping working. It flattens
the palette to white with near-black ink, force-opens every `<details>` and
every stage panel, hides interactive chrome (header, nav, replay and resample
controls, stage cards), disables reveal animations, and sets `break-inside:
avoid` on sections, cards and tables with `break-after: avoid` on headings.

If you add a collapsible or an interactive control, add it to the print rules.

## Accessibility contract

- Text meets **4.5:1**. Non-text, meaning icons, borders and focus rings, meets
  **3:1**. This is checked, not assumed.
- Measure contrast against the pixels that actually render. Gradients, blend
  modes and translucent fills all move the effective background.
- Every interactive element is reachable and operable by keyboard, and the
  `<details>` disclosures stay native rather than being reimplemented.
- The page works with JavaScript disabled: panels and disclosures fall back to
  open, and `.reveal` defaults to visible.
- The skip link is the first focusable element.
- Decorative SVG is `aria-hidden="true"` and `focusable="false"`; meaningful SVG
  carries a `<title>`.

## Writing style

The prose conventions are part of the design and reviewers do enforce them.

- No uppercase. Not in eyebrows, not in labels, not in buttons.
- No em dashes. Use a comma, a colon, a semicolon or a full stop.
- No leading-zero numbers. Write `1`, `2`, `3`, never `01`, `02`, `03`.
- Sentence case for headings and buttons.
- British spelling in prose, US spelling where a proper noun or a technical term
  demands it.
- Numbers that carry a claim must be true and current. `up to $500,000` is a
  ceiling and is fine; `a $500,000 open call` asserts possession and is not.

## Things that have bitten us

Read this before a large CSS change.

1. **`em` units resolve at the parent.** See the measure trap above.
2. **Equal specificity means source order decides.** `.faq-list--pay b` and
   `.faq-list b` are both (0,2,1), so the later rule won and rendered currency
   amounts at 10.4px. If two rules tie, order them deliberately and leave a
   comment saying why.
3. **`.sd__checks li` is a flex container**, so a bare `<b>` inside becomes its
   own flex item and breaks the bullet. Wrap inline emphasis in a `<span>` or
   use a grid with explicit columns.
4. **The arm artwork does not follow the palette.** Its colours are local
   variables inside the SVG's own `<style>` block because it is a physical
   object, not a diagram. Its contrast is with itself, not with the page. Leave
   `--b`, `--l`, `--h`, `--i`, `--m`, `--a`, `--k` and `--w` alone.
5. **Screenshot before and after.** The page is long, hand-maintained and has no
   tests, so a rendered diff is the only real check.
