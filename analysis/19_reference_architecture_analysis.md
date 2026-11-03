# 19 — تحليل المستودعات المرجعية وهندسة العمل بالذكاء الاصطناعي لـ CARE

> **الحالة: مقترح للمراجعة، لا قرار.** تحقيق للقراءة فقط. لم يُنشأ مستودع ولا فرع ولا PR، ولم يُطلق أي وكيل، ولم يُعدَّل كود تطبيق. الخطة السابقة `17_ai_engineering_architecture_plan.md` محفوظة كما هي وتُراجَع في §8.
>
> **وسوم الادعاءات:** `Verified` (قُرئ في ملف أو نُفِّذ أمر) · `Inferred` (استنتاج من أدلة، مع حدوده) · `Proposed` (اقتراح مني) · `Requires approval` (قرار لك).

## 0. تصحيح لافتراض في طلبك، وحدود التغطية

الطلب يفترض أن قراءتي السابقة اقتصرت على أشجار المجلدات وأسماء الملفات. **هذا غير دقيق (Verified):** قبل هذا التقرير كنت قد قرأت محتوى `AI-ENGINEERING-OS.md` و`BOOT-PROTOCOL.md` وقوالب Task/Evidence/Handoff و`AGENTS.md` (جزئياً) و`CLAUDE.md` و`governance.md` في SIRA، و`README` و`CLAUDE.md` و`conventions` و`principles` في Harmonic. لكنها كانت **قراءة جزئية** فعلاً، ولذلك وسّعتها هنا كما يلي.

| المستودع | الحالة المتحقَّق منها | ما قُرئ في هذا التقرير |
|---|---|---|
| `husam713/SIRA-Headless` (الرابط الفعلي `husam713/sira-headless`) | فرع `main` @ `2cb12ca`، clone سطحي (commit واحد، فلا يمكن دراسة تطور النمط عبر الزمن)، 756 ملفاً | `AGENTS.md` كاملاً، `CLAUDE.md`، `docs/AI-ENGINEERING-OPERATING-PROTOCOL.md` كاملاً (494 سطراً)، `AI-ENGINEERING-OS.md`، `STATE.md`، `ACCEPTANCE-GATES.md`، `GITHUB-GOVERNANCE.md`، `DECISIONS.md` (الرأس)، بنية `project-state.json`، رأس `HANDOFF.md`، `templates/ai/*` (5 قوالب + 4 overlays)، `schemas/ai/*` (4 مخططات: الحقول والـ enums)، `.claude/settings.json`، `.claude/rules/*` (4)، `.github/pull_request_template.md`، قائمة workflows، مثالان من `docs/tasks`، ADR-026 (الرأس) |
| `husam713/harmonic-incub-analysis` | فرع `main` @ `9134b9b`، clone سطحي، 14 ملفاً | **كل الملفات الـ 14** (الأكبر 73 سطراً) |
| `husam713/care_fe` | فرع `ccr-ed31e8b5-i4tl0y` @ `56e39aa` (أصل `develop` @ `96d7e93`) | ملفات الوكلاء الموجودة: `AGENTS.md`، `CLAUDE.md`، `.github/copilot-instructions.md`، `.github/instructions/*`، `.cursorrules`، `.entire/*`، `.codex/*`، `.github/hooks`، رأس `care-review.md` و`care-review.agent.md` |
| `husam713/care` | `develop` @ `67a4618`، clone سطحي | `CLAUDE.md`، قائمة `.github` |

**لم يُقرأ (Verified أنه غير مفحوص):** `docs/HANDOFF.md` (1324 سطراً)، `SOURCE-OF-TRUTH.md` (414)، `PROJECT-STATE.md` (342)، معظم `docs/STEP-*.md` و`docs/tasks/*` (قُرئ اثنان من 17)، `tools/*`، `frontend/tests/contract`، `artifacts/**`، `.github/workflows/*.yml` في SIRA (قُرئ وصفها في GITHUB-GOVERNANCE فقط)، `ROLE-OVERLAYS` (قُرئ رأس كل واحد). لم أقرأ `sira-workspace` ولا `sira-analysis` ولا `sira-group-reporting` (موجودة في حسابك لكنها خارج المرجعين المحددين؛ **`sira-workspace` تحديداً قد يحسم سؤال المظلة، ويلزم إذنك لقراءته**).

## 1. تحليل الأنماط في المستودعين المرجعيين (Phase A)

