# 09 — Code Quality & Engineering Assessment

## 1. نتائج تنفيذ فعلية (منفصلة عن الفحص الساكن)
| الفحص | الأمر | النتيجة | ملاحظة |
|---|---|---|---|
| Typecheck | `npx tsc --noEmit` | **نجح (exit 0)** | يتطلب أولاً `node scripts/generate-supported-browsers.mjs`؛ بدونه خطأ TS2307 في `BrowserWarning.tsx:7` (ملف مُولَّد مُتجاهَل في git) ⇒ خطوة اعتماد مخفية للمطور الجديد (موثقة في CI `linter.yml`) |
| Lint | `npx eslint ./src --quiet` ثم تقرير JSON كامل بـ `NODE_OPTIONS=--max-old-space-size=8192` | **0 أخطاء، 370 تحذيراً** في 1062 ملفاً | المحاولة الأولى بالذاكرة الافتراضية انهارت (exit 134)؛ نجحت بـ 8GB. أعلى القواعد: `react-hooks/set-state-in-effect` 119، `no-explicit-any` 80، `react-hooks/exhaustive-deps` 60، `react-hooks/incompatible-library` 36، `i18next/no-literal-string` 31، `react-hooks/refs` 17 |
| Dead code | `npx knip` | exit 0 | تلميح واحد: `@/supportedBrowsers` يمكن إزالته من `ignoreUnresolved` في `knip.json` |
| Unit tests | `npm run test:unit` | 20/20 نجحت | ملف اختبار واحد فقط، للـ Vite plugin `autoRegisterComponents` |
| Build | `npm run build` | **نجح (exit 0)**، Vite أنهى البناء في ≈ 15 ثانية، مع تحذير حجم chunks | حجم الحزمة لم يُحلَّل |
| E2E (Playwright) | `playwright test --project=chromium --workers=4 --retries=1` (مقابل Backend محلي على PostgreSQL، بعد `--project=setup --workers=1`) | **379 نجح، 29 فشل، 5 متذبذبة، 10 لم تعمل** من 413 (18 دقيقة) | انظر §6 |
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
## 6. نتائج E2E الفعلية (2026-11-02، بيئة سحابية، Chromium مثبّت مسبقاً)

**الإعداد:** Backend `husam713/care` @67a4618 على PostgreSQL 16 وRedis، بيانات `load_fixtures`، مدد JWT مرفوعة إلى 720 دقيقة (انتهاء التوكنات كان يُفشل التشغيل الأول بعد نحو 30 دقيقة)، واجهة من `npm run build` عبر `vite preview`، وضع `CI=1` (setup بـ worker واحد ثم 4 workers). استُخدم ملف إعداد محلي غير مُلتزَم به يضبط مسار Chromium لأن إصدار Playwright (1.61.1) يتوقع بناءً مختلفاً عن المثبّت.

| النتيجة | العدد |
|---|---:|
| setup | 8 / 8 نجحت |
| نجحت | 379 |
| فشلت | 29 |
| متذبذبة (نجحت بعد إعادة) | 5 |
| لم تعمل | 10 |

**تصنيف الفشل (أولي، لم يُحقَّق يدوياً كل حالة):**
| الفئة | الملفات | التقدير |
|---|---|---|
| بيئي: متصفح | `ServiceRequestCreate.spec.ts` (4) تفشل بـ `browserType.launch: Executable doesn't exist` لأن الاختبار يشغّل متصفحاً بنفسه خارج إعداد المشروع | بيئي (مسار Chromium) |
| بيئي: تخزين ملفات | `patientFiles` (4)، `userAvatar` (3)، `facilityImage` (2) | يطابق فشل 14 اختبار رفع في الـ Backend (لا bucket/MinIO) |
| غير محسوم | `DiagnosticReportReview` (7) تنتظر زر `Approve`، و`ServiceRequestReportWorkflow` (2)، `AdditionalReportObservations` (1)، `MultipleDiagnosticReports` (1)، `encounterShortcuts` (2)، وغيرها بمفرده | قد تكون حقيقية أو حساسة للحمل (حمل المعالج وصل ≈ 16 على 4 أنوية) أو لبيانات؛ **تحتاج إعادة تشغيل منفردة** |
| متذبذبة | `role.spec.ts`، `encounterFutureDateRestriction`، `encounterPlannedStatusTransition`، `enableWhen/boolean`، `observationDefinition` | حساسية للتوقيت والحمل |

**الخلاصة:** ≈ 92% نجاح (379/413) في بيئة محملة ومن دون تخزين ملفات. أقل من نصف حالات الفشل مبررة بيئياً؛ الباقي لم يُصنَّف نهائياً. لا يصح اعتبارها عيوباً في الكود قبل إعادة التشغيل منفردة.

## 7. نتائج اختبارات الـ Backend (Django)
`python manage.py test --keepdb --parallel 4` بإعدادات `config.settings.test`: **2005 اختبار في 56.6 ثانية، 1991 نجح، 14 فشل** (13 خطأ + فشل واحد). كل الـ14 في رفع الملفات (`test_file_upload_api`، `test_user_api` صور الملف الشخصي، `test_consent_api.test_for_attachments`) بسبب `ValueError: Invalid endpoint:` (لا إعداد bucket/MinIO) ثم خطأ ثانوي في قالب الخطأ (`favicon.ico` غير موجود لأن `collectstatic` لم يُشغَّل). أسباب بيئية لا عيوب كود؛ لم أتحقق بتشغيلها مع MinIO.
