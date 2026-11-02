# 17 — خطة هندسة العمل بالذكاء الاصطناعي لمشروع CARE (PROPOSAL)

> **الحالة: PROPOSAL للمراجعة. لا شيء هنا قرار معتمد.** كُتبت استجابة لتفكيرك بصوت عالٍ؛ لم يُنشأ مستودع ولم يُنسخ ملف إلى أي مستودع لك. المرجعان المقروءان (قراءة فقط): `husam713/SIRA-Headless` @`2cb12ca` و`husam713/harmonic-incub-analysis` @`9134b9b`. مستودع `sira-workspace` موجود في حسابك لكنني **لم أقرأه**.

## 0. إجابة سؤالك المباشر: محادثة جديدة؟ Opus؟
| السؤال | الجواب (دليل) |
|---|---|
| ما النموذج الذي يخدم هذه الجلسة؟ | `claude-sonnet-5-5` (من بيانات الجلسة: `session_context.model` و`last_served_model`). **تغييرك إلى Opus لم ينعكس على هذه الجلسة.** لا أستطيع تغيير النموذج من داخلها |
| حجم السياق | ≈ 415 ألفاً من 1,000,000 (ضغط تلقائي عند ≈ 900 ألف) ⇒ لا ضغط عاجل |
| هل تلزم محادثة جديدة لهذه المرحلة؟ | **لا** لمرحلة التخطيط والتوثيق. نعم قبل بدء التنفيذ (التعريب/الميزات)، **بعد** أن تصير الحالة مكتوبة في المستودع، لأن المحادثة الطويلة تتحمل خطر فقدان تفاصيل عند الضغط التلقائي |
| متى يفيد نموذج أقوى؟ | أقترح (دون ادعاء قياسات): نموذج أقوى لقرارات المعمارية، ومراجعة الأمن (مثل B-06)، والتحقق المستقل لـ PR؛ ونموذج عادي للأعمال الآلية (codemod، ترجمة مسودة، تشغيل اختبارات). الفصل بين من ينفّذ ومن يراجع (جلستان/نموذجان) يطابق نمط SIRA |
| كيف أبدأ جلسة بنموذج محدد؟ | يمكنني إنشاء جلسة جديدة بنموذج تختاره عند الطلب، أو تبدأها أنت من واجهة التطبيق |

## 1. ما لاحظته في مستودعيك (Verified: قراءة الملفات)
**SIRA-Headless** (756 ملفاً): الذاكرة الدائمة في المستودع لا في المحادثة. العناصر:
- `AGENTS.md` (161 سطراً) قواعد العمل الدائمة: بروتوكول بدء الجلسة المتناسب مع المهمة، تصنيف الأدلة (`CONFIRMED / STRONGLY INFERRED / TRANSFERRED EVIDENCE / UNKNOWN`)، «Architecture Locks»، نموذج التنفيذ `DISCOVER → AUDIT → PLAN → IMPLEMENT → VALIDATE → REVIEW → COMMIT → PUSH → PR → ACCEPTANCE GATE`.
- `CLAUDE.md` (52 سطراً، حدّه الأقصى 60): محوّل Claude Code فقط، يشير إلى `AGENTS.md`.
- `docs/STATE.md` (صفحة واحدة، أول قراءة في كل جلسة) + `project-state.json` (آلي) + `docs/DECISIONS.md` وADRs + `docs/HANDOFF.md`.
- `docs/AI-ENGINEERING-OS.md` (فهرس) و`docs/AI-ENGINEERING-OPERATING-PROTOCOL.md` (494 سطراً، المرجع المعياري).
- قوالب `templates/ai/` (Task Packet، Evidence Envelope، Handoff Packet، Boot Protocol، 4 أدوار: Program Control وLocal Implementation وIndependent GitHub Verification وDomain Governance) ومخططات JSON في `schemas/ai/` (4).
- مستوى استقلالية L3 داخل نطاق Task Packet فقط، و«قائمة توقف» محمية (دمج في main، نشر، DNS، أسرار، عمليات DB مدمّرة)، و`.claude/settings.json` يمنع أشكال الأوامر المحمية.
- `.claude/rules/*.md` قواعد تُحمَّل حسب مسار الملفات.

**harmonic-incub-analysis** (14 ملفاً): مستودع تحليل فقط. `governance/` (مبادئ P1–P10 واصطلاحات) و`registers/` (قرارات، أسئلة، تعارضات، افتراضات، مصطلحات، أصحاب مصلحة) و`sources/` و`domain/capability-map.md`. معرّفات ثابتة (`SRC- Q- CON- ASM- DEC- TERM- CAP- BR- REQ-`)، أنواع معلومة سبعة، وأسبقية: قرار معتمد > متطلب معتمد > مصدر شريك > تأكيد صاحب مصلحة > افتراض > اقتراح AI. «AI يقترح والإنسان يعتمد».