### 1.1 SIRA-Headless
| # | النمط | المصدر | ما يفعله ولماذا | نقاط القوة | الحدود | قابل لإعادة الاستخدام؟ | وسم |
|---|---|---|---|---|---|---|---|
| S1 | المستودع كذاكرة دائمة | `AGENTS.md` (الافتتاح)، `docs/AI-ENGINEERING-OS.md` «Purpose» و«Durable memory model» | ما يكفي ليعيد وكيل آخر بناء المشروع دون المحادثة | يحقق هدفك الأساسي | يعتمد على أن الملفات محدّثة | **نعم، جوهري** | Verified |
| S2 | بدء جلسة متناسب مع المهمة | `AGENTS.md` «Session Boot Protocol»، `CLAUDE.md` «Start of every session»، `templates/ai/BOOT-PROTOCOL.md` | جلسة عادية: `AGENTS.md` + `STATE.md` + Git؛ boot كامل فقط للحوكمة/الاسترجاع/العمليات المحمية | يوفر السياق | يتطلب أن `STATE.md` صادق | نعم | Verified |
| S3 | حاملات الحالة (state carriers) | `STATE.md` (74 سطراً)، `project-state.json` (704)، `PROJECT-STATE.md` (342)، `HANDOFF.md` (1324)، `SOURCE-OF-TRUTH.md` (414) | حالة آلية وبشرية وسجل تاريخي | الشمول | **التكرار يسبب انحرافاً:** `STATE.md:73` يعترف بأن الحاملات الأخرى «ما زالت تصف #64–#66 كفروع غير مدموجة»؛ والخطوة التالية المعلنة فيه هي توحيدها | **جزئياً**: حاملان فقط | Verified |
| S4 | تصنيف الادعاءات | `AGENTS.md` «Evidence Classification»؛ البروتوكول §«Claim and evidence classification» | `CONFIRMED / STRONGLY_INFERRED / TRANSFERRED_EVIDENCE / UNKNOWN` | يمنع الجزم دون دليل | لا يغطي القرارات والمتطلبات | نعم للمطالبات الهندسية | Verified |
| S5 | هرم سلطة الأدلة (10 مستويات) | البروتوكول §«Evidence authority hierarchy» | تنفيذ/وقت تشغيل > حالة Git > PRs مدموجة > CI > مخططات > ADR > مراجع > تقارير وكلاء > ذاكرة المحادثة > استنتاج النموذج | صريح ومرتب | طويل نسبياً | نعم، يُقتبس | Verified |
| S6 | فصل أبعاد: Authorization / Evidence Class / Acceptance / Canonical | البروتوكول §«Authorization, acceptance, merge state, and canonicality»؛ `canonical` boolean صريح | CI أخضر ≠ قبول؛ Draft PR ≠ قبول؛ الدمج ≠ قبول المالك | يمنع «الاعتماد الذاتي» | مفردات كثيرة لمالك واحد | نعم بتبسيط | Verified |
| S7 | Task Packet | `templates/ai/TASK-PACKET.md` + `schemas/ai/task-packet.schema.json` (204 سطراً، 19 حقلاً مطلوباً) | تفويض محدود + عقد النتيجة: baseline، نطاق، استثناءات، **شروط توقف**، تحقق، البوابة التالية | حدود واضحة | **لا تطبيق فعلي مطابق:** من 17 ملفاً في `docs/tasks/` لا يحتوي أي منها حقول القالب حرفياً (`Schema version`، `Execution profile`، `Stop conditions`، `Authorization text`، `Result contract` = 0 ملف لكل منها). أقرب مثال (`atlas-motion-port.md`) يستخدم ترويسة حرة: «Task: TP-… · Role · Profile» | القالب نعم، والمخطط يحتاج قراراً | Verified |
| S8 | Evidence Envelope | `templates/ai/EVIDENCE-ENVELOPE.md` + مخطط (104 أسطر) | ادعاء الوكيل عن نتيجته مع الصنف والقبول وcanonical | يفصل «ما أقوله» عن «ما أثبت» | **لم أجد أي نسخة فعلية** في `docs/` أو `artifacts/` (غير القالب وذكره في الوثائق) | يُدمج مع Handoff (§3) | Verified |
| S9 | Handoff Packet | `templates/ai/HANDOFF-PACKET.md` + مخطط (128 سطراً) | نقل بين الأدوار: baseline، إجراءات، ملفات، تحقق، مؤكد/مستنتج/تقرير-فقط/مجهول | يدعم الاستبدال بين الجلسات | `docs/HANDOFF.md` (1324 سطراً) شيء مختلف: وثيقة دخول لجلسة جديدة وليست Packets | نعم | Verified |
| S10 | Artifact Manifest | `schemas/ai/artifact-manifest.schema.json` + نسختان فعليتان (`artifacts/step-4/gj-visual-*-manifest.json`: `schemaVersion`، `sha256`، `size`) | جرد أدلة (لقطات) بتجزئات | **التطبيق الوحيد الفعلي للمخططات** | لا يُتحقق منه آلياً | نعم، مؤجل | Verified |
| S11 | ملفا تنفيذ `LOCAL` / `CLOUD_GITHUB` | البروتوكول §«Execution profiles»؛ مثال: `atlas-motion-port.md` (`LOCAL`، `localEvidenceRequired: true`) | يحدد أي أدلة يمكن للوكيل ادعاؤها | يمنع اختلاق حالة محلية | تعريف السحابة = أدلة المستودع/GitHub فقط، وهذا **لا يطابق حالتنا** (§5) | بتعديل | Verified |
| S12 | الأدوار | البروتوكول §«Operating roles» + 4 overlays: Program Control، Independent GitHub Verification، Domain/Design Governance، Implementation (دور منطقي واحد) | فصل التخطيط عن التنفيذ عن التحقق | بساطة: **دور تنفيذ واحد** | لا يعرّف تنفيذاً متوازياً ولا نماذج مختلفة | نعم بتصغير | Verified |
| S13 | قائمة التوقف والعمليات المحمية | `AGENTS.md` «Protected Operations»؛ `.claude/settings.json` (allow/deny/ask، وhook) | دمج/نشر/DNS/أسرار/عمليات DB مدمرة تحتاج المالك؛ أنماط أوامر تُرفض آلياً | **تطبيق ملموس بالأوامر** | يقرّ `GITHUB-GOVERNANCE.md` بأن الوكيل يعمل بتوكن المالك فلا يميّزه GitHub، فيبقى «الوكيل لا يدمج» قاعدة جهة العميل لا منصة | نعم، يُكيَّف | Verified |
| S14 | قواعد حسب المسار | `.claude/rules/{frontend,backend,graphql,governance}.md` بترويسة `paths:` | تُحمَّل عند لمس مسار معين: أوامر التحقق، الأقفال المعمارية | سياق مضغوط | تعمل داخل المستودع نفسه فقط | نعم | Verified |
| S15 | مفردات حالة التحقق | `AGENTS.md`، `PR template`، `ACCEPTANCE-GATES.md` | `PASS FAIL WARNING DEFERRED NOT RUN BLOCKED NOT APPLICABLE`؛ PASS لأمر نُفِّذ فعلاً | صدق التقارير | لا تعبّر عن «فشل معروف مسبقاً» | نعم + حقل Known-failure | Verified |
| S16 | بوابات القبول A–F | `docs/ACCEPTANCE-GATES.md` | مصدر مكتمل ← تحقق محلي ← Git ← CI ← تشغيل ← إنتاج | تدرج واضح | مرتبطة بأوامر pnpm الخاصة بـ SIRA | قالب عام، أوامر خاصة | Verified |
| S17 | سجل قرارات ADR | `docs/DECISIONS.md` (35 ADR قصيرة بحالة) + `docs/adr/` (3 طويلة) | قرارات «لا تُعاد مناقشتها دون دليل جديد» | بسيط | الاثنان معاً يوزّعان القرار | نعم: قصيرة فقط الآن | Verified |
| S18 | استرجاع الجلسة المقطوعة وbaseline lock | `BOOT-PROTOCOL.md` «Baseline lock» و«Interrupted-session recovery»؛ `BLOCKED_BASELINE_DRIFT` و`BLOCKED_SCOPE_EXPANSION_REQUIRED` | لا يُعاد تنفيذ المهمة عمياً؛ يتوقف عند الانجراف | يحمي من الازدواج والتجاوز | إجراء ثقيل | نعم | Verified |
| S19 | CI كدليل تنفيذي | `GITHUB-GOVERNANCE.md`: `frontend-ci.yml` (148 سطراً)، `backend-ci.yml` (87)، قالب PR (74)؛ rulesets مذكورة | بوابات آلية | موثّق | **لا يوجد أي CI أو سكربت يتحقق من ملفات الحوكمة أو المخططات** (Verified: لا `ajv`/`jsonschema` ولا ذكر لها في workflows)، وSIRA نفسه يقول إن ذلك خارج النطاق المصرّح به (`AI-ENGINEERING-OS.md` «Enforcement boundary») | لاحقاً | Verified (الـ rulesets نفسها: TRANSFERRED، لم أستعلم GitHub) |
| S20 | نموذج توزيع النماذج والتنفيذ المتوازي | بحث (`opus|sonnet|haiku|subagent|parallel|worktree|orchestrat`) في `AGENTS.md`، `CLAUDE.md`، `docs`، `templates`، `schemas`، `.claude` | **غير موجود كنمط.** لا سياسة لاختيار النموذج. وردت إشارتان فقط: «use an Explore agent» في `CLAUDE.md`، وuse عابر لوكيل فرعي في `docs/tasks/atlas-motion-port.md` عبر worktree | — | لا يقدّم أساساً لتوزيع النماذج | **لا** (ليس من SIRA) | Verified |

