# 11 — Consolidated Technical Specification (care_fe)

**مفتاح الوسم:** ✔ Verified from implementation · 📄 Officially documented · ≈ Inferred · ✖ Missing · ? Unverified

| # | البند | المواصفة | الوسم |
|---|---|---|---|
| 1 | النظام | SPA React 19/TS/Vite للـ EMR/HMS (CARE)، Digital Public Good | ✔ `package.json:4` |
| 2 | المعمارية | Client-rendered SPA + REST API خارجي؛ modular-by-feature؛ micro-frontend plugins | ✔ |
| 3 | المكدس | انظر 02 | ✔ |
| 4 | المكونات | `Routers`, `Providers`, `Utils/request`, `types`, `pages`, `components`, `hooks`, `atoms`, `context`, `lib/override` | ✔ |
| 5 | قاعدة البيانات | غير موجودة في المستودع؛ PostgreSQL حسب الوثائق | ✖ / 📄? |
| 6 | API | 422 تعريفاً في 65 ملفاً؛ `/api/v1/*`؛ JSON | ✔ (جهة العميل)، Backend ? |
| 7 | المصادقة | JWT access/refresh + 2FA (temp_token) + OTP للمرضى + reCAPTCHA؛ تخزين localStorage؛ تجديد دوري | ✔ |
| 8 | التفويض | صلاحيات نصية (`can_*`) من `user.permissions`؛ `isSuperAdmin`؛ صلاحيات على مستوى الكائن `objectPermissions` | ✔ الواجهة؛ الإنفاذ ? |
| 9 | التكاملات | Sentry، reCAPTCHA، OSM/Nominatim، CDN/S3، SBOM، سجل الإضافات | ✔ |
| 10 | البنية التحتية | Docker multi-stage + nginx؛ Cloudflare preview؛ لا IaC | ✔ |
| 11 | النشر | CI/CD عبر GitHub Actions؛ إعداد وقت البناء عبر `REACT_*` | ✔ |
| 12 | الأمن | انظر 08؛ CSP تقريرية فقط؛ DOMPurify على Markdown | ✔ |
| 13 | الأداء | غير مقاس | ? |
| 14 | العمليات | Sentry اختياري؛ لا health/metrics في FE | ✔ |
| 15 | القيود | لا عربية/RTL؛ BE-coupling؛ اختبارات وحدة شبه معدومة؛ لغات: en, hi, kn, ml, mr, ta | ✔ |

## تفاصيل إضافية
- **حالة اللغة:** `fallbackLng: "en"`، `supportedLngs: Object.keys(LANGUAGES)` (`src/i18n.ts:155-159`)؛ اللغات المسموحة تُضبط عبر `REACT_ALLOWED_LOCALES` ✔.
- **الإصدار:** `build:meta` يولد معلومات الإصدار؛ `src/lib/appVersion.ts` يخزن `app-version` في localStorage ويُستعمل كـ `buster` للكاش المحفوظ (`queryClient.ts:44`) ✔.
- **PWA:** `vite-plugin-pwa` + service worker مخصص (`src/service-worker.ts`) ✔؛ سياسة التخزين المؤقت لم تُقرأ تفصيلاً ?.
- **الإضافات:** manifest يعرّف routes/components/tabs/devices/overrides (`pluginTypes.ts`، 232 سطراً) ✔.
