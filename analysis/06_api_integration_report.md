# 06 — API & Integration Report

## 1. المنهج
استخراج آلي لتعريفات `{path, method}` من 65 ملف `src/**/*Api.ts` (سكربت يقرأ كتل `key: { ... path: "..." ... HttpMethod.X }`) + قراءة يدوية لطبقة الطلبات. الناتج الخام: `evidence/api_inventory_raw.tsv`. **حدود الطريقة:** قد يفوتها تعريفات تُبنى بدوال مساعدة (`API(...)` في `utils.ts:5`) أو ملفات لا تنتهي بـ `Api.ts`؛ 8 تعريفات لم يُكتشف لها method (تستخدم `API()`/أشكال أخرى).

## 2. الإحصاء
| Method | العدد |
|---|---:|
| GET | 155 |
| POST | 181 |
| PUT | 54 |
| DELETE | 21 |
| PATCH | 3 |
| غير مكتشف | 8 |
| **المجموع** | **422** |

| النطاق | عدد التعريفات |
|---|---:|
| emr | 111 |
| inventory | 38 |
| billing | 37 |
| scheduling | 30 |
| questionnaire | 25 |
| tokens | 23 |
| device | 16 |
| valueSet / organization / location / facility | 15 لكلٍّ |
| user / facilityOrganization / auth | 12 لكلٍّ |
| consent | 8 |
| files | 7 |
| resourceRequest / base | 6 لكلٍّ |
| patient / healthcareService | 5 لكلٍّ |
| otp / notes | 4 لكلٍّ |
| careTeam / file | 1 لكلٍّ |

نمط المسارات: `/api/v1/<resource>/...` (مثال: `/api/v1/patient/{patientId}/consent/`). كل المسارات نسبية إلى `careConfig.apiUrl`.

## 3. سلوك العميل (Verified)
| الجانب | التفصيل | الدليل |
|---|---|---|
| Base URL | `careConfig.apiUrl` ما لم يحدد route `baseUrl` | `query.ts:13` |
| Auth | `Authorization` من `localStorage` ما لم يكن `noAuth` | `utils.ts:56-80` |
| Content-Type | JSON افتراضياً، FormData بلا ترويسة | `utils.ts:63-66` |
| Errors | `HTTPError{status, silent, cause}`؛ 404/413/400/406 معالجة، `token_not_valid` ⇒ `/session-expired` | `errorHandler.ts:8-60` |
| Silent | `silent: bool | (res)=>bool` | `query.ts:44-48` |
| Pagination | `query.paginated` يجلب كل الصفحات (`maxPages`) | `query.ts` |
| Batch | `Utils/request/batch.ts` | غير مقروء بالتفصيل |
| Upload | `XMLHttpRequest` في `uploadFile.ts` (لتقدم الرفع) | `uploadFile.ts:18-74` |
| Cancellation | AbortSignal من React Query | `query.ts:73` |
| Retry | معطل افتراضياً (`retry:false`) | `queryClient.ts:19` |

## 4. الملاحظات
1. **عقد غير مُنفَّذ**: الأنواع تُكتب يدوياً (`Type<T>()`) ولا تُولَّد من OpenAPI ⇒ خطر انحراف عن الخادم (Medium). لم يُعثر على ملف OpenAPI في المستودع.
2. **تعليق دلالي على اقتران**: `chargeItemDefinitionApi.ts:34` — المسار مؤقت "حتى يجهز BE".
3. **ترميز المعاملات**: `redirect` في `/session-expired?redirect=${window.location.href}` بلا ترميز (`errorHandler.ts` `handleSessionExpired`) ⇒ قد يقتطع query (Low).
4. **تمرير الصلاحيات**: لا تتحقق الطبقة من الصلاحيات؛ الإنفاذ للخادم (Unverified).
5. **استدعاءات مفرطة محتملة**: `query.paginated` يجلب كل الصفحات؛ لم يُحصَ استخدامه — Unverified.
6. **WebSockets / GraphQL / Webhooks**: لا شيء مرصود (grep `WebSocket` لم يُنفَّذ بشمول؛ `postMessage` يظهر فقط في service worker).
7. ‏**API للإضافات**: `plugConfigApi.list` تُحدّد remote URLs (انظر 08-S03).

## 5. جدول مفصَّل لكل Endpoint
غير مُنتَج يدوياً (422 صفاً). المرجع الخام: `evidence/api_inventory_raw.tsv` (file, key, line, method, path). حقول Authorization/Validation على الخادم = **Unverified**.