### 1.2 Harmonic (مستودع تحليل فقط)
| # | النمط | المصدر | ما يفعله | القوة | الحدود | قابل لإعادة الاستخدام؟ |
|---|---|---|---|---|---|---|
| H1 | «المستودع هو الحقيقة» | `README.md`، `CLAUDE.md`، `governance/principles.md` P5 | المحادثة ليست مصدراً | واضح وقصير (29 سطراً) | مصمم لمحلل أعمال واحد | نعم |
| H2 | سجلات بمعرّفات ثابتة | `governance/conventions.md`: `SRC- STK- TERM- Q- CON- ASM- DEC- CAP- BR- REQ-`، لا إعادة استخدام ولا حذف | تتبع وإحالة | يناسب التحليل | بلا ربط بكود/اختبارات | نعم |
| H3 | 7 أنواع معلومة | `conventions.md` (FACT/ASSUMPTION/QUESTION/CONFLICT/PROPOSAL/DECISION/REQUIREMENT) | تمييز المقترح من المعتمد | بساطة (DEC-003 يبرر 7 بدل 12) | **يختلف عن تصنيف SIRA** | نعم للمتطلبات والقرارات |
| H4 | أسبقية المصادر | `principles.md`، `CLAUDE.md` | DEC معتمد > REQ معتمد > مصدر شريك > تأكيد > افتراض > اقتراح AI | يحسم التعارض | يغطي وثائق الأعمال لا الكود | نعم مع هرم SIRA للأدلة |
| H5 | AI يقترح والإنسان يعتمد | `principles.md` P4 + جدول الصلاحيات؛ `CLAUDE.md` «Never promote ASSUMPTION/PROPOSAL…» | صلاحيات واضحة | مطابق لهدفك | — | نعم |
| H6 | لا حل صامت للتعارض | P2، `registers/conflicts.md` (10 تعارضات، كل منها بـ FACT/قراءات ممكنة/من يقرر) | تسجيل التعارض | قيّم لتعارض الكود/الوثائق | — | نعم |
| H7 | قالب قرار DEC | `conventions.md` | Options/Decision/Rationale/Consequences/Impacts | موجز | — | نعم |
| H8 | «Register updates» في نهاية كل جلسة | `CLAUDE.md` | تسليم قابل للالتزام | يُبقي السجلات حية | يعتمد على انضباط الوكيل | نعم |
| H9 | الحد الأدنى من الأدوات | P10، DEC-002 («Revisit when» بمعايير قياس) | لا أداة قبل ألم متكرر | **يعارض ثقل SIRA** | — | نعم كقاعدة تصميم |
| H10 | الوثيقة المولَّدة مخرَج لا مصدر | P6 | PDF/عرض يولَّد من السجلات | يمنع نسخاً متعارضة | — | نعم |
| — | **قيد:** لا ربط بالكود | — | لا Task Packets ولا اختبارات ولا baseline | — | يلزم جسر لتنفيذ CARE | — |