## 2. كيف أفهم طلبك (صحّحني إن أخطأت)
تريد **نظاماً واحداً** يجمع: (أ) تحليل متكاملاً لـ CARE على نمط Harmonic (سجلات ومعرّفات وأسبقية)، و(ب) تنفيذاً منظماً على نمط SIRA (Task Packets وبوابات واختبارات وتسليم بين الجلسات)، بحيث لا تعتمد على ذاكرة المحادثة ولا على أداة بعينها، ويمكنك نقل العمل إلى أي وكيل. وتريد استنساخ **الطريقة** (لا بالضرورة المحتوى) إلى مستودع يحمل تحليل CARE.

## 3. تحويل نمطيك إلى CARE (مقترح)
| مكوّن | مصدره | التكييف مع CARE |
|---|---|---|
| `AGENTS.md` + محوّل `CLAUDE.md` قصير | SIRA | قواعد خاصة بـ CARE: مستودعان، بيانات صحية، قائمة توقف مناسبة (§6) |
| `docs/STATE.md` + `project-state.json` | SIRA | يسجلان: أي commit في `care_fe` و`care`، أي PR مفتوح، أي بوابة مفتوحة، آخر نتائج الاختبارات |
| Task Packet / Evidence Envelope / Handoff Packet + المخططات | SIRA | تنسخ شبه حرفية؛ يتغير حقل `Repository` ليقبل مستودعين، ويضاف حقل «مجموعة الاختبارات المطلوبة» |
| السجلات والمعرّفات (`Q- CON- ASM- DEC- REQ-`…) | Harmonic | تُغذّى من تحليلنا: نتائج `S-*` و`B-*` و`Q-*` الحالية تتحول إلى سجل نتائج بمعرّفات ثابتة؛ `unresolved_questions.md` يصير `registers/questions.md` |
| أسبقية الأدلة | الاثنان | **اختلاف يجب حسمه:** SIRA يصنف ادعاءات هندسية (CONFIRMED…) وHarmonic يصنف أنواع معلومة (FACT/ASSUMPTION/…). اقتراحي: الأول لأي ادعاء عن الكود والاختبارات، والثاني للمتطلبات والقرارات، مع جدول مقابلة |
| الأدوار | SIRA | Program Control (تخطيط)، Implementation (تنفيذ)، Independent Verification (مراجعة بجلسة/نموذج مختلف)، Domain Governance (أمن/خصوصية/مصطلحات سريرية) |
| ملفات القواعد حسب المسار | SIRA | `rules/frontend.md`، `rules/backend.md`، `rules/i18n-rtl.md`، `rules/security-health.md`، `rules/governance.md` |

## 4. أين يعيش النظام؟ (قرار مفتوح، بدائل)
مشكلة جوهرية: `care_fe` و`care` **forks**، و`AGENTS.md`/`CLAUDE.md` فيهما من upstream (وفي `care_fe` تعليمات «star the repo» للوكلاء، S-10). وضع حوكمتنا داخلهما يسبب تعارضات عند المزامنة ويخلط تعليمات upstream بتعليماتك.

| البديل | الوصف | المزايا | العيوب |
|---|---|---|---|
| **A (أوصي به)**: مستودع مظلة `care-workspace` | يحوي الحوكمة والتحليل والسجلات والمهام، و`care_fe` و`care` كـ submodules مثبتة على commits | حوكمتك منفصلة عن upstream؛ نقطة دخول واحدة لأي وكيل؛ يثبّت تركيبة متوافقة من الواجهة والخادم؛ يطابق فكرة workspace | submodules تحتاج انضباطاً؛ مستودع إضافي |
| B: مستودع تحليل فقط على نمط Harmonic (`care-analysis`) والكود في forks منفصلة | مطابق حرفياً لـ Harmonic | بساطة، فصل صارم بين التحليل والتنفيذ | التنفيذ والاختبار بلا حوكمة مشتركة؛ تُكرَّر ملفات الحالة |
| C: داخل `care_fe` | كل شيء في فرع/مجلد | لا مستودع جديد | يتعارض مع upstream؛ يحمل تعليمات غير مرغوبة |

