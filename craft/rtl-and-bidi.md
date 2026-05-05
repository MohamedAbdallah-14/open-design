# RTL and bidirectional craft rules

Universal rules for right-to-left layout and bidirectional text. The
active `DESIGN.md` decides brand visual language; this file decides
how that language behaves when the script reads from the right or
mixes direction within a line.

> Grounded in primary sources: Unicode UAX #9 revision 51 (Sept 2025)
> + Unicode 17.0, CSS Logical Properties Level 1, Tailwind v4.0/v4.2
> changelogs, W3C alreq, Material 3 RTL guidance, Apple HIG
> internationalization, WebKit Bugzilla 50949.

## Logical properties first

Hardcoded `left` / `right` is a bug for any layout that might render
RTL. Use logical properties on the inline axis. Use them on the block
axis when the writing-mode varies; physical otherwise.

| Logical | LTR resolves to | RTL resolves to |
|---|---|---|
| `margin-inline-start` / `padding-inline-start` / `inset-inline-start` | left | right |
| `margin-inline-end` / `padding-inline-end` / `inset-inline-end` | right | left |
| `border-inline-start` | border-left | border-right |
| `border-start-start-radius` | border-top-left-radius | border-top-right-radius |
| `text-align: start` / `text-align: end` | left / right | right / left |
| `inline-size` / `block-size` | width / height | width / height |

Browser support: core inline-axis logical properties are Baseline
Widely Available (Chrome 87, Safari 14.1, Firefox 66; ≥95% global as
of 2026-05).

**Tailwind v4 changes the answer for new projects.** v4.0 (2025-01-22)
folded inline-axis logical utilities into core (`ms-*`, `me-*`, `ps-*`,
`pe-*`, `start-*`, `end-*`). v4.2 (2026-02-18) added the block-axis
set (`mbs-*`, `mbe-*`, `pbs-*`, `pbe-*`) and renamed the inset
utilities: `start-*` / `end-*` are deprecated (still work) in favor
of `inset-s-*` / `inset-e-*`. The `tailwindcss-rtl` plugin is obsolete.
Don't write `[dir="rtl"]:` overrides for spacing on Tailwind v4.

## Bidirectional text

UAX #9 rev 51 (Sept 2025) is a version stamp for Unicode 17.0. No
algorithm change; `max_depth = 125` is permanently locked forward.

The five isolate-and-formatting characters: U+2066 LRI, U+2067 RLI,
U+2068 FSI, U+2069 PDI, U+202C PDF.

**Use `<bdi>` in HTML and the U+2068 / U+2069 control pair for plain
text.** UAX #9 §2.7: *"where available, markup should be used instead
of the explicit formatting characters."* `<bdi>` has been Baseline
Widely Available since January 2020.

`dir="auto"` lets the browser detect direction from the first strong
directional character — useful for user-generated content where the
direction isn't known up front.

