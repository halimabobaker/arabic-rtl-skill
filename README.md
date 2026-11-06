# arabic-rtl-skill

A skill that teaches coding agents (Claude Code, Cursor, Codex, Copilot, …) to build Arabic and RTL interfaces correctly — the first time.

<div dir="rtl">

مهارة تُمكّن أدوات البرمجة بالذكاء الاصطناعي من بناء واجهات عربية سليمة من المحاولة الأولى: اتجاه صحيح، وخطوط ملائمة، وأرقام وتواريخ وعملات بالصيغة الصحيحة، ونصوص عربية طبيعية، وبحث عربي فعّال.

</div>

## The problem

Ask any AI agent for an Arabic screen and you get:

- `padding-left` / `EdgeInsets.only(left:)` everywhere, so the layout breaks in RTL
- back arrows pointing the wrong way, media controls mirrored that shouldn't be
- phone numbers and emails scrambled inside Arabic text
- `letter-spacing` and italics that break Arabic letter connections
- `count == 1 ? 'ملف' : 'ملفات'` — Arabic has **six** plural forms
- prices with 2 decimals for currencies that use 3 (LYD, KWD, TND, …)
- validation that rejects `٠٩١٢٣٤٥٦٧٨` because the user typed Arabic-Indic digits
- search that can't find «احمد» when the data says «أحمد»
- copy that reads like a literal machine translation

This skill gives the agent the rules, the platform-specific APIs, and a review checklist so it gets these right without being told.

## Before / after

**Flutter** — what agents usually write:

```dart
Padding(
  padding: const EdgeInsets.only(left: 16),
  child: Row(children: [
    SvgPicture.asset('assets/arrow.svg'),
    Text(
      count == 1 ? 'ملف واحد' : '$count ملفات',
      textAlign: TextAlign.left,
      style: const TextStyle(letterSpacing: 1.2),
    ),
  ]),
)

TextField(keyboardType: TextInputType.phone)
```

With the skill:

```dart
Padding(
  padding: const EdgeInsetsDirectional.only(start: 16),
  child: Row(children: [
    Transform.flip(
      flipX: Directionality.of(context) == TextDirection.rtl,
      child: SvgPicture.asset('assets/arrow.svg'),
    ),
    Text(l10n.filesCount(count), textAlign: TextAlign.start),
  ]),
)

TextField(
  keyboardType: TextInputType.phone,
  textDirection: TextDirection.ltr,
  inputFormatters: [WesternDigitsFormatter()],
)
```

Padding and alignment follow the reading direction, the arrow points the right way, all six Arabic plural forms come from the ARB file, letter connections stay intact, and the phone field stays LTR and accepts `٠٩١…`.

**CSS** — before → after:

```css
.card  { padding-left: 16px; text-align: left; letter-spacing: .05em; text-transform: uppercase; }
.badge { position: absolute; right: 8px; }
```

```css
.card  { padding-inline-start: 16px; text-align: start; letter-spacing: .05em; text-transform: uppercase; }
.badge { position: absolute; inset-inline-end: 8px; }
:lang(ar) .card { letter-spacing: 0; text-transform: none; line-height: 1.7; }
```

## What's covered

| File | Contents |
|---|---|
| [`SKILL.md`](arabic-rtl/SKILL.md) | 11 core rules + review checklist |
| [`flutter.md`](arabic-rtl/references/flutter.md) | Directional widgets, icons, gestures, inputs, ARB plurals, testing |
| [`web.md`](arabic-rtl/references/web.md) | `dir`, CSS logical properties, Tailwind, forms, React, component libraries |
| [`native.md`](arabic-rtl/references/native.md) | React Native, Android Views/Compose, UIKit/SwiftUI |
| [`copy.md`](arabic-rtl/references/copy.md) | Writing natural, gender-neutral Arabic UI text; plurals; UI glossary |
| [`numbers-dates.md`](arabic-rtl/references/numbers-dates.md) | Digit systems, input normalization, currency, Hijri, regional month names |
| [`search.md`](arabic-rtl/references/search.md) | Arabic text normalization (JS + Dart), database notes |

The agent loads only `SKILL.md` up front and reads a reference when it touches that platform.

## Install

**Claude Code** — copy the `arabic-rtl` folder into your skills directory:

```bash
git clone https://github.com/halimabobaker/arabic-rtl-skill.git
```

```bash
cp -r arabic-rtl-skill/arabic-rtl ~/.claude/skills/
```

Use `.claude/skills/` inside a project instead to share it with your team through the repo.

**Other agents** — add the contents of `arabic-rtl/SKILL.md` to your agent's rules file (`AGENTS.md`, `.cursor/rules/`, `.github/copilot-instructions.md`) and keep the `references/` folder in the repo so the links resolve.

## Usage

It triggers automatically whenever the work involves Arabic or RTL. You can also ask directly:

- "Review this app for RTL issues"
- "Build the checkout screen in Arabic"
- "Add Arabic search to the products list"

## Contributing

Corrections and additions are welcome — especially regional conventions (digits, month names, currency, wording) for your country. Please include a source or a real-world example with each change.