بنية مقترحة للبديل A:
```
care-workspace/
  AGENTS.md  CLAUDE.md(<60 سطراً)  README.md
  docs/STATE.md  project-state.json  DECISIONS.md  adr/  HANDOFF.md
  governance/  principles.md  conventions.md  evidence-classes.md  protected-operations.md  roles/
  registers/   findings.md  questions.md  assumptions.md  conflicts.md  decisions.md  glossary.md
  analysis/    (ينقل من 00–16 + backend/ + diagrams/ + evidence/ كما هي)
  tasks/       (Task Packets)   evidence/ (Envelopes)   handoffs/
  schemas/ai/  templates/ai/   .claude/rules/ + settings.json
  runtime/     RUNBOOK.md  scripts/  (إعادة إنتاج بيئة التشغيل، §5)
  repos/care_fe  repos/care   (submodules)
```

## 5. التوثيق الذي يجعلك مستقلاً عن ذاكرة المحادثة
أهم ما أنتجناه ويضيع مع المحادثة هو **معرفة تشغيل البيئة**. يجب أن يُكتب في `runtime/RUNBOOK.md` مع سكربتات (كلها جرّبناها فعلاً في هذه الجلسة):
1. تشغيل PostgreSQL 16 وRedis، إنشاء قاعدة `care` وكلمة مرور `postgres`.
2. بيئة Python: `uv venv` ثم تثبيت `Pipfile.lock` مع **تعديلين لازمين**: `psycopg-binary` بدل `psycopg[c]` (لا `pg_config`)، و`python-magic==0.4.27` من PyPI (الأصل من فهرس خارجي). أضف حزم dev (`django_extensions`…) لأن `config.settings.local` يحتاجها.
3. `migrate` ثم `load_fixtures` ثم `runserver :9000` بمتغيرات: `DATABASE_URL`, `REDIS_URL`, `DJANGO_DEBUG=True`، ورفع `JWT_ACCESS_TOKEN_LIFETIME` و`JWT_REFRESH_TOKEN_LIFETIME` إلى 720 للتشغيل الطويل (وإلا انتهت التوكنات بعد ≈ 30 دقيقة وفشل E2E).
4. الواجهة: `.env.local` (`REACT_CARE_API_URL`)، `npm ci`، **توليد `src/supportedBrowsers.ts` قبل `tsc`**، `npm run build`.
5. Playwright: `db-snapshot` بعد الـ fixtures، `CI=1` لتشغيل `--project=setup --workers=1` ثم `--project=chromium --workers=4`، وملف إعداد محلي غير مُلتزَم به يضبط مسار Chromium لأن إصدار Playwright لا يطابق المثبّت.
6. **فخاخ**: `pkill -f <نص>` يقتل الصدفة نفسها إن ورد النص في أمرها؛ استخدم `pkill -x` أو معرّفات العمليات. والحاوية قد تعاد تشغيلها فتتوقف الخدمات (الملفات تبقى).
7. خط الأساس الرقمي (Baseline) الحالي لكل بوابة (§7) مسجلاً بتاريخه وcommit.

## 6. قائمة التوقف المقترحة (Protected Operations) لـ CARE
دمج في `develop`/الفرع الافتراضي لأي fork؛ أي دفع أو PR إلى `ohcnetwork/*` (upstream)؛ **أي إجراء على حساب GitHub الشخصي لا يرد في Task Packet** (مثل إضافة نجمة، دروس S-10)؛ نشر/إنتاج؛ عمليات قاعدة بيانات مدمّرة؛ تغيير أسرار أو إعدادات JWT/`SECRET_KEY` في بيئة حقيقية؛ استخدام بيانات مرضى حقيقية في التطوير؛ تعديل تعليمات الوكلاء (`AGENTS.md`/workflows `*.lock.yml`) دون مراجعة؛ حذف أي فرع/وسم استرجاع.

## 7. بوابات الجودة وخط الأساس (Baseline، 2026-11-02)
| البوابة | الأمر | الأساس المقاس |
|---|---|---|
| Typecheck | `npx tsc --noEmit` | exit 0 (بعد توليد `supportedBrowsers.ts`) |
| Lint | `eslint ./src` | 0 أخطاء، 370 تحذيراً (الهدف: عدم الزيادة) |
| Unit (FE) | `npm run test:unit` | 20/20 |
| Build | `npm run build` | exit 0 |
| Backend | `manage.py test --keepdb --parallel 4` | 1991/2005 (14 فشل بيئي: لا bucket) |
| E2E | setup ثم chromium | 379/413؛ ‏15 فشل مستمر في تقارير التشخيص (سبب غير محسوم) + 13 بيئي |
| أمن التبعيات | `npm audit --omit=dev` | 13 → 4 بعد PR #2 |
| i18n | قاعدة `no-literal-string` | 31 تحذيراً |
`PASS` لا يُسجَّل إلا لأمر نُفِّذ فعلاً على commit مذكور (قاعدة SIRA).

