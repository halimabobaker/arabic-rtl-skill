# Flutter

## App setup

```yaml
# pubspec.yaml
dependencies:
  flutter_localizations:
    sdk: flutter
  intl: any
flutter:
  generate: true
```

```dart
MaterialApp(
  locale: const Locale('ar'),
  supportedLocales: const [Locale('ar'), Locale('en')],
  localizationsDelegates: const [
    AppLocalizations.delegate,
    GlobalMaterialLocalizations.delegate,
    GlobalWidgetsLocalizations.delegate,
    GlobalCupertinoLocalizations.delegate,
  ],
  theme: ThemeData(fontFamily: 'IBMPlexSansArabic'),
)
```

`GlobalWidgetsLocalizations` is what sets `Directionality` to RTL. Do not wrap screens in `Directionality(textDirection: TextDirection.rtl)` manually; read the current direction with `Directionality.of(context)`.

## Replace physical with directional

| Avoid | Use |
|---|---|
| `EdgeInsets.only(left: 8)` | `EdgeInsetsDirectional.only(start: 8)` |
| `EdgeInsets.fromLTRB(a, b, c, d)` | `EdgeInsetsDirectional.fromSTEB(a, b, c, d)` |
| `Alignment.centerLeft` / `topRight` | `AlignmentDirectional.centerStart` / `topEnd` |
| `Positioned(left: 0)` | `PositionedDirectional(start: 0)` |
| `BorderRadius.only(topLeft: r)` | `BorderRadiusDirectional.only(topStart: r)` |
| `Border(left: side)` | `BorderDirectional(start: side)` |
| `TextAlign.left` / `right` | `TextAlign.start` / `end` |
| `Align(alignment: Alignment.left…)` | `Align(alignment: AlignmentDirectional…)` |

`EdgeInsets.all`, `EdgeInsets.symmetric`, `Alignment.center`, and `Row`/`Column` with `MainAxisAlignment.start`/`CrossAxisAlignment.start` are already direction-safe. `Row` lays out children from the start edge automatically — do not reverse child lists for RTL.

## Icons and images

- Many Material directional icons (`Icons.arrow_back`, `Icons.arrow_forward`, `Icons.chevron_left`, `Icons.chevron_right`, `Icons.arrow_back_ios`, …) are declared with `matchTextDirection: true` and mirror automatically. Prefer `BackButton` / `Icons.adaptive.arrow_back` for back navigation.
- Custom `IconData` and SVG/PNG arrows do not mirror. For icon fonts, declare `IconData(..., matchTextDirection: true)`. For images, use `Image(matchTextDirection: true)`. For other widgets:

```dart
final isRtl = Directionality.of(context) == TextDirection.rtl;
Transform.flip(flipX: isRtl, child: SvgPicture.asset('assets/arrow.svg'));
```

- Never mirror media playback icons, logos, or checkmarks.

## Motion and gestures

Hardcoded horizontal offsets do not follow direction:

```dart
final isRtl = Directionality.of(context) == TextDirection.rtl;
final begin = Offset(isRtl ? -1 : 1, 0);
```

- Flip the sign of `details.delta.dx` / `primaryVelocity` in custom swipe logic.
- `Dismissible` uses `DismissDirection.startToEnd` / `endToStart`, which already follow direction — use those, not physical assumptions.
- Horizontal `ListView`, `PageView`, and `TabBar` follow `Directionality` already; do not set `reverse: true` to "fix" RTL.
- `CustomPainter` draws in physical coordinates: pass `TextDirection` in and mirror manually (e.g. progress drawn from `size.width` toward 0).

## Text and inputs

```dart
TextField(
  keyboardType: TextInputType.phone,
  textDirection: TextDirection.ltr,
  textAlign: TextAlign.start,
  inputFormatters: [WesternDigitsFormatter()],
)
```

- Phone, email, URL, password, OTP, IBAN, and card fields: `textDirection: TextDirection.ltr`.
- Mixed-direction strings in `Text`: wrap the LTR part with Unicode isolates — `'⁦$phone⁩'` (LRI…PDI), or `'⁨$name⁩'` (FSI…PDI) when the direction is unknown.
- User-generated text of unknown language: detect with `intl`'s `Bidi.detectRtlDirectionality(text)` and set `textDirection` accordingly.
- Set `height: 1.5`+ in the text theme for Arabic; never set `letterSpacing` on Arabic styles (Material's default `letterSpacing` values in `TextTheme` should be zeroed for the Arabic theme); avoid `FontStyle.italic`.

Normalize digits as the user types, so validators and `int.parse` work:

```dart
class WesternDigitsFormatter extends TextInputFormatter {
  static final _eastern = RegExp('[٠-٩۰-۹]');

  @override
  TextEditingValue formatEditUpdate(TextEditingValue oldValue, TextEditingValue newValue) {
    final text = newValue.text.replaceAllMapped(_eastern, (m) {
      final c = m[0]!.codeUnitAt(0);
      return String.fromCharCode(c - (c >= 0x06F0 ? 0x06F0 : 0x0660) + 0x30);
    });
    return newValue.copyWith(text: text);
  }
}
```

Each replacement is a single code unit for a single code unit, so the existing selection stays valid.

## Localized strings (ARB)

```json
{
  "filesCount": "{count, plural, =0{لا توجد ملفات} =1{ملف واحد} =2{ملفان} few{{count} ملفات} many{{count} ملفًا} other{{count} ملف}}",
  "@filesCount": { "placeholders": { "count": { "type": "int" } } },
  "welcome": "مرحبًا {name}",
  "@welcome": { "placeholders": { "name": { "type": "String" } } }
}
```

- Provide all six plural branches for Arabic, not just `=1` and `other`.
- Never build UI text as `'${l10n.hello} $name'`; put the placeholder inside the message.

## Numbers and dates

```dart
NumberFormat.decimalPattern('ar').format(1234.5);
NumberFormat.currency(locale: 'ar', name: 'LYD', symbol: 'د.ل', decimalDigits: 3).format(12.5);
DateFormat.yMMMMd('ar').format(date); // requires initializeDateFormatting('ar') if not using flutter_localizations
```

Dart `intl` resolves locales differently from browsers/ICU. With `intl` 0.20: `ar`, `ar_LY`, `ar_SA` → `1,234.5`; `ar_EG` → `١٬٢٣٤٫٥` — whereas JS gives Arabic-Indic digits for `ar-SA` and `1.234,5` for `ar-LY`. Check the output and, if the product requires one digit system, convert the formatted string explicitly. See [numbers-dates.md](numbers-dates.md).

`package:intl` exports its own `TextDirection`, which clashes with Flutter's. Import it as `import 'package:intl/intl.dart' hide TextDirection;` in widget files.

## Testing

- Run every screen with `locale: const Locale('ar')`.
- Widget tests: wrap the widget under test in `Directionality(textDirection: TextDirection.rtl, ...)` and assert positions (`tester.getTopLeft`) for elements that must sit on the start edge.
- Search the codebase for regressions: `EdgeInsets.only(left`, `EdgeInsets.only(right`, `fromLTRB`, `Alignment.centerLeft`, `Alignment.centerRight`, `topLeft`, `topRight`, `bottomLeft`, `bottomRight`, `Positioned(`, `TextAlign.left`, `TextAlign.right`, `letterSpacing`.
