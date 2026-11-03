# 03 — Architecture Report (Frontend فقط)

**الحكم:** Single-Page Application (SPA) ثابتة تُخدَم من nginx وتتحدث إلى Backend REST خارجي (`careConfig.apiUrl`) عبر `fetch` + TanStack Query. بنية داخلية **modular-by-feature** مع **Micro-frontend plugin system** اختياري. **وسم الثقة:** Verified من الكود للطبقات الداخلة في المستودع؛ كل ما وراء الـ API = Unverified.

## 1. الأسلوب المعماري
| الجانب | الاستنتاج | الدليل |
|---|---|---|
| النمط | SPA / Static front + REST API (Client-rendered، لا SSR) | `Dockerfile` (build ثم nginx)، `index.html` |
| Routers | ثلاثة: `PublicRouter` (غير مسجَّل)، `PatientRouter` (مريض OTP)، `AppRouter` (موظف) — يُختار بحسب حالة المصادقة | `src/Routers/*.tsx`, `src/Providers/AuthUserProvider.tsx:211-215` |
| Routes | 144 مساراً فريداً في 8 ملفات routes + 3 routers | `analysis/evidence/routes_all.txt` |
| إضافات Routes | `usePluginRoutes()` و`useOrganizationRoutes()` تُدمج وقت التشغيل | `AppRouter.tsx:102-115` |
| Server state | TanStack Query، `retry:false`، `refetchOnWindowFocus:false`؛ أخطاء مركزية | `queryClient.ts:17-31` |
| Persistence | الاستعلامات ذات `meta.persist===true` فقط تُحفظ في `localStorage` | `queryClient.ts:36-44` (مستعملة في موضعين فقط: `useTagConfig.ts:27`, `ManageResponseTemplatesSheet.tsx:101`) |
| Client state | Jotai atoms (8) + Contexts (Permission، Shortcut) | `src/atoms`, `src/context` |
| API typed contract | كائنات route ثابتة `{path, method, TRes, TBody}` + `query()/mutate()` | `src/Utils/request/query.ts`, `utils.ts:5-15` |
| Override framework | استبدال مكونات من الإضافات بشروط وأولويات | `src/lib/override`, `PluginEngine.tsx:101-122` |
| الصلاحيات | تُحمَّل من الـ API (`user.permissions`) وتُفحص عبر `usePermissions().hasPermission`؛ `isSuperAdmin` يتجاوز الكل | `PermissionContext.tsx:34-44` |

## 2. تسلسل الطلب (Request lifecycle) — Verified
`Component → useQuery({queryFn: query(route,opts)}) → callApi → makeUrl(path+pathParams+query) → makeHeaders (Authorization: Bearer من localStorage) → fetch(apiUrl+path) → getResponseBody → HTTPError عند !ok → QueryCache.onError → handleHttpError (toast / redirect إلى /session-expired / رسائل Pydantic)`.
- `src/Utils/request/query.ts:9-60`, `utils.ts:56-80`, `errorHandler.ts:8-60`.
- نموذج الأخطاء يفترض أن الـ Backend يعيد أخطاء Pydantic/DRF (`code:"token_not_valid"`) → اقتران بالـ Backend (Medium).

## 3. المصادقة (Verified من جهة الـ Client)
1. تسجيل الدخول → إما `JwtTokenObtainPair {access, refresh}` أو `MfaAuthenticationToken {temp_token}` ⇒ مسار `/2fa` (`AuthUserProvider.tsx:31-38, 95-136`).
2. التخزين: `localStorage` لـ access/refresh/patient token/`mfa_temp_token` (`:47-48,99,114-115,131-132,141-143`).
3. التجديد: `useQuery` بـ `refetchInterval = careConfig.auth.tokenRefreshInterval` (`:70-77`)؛ عند الفشل تُحذف المفاتيح (`:81-85`).
4. مزامنة بين التبويبات عبر storage events (مفاتيح مراقبة عند `:182-184`).
5. مريض: تدفق OTP منفصل (`PatientRouter`, `types/otp`).
6. إعادة التوجيه بعد الدخول تُحقَّق same-origin (`AuthUserProvider.tsx:224-238`).

## 4. الصلاحيات (RBAC) — جهة العميل
- ثوابت `PERMISSION_*` (68 تصديراً) في `src/common/Permissions.tsx` بأسماء مثل `can_create_patient`، ‏`can_view_clinical_data`.
- فحص الواجهة **تجميلي/UX**؛ الإنفاذ الحقيقي في الـ Backend (**Unverified**).
- 74 ملفاً تستخدم `useAuthUser/usePermissions/useHasPermission`.

## 5. الإضافات (Module Federation)
- قائمة التطبيقات المفعّلة تأتي من (أ) `REACT_ENABLED_APPS` وقت البناء و(ب) API `plugConfigApi.list` وقت التشغيل ثم تُدمج (`mergePlugConfigs`).
- لكل إضافة: `config.meta.url` يُتحقَّق بـ `z.url()` فقط، ثم `setFederationRemote` + `getFederationRemote(slug,"./manifest")` (`PluginEngine.tsx:25-56`) ⇒ **تنفيذ كود JS من عنوان يحدده الـ Backend داخل أصل التطبيق**.
- الـ manifest يصدّر routes، components، tabs، devices، overrides (`pluginTypes.ts`).

## 6. النشر
Build ثابت → `build/` → nginx:alpine. `_headers` تُولَّد من متغير `HEADERS` (`scripts/generate-headers.ts`). Preview على Cloudflare. لا Backend في الحاوية. (انظر 10)

## 7. التكامل الخارجي (جهة المتصفح)
| الجهة | الغرض | الدليل |
|---|---|---|
| `careConfig.apiUrl` (CARE API) | كل البيانات | `care.config.ts:37-54` (`REACT_CARE_URL_MAP` لربط origin→API) |
| Sentry | تتبع أخطاء | `Integrations/Sentry.tsx:19` |
| Google reCAPTCHA | حماية تسجيل الدخول/OTP | `care.config.ts:88` |
| `sbom.ohc.network` | عرض SBOM/التراخيص | `care.config.ts:55` |
| `cdn.ohc.network` + S3 buckets | شعارات وملفات المرضى | `.env`, `nginx.conf:12` |
| Nominatim/OSM + `mapFallbackUrlTemplate` | الخرائط/العناوين | grep + `care.config.ts:84` |
| `raw.githubusercontent.com/.../care_apps_registry` | متجر التطبيقات | `.env` |
| `wa.me` | روابط واتساب | grep (4 مواضع) |

## 8. الفجوات
- لا يمكن رسم ERD أو تتبع Business Logic خلف الـ API.
- لا Deployment architecture حقيقي للإنتاج (مجرد nginx + CI).
- الرسوم: `diagrams/*.mmd`.
