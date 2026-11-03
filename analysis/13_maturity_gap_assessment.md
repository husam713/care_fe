# 13 — Maturity, Gap & Risk Assessment

مقياس 1–5: 1 = ad hoc، 2 = أولي، 3 = مُعرَّف، 4 = مُدار/مُقاس، 5 = مُحسَّن باستمرار. **الدرجة تخص `care_fe` فقط.** أي مجال يعتمد على Backend لا يُقيَّم ("غير قابل للحكم").

| Area | Score | Current State | Evidence | Gap | Risk | Recommendation |
|---|---:|---|---|---|---|---|
| Functional completeness (FE) | 4 | 18 modules، 144 مساراً، 422 تعريف API، نشاط commits عالٍ | 04, 06 | بعض ميزات تنتظر BE (TODOs)؛ لا عربية | Low | إغلاق TODOs المقترنة بـ BE |
| Architecture (FE) | 4 | طبقات واضحة، API typed، plugin framework | 03 | تقسيم ملفات ضخمة؛ نموذج ثقة الإضافات غير موثق | Medium | توثيق trust model |
| Code quality | 4 | tsc/eslint/knip نظيفة، لا `@ts-ignore` | 09 | `any` ×81، ملفات 1–3K سطر | Medium | تقسيم تدريجي |
| Security (FE) | 2–3 | DOMPurify مُستعمل، redirect آمن | 08 | S-01..S-05 (JWT localStorage، CSP تقريرية، ثغرات تبعية، إضافات) | **High** | خطة معالجة 14 |
| Database design | — | خارج النطاق | 05 | — | غير قابل للحكم | تحليل BE |
| Documentation | 4 | AGENTS/CLAUDE/docs/instructions غنية | 00 | لا مواصفة API/OpenAPI؛ توجيهات الوكلاء فيها أوامر خارج المهمة (S-10) | Low | OpenAPI + سياسة وكلاء |
| Testing | 3 | 109 E2E، 1 unit | 09 | غياب unit للمنطق؛ E2E تحتاج BE؛ coverage غير مقاس | Medium | unit tests لـ utils/hooks/validators |
| Deployment (FE) | 3 | Docker+nginx، CI غني | 10 | بناء غير حتمي جزئياً؛ إعداد وقت البناء؛ CSP | Medium | `npm ci`، runtime config |
| Scalability (FE) | 4 | static + CDN | 10 | حجم الحزمة غير مقاس | Low | تحليل bundle |
| Maintainability | 3 | modular لكن ضخم (233K سطر) | 00 | مكونات عملاقة | Medium | refactor مرحلي |
| Extensibility | 4 | plugins + overrides | 03 | حوكمة/توقيع الإضافات | Medium | allowlist/SRI |
| Accessibility/i18n | 3 | 388 aria، 599 ملف i18n | 07 | لا RTL/العربية | Medium (للمستخدم العربي: High) | خطة RTL |

## Gap Analysis المُرتَّب
**Critical:** لا شيء مُتحقَّق كاستغلال فعلي؛ لكن S-01 (critical حسب npm) يحتاج تحقق تعرض.
**High:** S-01 تحديث DOMPurify؛ S-03 نموذج ثقة الإضافات؛ غياب العربية/RTL (إذا كان الاستهداف عربياً).
**Medium:** S-02 JWT، S-04 CSP، S-05 تبعيات DoS، اختبارات وحدة، حتمية البناء، توليد الأنواع من OpenAPI.
**Low:** S-06..S-12، تنظيف `console.*`/`any`.