## 8. مسار العمل لكل نوع مهمة (مقترح)
- **تحليل:** مصدر/سؤال → استخراج وتصنيف → تحدٍّ → سجل (Q/CON/ASM) → قرار بشري (DEC) → تحديث السجلات (نمط Harmonic).
- **تغيير/ميزة:** طلب → **تحليل أثر** (قائمة الملفات والـ APIs والصلاحيات والـ i18n المتأثرة قبل أي تعديل) → REQ بمعايير قبول قابلة للاختبار → Task Packet → تنفيذ بفرع مستقل → بوابات §7 → مراجعة مستقلة → قبول المالك → تحديث `STATE.md`.
- **تعريب:** خطة 16 تتحول إلى Task Packets (AR-1…AR-10) كل منها بنطاق وبوابات؛ تقع الأرقام والتقويم والمستخدمون في سجل القرارات بعد موافقتك.
- **أمن:** كل ثغرة تأخذ معرّفاً ومالكاً وبوابة إغلاق (إعادة `npm audit`/اختبار).
- مبدأ التسليم: **الجلسة قابلة للاستبدال**، `Task ID` يبقى. أي جلسة تبدأ بقراءة `STATE.md` ثم Task Packet.

## 9. مراحل مقترحة (كل مرحلة بوابة مراجعة منك)
| المرحلة | المحتوى | مخرجات | بوابة |
|---|---|---|---|
| G0 | تأسيس الحوكمة: إنشاء `care-workspace`، نقل التحليل، تحويل النتائج والأسئلة إلى سجلات، `AGENTS.md` و`STATE.md` والقوالب والمخططات، `RUNBOOK.md` | مستودع قابل للتشغيل من أي وكيل | مراجعتك لـ AGENTS/القائمة المحمية/الأسبقية |
| G1 | اختبار النظام: جلسة جديدة (نموذج مختلف) تُجري «تشغيلاً تجريبياً» من `STATE.md` فقط على مهمة صغيرة | تقرير فجوات التوثيق | قبولك |
| A | إغلاق فجوات التحليل: مراجعة ViewSets (B-06)، 15 فشل التشخيص، `git fetch --unshallow` ومقارنة upstream، فحص أسرار، قراءة معمّقة للاستبيانات/الأدوية/الفوترة/المخزون | سجلات محدّثة | قبولك |
| S | أمن: PR #2 (جاهز)، ثم مهام الـ Backend (`SECRET_KEY` إلزامي، HSTS، `ALLOWED_HOSTS`، تفويض افتراضي مغلق) | PRs صغيرة | مراجعة مستقلة |
| L | التعريب (خطة 16) المرحلتان 0 و1 | عربية جزئية قابلة للاستخدام للموظفين | مراجع طبي للمصطلحات |
| F | الميزات الجديدة عبر مسار §8 | حسب REQ | بوابات §7 |

## 10. أسئلة للمراجعة (ليست قرارات)
1. هل البديل A (مظلة) مناسب أم تفضّل B على نمط Harmonic حرفياً؟ وهل `sira-workspace` هو النموذج المقصود للمظلة (لم أقرأه)؟
2. هل توحَّد أنواع الأدلة بين SIRA وHarmonic كما في §3؟
3. من يملك صلاحية القبول لكل نوع (أنت وحدك؟ مراجع طبي؟) وكم دوراً تريد فعلاً؟ قد يكفي Program Control وImplementation وVerification.
4. سياسة الـ upstream: مزامنة دورية أم تفرّع دائم؟ تؤثر على جدوى codemod الـ RTL.
5. هل تريد تثبيت `CLOUD_GITHUB` كملف التنفيذ الافتراضي، علماً أننا أثبتنا أن الحاوية السحابية تشغّل الحزمة كاملة (Postgres/Redis/Playwright)؟
6. حدّ المحادثات: جلسة لكل Task Packet؟

## 11. مدخلات سابقة أعطيتها (مؤقتة، غير مسجّلة كقرارات)
أرقام غربية؛ تقويم ميلادي؛ الموظفون أولاً ثم دراسة المرضى. تُدرج في سجل القرارات فقط بعد موافقتك الصريحة.

## 12. ما لم أفعله عمداً
لم أنشئ مستودعاً، ولم أنسخ ملفات من SIRA/Harmonic إلى أي مكان، ولم أعدّل مستودعيك المرجعيين، ولم أحوّل أياً من أفكارك إلى قرار. كل ما سبق قابل للتعديل أو الرفض.
