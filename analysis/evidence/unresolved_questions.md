# Unresolved Questions & Required Next Evidence

1. ~~Backend~~ حُسمت جزئياً في المرحلة الثانية (`analysis/backend/`): الصلاحيات تُفرض خادمياً (RBAC على شجرة Organizations)؛ العزل منطقي لا فيزيائي؛ `plug_config` يكتبه `is_staff` فقط؛ نماذج ≈ 89 و295 migration. **المتبقي:** تشغيل الاختبارات، مراجعة كل viewset (B-06)، فحص الفهارس والقيود في PostgreSQL الحقيقي، تدقيق `plugs/` و`care/emr/fhir` و`reports`.
2. **Git history**: `git fetch --unshallow` ثم مقارنة merge-base/diff مع upstream.
3. **هل `husam713/care_fe` fork رسمي؟** علم fork في GitHub لم يُفحص.
4. ~~E2E~~ نُفِّذت (379/413). **المتبقي:** إعادة تشغيل منفردة لإخفاقات `DiagnosticReportReview`/`ServiceRequestReportWorkflow`/`encounterShortcuts`، وتشغيل اختبارات الرفع مع MinIO.
5. **Build**: نجح؛ المتبقي تحليل حجم الحزمة (code splitting).
6. ~~ESLint warnings~~ — حُسمت: 370 تحذيراً (انظر 09).
7. **Workflows الوكلاء (`*.lock.yml`, `.github/aw`, `.entire`, `.codex`, `.opencode`)**: صلاحيات الرموز، مصادر المدخلات (prompt injection)، أسرار.
8. **S-01 exploitability**: PoC آمن محلياً على إعداد TemplateBuilder (`WHOLE_DOCUMENT` + shadow DOM).
9. **Questionnaire/Medication/Billing/Inventory**: قراءة معمّقة (أكبر الملفات) وتتبع كامل لمسار الميزة.
10. **سياسة الـ service worker/PWA**: ما يُخزَّن مؤقتاً (هل بيانات API؟).
11. **Secrets scanning**: تشغيل gitleaks/trufflehog على التاريخ الكامل بعد unshallow.
12. **الرسوم غير المنتجة**: `components.mmd` و`data_flow.mmd` (مؤجلة). `database_erd.mmd` أُنتج جزئياً من الـ Backend (30 نموذجاً من ~89).
