# Arabic search

Users type the same word many ways: «أحمد» / «احمد», «مدرسة» / «مدرسه», «مستشفى» / «مستشفي», with or without diacritics, with tatweel («مـــرحبا»), with Eastern or Western digits, or on a Persian keyboard (ی، ک). Exact matching fails on all of these.

Normalize **both** the stored/indexed text and the query with the same function. Never show normalized text to users — keep the original for display.

## Normalization

| Step | From | To |
|---|---|---|
| Remove diacritics (tashkeel) | ً ٌ ٍ َ ُ ِ ّ ْ ٰ and Quranic marks | — |
| Remove tatweel | ـ | — |
| Unify alef | أ إ آ ٱ | ا |
| Alef maqsura | ى | ي |
| Ta marbuta | ة | ه |
| Persian/Urdu letters | ی ک | ي ك |
| Digits | ٠-٩ ۰-۹ | 0-9 |
| Optional, aggressive | ؤ → و، ئ → ي | |

The optional step increases recall but merges some distinct words; enable it for names and fuzzy search, not for exact lookups.

```js
const DIACRITICS = /[ؐ-ًؚ-ٰٟۖ-ۜ۟-۪ۨ-ۭ]/g;

export function normalizeArabic(text, { aggressive = false } = {}) {
  let s = text
    .normalize('NFC')
    .replace(DIACRITICS, '')
    .replace(/ـ/g, '')
    .replace(/[آأإٱ]/g, 'ا')
    .replace(/ى/g, 'ي')
    .replace(/ة/g, 'ه')
    .replace(/ی/g, 'ي')
    .replace(/ک/g, 'ك')
    .replace(/[٠-٩]/g, (d) => String(d.charCodeAt(0) - 0x0660))
    .replace(/[۰-۹]/g, (d) => String(d.charCodeAt(0) - 0x06F0));
  if (aggressive) s = s.replace(/ؤ/g, 'و').replace(/ئ/g, 'ي');
  return s.toLowerCase().replace(/\s+/g, ' ').trim();
}
```

```dart
final _diacritics = RegExp('[ؐ-ًؚ-ٰٟۖ-ۜ۟-۪ۨ-ۭ]');

String normalizeArabic(String text, {bool aggressive = false}) {
  var s = text
      .replaceAll(_diacritics, '')
      .replaceAll('ـ', '')
      .replaceAll(RegExp('[آأإٱ]'), 'ا')
      .replaceAll('ى', 'ي')
      .replaceAll('ة', 'ه')
      .replaceAll('ی', 'ي')
      .replaceAll('ک', 'ك')
      .replaceAllMapped(RegExp('[٠-٩]'), (m) => '${m[0]!.codeUnitAt(0) - 0x0660}')
      .replaceAllMapped(RegExp('[۰-۹]'), (m) => '${m[0]!.codeUnitAt(0) - 0x06F0}');
  if (aggressive) s = s.replaceAll('ؤ', 'و').replaceAll('ئ', 'ي');
  return s.toLowerCase().replaceAll(RegExp(r'\s+'), ' ').trim();
}
```

## Beyond normalization

- **Definite article:** «المدرسة» vs «مدرسة». For prefix/contains search on short fields, also match the query with a leading «ال» stripped (and attached prefixes «و»، «ب»، «ل» for longer text) — or use a stemmer.
- **Databases:**
  - PostgreSQL: store a normalized column (generated column or trigger) and index it; `pg_trgm` on that column for fuzzy matching. Built-in full-text search has no Arabic stemmer.
  - Elasticsearch/OpenSearch: the built-in `arabic` analyzer (`arabic_normalization` + `arabic_stem`) covers most of the above.
  - SQLite FTS5 / MySQL: store and query a normalized column.
  - Firestore: no text search — store a normalized field (or token array) and query on it, or use an external index.
- **Highlighting:** match on normalized text but highlight in the original; build an index map between the two strings since diacritics removal changes offsets.
- **Sorting:** use `Intl.Collator('ar')` / locale-aware collation, not code-point order.
