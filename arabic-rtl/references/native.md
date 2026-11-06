# React Native, Android, iOS

## React Native

- Enable RTL: `I18nManager.allowRTL(true)`; switching direction at runtime requires `I18nManager.forceRTL(isRtl)` followed by an app reload (e.g. `expo-updates` `reloadAsync()` or `react-native-restart`).
- Styles: `marginStart`/`marginEnd`, `paddingStart`/`paddingEnd`, `start`/`end`, `borderStartWidth`, `borderTopStartRadius`. Avoid `marginLeft`, `left`, etc.
- `flexDirection: 'row'` already flips under RTL. Do not swap to `row-reverse` manually.
- `transform: [{ translateX }]` and `Animated` x-values do not flip — multiply by `I18nManager.isRTL ? -1 : 1`.
- Directional images: `style={{ transform: [{ scaleX: I18nManager.isRTL ? -1 : 1 }] }}`.
- `TextInput` for phone/email: `style={{ writingDirection: 'ltr', textAlign: 'left' }}` (here `left` is intentional — the content is LTR).

## Android (Views)

- `android:supportsRtl="true"` in `<application>`.
- Use `start`/`end` attributes: `layout_marginStart`, `paddingEnd`, `layout_alignParentStart`, `drawableStart`, `layout_toEndOf`, `gravity="start"`.
- Vector drawables that indicate direction: `android:autoMirrored="true"`.
- `TextView` with mixed content: `BidiFormatter.getInstance().unicodeWrap(text)`.
- Phone/email `EditText`: `android:textDirection="ltr"` and `android:layoutDirection="ltr"`.
- Lint: enable `RtlHardcoded`, `RtlSymmetry`, `RtlCompat`, `RtlEnabled` and treat them as errors.

## Jetpack Compose

- `Modifier.padding(start = 8.dp)`, `Alignment.Start`/`End`, `Arrangement.Start`, `TextAlign.Start`. `absolutePadding`/`absoluteOffset` only for things that must not mirror.
- Read direction: `LocalLayoutDirection.current == LayoutDirection.Rtl`.
- Directional icons: `Icons.AutoMirrored.Filled.ArrowBack` (and other `AutoMirrored` icons); for custom vectors set `autoMirror = true` in `ImageVector.Builder`.
- Force LTR for a subtree (phone input):
  ```kotlin
  CompositionLocalProvider(LocalLayoutDirection provides LayoutDirection.Ltr) { PhoneField() }
  ```
- Canvas drawing and `Offset` x-values are physical — mirror manually.

## iOS (UIKit)

- Use `leading`/`trailing` anchors and constraints, never `left`/`right`.
- `UIStackView` follows `semanticContentAttribute` automatically.
- Directional images: `image.imageFlippedForRightToLeftLayoutDirection()`, or set "Direction: Left to Right, Mirrors" in the asset catalog.
- Phone/email fields: `textField.semanticContentAttribute = .forceLeftToRight` and `textAlignment = .left`.
- `NSTextAlignment.natural` for labels.

## SwiftUI

- `.padding(.leading, 8)`, `HStack` and `.frame(alignment: .leading)` are direction-aware.
- Directional custom images: `.flipsForRightToLeftLayoutDirection(true)`.
- SF Symbols with directional variants (`chevron.backward`, `arrow.forward`) mirror automatically — prefer `backward`/`forward` names over `left`/`right`.
- Force LTR subtree: `.environment(\.layoutDirection, .leftToRight)`.
- Preview RTL: `.environment(\.layoutDirection, .rightToLeft)` and `.environment(\.locale, Locale(identifier: "ar"))`.
- Use `Text(value, format: .number)` / `FormatStyle` with the Arabic locale for numbers and dates.