(`AGENTS.md` في Harmonic مجرد مؤشر إلى `CLAUDE.md`، بعكس SIRA حيث `AGENTS.md` هو الأصل و`CLAUDE.md` محوّل. وهذا مثال على تباين البنية بين مستودعيك.)

## 2. العام مقابل الخاص

| فئة | عام (يُعاد استخدامه) | خاص بالمشروع (لا يُنسخ) |
|---|---|---|
| SIRA | فصل الأبعاد، Task Packet/Handoff، boot متناسب، baseline lock، مفردات التحقق، قائمة التوقف، قواعد المسار، hook فحص المسافات، قالب PR | WordPress/WPGraphQL/Next، ADR-001..035، أقفال المعمارية (`Architecture Locks`)، ADR-030 seed، `tools/` (SSH/CMS)، Cloud Run، DNS، مفاتيح HMAC/Application Password، حدود tenant |
| Harmonic | معرّفات وسجلات، 7 أنواع، أسبقية، قالب DEC، Register updates، تسجيل التعارضات | شركاء الحاضنة ومصادرهم `SRC-001..006`، `CAP-*`، المسرد |

## 3. معمارية هندسة عمل عامة مقترحة (Phase B) — `Proposed`

طبقات (كل طبقة ملف أو اثنان، لا أكثر):

| طبقة | الغرض | ملف/قالب | المصدر |
|---|---|---|---|
| L0 دستور | قواعد دائمة، أقفال معمارية، قائمة التوقف | `AGENTS.md` + محوّل `CLAUDE.md` (<60 سطراً) | SIRA S1/S13 |
| L1 حالة | أين نحن الآن | `docs/STATE.md` (صفحة) + `project-state.json` (صغير، آلي) | SIRA S3 مقلَّص |
| L2 سجلات | ذاكرة التحليل والقرارات | `registers/{decisions,questions,findings,assumptions,conflicts,glossary}.md` بمعرّفات | Harmonic H2–H7 |
| L3 وحدة عمل | تفويض ثم تسليم | `tasks/<ID>.md` (Task Packet) و`handoffs/<ID>.md` | SIRA S7/S9 |
| L4 أدوار | من يملك ماذا | 3 أدوار + overlay اختياري (§4) | SIRA S12 |
| L5 مفردات | لغة التقارير | ادعاءات (4 SIRA) · حالة تحقق (7 SIRA) · أنواع سجل (7 Harmonic) · مراحل قبول | S4/S15، H3 |
| L6 حماية | منع الأفعال | `.claude/settings.json` (deny/ask)، قواعد المستودع على GitHub، `protected-operations.md` | S13 |
| L7 تشغيل | إعادة إنتاج البيئة | `runtime/RUNBOOK.md` + سكربتات | جديد (§5) |
| L8 محوّلات | أدوات أخرى | `CLAUDE.md`، `.cursorrules`، `copilot-instructions.md` تشير كلها إلى `AGENTS.md` | SIRA CLAUDE.md/AGENTS.md |

### 3.1 جرد القوالب (Template inventory) — لتجنب عبء الورق
| القالب | التوصية | السبب |
|---|---|---|
| `AGENTS.md` + محوّل | **أبقِ** | أساس |
| `STATE.md` | **أبقِ** (أول قراءة) | أقصر حامل فعّال |
| `project-state.json` | **أبقِ مقلَّصاً** (≈ 30 سطراً: baselines، PRs، بوابات مفتوحة، آخر نتائج الاختبار) | SIRA وصل 704 سطر (23 بوابة مفتوحة) وتكرر السرد |
| `PROJECT-STATE.md`, `HANDOFF.md` الضخم, `SOURCE-OF-TRUTH.md` | **لا تنقل** | ثبت الانحراف بينها (S3) |
| سجل القرارات DEC/ADR | **أبقِ** (قصير؛ ADR طويلة عند الحاجة فقط) | S17/H7 |
| Task Packet (Markdown) | **أبقِ مبسطاً** (≈ 12 حقلاً) | S7: المستعمل فعلاً أقل من القالب |
| Evidence Envelope | **ادمج في Handoff** كقسم «Evidence» | S8: لا نسخ فعلية |
| Handoff Packet | **أبقِ** | S9 |
| Artifact Manifest | **أجّل** (يلزم عند أدلة لقطات RTL/E2E) | S10 |
| JSON Schemas (4) | **أجّل** حتى وجود مدقق | S19: غير مفعّلة |
| Boot Protocol | **أبقِ نسخة قصيرة** (قسم في `AGENTS.md`) | S2 يكفي |
| Role overlays (4) | **اختصر إلى 3 فقرات قصيرة** في `governance/roles.md` | حجمها 26–91 سطراً لكل منها |
| Risk register | **لا تنشئ**: `registers/findings.md` يؤدي الغرض | تفادي التكرار |
| Model allocation policy | **جدول داخل `roles.md`** لا ملف مستقل | S20: لا سابقة |
| Verification report | **قسم في Handoff للمراجع المستقل** | — |
| Human approval record | **سجل صغير** `docs/approvals.md` (سطر لكل بوابة: من/متى/على أي SHA) | S6 «القبول لكل PR عند head محدد» |
| Runbook | **أنشئ** | جديد، مدعوم بتجربتنا |
| Phase completion report | **ضمن Handoff** | — |
| قوالب Harmonic: stakeholders, capability-map, sources | **أجّل** (لا حاجة الآن؛ `sources/register.md` قد يُستعمل لتتبع commits upstream) | — |

## 4. هيكل الأدوار والوكلاء (Phase C-1) — `Proposed`

الأدوار **وظيفية**، لا تعني بالضرورة جلسات أو نماذج منفصلة، لكن **الاستقلال في التحقق يتطلب جلسة منفصلة السياق**.

