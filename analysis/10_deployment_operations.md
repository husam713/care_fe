# 10 — Deployment, Operations & Scalability (Frontend)

## 1. البناء والتشغيل (Verified من الملفات)
```
Dockerfile:  node:22-bookworm-slim → npm install --ignore-scripts → npm run postinstall → npm run build
             → nginx:stable-alpine ← COPY /app/build + nginx/nginx.conf  (EXPOSE 80)
```
- `npm run build` = `build:meta` (يولّد version) + `supported-browsers` + `vite build`؛ `postbuild` يولّد `_headers` من `HEADERS`.
- ذاكرة Node 4GB (`NODE_OPTIONS=--max-old-space-size=4096`). التوثيق يقول البناء > 2 دقيقة.
- الإعداد وقت **البناء** عبر متغيرات `REACT_*` (انظر `.example.env` ‏206 سطراً): `REACT_CARE_API_URL`, `REACT_ENABLED_APPS`, `REACT_ALLOWED_LOCALES`, `REACT_RECAPTCHA_SITE_KEY`, … ⇒ صورة Docker واحدة لا تصلح لبيئات متعددة إلا بإعادة بناء (أو `REACT_CARE_URL_MAP` لربط origin→API، `care.config.ts:37-54`).
- `UPGRADING.md`: تغيير كاسر موثق (ملفات `.env` للـ whitelabel منذ v11.3).

## 2. CI/CD (32 ملف workflow)
| Workflow | الغرض |
|---|---|
| `linter.yml` | `typecheck` + `lint --quiet` + `knip` على ARM |
| `unit-tests.yml` | `npm run test:unit` |
| `playwright.yaml` | build + E2E؛ يستنسخ `ohcnetwork/care` ويشغّل حاويات Docker للـ Backend داخل CI (`playwright.yaml` ≈ :100-125) على `main, develop, production` |
| `codeql-analysis.yml`, `ossar-analysis.yml` | فحص أمني |
| `deploy.yaml` | بناء صورة Docker (buildx) |
| `release.yml` | إصدار عند push فروع معينة |
| `cloudflare-preview*.yml` | معاينات PR |
| `pr-*`, `issue-automation`, `labeler`, `stale`… | أتمتة |
| `care-review*.md/.lock.yml`, `daily-playwright-improver*`, `thank-you-note*` | **Agentic workflows (gh-aw)** — لم تُراجع صلاحياتها |

## 3. الترويسات
`nginx.conf:4-12`: HSTS، X-Frame-Options، nosniff، Referrer-Policy، Permissions-Policy، CSP Report-Only. نشر Netlify/Cloudflare يعتمد `_headers` بلا CSP (انظر S-04). ملاحظة: الـ nginx يخدم `expires 7d` لكل المسارات و6h لـ `index.html` — ينبغي التحقق من أن ملفات الأصول مُصمَّمة بـ hash.

## 4. المتطلبات التشغيلية
| متطلب | الحالة |
|---|---|
| Backend CARE (API) | خارجي، مطلوب |
| S3-compatible storage | خارجي (عبر Backend + `CARE_CDN_URL`) |
| تخزين مؤقت/CDN للـ static | غير مُحدد |
| Observability | Sentry اختياري؛ لا metrics |
| Backup/Recovery | **غير ذي صلة بالـ FE** (يقع على Backend/DB) |
| Scaling | SPA ثابتة ⇒ قابلة للتوسع أفقياً بسهولة عبر CDN؛ عنق الزجاجة في الـ API |

## 5. أداء (مؤشرات، لم تُقَس)
حزمة كبيرة محتملة: ≈ 90 تبعية إنتاجية، `react-pdf`/`jspdf`/`recharts`/`reactflow`/`mdxeditor` — قد تتطلب code splitting (لم يُتحقق: لا تحليل حجم). فهرس `src/CAREUI/icons/UniconPaths.json` (4857 سطراً).

## 6. الجاهزية للإنتاج
| الجانب | التقييم |
|---|---|
| FE نفسه | قابل للبناء والنشر، فحوص جودة نظيفة؛ مخاطر مفتوحة: S-01..S-05 |
| النظام CARE كاملاً | **غير قابل للحكم** دون Backend/DB/Infra |
