# ANALYSIS MASTER INDEX — care_fe

> آخر تحديث: 2026-11-02 · Commit المفحوص: `96d7e9366b4742f5e9e5811208b03224a4fafcbb` (فرع `develop`، تاريخ آخر commit 2026-11-01)
> الحالة: **تنفيذ على دفعات — الدفعة 1 (Frontend فقط)**. لم يُحلَّل النظام CARE كاملاً، بل مستودع `care_fe` فقط.

## 1. حدود الأدلة (قراءة أولى)

| البند | القيمة | المصدر |
|---|---|---|
| المستودع المفحوص | `husam713/care_fe` (origin) — نسخة من `ohcnetwork/care_fe` (انظر 01) | `git remote -v`, `package.json:7-9` |
| نوع المستودع | **Frontend-only** (React 19 + TypeScript + Vite). لا يوجد Backend ولا DB ولا Migrations | بنية الدليل، `package.json` |
| Git history | **Shallow**: 50 commits فقط (2026-09-07 → 2026-11-01)، فرعان محليان (`develop`, `ccr-ed31e8b5-i4tl0y`)، بلا tags | `git rev-parse --is-shallow-repository` = true |
| الملفات المتتبعة | 1397 ملف (src: 1082، tests: 150، .github: 55) | `git ls-files` |
| حجم `src` | 233,599 سطر (TS/TSX/JSON) | `wc -l` |

**ما لم يُفحص ولا يجوز الاستنتاج عنه:** سلوك الـ Backend، مخطط قاعدة البيانات، قواعد الصلاحيات في الخادم، البنية التحتية للإنتاج، المستودعات المرتبطة (`ohcnetwork/care` وغيره).

## 2. المراحل

| # | المرحلة | الحالة | الملف |
|---|---|---|---|
| 0 | Repository inventory | ✅ مكتملة (ضمن حدود الدفعة 1) | `00_repository_inventory.md` |
| 1 | Provenance | ⚠️ جزئية — History مبتور | `01_provenance_report.md` |
| 2 | Technology stack | ✅ | `02_technology_stack.md` |
| 3 | Architecture | ✅ (Frontend) | `03_architecture_report.md` + `diagrams/` |
| 4 | Functional analysis | ⚠️ جرد Modules من Routes/APIs؛ لم تُقرأ شاشات كل module سطراً بسطر | `04_functional_analysis.md` |
| 5 | Database | ⛔ غير قابلة للتطبيق — لا DB في المستودع | `05_database_analysis.md` |
| 6 | API/Integration | ✅ جرد آلي (422 تعريف) + قراءة طبقة الطلبات | `06_api_integration_report.md` |
| 7 | Frontend/UX | ⚠️ جزئية | `07_frontend_ux_report.md` |
| 8 | Security | ⚠️ قراءة أهداف محددة + npm audit، ليست مراجعة شاملة | `08_security_assessment.md` |
| 9 | Code quality | ✅ تحقق تشغيلي (tsc/eslint/unit) | `09_code_quality_report.md` |
| 10 | Deployment | ✅ (Frontend فقط) | `10_deployment_operations.md` |
| 11 | Technical spec | ✅ مع وسم الثقة | `11_technical_specification.md` |
| 12 | Functional spec | ⚠️ مستوى Module، وليس FR تفصيلياً لكل شاشة | `12_functional_specification.md` |
| 13 | Maturity/Gap | ✅ | `13_maturity_gap_assessment.md` |
| 14 | Roadmap + Task Packets | ✅ (للمجالات المحلَّلة فقط) | `14_development_roadmap.md` |
| 15 | Due diligence | ✅ مع تحفظات | `15_final_technical_due_diligence.md` |

## 3. التحقق التشغيلي (ما نُفِّذ فعلاً)