| الدور | المسؤولية | السلطة | السياق اللازم | المخرجات | توازٍ؟ | النموذج المقترح | كيف يُتحقق من مخرجاته |
|---|---|---|---|---|---|---|---|
| **Program Control** (مخطِّط/منسِّق) | يحوّل هدفك إلى Task Packets، يختار البوابة التالية، يحدّث `STATE.md` | يقترح فقط؛ لا يعتمد ولا يدمج | `STATE.md` + السجلات + الحزمة الحالية | حزم مهام، مقترحات DEC | لا (واحد) | أقوى نموذج متاح (قرارات معمارية) | أنت |
| **Implementation** (يشمل الاستقصاء كوضع داخلي) | ينفّذ ضمن نطاق الحزمة: تحليل، كود، اختبارات | فرع واحد، commits، PR مسودة؛ لا يدمج | الحزمة + الملفات المعنية + قواعد المسار | PR + Handoff | **نعم** عبر حزم متباعدة الملفات في worktrees | نموذج سريع للمهام المحدودة؛ **قوي للأمان/الصلاحيات** | Independent Verification + بوابات §6 |
| **Independent Verification** | يعيد التحقق دون تصديق تقرير المنفذ | قراءة فقط؛ لا يصلح | رأس PR + CI + الحزمة، **بلا** محادثة المنفِّذ | تقرير تحقق: ما أُعيد تشغيله فعلاً | نعم (مع منفّذ آخر) | نموذج **مختلف عن المنفِّذ** إن أمكن | أنت عند القبول |
| *(اختياري عند الطلب)* **Domain Governance** | أمن، خصوصية مرضى، مصطلحات سريرية عربية، i18n | يقترح؛ **لا يقبل** المصطلحات الطبية | الحزمة + المسرد | مقترحات + أسئلة | لا | قوي + **مراجع بشري مؤهل** للمصطلحات | مراجع طبي بشري |

لماذا ثلاثة لا أربعة: SIRA نفسه لديه دور تنفيذ واحد (S12)، و«Investigator» الذي اقترحه ChatGPT هو **وضع** داخل Implementation/Program Control (يمكن ألا يكتب الكود). فصله كدور مستقل يضيف تسليمات بلا استقلال فعلي. (`Inferred`)

## 5. القدرات الفعلية للتنسيق في هذه البيئة (Verified من تعريفات الأدوات في هذه الجلسة؛ لم تُمارَس)

| القدرة | ما هو متاح | ملاحظة |
|---|---|---|
| وكلاء فرعيون (Agent tool) | أنواع: `claude`، `Explore`، `Plan`، `general-purpose`، `claude-code-guide`، `statusline-setup`، و`fork`. معامل `model`: `sonnet`/`opus`/`haiku`/`fable` (يُتجاهل مع `fork` الذي يرث نموذج الجلسة). يعمل في الخلفية افتراضياً؛ عزل `worktree` متاح | وصف الأداة نفسه يقول **لا تُطلَق وكلاء إلا بطلب صريح منك** (كلفة وبدء بارد). لذلك لا تنسيق متوازٍ تلقائي |
| تعريف وكلاء مخصصين | عبر `.claude/agents/*.md` (ترويسة تحدد النموذج والأدوات) | **لا يوجد أي `.claude/agents` ولا `skills` في أي من المستودعات الأربعة** (Verified بـ `ls`) |
| جلسات مستقلة | `create_session` (نموذج، prompt، مستودع، وضع صلاحيات، وسوم) + `send_message` + `list_events`/`get_session` | **الأقرب لنمط SIRA** (محادثات منفصلة بأدوار)، ويحقق استقلال المراجع تماماً لأنه بلا سياق المنفذ |
| تتبع المهام | `TaskCreate/Update` داخل الجلسة | تتبع تقدم لا سجل دائم |
| مراقبة PR | `subscribe_pr_activity` | حدث PR يذهب لجلسة واحدة |
| جدولة | `create_trigger` / `send_later` | صالحة لـ check-ins |
| النطاق | كل جلسة تُضاف إليها المستودعات بـ `add_repo` | **ملاحظة جديدة (Verified):** عند إرفاق مستودع تُحمَّل `CLAUDE.md` الخاصة به تلقائياً كتعليمات (حدث ذلك لـ SIRA وHarmonic في هذه الجلسة). فإرفاق مستودع من upstream يحمل تعليماته (ومنها تعليمة «star the repo» في `care_fe`) |
| يحتاج إعداداً إضافياً | وكلاء `.claude/agents`، قواعد deny في `settings.json`، rulesets على GitHub، أي مدقق لمخططات الحوكمة | لا يتفعل بتغيير النموذج وحده |

**الخلاصة (`Verified` للأولى، `Inferred` للباقي):** تبديل نموذج الـ orchestrator لا يمكّن تلقائياً تعدد الوكلاء. الجلسة الحالية على `claude-sonnet-5-5` (من `get_session`)، وليس لدي دليل على كيفية سلوك أي نموذج آخر في التنسيق دون تجربة.

## 6. تقييم نمط توزيع النماذج الذي تنوي استخدامه (Phase C-4) — `Inferred/Proposed`
النمط: نموذج قوي للمعمارية والتخطيط والتنسيق ومراجعة المخاطر العالية؛ نماذج أسرع لاستقصاء محدود وتنفيذ محدود؛ تحقق مستقل قبل القبول.

