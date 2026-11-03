# 07 — Frontend & UX Report

> تغطية جزئية: مؤشرات آلية + قراءة Routers. لا مراجعة شاشة-بشاشة، ولا تشغيل للتطبيق في متصفح (لا Backend).

## 1. الهيكل
- **Layouts:** Sidebar + محتوى؛ المسارات التي تُخفى فيها الـ Sidebar محددة بقائمة regex (`AppRouter.tsx:31-60`، ≥ 12 نمطاً لشاشات الطلبات/التحويلات/الـ tokens).
- **Routers:** Public (دخول، إعادة تعيين كلمة المرور، صفحة هبوط، حجز مواعيد عام)، Patient (`/patient/home`, `/patient/:id/:tab`)، App (الموظفون) + Admin (`/admin/*` يُحسب بـ `startsWith`).
- **Routes:** 144 مساراً (انظر `evidence/routes_all.txt`). أكبر مجموعات: `facility/patient` 26، `organization` 19، `admin` 16، `facility/billing` 14.
- **Design system:** shadcn/ui (92 ملفاً في `components/ui`) — مكتوب في التوجيهات "لا تعدّل مباشرة".
- **Forms:** react-hook-form + zod؛ ‏Questionnaire Engine ديناميكي (23 نوع سؤال في `QuestionTypes/`).

## 2. الحالات والجودة (مؤشرات)
| المؤشر | القيمة |
|---|---:|
| استعمال `aria-*` | 388 موضعاً |
| `sr-only` | 52 |
| ملفات تستخدم Skeleton (حالات تحميل) | 174 |
| ملفات تذكر ErrorBoundary | 8 |
| ملفات تستخدم `useTranslation` | 599 |
| دعم RTL (`rtl`/`dir`) | **0** |
| لغات `public/locale` | en (6856 مفتاحاً)، hi، kn، ml، mr، ta |
| العربية | **غير مدعومة** (لا `ar.json`، لا RTL) |

- التحقق من حالات الأخطاء الفارغة/المكسورة للتنقل لم يُجرَ.
- `eslint-plugin-i18next` + `no-undefined-translation-keys` تفرض i18n وقت lint (`package.json:devDependencies`) ⇒ نقطة قوة (Verified أن الحزمتين مثبتتان؛ تفعيل القواعد في `eslint.config.mjs` لم يُقرأ).
- **للاستخدام العربي:** يتطلب إضافة `ar` إلى `LANGUAGES` (`src/i18n.ts`)، وRTL (Tailwind logical properties)، وإعادة فحص الطباعة (PDF/Print) ‏— عمل غير بسيط نظراً لحجم الشاشات.

## 3. رحلات مستخدم أساسية (من تعريف Routes)
1. موظف: Login → (2FA) → Dashboard `/` → Facility → Patient search → Patient → Encounter → أسئلة/أدوية/تشخيصات → Billing.
2. مريض: Landing → Facilities → حجز موعد (`/facility/:id/appointments/:staffId/book-appointment`) → OTP → تأكيد.
3. مدير: `/admin/*` (مؤسسات، أدوار، Questionnaires، ValueSets).

## 4. مخاطر UX/صيانة
- ملفات مكونات ضخمة (≥1000 سطر): `QuestionnaireEditor.tsx` 3258، `MedicationRequestQuestion.tsx` 2403، `PrintChargeItems.tsx` 1677، `InvoiceShow.tsx` 1608، `AddSupplyDeliveryForm.tsx` 1451، `AppointmentDetail.tsx` 1303، `PatientRegistration.tsx` 1198 ⇒ صعبة الاختبار والمراجعة.
- كثرة مسارات بقائمة regex لإخفاء الـ Sidebar تشير إلى تعقيد layout قابل للتفكك.

## 5. جرد الشاشات
مُوثَّق على مستوى مجموعات Route فقط (04). الجرد الفردي (Screen ID لكل شاشة) **مؤجل**.
