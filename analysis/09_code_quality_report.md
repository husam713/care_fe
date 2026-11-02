# 09 — Code Quality & Engineering Assessment

## 1. نتائج تنفيذ فعلية (منفصلة عن الفحص الساكن)
| الفحص | الأمر | النتيجة | ملاحظة |
|---|---|---|---|
| Typecheck | `npx tsc --noEmit` | **نجح (exit 0)** | يتطلب أولاً `node scripts/generate-supported-browsers.mjs`؛ بدونه خطأ TS2307 في `BrowserWarning.tsx:7` (ملف مُولَّد مُتجاهَل في git) ⇒ خطوة اعتماد مخفية للمطور الجديد (موثقة في CI `linter.yml`) |
| Lint | `npx eslint ./src --quiet` ثم تقرير JSON كامل بـ `NODE_OPTIONS=--max-old-space-size=8192` | **0 أخطاء، 370 تحذيراً** في 1062 ملفاً | المحاولة الأولى بالذاكرة الافتراضية انهارت (exit 134)؛ نجحت بـ 8GB. أعلى القواعد: `react-hooks/set-state-in-effect` 119، `no-explicit-any` 80، `react-hooks/exhaustive-deps` 60، `react-hooks/incompatible-library` 36، `i18next/no-literal-string` 31، `react-hooks/refs` 17 |
| Dead code | `npx knip` | exit 0 | تلميح واحد: `@/supportedBrowsers` يمكن إزالته من `ignoreUnresolved` في `knip.json` |
| Unit tests | `npm run test:unit` | 20/20 نجحت | ملف اختبار واحد فقط، للـ Vite plugin `autoRegisterComponents` |
| E2E | Playwright | **لم تُنفَّذ** | تحتاج Backend (9000) وبناء إنتاجي |
| Build | `npm run build` | **لم يُنفَّذ** | |
| Audit | `npm audit` | 43 ثغرة (انظر 08) | |

## 2. مؤشرات ساكنة (grep على `src/`)
| المؤشر | العدد |
|---|---:|
| `TODO/FIXME/HACK/XXX` | 31 |
| `eslint-disable` | 34 |
| `@ts-ignore/@ts-expect-error/@ts-nocheck` | **0** |
| استخدام `any` (`: any`, `as any`, `<any>`) | 81 |
| `console.log/warn/error` | 57 |
| ملفات TS/TSX | 1223 (796 + 427) |
| ملفات ≥1000 سطر (غير بيانات) | ≥ 7 |

## 3. نقاط القوة (Verified)
- ‏TypeScript strict، بلا `@ts-ignore`؛ typecheck بلا أخطاء وlint بلا أخطاء (مع 370 تحذيراً).
- مجموعة أدوات جودة متقدمة: knip، husky + lint-staged، قواعد i18n في lint، CodeQL، OSSAR، Snyk، react-doctor.
- طبقة API موحدة ومنمذجة (`query`/`mutate`) ومعالجة أخطاء مركزية.
- اختبارات E2E واسعة: 109 ملف `*.spec.ts` موزعة على facility (billing، patient، settings…)، auth، admin، organization.
- توثيق للوكلاء والمطورين (`AGENTS.md`, `docs/*.md`, `tests/PLAYWRIGHT_GUIDE.md`, `.github/instructions/*`).

## 4. نقاط الضعف
1. **لا اختبارات وحدة للمنطق** (1 ملف فقط، لـ build plugin) ⇒ كل التحقق عبر E2E الثقيلة (تحتاج Backend).
2. **مكونات عملاقة** (3258 سطراً لمحرر الاستبيانات…) ⇒ مخاطر صيانة/اختبار.
3. **ملف مُولَّد مطلوب قبل tsc** (`supportedBrowsers.ts`).
4. **أنواع مكتوبة يدوياً** للـ API بلا توليد من مخطط (انظر 06).
5. `any` ×81 و`console.*` ×57 في كود إنتاجي؛ 370 تحذير ESLint، منها 119 `set-state-in-effect` و60 `exhaustive-deps` (مؤشرات على أنماط effects قابلة لإعادة العمل).
6. حتمية البناء: `Dockerfile` يستعمل `npm install` وبعض optionalDependencies بـ `latest`.
7. اقتران بخطأ/ميزات BE غير جاهزة بتعليقات TODO (`chargeItemDefinitionApi.ts:34`, `CloneQuestionnaireSheet.tsx:98`).

## 5. لم يُقَس
تغطية الاختبارات (coverage)، الدوائر الاعتمادية (circular imports)، التكرار (duplication)، أداء الحزمة (bundle size).