| ما يدعمه | أين قد يكون غير فعّال أو غير آمن |
|---|---|
| تركيز الكلفة على القرارات المكلفة | **تنفيذ الأمن/الصلاحيات بنموذج سريع**: B-06 (التفويض اختياري) مثال لا يصلح لـ «محدود». يجب أن يُصنَّف حسب **المخاطر لا الحجم** |
| الاستقلال في التحقق | مراجع من **نفس العائلة** قد يشترك في الأخطاء؛ المراجع الأضعف من المنفِّذ يضعف الضمان؛ مراجع يرث السياق (`fork`) **ليس مستقلاً** |
| توازي المهام المتباعدة | عبء التنسيق؛ تعارضات في الملفات المشتركة (`package-lock.json`, `en.json`, `Permissions.tsx`)؛ يحتاج worktrees وحزماً بلا تداخل |
| وضوح الأدوار | **خطر السلطة:** الوكلاء كلهم يعملون بتوكن مالك GitHub الواحد (يقر SIRA بذلك في `GITHUB-GOVERNANCE.md`). نموذج «آمن» لا يغني عن deny-rules وrulesets |
| الاستمرارية | الحاوية السحابية أُعيد تشغيلها أثناء عملنا (uptime 3 دقائق؛ Verified) وفقدت الخدمات؛ يلزم RUNBOOK وسكربتات |
| — | الكلفة: كل وكيل فرعي يبدأ بارداً؛ لا نملك قياساً للتوفير الفعلي، والادعاء بأن «السريع أرخص» هنا `UNKNOWN` |

توصية عملية (`Proposed`): ابدأ بـ **مسار واحد متسلسل** (Program Control ← Implementation ← Verification) بجلسات منفصلة، وفعّل التوازي لاحقاً لحزم أثبتت أنها مستقلة الملفات.

## 7. التكييف مع CARE (Phase C) — حقائق تحدد التصميم

| الحقيقة | الدليل | الأثر على التصميم |
|---|---|---|
| مستودعان forks + upstream نشط جداً | `care_fe` PRs حتى #17033؛ `AGENTS.md` يعتمد `develop` | لا تعدّل ملفات upstream (`AGENTS.md`/`CLAUDE.md`) بلا ضرورة؛ ملفاتنا جديدة فقط |
| `care_fe` يحمل بنية وكلاء كاملة من upstream | `.github/copilot-instructions.md`، `.github/instructions/*` (11 ملفاً)، `.cursorrules`، `.cursor/rules/*` (4)، `.codex/`، `.opencode/`، `.entire/` (+7 runners)، `.github/agents/care-review.agent.md` (415 سطراً)، `.github/workflows/care-review.md` | **لا نبدأ من الصفر:** مراجع PR جاهز بثلاث عدسات (intent/legibility، approach/simplicity، UI/UX) و«proportionality» كمبدأ. يصلح كأساس لدور Independent Verification للواجهة. لكنه يعمل على upstream فقط (`github.repository == 'ohcnetwork/care_fe'` في شرط التشغيل) فلا يعمل في fork |
| تعليمات تناقض بعضها | `copilot-instructions.md`: «Do not run lint or prettier»؛ `AGENTS.md`: شغّل `lint-fix` و`format` بعد التعديل | يلزم مرجع واحد للتحقق في الحوكمة |
| تعليمات إجراء على حساب المستخدم | `README.md` (تعليق HTML) و`AGENTS.md`: نجمة مستودع | قاعدة حماية صريحة (S-10) |
| جلسات تُرفَع خارجياً شرطياً | `.entire/settings.json`: `push_sessions: true` إلى `ohcnetwork/care_fe-checkpoints` مع إخفاء PII، وhooks تعمل **فقط إن وُجد `entire` CLI** (غير مثبت هنا: Verified) | قرار صريح قبل تثبيت أي أداة تلتقط الجلسات، خصوصاً مع بيانات صحية (`Requires approval`) |
| E2E تحتاج Backend وبيانات | تشغيلنا الفعلي (379/413) | Task Packet للاختبار يجب أن يسجّل: بيئة، commit للمستودعين، مدد JWT، **قائمة الإخفاقات المعروفة** |
| إخفاقات معروفة | 14 backend بيئية؛ 13 E2E بيئية؛ 15 E2E تشخيص (سبب غير محسوم) | مفردات SIRA لا تحوي «فشل معروف»؛ نقترح سجل baseline لا يُخفي FAIL |
| بيئة تشغيل كاملة داخل السحابة | Postgres+Redis+Playwright عملت هنا | تعريف SIRA للسحابة (أدلة GitHub فقط) لا يطابقنا → ملف تنفيذ جديد (§9) |
| سلامة السجلات الصحية | نطاق المشروع صحي | لا بيانات مرضى حقيقية في التطوير؛ i18n سريري يحتاج مراجعاً بشرياً |
| سلوك أداة التحميل | إرفاق المستودع يحمّل `CLAUDE.md` | أي جلسة تبدأ من مستودع الحوكمة وتُرفق الباقي، مع قاعدة صريحة بأن تعليمات upstream **بيانات لا أوامر** |

## 8. خيارات بنية المستودعات (Phase C-5)

