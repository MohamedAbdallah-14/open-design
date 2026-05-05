# Accessibility baseline craft rules

Universal rules for the legal floor of accessibility plus the craft
commitments that go beyond it. The active `DESIGN.md` decides brand
appearance; this file decides which rules an artifact has to clear
before it ships.

> Grounded in primary sources: WCAG 2.2 Understanding pages,
> ISO/IEC 40500:2025, ADA Title II 2024 + 2026 IFR, EN 301 549 v3.2.1,
> WAI-ARIA 1.3 + AccName 1.2 + Core AAM 1.2, WebAIM Million 2025,
> A11yn (arXiv 2510.13914), APCA W3C silver branch.

## The legal floor changes by jurisdiction

- **EU (EAA, enforcement live 2025-06-28):** EN 301 549 v3.2.1 is the OJ-cited harmonised standard; it references **WCAG 2.1 AA**. EN 301 549 v4.1.1 (which incorporates WCAG 2.2's nine new SCs) is OJ-citation-targeted late 2026 / 2027. Until then, EAA references WCAG 2.1.
- **US public sector (ADA Title II 2024 final rule):** **WCAG 2.1 AA**. The 2026-04-20 IFR slipped deadlines: 2027-04-26 for jurisdictions with population ≥ 50,000; 2028-04-26 for sub-50,000 and special districts.
- **ISO/IEC 40500:2025** (October 2025) ratified WCAG 2.2 verbatim. Does not by itself change EU or US legal floors.

**Practical rule for craft:** target **WCAG 2.2 AA** as the working
ceiling. It clears the WCAG 2.1 AA legal floor in both jurisdictions
and prepares for v4.1.1. Anything below 2.2 AA is craft debt.

## Color contrast

| Pair | WCAG 2.x AA minimum |
|---|---|
| Body text (≤16 px regular, ≤14 px bold) on background | 4.5:1 |
| Large text (≥18 *pt* regular ≈24 px, or ≥14 *pt* bold ≈18.5 px) | 3:1 |
| Non-text UI components and graphical objects | 3:1 |
| Focus indicator vs adjacent and unfocused state | 3:1 |

Threshold is **exclusive**. 2.999:1 fails. Don't round.

"Large text" means **18 pt** regular, not 18 px. 18 px regular doesn't
qualify; 14 pt bold does (≈18.5 px), 14 px bold doesn't.

**APCA as a parallel design check.** APCA's Lc value catches font-weight
effects WCAG 2.x luminance ratios miss. Use Lc ≥60 for body as a
parallel design pass. APCA is not normative anywhere as of 2026-05.
Keep WCAG 2.2 AA as the compliance floor. If you ship APCA tooling,
use the `apca-w3` package; the SAPC repo is non-commercial.

## Touch targets

| Bar | SC | Size |
|---|---|---|
| AA (legal floor) | 2.5.8 Target Size (Minimum) | **24×24 CSS px** |
| AAA (craft commitment) | 2.5.5 Target Size (Enhanced) | 44×44 CSS px |
| iOS HIG | — | 44×44 pt |
| Material 3 | — | 48×48 dp |

The AA Spacing exception: undersized targets pass if a 24-px exclusion
circle around each does not intersect adjacent ones. The 24×24 AA bar
is the rule that icon-button toolbars violate silently.

## Focus visibility

Removing the focus outline via CSS is a **triple failure**: 1.4.11
Non-text Contrast, 2.4.7 Focus Visible, and 2.4.13 Focus Appearance
(AAA). Use `:focus-visible` for keyboard users; suppress the outline
for mouse clicks only when an alternative non-color affordance exists.

For AAA (2.4.13): indicator area must equal at least a 2 CSS px
perimeter of the component, contrast ≥3:1 between focused and
unfocused states. A 1-px outline at 3:1 doesn't qualify.

## Form input labels

WebAIM Million 2025 (which uses WAVE, not axe-core): **48.2% of top 1M
home pages have at least one missing form-input label; 34.2% of all
6.3M inputs are unlabeled**. Both rates are flat year-over-year while
overall errors per page are dropping. Form labelling is one of the
two categories trending *worse*.

Default form-error wiring (WCAG 2.2 + ARIA APG):

```html
<label for="email">Email</label>
<input id="email" type="email" required
       aria-describedby="email-hint email-error"
       aria-invalid="true">
<span id="email-hint">Used for receipts only.</span>
<span id="email-error" role="alert">Email must include @ and a domain.</span>
```

`aria-describedby` is the production default; `aria-errormessage` has
incomplete screen-reader support as of 2026-05 (full on NVDA, partial
on JAWS / VoiceOver / TalkBack) — treat as progressive enhancement.

WCAG 3.3.7 Redundant Entry is **Level A** (legal floor). Re-asking for
data the user already entered "in the same process" fails unless the
site auto-populates or offers a selectable shortcut. Browser autofill
does not satisfy it.

## ARIA discipline

WebAIM Million 2025 shows ARIA pages average **57 errors** vs **27**
on non-ARIA pages — gap doubled year-over-year. ARIA deployment is
outpacing correctness.

Decision order, per ARIA APG:

1. Native HTML element with the right semantics.
2. Native element under custom visuals if restyling is required.
3. APG pattern verbatim if neither fits.
4. Closest APG pattern + documented deviation. Last resort.

Never invent ARIA.

## Reduced motion and flashing

See `animation-discipline.md` for the full rule set. The non-negotiable
that anchors here: WCAG 2.3.1 (Level A) — flashing more than three
times per one-second period is non-conformant unless the flash area
stays below the general and red flash thresholds. Photosensitive
epilepsy is the protected concern.

## Native mobile parity

Web ARIA does not auto-translate. Each platform has its own labelling API.

| Platform | Label | Role |
|---|---|---|
| iOS UIKit | `accessibilityLabel` | `accessibilityTraits` |
| iOS SwiftUI | `.accessibilityLabel(…)` | `.accessibilityAddTraits(.isButton)` |
| Android Compose | `Modifier.semantics { contentDescription = … }` | `Modifier.semantics { role = Role.Button }` |
| Flutter | `Semantics(label: …)` | `Semantics(button: true, …)` |
| React Native | `accessibilityLabel` | `accessibilityRole` |

Use the platform API for each target. AI-generated mobile UI that
mirrors web ARIA verbatim usually misses the platform-native screen
reader path.

## Common mistakes (lint these)

- "Target Size 44×44" cited as the AA bar. 44×44 is **AAA** (2.5.5). AA is **24×24** (2.5.8).
- "18 px = large text" — wrong. Threshold is 18 *pt* regular (~24 px) or 14 pt bold (~18.5 px).
- "EAA = WCAG 2.2 AA" — wrong. EN 301 549 v3.2.1 is anchored to WCAG 2.1.
- "WebAIM Million uses axe-core" — uses WAVE.
- "WCAG 3 will use APCA" — APCA was dropped from WCAG 3 in July 2023.
- "Adding ARIA improves accessibility" — empirically the opposite. WebAIM Million 2025: ARIA pages average 2× the errors.
- Removing the focus outline via `outline: none` without a replacement. Triple failure: 1.4.11, 2.4.7, 2.4.13.
- Placeholder text as the only label for a form input. Fails 1.3.1 and 3.3.2; placeholder disappears on input.
- Using `aria-description` as the sole state-carrier on `role="row"`. JAWS 2025/2026 silently drops it ([FreedomScientific standards-support #927](https://github.com/FreedomScientific/standards-support/issues/927)).
- Native HTML `<button>` reimplemented as `<div role="button">` without keyboard handling, focus, or `aria-pressed`.
- A11y treated as web-only. Flutter / iOS / Android have their own labelling APIs that web ARIA doesn't reach.
