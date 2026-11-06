# Numbers, currency, dates

## Digit systems

| System | Digits | Typical use |
|---|---|---|
| Western (latn) | 0 1 2 3 4 5 6 7 8 9 | Libya, Tunisia, Algeria, Morocco; many modern apps everywhere |
| Arabic-Indic (arab) | ٠ ١ ٢ ٣ ٤ ٥ ٦ ٧ ٨ ٩ | Egypt, Levant, Iraq, Gulf (common, not universal) |
| Extended Arabic-Indic (arabext) | ۰ ۱ ۲ ۳ ۴ ۵ ۶ ۷ ۸ ۹ | Persian/Urdu keyboards — Arabic users still type them |

Pick one per product (ask if unclear, default to the target country's convention) and never mix systems on one screen.

Locale defaults differ by region. With current ICU (Node 22):

```
ar          → 1,234,567.891
ar-EG/ar-SA → ١٬٢٣٤٬٥٦٧٫٨٩١
ar-LY/ar-TN/ar-MA → 1.234.567,891   ← dot for thousands, comma for decimals
ar-u-nu-latn → 1,234,567.891         ← force Western digits
ar-EG-u-nu-arab → ١٬٢٣٤٬٥٦٧٫٨٩١       ← force Arabic-Indic digits
```

Force the digit system explicitly with the `-u-nu-latn` / `-u-nu-arab` extension instead of relying on defaults. Other libraries and platforms (Dart `intl`, Android, iOS) can resolve the same locale differently — check real output.

Formatted Arabic output often contains invisible bidi marks (U+200F RLM, U+061C ALM). Do not compare formatted strings with `==` in tests or parse them back; compare numbers, or strip `[‎‏؜]` first.

## Normalizing input

Every numeric input (phone, OTP, amount, national ID, quantity) must convert Eastern digits before validation or parsing:

```js
export function toWesternDigits(s) {
  return s
    .replace(/[٠-٩]/g, (d) => String(d.charCodeAt(0) - 0x0660))
    .replace(/[۰-۹]/g, (d) => String(d.charCodeAt(0) - 0x06F0))
    .replace(/٫/g, '.')   // Arabic decimal separator ٫
    .replace(/٬/g, '');   // Arabic thousands separator ٬
}
```

```dart
String toWesternDigits(String s) => s
    .replaceAllMapped(RegExp('[٠-٩]'), (m) => '${m[0]!.codeUnitAt(0) - 0x0660}')
    .replaceAllMapped(RegExp('[۰-۹]'), (m) => '${m[0]!.codeUnitAt(0) - 0x06F0}')
    .replaceAll('٫', '.')
    .replaceAll('٬', '');
```

Apply on the server too — never trust that the client normalized.

## Currency

- Minor units: LYD, KWD, BHD, OMR, JOD, TND → 3 decimals (1 dinar = 1000 dirham/fils/millime). SAR, AED, EGP, QAR, MAD → 2. IQD → 0 in display.
- Store money as integers in the minor unit (LYD 12.500 → 12500) or as decimal types; never floats.
- Format with the currency formatter so symbol, position, and decimals come from locale data:
  ```js
  new Intl.NumberFormat('ar-LY', { style: 'currency', currency: 'LYD' }).format(12.5) // ‏12,500 د.ل.‏
  ```
- Amount in words (تفقيط) for invoices/cheques must follow Arabic number grammar (gender agreement, dual, 3–10 plural, 11–99 singular accusative). Use a tested library; don't hand-roll it inline.

## Plurals with numbers

Arabic CLDR plural categories (`new Intl.PluralRules('ar').select(n)`):

| Category | n | Noun form | Example (ملف) |
|---|---|---|---|
| zero | 0 | plural/negation | لا توجد ملفات |
| one | 1 | singular | ملف واحد |
| two | 2 | dual | ملفان |
| few | 3–10, 103–110… | plural | ٣ ملفات |
| many | 11–99, 111–199… | singular accusative | ١١ ملفًا |
| other | 100–102, 200… | singular | ١٠٠ ملف |

## Dates and time

- Format with `Intl.DateTimeFormat` / `DateFormat` / platform formatters and the user's locale; never hardcode month names.
- Month names vary by region and the formatter handles it: `ar-TN`/`ar-DZ` → جانفي، فيفري…; Levant/Iraq (`ar-SY`, `ar-LB`, `ar-JO`, `ar-IQ`) → كانون الثاني، شباط…; most others → يناير، فبراير…
- Hijri calendar where expected (Saudi Arabia, religious dates): `new Intl.DateTimeFormat('ar-SA-u-ca-islamic-umalqura', { dateStyle: 'long' })` → ٢٦ رجب ١٤٤٧ هـ. Umm al-Qura is the official Saudi calendar; other Hijri variants differ by a day or two.
- Week start and weekend vary (Saturday or Sunday start; Friday–Saturday weekend in most Arab countries). Take them from locale data (JS: `const l = new Intl.Locale('ar-LY'); (l.getWeekInfo?.() ?? l.weekInfo)` → `{ firstDay: 6, weekend: [5, 6] }`, i.e. Saturday start, Friday–Saturday weekend; Dart `MaterialLocalizations.firstDayOfWeekIndex`) instead of hardcoding Monday.
- Time: use the locale's AM/PM markers (ص / م) via the formatter.
- Relative time: `new Intl.RelativeTimeFormat('ar', { numeric: 'auto' })` → «أمس»، «قبل 3 ساعات», which applies the plural rules correctly.

## Phone numbers

- Store in E.164 (`+218912345678`), display in local format, always LTR-isolated in Arabic text.
- Accept input with Eastern digits, spaces, dashes, leading `00` or `+`, or a leading national `0`; normalize before validating.