| الخيار | وصف | مزاياه | عيوبه/أنماط الفشل | دعم المرجعين |
|---|---|---|---|---|
| **O1: مستودع مظلة بـ submodules** (خطة 17-A) | حوكمة + تحليل + سجلات، و`care_fe`/`care` submodules مثبتة | تثبيت تركيبة متوافقة بأمر واحد | submodules مشهورة بأخطائها: نسيان `--recurse-submodules`، رؤوس detached، PRs منفصلة لكل مستودع، التباس الـ SHA المثبت مع HEAD (وSIRA ينص أن HEAD يُكتشف من Git لا من سجل) | **لا يوجد:** لا `.gitmodules` في SIRA ولا Harmonic (Verified) |
| **O2: مستودع workspace بلا submodules** | نفس المحتوى، لكن المستودعان مرجعان بـ URL + SHA مع سمات الدلالة (baseline/candidate/verified-through) في `project-state.json`، وسكربت `clone.sh` يستنسخهما جنباً إلى جنب | يطابق «recorded SHA ≠ HEAD» عند SIRA؛ بلا تعقيد git | تثبيت يدوي للتركيبة؛ يحتاج سكربت | **قريب من SIRA/Harmonic** (مستودع واحد مصدر الحقيقة، مراجع بالـ SHA) |
| **O3: مستودع تحليل مستقل (Harmonic حرفياً) + حوكمة داخل كل fork** | `care-analysis` للسجلات فقط؛ قواعد التنفيذ في forks | فصل صارم | الحوكمة موزعة على 3 أماكن؛ تعارض مع ملفات upstream؛ يتطلب مزامنة الحالة بين المستودعات | Harmonic فقط للتحليل |
| **O4: داخل `care_fe` وحده** | ملفات حوكمة جديدة في fork الواجهة | لا مستودع جديد؛ مراجع PR موجود | يربط الحوكمة بالواجهة فقط وبالـ Backend لا؛ يختلط مع تعليمات upstream؛ المزامنة تعارض إن لمسنا `AGENTS.md` | SIRA (لكنه مستودع أصلي لا fork) |

**التوصية (`Proposed`): O2.** البديل الأقوى: **O1** إن كان مهماً لك أن يستنسخ الآخرون كل شيء بأمر واحد مع تثبيت تلقائي. أبرز المقايضة: O2 يتطلب `clone.sh` ويدوية التثبيت، لكنه يتجنب أخطاء submodules ويطابق سابقة SIRA (SHA مسجَّل بمعناه). **لا يتبع أي من المرجعين نمط submodules**، ولذلك فتوصيتي السابقة في خطة 17 (البديل A) كانت **غير مسنودة بالمراجع**.

قيد تشغيلي يخص كل الخيارات خارج O4 (`Verified`): Claude Code يحمّل `CLAUDE.md` للمستودع الذي تبدأ منه الجلسة؛ فيلزم أن **تبدأ الجلسات من مستودع الحوكمة** وتُرفق الـ forks كمستودعات إضافية، وإلا لم تُحمَّل قواعدنا تلقائياً.

## 9. مواءمة مع CARE: ما يتغير في القوالب العامة
- **ملف التنفيذ:** SIRA يعرّف `LOCAL` و`CLOUD_GITHUB` (السحابة = أدلة المستودع فقط). في CARE أثبتنا تشغيلاً كاملاً سحابياً. اقتراح: إضافة `CLOUD_RUNTIME` (سحابة بقدرات تشغيل، وأدلتها قابلة للإعادة عبر RUNBOOK) أو إعادة تعريف `CLOUD_GITHUB`. (`Requires approval`)
- **حقول Task Packet الإضافية:** `repos[]` (أي مستودع/SHA)، `testSuites[]`، `knownFailuresBaseline`، `phiTouching: true|false`، `upstreamImpact: none|candidate|required`.
- **قواعد المسار لـ CARE:** `frontend.md` (tsc, eslint, build, E2E أوامر وحدود)، `backend.md` (manage.py test، الإعدادات)، `i18n-rtl.md` (الأرقام الغربية، RTL، قاموس المصطلحات)، `security-health.md` (deny/ask + بيانات الصحة)، `governance.md`.
- **بوابات القبول:** تعديل A–F ليشمل E2E ببيانات fixture، وفحص لقطات RTL، ومراجعة المصطلحات.
- **ما نعيد استخدامه من upstream:** `care-review.agent.md` كمرجع لدور المراجع، و`.github/instructions/*` كمعايير ترميز (تُحال ولا تُنسخ).

## 10. مراجعة الخطة السابقة `17_ai_engineering_architecture_plan.md` (Phase D)

| بند في خطة 17 | الحكم | السبب |
|---|---|---|
| مستودع مظلة | **مدعوم جزئياً** كفكرة مستودع حوكمة واحد (SIRA/Harmonic) | |
| **submodules** | **يُراجَع: لا دليل مرجعي** (O2 بديلاً) | `.gitmodules` غائب في المرجعين |
| `AGENTS.md` + محوّل `CLAUDE.md` قصير | **مدعوم** (SIRA S1، حدّ 60 سطراً) | |
| `STATE.md` + `project-state.json` | **مدعوم مع تقليص** | S3: انحراف حاملات الحالة؛ JSON وصل 704 سطر |
| Task Packet/Evidence/Handoff + مخططات | **مدعوم جزئياً:** القوالب موجودة لكن التطبيق الفعلي ضعيف (0/17 مطابقة؛ لا Envelope فعلي؛ لا مدقق) | ادمج Envelope في Handoff، أجّل المخططات |
| 4 أدوار (PC, Impl, IV, DG) | **يُراجَع إلى 3 + overlay** | §4 |
| سجلات Harmonic ومعرّفاتها | **مدعوم** (H2–H7) | |
| توحيد أنواع الأدلة | **معقول لكن غير محسوم:** أقترح 4 فئات SIRA للمطالبات الهندسية و7 أنواع Harmonic للمتطلبات/القرارات | `Requires approval` |
| RUNBOOK + سكربتات | **مدعوم بأدلة جلستنا** (انهيار الحاوية، انتهاء JWT، فخ `pkill -f`) | جديد وليس من المرجعين |
| قائمة التوقف | **مدعوم** (S13) مع إضافة: إجراءات على حسابات/خدمات خارج الحزمة، وأي إرسال لجلسات خارجية، وتعديل ملفات الحوكمة نفسها | |
| بوابات الجودة وخط الأساس | **مدعوم** (S15/S16) + سجل إخفاقات معروفة | |
| مراحل G0…F | **تُراجَع** (§11) | G0 كبيرة؛ وG1 اختبار جلسة جديدة قيّم |
| **أنماط ناقصة في الخطة 17** | stop conditions كرموز (`BLOCKED_BASELINE_DRIFT`, `BLOCKED_SCOPE_EXPANSION_REQUIRED`)؛ «HEAD يُكتشف من Git»؛ فصل Authorization/Acceptance/Canonical؛ استرجاع الجلسة المقطوعة؛ نموذج «Stage report» و`CURRENT PROJECT STATE`؛ بطاقة `External Admin Action`؛ منع قراءة `.env` عبر deny؛ hook للتحقق بعد commit؛ «لا إعادة كتابة للتاريخ» | S6/S13/S18 |
| **مفرط التعقيد في الخطة 17** | JSON Schemas الآن؛ 4 حاملات حالة؛ Envelope منفصل؛ خريطة قدرات وأصحاب مصلحة | يمكن تأجيلها |
| ملاحظة على الخطة 17 §0 | ذكرتُ «تغيير النموذج لم ينعكس»؛ ما زال صحيحاً (`get_session`: `claude-sonnet-5-5`) | |

