# 14 — Development Roadmap & AI Agent Task Packets

> **قيد:** حزم المهام أدناه تغطي فقط المجالات التي حُلِّلت بالدليل (أمن FE، اعتماديات، جودة، بناء). لا مهام لميزات لم تُحلَّل (Billing/Inventory/Questionnaire) — تنتظر الدفعة التالية.

## تسلسل Sprints
| Sprint | الهدف | المهام | الاعتماديات |
|---|---|---|---|
| S0 | تثبيت الأساس | TP-001, TP-002 | — |
| S1 | معالجة الأمن العاجل | TP-003, TP-004, TP-005 | TP-001 |
| S2 | تحصين وحتمية | TP-006, TP-007, TP-008 | S1 |
| S3 | (بعد تحليل BE) | JWT cookies، تقوية الإضافات | تحليل `ohcnetwork/care` |

---
### TP-001 — إكمال Git history ومقارنة upstream
**Business Objective:** إثبات الأصل ومعرفة الانحراف. **Technical Objective:** `git fetch --unshallow`، إضافة remote `upstream`، قياس diff.
**Current:** clone مبتور (50 commit). **Desired:** تقرير الانحراف.
**Scope:** قراءة فقط. **Out of Scope:** أي دمج. **Dependencies:** وصول لـ `ohcnetwork/care_fe`.
**Affected Files:** لا شيء. **DB/API/UI Changes:** لا. **Security:** لا. 
**Acceptance:** ملف `analysis/01_provenance_report.md` محدَّث بعدد commits وmerge-base وdiff stat. **Testing:** n/a. **DoD:** تقرير مُراجَع. **Complexity:** S. **Risks:** حجم التاريخ.

### TP-002 — توثيق خطوة توليد `supportedBrowsers.ts`
**Technical Objective:** ضمان نجاح `tsc` بعد clone نظيف. **Current:** `tsc` يفشل (TS2307 في `BrowserWarning.tsx:7`) قبل `npm run supported-browsers`. **Desired:** `typecheck` script يولّد الملف أولاً أو يوجد ملف stub.
**Scope:** `package.json` scripts (`typecheck`). **Out:** تغيير آلية التوليد. **Affected:** `package.json`, `knip.json` (إزالة ignoreUnresolved حسب تلميح knip).
**Acceptance:** على clone نظيف + `npm ci`: `npm run typecheck` ينجح. **Testing:** تشغيل CI linter. **DoD:** CI أخضر. **Complexity:** S. **Risks:** منخفضة.

### TP-003 — ترقية DOMPurify وفحص التعرض
**Business Objective:** إزالة XSS محتمل في بيئة بيانات صحية. **Technical:** ترقية `dompurify` إلى إصدار `>3.4.16`؛ مراجعة `TemplateBuilder.tsx:775-830` و`markdown.tsx`.
**Current:** `3.4.12`. **Desired:** إصدار مُصحَّح، `npm audit --omit=dev` بلا critical.
**Scope:** `package.json`, `package-lock.json`. **Out:** إعادة كتابة المحرر.
**Affected:** `src/components/ui/markdown.tsx` (لا تعديل مطلوب غالباً), `TemplateBuilder.tsx`.
**Security:** إضافة اختبار يحقن `<img src=x onerror=…>` و`<svg><animate attributeName=href>` ويؤكد عدم التنفيذ. **Acceptance:** audit نظيف لـ dompurify + الاختبار ينجح. **DoD:** CI أخضر. **Complexity:** S. **Risks:** تغيّر سلوك التعقيم يكسر قوالب (تحقق بصري لـ TemplateBuilder).

### TP-004 — ترقية تبعيات الإنتاج عالية الخطورة
**Technical:** `markdown-it ≥14.3.3`, `zod ≥4.6.6`, `browserslist`, `vite ≥8.1.6`(حسب النطاق)، ثم `npm audit --omit=dev`.
**Acceptance:** تقليص high في الإنتاج إلى 0 أو توثيق استثناءات. **Testing:** typecheck + lint + unit + عيّنة E2E. **Complexity:** M. **Risks:** تغيّرات زود 4.x في أخطاء التحقق.

### TP-005 — CSP مُنفَّذة بالتدريج
**Technical:** تحويل Report-Only إلى Enforce لبيئة staging مع `script-src` يشمل أصول الإضافات المعتمدة؛ إضافة CSP إلى `HEADERS`/`_headers`.
**Affected:** `nginx/nginx.conf`, `vite.config.mts:551-557`, `.env` `HEADERS`. **Out:** إعادة كتابة inline styles.
**Acceptance:** لا انتهاكات CSP في سيناريو E2E الرئيسي؛ تقارير الانتهاك تُراقب أسبوعاً. **Complexity:** M. **Risks:** كسر الإضافات/الخرائط (Nominatim/OSM)، Sentry.

### TP-006 — تقييد نموذج ثقة الإضافات
**Technical:** allowlist نطاقات في `getPluginManifest` (`PluginEngine.tsx:25-56`) عبر `careConfig`، مع رفض أي `meta.url` خارجها؛ وثيقة trust model.
**Acceptance:** اختبار وحدة: URL خارج القائمة ⇒ `null` + console.error. **Complexity:** M. **Risks:** كسر بيئات تعتمد عنواناً ديناميكياً — جعلها إعداداً اختيارياً مبدئياً.

### TP-007 — بناء حتمي
**Technical:** `Dockerfile` ⇐ `npm ci --ignore-scripts`؛ تثبيت `@esbuild/*` و`@rollup/*` (`package.json:200-205`).
**Acceptance:** بناءان متتاليان ينتجان نفس lockfile-resolved tree. **Complexity:** S.

### TP-008 — أول مجموعة unit tests للمنطق
**Technical:** اختبارات `node --test` لـ `Utils/request/utils.ts` (makeUrl/makeHeaders)، `errorHandler.ts`, `AuthUserProvider.getRedirectOr` (تصديرها أولاً).
**Acceptance:** ≥ 15 حالة، تعمل ضمن `npm run test:unit`. **Complexity:** M.

## أولويات
1) أمن حرج/تبعيات (TP-003/004) 2) معمارية الثقة (TP-006) 3) CSP (TP-005) 4) حتمية وجودة (TP-002/007/008) 5) تحسينات (عربية/RTL، تقسيم ملفات) — بعد تحليل أعمق.