U+2066-U+2069 are interoperable across modern Chrome, Firefox, and
Safari. The remaining gap is the CSS form `unicode-bidi: plaintext`
([WebKit Bugzilla #50949](https://bugs.webkit.org/show_bug.cgi?id=50949),
still open as of 2026-05). Don't rely on it as a primary mechanism.

## What mirrors and what doesn't

Mirroring isn't universal. The rules below are unanimous across
Material 3 RTL guidance and Apple HIG internationalization.

**Must mirror:**

- Directional arrows (back / forward / next / previous), navigation rail position, tab order, slider fill direction, progress-bar fill, calendar-grid weekday order.
- Checkbox-and-label position. Label sits to the right in LTR, to the left in RTL.
- Phone-number and IBAN affordances when the surrounding paragraph is RTL but the value itself is LTR — wrap the value in `<bdi>` so the digits don't reflow.

**Must not mirror:**

- Clock faces. Clockwise is universal.
- Circular refresh / sync / reload icons. Same reason.
- Media playback controls (play / pause / fast-forward / rewind). They represent tape direction, not reading direction.
- Charts and graphs. X-axis stays mathematical, not linguistic.
- Photographs, brand logos, physical-object icons (camera, keyboard, headphones). Identity over direction.

**Numerals are not a mirroring decision.** They follow locale, not
paragraph direction. Arabic-Indic digits carry bidi class **AN**, not
EN — affects how they sit inside mixed-direction lines but does not
flip them.

**Single live conflict between platforms:** the search icon. SF Symbols
ships an RTL `magnifyingglass` variant (Apple flips it). Material 3
says don't flip the magnifying glass (handle stays bottom-right).
Decide per-platform; don't synthesize a single rule.

## Typography rules anchored here

Two RTL-coupled typography rules sit in this file because they cause
breakage at the layout level. The full Arabic / Hebrew typography
guide (font picks, harakat line-height, OpenType shaping, mixed-script
fallback chains) belongs in a future `craft/arabic-hebrew-typography.md`.

- **Never apply CSS `letter-spacing` to Arabic runs.** alreq treats
  letter-spacing as a boundary concept, not a uniform tracking value.
  Applying tracking breaks the cursive joining the script depends on.
- **Body type for Arabic runs ~14-18 px with line-height 1.5-1.75** to
  give harakat (diacritics) clearance. Latin defaults are too tight.

## Native mobile RTL parity

Web RTL handling does not auto-translate to mobile. Each platform has
its own direction primitive.

| Platform | Direction primitive | Spacing |
|---|---|---|
| iOS UIKit | `semanticContentAttribute = .forceRightToLeft` | `NSDirectionalEdgeInsets` |
| iOS SwiftUI | `.environment(\.layoutDirection, .rightToLeft)` | `EdgeInsets` with `leading` / `trailing` |
| Android Compose | `CompositionLocalProvider(LocalLayoutDirection provides LayoutDirection.Rtl)` | `PaddingValues` accepts start / end |
| Flutter | `Directionality(textDirection: TextDirection.rtl)` | `EdgeInsetsDirectional.fromSTEB(...)` |
| React Native | `I18nManager.forceRTL(true)` (requires native reload; no `forceLTR` parity, no `react-native-web` support) | `marginStart` / `marginEnd` |

The rule across all platforms: prefer the directional primitive over
the absolute one. `EdgeInsets.left/right` in Flutter, `paddingLeft` /
`paddingRight` in Android, leading-vs-trailing in iOS — these are bugs
waiting for an Arabic deployment.

## Forms in RTL

Form fields commonly mix scripts. Three rules cover most of it.

- **`<input dir="auto">`** for any field whose value's direction is uncertain (search boxes, comment fields, free-text inputs). The browser detects from the first strong directional character.
- **Force LTR on intrinsically-LTR fields** even inside an RTL paragraph: email, URL, phone, IBAN, credit-card. `<input type="email" dir="ltr">`.
- **Wrap rendered values in `<bdi>`** when displaying mixed-script content (a username inside a paragraph, a model number inside a description). Stops the surrounding direction from rearranging the embedded value.

## Common mistakes (lint these)

- Hardcoded `left` / `right` / `text-align: left` in new CSS. Every hardcoded directional property is a bug in RTL.
- "Use `text-justify: kashida` for Arabic" — no browser implements it. The standard CSS `justify` form adds inter-word spacing and looks unnatural; kashida is the right form, but you can't ship it on the web.
- "Tailwind v4.2 logical-utility rename is `inline-s-*` / `inline-e-*`" — wrong family. Those are size utilities. The inset rename is `inset-s-*` / `inset-e-*`.
- "WebKit doesn't support U+2066-U+2069" — wrong, they're interoperable across modern browsers. The "still missing" claim traces to a stale 2015 W3C test snapshot.
- "Use `unicode-bidi: plaintext`" — broken in WebKit ([Bugzilla #50949](https://bugs.webkit.org/show_bug.cgi?id=50949)). Use `<bdi>` for HTML and U+2068 / U+2069 for plain text.
- Italics on Arabic or Hebrew text. Neither script has an italic tradition.
- CSS `letter-spacing` applied to Arabic. Breaks cursive joining.
- Lorem Ipsum used for RTL prototyping. Arabic word lengths, connection behaviors, and vertical extents differ; use real Arabic / Hebrew text.
- Flutter `EdgeInsets.left/right` in code that needs to render RTL. Use `EdgeInsetsDirectional.start/end`.