## 11. خطة مراحل منقَّحة (Phase 6) — `Proposed`

| المرحلة | المحتوى | بوابة |
|---|---|---|
| **قرار المعمارية** (الآن) | تراجع هذا التقرير وتقرر §12 | قرارك |
| **G0-min (الحد الأدنى)** | مستودع حوكمة واحد (O2): `AGENTS.md` + `CLAUDE.md` (<60 سطراً) + `docs/STATE.md` + `project-state.json` مقلَّص + `governance/{principles,conventions,roles,protected-operations}.md` + `.claude/settings.json` (deny/ask) + قالبان فقط (Task Packet مبسَّط، Handoff) + `registers/{decisions,questions,findings}.md` بمعرّفات + نقل `analysis/` كما هو + `runtime/RUNBOOK.md` وسكربتان (`clone.sh`، `up.sh`) + سجل baseline للاختبارات | مراجعتك |
| **G1 اختبار الاستقلال** | جلسة جديدة بلا سياق تقرأ `STATE.md` فقط وتؤدي مهمة صغيرة (مثلاً إغلاق أحد بنود B-06) وتُرجع Handoff | تقرير فجوات توثيق |
| **A إغلاق التحليل** | مراجعة ViewSets (B-06)، سبب إخفاقات التشخيص (15)، `unshallow` ومقارنة upstream، فحص أسرار | سجلات محدّثة |
| **S أمن** | PR #2 (قائم)، مهام الـ Backend (`SECRET_KEY` إلزامي…) | مراجعة مستقلة |
| **L تعريب** | خطة 16: مرحلتان 0 و1 كحزم AR-* | مراجع طبي للمصطلحات |
| **F ميزات** | طلب ← تحليل أثر ← REQ ← حزمة ← بوابات | حسب الميزة |

**يُؤجَّل صراحة:** JSON Schemas ومدقق CI للحوكمة، Artifact Manifest، ADR طويلة، overlays مفصلة، تنفيذ متوازٍ متعدد الوكلاء، وكلاء `.claude/agents` مخصصون، أي تثبيت لأدوات التقاط الجلسات (`entire`)، `sira-workspace`/`sira-analysis` ما لم تأذن بقراءتهما.

## 12. قرارات تحتاج موافقتك (`Requires approval`)
1. **بنية المستودعات:** O2 (موصى به) أم O1 أم غيره؟ وهل أقرأ `sira-workspace` أولاً لأنه قد يكون النموذج المقصود؟
2. **توحيد مفردات الأدلة** (SIRA للمطالبات الهندسية + Harmonic للمتطلبات)؟
3. **ملف تنفيذ جديد** `CLOUD_RUNTIME` أم إعادة تعريف `CLOUD_GITHUB`؟
4. **عدد الأدوار:** 3 + overlay؟
5. **بوابات إضافية فعلاً ضرورية** (فوق ما حددتَ: المعمارية، الأمن، تغيير النطاق، الإنتاج، إتمام المرحلة): (أ) أي إجراء على حساب أو خدمة خارج الحزمة؛ (ب) أي إجراء موجه إلى upstream (`ohcnetwork/*`)؛ (ج) أي لمس لبيانات مرضى؛ (د) اعتماد المصطلحات السريرية العربية بمراجع بشري؛ (هـ) أي تعديل لملفات الحوكمة وقواعد deny نفسها؛ (و) ترحيلات قاعدة البيانات في الـ Backend؛ (ز) تفعيل أدوات التقاط/رفع الجلسات؛ (ح) إطلاق وكلاء/جلسات إضافية (كلفة).
6. **سياسة upstream:** مزامنة دورية أم تفرّع دائم (تؤثر على كل ما يلمس `AGENTS.md` أو مئات ملفات RTL).
7. **سجل الإخفاقات المعروفة:** هل تقبل ملف baseline يفرّق «FAIL جديد» عن «FAIL معروف» دون تغيير مفردات SIRA؟
8. **المراجع الطبي:** من يعتمد المصطلحات السريرية العربية؟
9. **توزيع النماذج:** تتبنى نمط §6 كبداية متسلسلة؟

## 13. ما لم يُفعل
لم أنشئ مستودعاً أو فرعاً أو PR، ولم أنسخ ملفاً من SIRA/Harmonic إلى أي مكان، ولم أطلق وكيلاً أو جلسة، ولم أعدّل كود التطبيق. أُضيف هذا الملف فقط إلى الفرع القائم لـ PR #1. المستودعان المرجعيان لم يُعدَّلا (`git status` نظيف).
