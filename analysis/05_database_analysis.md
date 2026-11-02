# 05 — Database & Data Model

## الحكم
**لا توجد قاعدة بيانات أو ORM أو Migrations في هذا المستودع.** لا يجوز تقديم أي جدول كحقيقة. (بحث: لا `.sql`، لا `migrations/`، لا نماذج ORM.)

## ما هو موجود فعلاً
| الفئة | الموقع | العدد | الطبيعة |
|---|---|---|---|
| Frontend types / API data contracts | `src/types/**` | 153 ملفاً / 12,278 سطراً | واجهات TypeScript لاستجابات/طلبات الـ API (Pydantic-shaped) |
| Domain folders | `src/types/{emr,billing,inventory,scheduling,questionnaire,tokens,device,...}` | 25 نطاقاً | تدل على نموذج بيانات الخادم لكنها **ليست** مخططاً |
| Enumerations/consts | داخل الأنواع | — | قد تعكس enums الخادم |
| Zod schemas | `src/lib/zod.ts`, داخل النماذج | — | تحقق من جهة العميل |
| Client persistence | `localStorage` (JWT، تفضيلات، queries موسومة) | — | ليس مصدر حقيقة |

## كيانات مستنتجة (Inferred — ليست جداولاً مُتحقَّقة)
من أسماء نطاقات `src/types` ومسارات الـ API: Patient، Encounter، Facility، FacilityOrganization، Organization، Location، User/Role/Permission، Questionnaire/QuestionnaireResponse، MedicationRequest/Statement/Administration/Dispense، AllergyIntolerance، Symptom، Diagnosis، Observation/ObservationDefinition، ServiceRequest، Specimen/SpecimenDefinition، DiagnosticReport، ActivityDefinition، Account/Invoice/ChargeItem/ChargeItemDefinition/Payment، Product/ProductKnowledge/SupplyDelivery/DeliveryOrder، Schedule/Slot/Appointment، TokenCategory/Token/Queue، Device/ServiceHistory، Consent، ResourceRequest، ValueSet، TagConfig، Template/Report، PlugConfig.
**الثقة: Low–Medium.** مصطلحات تشبه FHIR (MedicationRequest، Observation…) تدل على تأثر بـ FHIR لكن لا تثبت التزاماً به.

## بعد المرحلة الثانية
حُلِّل الـ Backend في `backend/B02_database_and_data_model.md` (≈ 89 نموذجاً، 295 migration، PostgreSQL) وأُنتج `diagrams/database_erd.mmd` (جزئي). الكيانات المستنتجة أعلاه صحيحة في معظمها.
