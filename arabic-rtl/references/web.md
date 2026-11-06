# Web: HTML, CSS, Tailwind, React

## Document direction

```html
<html lang="ar" dir="rtl">
```

- Set `dir` with the HTML attribute, not CSS `direction` — the attribute drives bidi, form controls, scrollbars, and `:dir()` selectors.
- For a bilingual app, switch `lang` and `dir` on `<html>` together when the locale changes.
- User-generated content of unknown language: `<p dir="auto">`, and `<bdi>` for inline names/values inside a sentence.

## CSS logical properties

| Avoid | Use |
|---|---|
| `margin-left` / `margin-right` | `margin-inline-start` / `margin-inline-end` |
| `padding-left: 8px; padding-right: 8px` | `padding-inline: 8px` |
| `left: 0` / `right: 0` | `inset-inline-start: 0` / `inset-inline-end: 0` |
| `text-align: left` | `text-align: start` |
| `border-left` | `border-inline-start` |
| `border-top-left-radius` | `border-start-start-radius` |
| `float: left` | `float: inline-start` |

Flexbox and grid already follow `dir`; `justify-content: flex-start` and `grid-column: 1` are direction-safe. Do not use `flex-direction: row-reverse` to "fix" RTL.

Not logical, must be handled manually: `transform: translateX()`, `background-position`, `box-shadow` / `text-shadow` x-offsets, `clip-path`, gradients with `to right`, and JS that reads `clientX` or `scrollLeft`.

```css
[dir="rtl"] .slide-in { --dir: -1; }
.slide-in { transform: translateX(calc(100% * var(--dir, 1))); }

:dir(rtl) .icon-directional { transform: scaleX(-1); }
```

`scrollLeft` is 0 at the start edge and negative toward the end in RTL in current browsers — account for the sign in custom carousels.

## Tailwind

- Use logical utilities: `ms-*`, `me-*`, `ps-*`, `pe-*`, `start-*`, `end-*`, `text-start`, `text-end`, `rounded-s-*`, `rounded-e-*`, `border-s`, `border-e`.
- Use `rtl:` / `ltr:` variants only for what logical utilities can't express: `rtl:-scale-x-100` for directional icons, `rtl:-translate-x-full` for transforms.
- Prefer `gap-*` over `space-x-*` (which needs `rtl:space-x-reverse`).

## Typography

```css
:lang(ar) {
  font-family: "IBM Plex Sans Arabic", "Noto Sans Arabic", system-ui, sans-serif;
  line-height: 1.7;
  letter-spacing: 0;
  font-style: normal;
  text-transform: none;
}
```

- Load the Arabic subset of the font (Google Fonts serves it via `subset=arabic`).
- Override `letter-spacing` and `text-transform: uppercase` coming from design-system defaults (buttons, labels, overlines).
- Arabic text often renders visually smaller at the same `font-size` — bump body size by ~1–2px if the design was tuned for Latin.

## Forms

```html
<input type="tel" dir="ltr" inputmode="numeric" autocomplete="tel">
<input type="email" dir="ltr" autocomplete="email">
```

- Phone, email, URL, password, OTP, IBAN, card: `dir="ltr"`. Keep the label in Arabic.
- Normalize digits before validation (see [numbers-dates.md](numbers-dates.md)); `pattern="[0-9]*"` rejects ٠-٩.
- Placeholder text in an LTR input that is Arabic will look misaligned — use an LTR example value (`09X XXX XXXX`) or put guidance in the label.

## React / JS

- Read direction from the locale, not from a hardcoded boolean: 
  ```js
  const loc = new Intl.Locale(locale);
  const dir = (loc.getTextInfo?.() ?? loc.textInfo)?.direction
    ?? (['ar', 'he', 'fa', 'ur'].includes(loc.language) ? 'rtl' : 'ltr');
  ```
  Engines expose either `getTextInfo()` or the older `textInfo` getter, and some neither — hence both checks and the fallback.
- Component libraries: enable their RTL mode (MUI: `theme.direction = 'rtl'` + `stylis-plugin-rtl`; Ant Design: `ConfigProvider direction="rtl"`; Radix: `DirectionProvider dir="rtl"`).
- Charts: keep numeric labels readable; decide once whether the time axis runs right-to-left and apply it across all charts.
- Mixed strings: `⁦…⁩` (LRI…PDI) around phone numbers/emails inside Arabic template strings, or render them in `<bdi>` / `<span dir="ltr">`.

## Testing

- Toggle `dir="rtl"` on `<html>` and walk every page: navigation, drawers, dropdown alignment, tooltips and popovers (placement `start`/`end`, not `left`/`right`), toasts, carousels, breadcrumbs, pagination arrows.
- Grep for regressions: `margin-left`, `margin-right`, `padding-left`, `padding-right`, `left:`, `right:`, `text-align: left`, `ml-`, `mr-`, `pl-`, `pr-`, `left-`, `right-`, `text-left`, `text-right`, `rounded-l`, `rounded-r`, `border-l`, `border-r`, `tracking-`, `uppercase`.
