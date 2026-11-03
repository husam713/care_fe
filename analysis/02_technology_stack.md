# 02 — Technology Stack

> المصدر: `package.json`, `vite.config.mts`, `Dockerfile`, `nginx/nginx.conf`, `tsconfig.json`. الإصدارات من `package.json` (مدى semver) وما ثُبِّت فعلاً في `node_modules` عند الإشارة.

## 1. الجدول
| الطبقة | التقنية | الاستخدام (دليل) | ضرورية؟ |
|---|---|---|---|
| Language | TypeScript ^6.0.3 (strict، target ES2022) | `tsconfig.json`؛ 796 tsx + 427 ts | أساسية |
| UI | React ^19.2.6 | كل `src` | أساسية |
| Build | Vite ^8.0.14 (+ `@vitejs/plugin-react`) | `vite.config.mts` | أساسية |
| Styling | Tailwind CSS v4 (`@tailwindcss/vite`)، `tw-animate-css` | `src/style/`, `tailwind.config.js` | أساسية |
| Components | shadcn/ui نمط + Radix UI (≈25 حزمة)، `lucide-react`، `cmdk`, `vaul`, `sonner` | `src/components/ui/` (92 ملف) | أساسية |
| Routing | `raviger` ^5.3 | `src/Routers/` (3 routers + 8 ملفات routes) | أساسية |
| Server state | TanStack Query ^5.100 + persist client (localStorage) | `src/Utils/request/queryClient.ts` | أساسية |
| Client state | Jotai ^2.20 (`src/atoms/` 8 ملفات)، React Context (2) | | أساسية |
| Forms | react-hook-form ^7.76 + zod ^4.4 + `@hookform/resolvers` | | أساسية |
| i18n | i18next ^26 + react-i18next + http-backend + language-detector | `src/i18n.ts`, `public/locale` | أساسية |
| Micro-frontend | `@originjs/vite-plugin-federation` | `src/PluginEngine.tsx` | اختيارية (تُفعَّل بالإعداد) |
| PWA | `vite-plugin-pwa` (Workbox) + `src/service-worker.ts` | `vite.config.mts:456` | اختيارية |
| Rich text / Docs | `@mdxeditor/editor`, `lexical`, `markdown-it`(+mark, task-lists), `dompurify` | `components/ui/markdown.tsx` | أساسية للملاحظات/القوالب |
| PDF/Print | `jspdf`, `react-pdf` | 1 و3 استيرادات | وظيفية |
| Charts/Graph | `recharts` (1)، `reactflow` (1) | | وظيفية |
| Clinical | `fhirpath` ^4.11 (1 استيراد)، `decimal.js`, `libphonenumber-js` | | وظيفية |
| Media/Scan | `react-webcam`, `@yudiel/react-qr-scanner`, `browser-image-compression`, `react-easy-crop` | | وظيفية |
| Maps | `pigeon-maps` + Nominatim/OSM | استدعاء `nominatim.openstreetmap.org` (1) | وظيفية |
| Anti-abuse | `react-google-recaptcha` | `care.config.ts:88` | وظيفية |
| Monitoring | `@sentry/browser` | `src/Integrations/Sentry.tsx`, `care.config.ts:97-100` | اختيارية |
| Test | Playwright ^1.60 (E2E)، `node --test` (unit، ملف واحد) | `tests/`, `plugins/*.test.ts` | |
| Quality | ESLint 9، Prettier 3، knip، husky + lint-staged، Snyk، CodeQL، react-doctor | `package.json:56-62`, `.github/workflows` | |
| Runtime hosting | nginx:stable-alpine (static) | `Dockerfile` | |
| Hosting إضافي | Netlify (README)، Cloudflare Pages preview | `.github/workflows/cloudflare-preview*.yml` | |

## 2. ما غير موجود في المستودع (Backend حسب الوثائق فقط)
Django/Python 3.13، PostgreSQL، Redis، S3/LocalStack (`CARE_CDN_URL` في `.env`). **Unverified**.

## 3. ملاحظات
- **Direct dependencies:** 90 prod + 42 dev + 4 optional. Transitive: غير مجرد (≈ 699 حزمة مثبتة عند `npm ci`).
- `optionalDependencies` تستخدم `"latest"` لـ `@esbuild/linux-*` (`package.json:200-205`) → بناء غير حتمي (risk: Low).
- ‏`postinstall` يشغّل `tsx scripts/install-platform-deps.ts` (سكربت لم يُراجَع محتواه — لم يُنفَّذ؛ استُخدم `--ignore-scripts`).
- ‏`vite 8.1.4` مثبتة وضمن نطاق ثغرات dev-server (انظر 08).
- ‏`zod 4.4.3` ضمن نطاق ثغرة إجهاد الذاكرة `<4.6.6` (انظر 08).
- مكونات تُزال بـ tree-shaking: `plugins/treeShakeCareIcons.ts`، ‏`autoRegisterComponents.ts` (Vite plugins محلية).
- ‏`src/CAREUI/icons/UniconPaths.json` (4857 سطراً) مولَّد/بيانات.

## 4. مصفوفة المسؤوليات (Frontend)
| المجلد | المسؤولية |
|---|---|
| `src/Routers` | اختيار Router حسب حالة المصادقة + دمج Routes الإضافات |
| `src/Providers` | `AuthUserProvider` / `PatientUserProvider` |
| `src/Utils/request` | `callApi`, `query`, `mutate`, `batch`, `uploadFile`, معالجة الأخطاء |
| `src/types/*` | عقود البيانات + مسارات API (65 ملف `*Api.ts`) |
| `src/pages`, `src/components` | الشاشات والمكونات (≈ 800 ملف) |
| `src/common/Permissions.tsx` | ثوابت الصلاحيات (68 export) |
| `src/lib/override` | آلية استبدال مكونات من الإضافات |
| `plugins/` | Vite plugins محلية |
| `scripts/` | بناء/DB snapshot لـ Playwright |