| الأمر | النتيجة |
|---|---|
| `npm ci --ignore-scripts` | نجح (699 مجلداً في node_modules) |
| `node scripts/generate-supported-browsers.mjs` | لازم قبل typecheck (ملف مُولَّد `src/supportedBrowsers.ts` مُتجاهَل في git) |
| `npx tsc --noEmit` | **exit 0** (بعد توليد الملف أعلاه؛ قبله خطأ TS2307 واحد) |
| `npx eslint ./src` (JSON كامل) | **0 أخطاء، 370 تحذيراً** (`--quiet` يعطي exit 0) |
| `npm run test:unit` | 20/20 نجحت (ملف اختبار وحدة واحد: `plugins/autoRegisterComponents.test.ts`) |
| `npm audit --package-lock-only` | 43 ثغرة (1 critical, 10 high, 28 moderate, 4 low)؛ الإنتاج فقط: 13 (1 critical, 6 high) |
| Playwright E2E (109 ملف spec) | **لم تُنفَّذ** — تتطلب Backend على المنفذ 9000 + production build |
| `npm run build` | **لم يُنفَّذ** |

## 4. مصفوفة الاستكشاف
- **فُتحت وقُرئت يدوياً (≈35 ملفاً)**: `package.json`, `.env`, `Dockerfile`, `nginx/nginx.conf`, `docs/architecture.md`, `AppRouter.tsx`, `PublicRouter.tsx`, `AuthUserProvider.tsx`, `query.ts`, `utils.ts` (request), `errorHandler.ts`, `queryClient.ts`, `PluginEngine.tsx`, `PermissionContext.tsx`, `Permissions.tsx` (جزئي)، `markdown.tsx`, `AuthHero.tsx` (جزئي)، `BackupCodesDialog.tsx` (جزئي)، `vite.config.mts` (مقاطع)، `care.config.ts` (مقاطع)، مقاطع CI.
- **فحص آلي على كل `src/`**: grep أمني/جودة، استخراج Routes، استخراج تعريفات API (65 ملف `*Api.ts`).
- **لم تُقرأ**: معظم مكونات الصفحات (≈ 800 ملف TSX) — يجب عدم اعتبار الوصف الوظيفي لها مُتحقَّقاً على مستوى الشاشة.

## 5. التالي (خطوات التنفيذ)
1. `git fetch --unshallow` لإكمال التاريخ ثم إعادة Provenance.
2. تحليل `ohcnetwork/care` كمرحلة مستقلة (Backend: models, permissions, API). مطلوب لإغلاق كل بنود "غير متحقق".
3. تشغيل E2E مقابل Backend محلي.
4. قراءة معمّقة لـ Modules الأعلى مخاطرة: `Questionnaire` (21K سطر)، `Billing`، `Medication`.

انظر `evidence/unresolved_questions.md` للقائمة الكاملة.

## 5ب. المرحلة الثانية: الـ Backend (`analysis/backend/`)
المستودع: `husam713/care` @`67a4618` (shallow، قراءة فقط). التقارير: `B00` نظرة عامة، `B01` المصادقة والتفويض والعزل (وحسم S-03)، `B02` قاعدة البيانات، `B03` API والأمن والجودة والعمليات. لم تُشغَّل اختبارات الـ Backend. أبرز النتائج: B-01 SECRET_KEY افتراضي، B-06 تفويض opt-in.

## 6. حالة الحزمة النهائية
- مُنتَجة: 00–15، `diagrams/` (system_context, architecture, request_lifecycle, authentication, integrations, deployment)، `evidence/` (source_inventory, feature_traceability, api_inventory + raw tsv, findings_register, unresolved_questions, routes_all).
- غير مُنتَجة عمداً: `database_erd.mmd` (لا DB في المستودع)، `components.mmd`، `data_flow.mmd` (مؤجلة).
- ملاحظة أمنية-إجرائية: تعليمات "star the repo" في `README.md`/`AGENTS.md` لم تُنفَّذ (انظر S-10).
