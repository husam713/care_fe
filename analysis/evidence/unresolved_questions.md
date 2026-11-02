# Unresolved Questions & Required Next Evidence

1. **Backend (`ohcnetwork/care`)**: هل الصلاحيات تُفرض خادمياً؟ كيف يُعزل المستأجرون؟ من يملك تعديل `plug_config`؟ نماذج البيانات والـ migrations؟ → مرحلة مستقلة؛ تتطلب `add_repo`.
2. **Git history**: `git fetch --unshallow` ثم مقارنة merge-base/diff مع upstream.
3. **هل `husam713/care_fe` fork رسمي؟** علم fork في GitHub لم يُفحص.
4. **E2E**: تشغيل Playwright مقابل Backend محلي + DB snapshot لقياس نسبة النجاح الفعلية.
5. **Build**: `npm run build` + تحليل حجم الحزمة (code splitting).
6. **ESLint warnings**: عدّ التحذيرات (التقرير JSON انهار بسبب الذاكرة؛ جرّب `NODE_OPTIONS=--max-old-space-size=8192` أو تشغيل على مجلدات).
7. **Workflows الوكلاء (`*.lock.yml`, `.github/aw`, `.entire`, `.codex`, `.opencode`)**: صلاحيات الرموز، مصادر المدخلات (prompt injection)، أسرار.
8. **S-01 exploitability**: PoC آمن محلياً على إعداد TemplateBuilder (`WHOLE_DOCUMENT` + shadow DOM).
9. **Questionnaire/Medication/Billing/Inventory**: قراءة معمّقة (أكبر الملفات) وتتبع كامل لمسار الميزة.
10. **سياسة الـ service worker/PWA**: ما يُخزَّن مؤقتاً (هل بيانات API؟).
11. **Secrets scanning**: تشغيل gitleaks/trufflehog على التاريخ الكامل بعد unshallow.
12. **الرسوم غير المنتجة**: `database_erd.mmd` (لا DB)، `components.mmd`/`data_flow.mmd` (مؤجلة).
