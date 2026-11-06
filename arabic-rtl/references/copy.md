# Writing Arabic UI copy

## Principles

- **Write in Arabic, don't transliterate English structure.** "You have no notifications yet" → «لا توجد إشعارات بعد», not «أنت ليس لديك إشعارات حتى الآن».
- **Register:** Modern Standard Arabic (فصحى مبسطة) by default — short, plain, no ornate or classical vocabulary. Use dialect only if the brand voice explicitly does, and then consistently.
- **Gender-neutral by default.** Arabic imperatives and second-person forms are gendered. Prefer:
  - Verbal nouns for buttons and titles: «تسجيل الدخول»، «إضافة إلى السلة»، «حفظ».
  - Impersonal or passive phrasing for messages: «تم حفظ التغييرات», «يرجى إدخال رقم الهاتف».
  - If addressing the user is unavoidable and gender is known, use the correct form; otherwise masculine-default is common but «يرجى…»/«الرجاء…» avoids it.
- **Consistency:** pick verb or verbal-noun style for actions and use it across the app. Don't mix «احفظ» and «إلغاء» side by side.
- **Punctuation:** Arabic comma «،», semicolon «؛», question mark «؟». Use «…» quotes or standard double quotes consistently. No space before punctuation.
- **Length:** Arabic strings are often shorter in characters but taller; keep labels short, avoid fixed-width containers.
- **Numbers in text** follow the product's digit system (see [numbers-dates.md](numbers-dates.md)) and the plural rules below.
- **Don't translate brand and product names** unless an official Arabic name exists; keep them in Latin script, LTR-isolated.
- **Tone for errors:** state what happened and what to do, without blame: «تعذّر الاتصال بالخادم. يرجى المحاولة مرة أخرى.»

## Plurals

Arabic needs six forms. Always write ICU messages with all of them:

```
{count, plural,
  =0 {لا توجد رسائل}
  =1 {رسالة واحدة}
  =2 {رسالتان}
  few {{count} رسائل}
  many {{count} رسالةً}
  other {{count} رسالة}
}
```

Rules of thumb for the noun after the number:
- 3–10 → plural, genitive (٣ رسائل، ١٠ أيام)
- 11–99 → singular, accusative with tanween fatha (١١ يومًا، ٢٥ ملفًا، ٥٠ رسالةً — nouns ending in ة take the tanween without an added alef)
- 100, 1000, … → singular, genitive (١٠٠ يوم، ١٠٠٠ ملف)

Number–noun gender agreement (٣ أيام vs ٣ ساعات written as words: ثلاثة أيام / ثلاث ساعات) only matters when numbers are written as words — prefer digits in UI.

## No concatenation

Bad: `"Welcome " + name + ", you have " + n + " orders"`.
Good: one message with placeholders and plural: `«مرحبًا {name}، لديك {count, plural, …}»`.

Also avoid reusing one string in different grammatical slots (e.g. "New" for both «جديد» and «جديدة»): Arabic adjectives agree with the noun's gender, so each context needs its own key.

## Glossary

| English | Arabic |
|---|---|
| Sign in / Log in | تسجيل الدخول |
| Sign out | تسجيل الخروج |
| Sign up / Create account | إنشاء حساب |
| Forgot password? | نسيت كلمة المرور؟ |
| Password | كلمة المرور |
| Email | البريد الإلكتروني |
| Phone number | رقم الهاتف |
| Username | اسم المستخدم |
| Verification code | رمز التحقق |
| Resend code | إعادة إرسال الرمز |
| Home | الرئيسية |
| Profile | الملف الشخصي |
| Account | الحساب |
| Settings | الإعدادات |
| Notifications | الإشعارات |
| Language | اللغة |
| Search | بحث |
| Search… (placeholder) | ابحث… |
| Filter | تصفية |
| Sort by | ترتيب حسب |
| Save | حفظ |
| Cancel | إلغاء |
| Delete | حذف |
| Edit | تعديل |
| Add | إضافة |
| Confirm | تأكيد |
| Submit / Send | إرسال |
| Next | التالي |
| Back | رجوع |
| Skip | تخطٍّ |
| Done | تم |
| Close | إغلاق |
| Retry | إعادة المحاولة |
| Loading… | جارٍ التحميل… |
| Upload | رفع |
| Download | تنزيل |
| Share | مشاركة |
| Copy | نسخ |
| Copied | تم النسخ |
| More | المزيد |
| Show all | عرض الكل |
| Required | مطلوب |
| Optional | اختياري |
| Something went wrong | حدث خطأ ما |
| No results found | لا توجد نتائج |
| No internet connection | لا يوجد اتصال بالإنترنت |
| Cart | السلة |
| Checkout | إتمام الشراء |
| Order | طلب |
| My orders | طلباتي |
| Payment | الدفع |
| Total | الإجمالي |
| Subtotal | المجموع الفرعي |
| Discount | خصم |
| Delivery | التوصيل |
| Address | العنوان |
| Help | المساعدة |
| Contact us | تواصل معنا |
| Terms and conditions | الشروط والأحكام |
| Privacy policy | سياسة الخصوصية |
| About | حول التطبيق |
| Today / Yesterday | اليوم / أمس |
| Welcome back | مرحبًا بعودتك |
| Are you sure? | هل أنت متأكد؟ (gendered — prefer «هل تريد المتابعة؟» or a specific question) |
| Successfully saved | تم الحفظ بنجاح |

Notes:
- «تخطٍّ» is the verbal noun (skip); many apps use «تخطي» — pick one and be consistent.
- Use «تنزيل» for download and «رفع» for upload; «تحميل» is ambiguous across regions (used for both).
