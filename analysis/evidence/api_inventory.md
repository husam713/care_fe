# API Inventory

الملف الخام (422 صفاً): [`api_inventory_raw.tsv`](./api_inventory_raw.tsv) — الأعمدة: `file, key, line, method, path`.

## إعادة التوليد
السكربت المستخدم يقرأ كل `src/**/*Api.ts`، يلتقط اسم المفتاح (`key: {`) والسطر الأول الحاوي `path: "..."` وأقرب `HttpMethod.X` (حتى 4 أسطر لاحقة). حدوده: لا يلتقط تعريفات `API(HttpMethod.X, "path")` المعرّفة بدالة (انظر `src/Utils/request/utils.ts:5`)؛ 8 مسارات بلا method مكتشفة:

- `careTeam/careTeamApi.ts:8 setCareTeam`
- `consent/consentApi.ts:13,29,39,49,55` (create, update, addVerification, removeVerification, upsert)
- `device/deviceApi.ts:67,73` (service_history create/update)

## الإحصاء
POST 181 · GET 155 · PUT 54 · DELETE 21 · PATCH 3 · غير مكتشف 8 = **422**.

## ملاحظة
حقول Authentication/Authorization/Validation الخادمية **غير متاحة** من هذا المستودع.
