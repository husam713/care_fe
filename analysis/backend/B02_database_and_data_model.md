# B02 — Database & Data Model

**المصدر:** تعريفات النماذج والـ migrations في `husam713/care` @67a4618. لم تُشغَّل migrations ولم يُفحص مخطط PostgreSQL الفعلي؛ لذا هذا **مخطط من الكود (ORM)** وليس مخططاً حياً.

## 1. الأرقام
- ≈ 89 نموذجاً فعلياً (استخراج آلي بالـ regex؛ قد يخطئ ± بضعة نماذج). التوزيع: `care/emr/models` ≈ 75، ‏`care/facility/models` 3، ‏`care/users/models.py` ≈ 8 (User, PlugConfig, UserFlag, Skill, UserSkill, MobileOTP…)، ‏`care/security/models` 4.
- ‏295 ملف migration (facility 172 تاريخي، emr 86، users 27، security 6، sites 4).
- محرك: PostgreSQL (psycopg3)؛ `ATOMIC_REQUESTS=True` و`CONN_MAX_AGE` افتراضي 60 في deployment (`deployment.py:15-17`).

## 2. أنماط تصميم (Verified)
| النمط | الدليل |
|---|---|
| قاعدة موحدة `BaseModel` ← `EMRBaseModel`: `external_id` (UUID)، `history` JSON، `meta` JSON، `created_by`/`updated_by` | `care/emr/models/base.py:7-24` |
| **Soft delete** عبر `deleted=True` (لا حذف فعلي) | `EMRDestroyMixin.perform_destroy` في `viewsets/base.py` |
| **Slug بنطاق Facility/Instance** (`f-<facility_uuid>-slug` / `i-slug`) | `SlugBaseModel` `base.py:27-60` |
| تخزين مشتق للتفويض/الأداء: `organization_cache`, `facility_organization_cache`, `users_cache` (ArrayField) على Patient/Encounter… | `security/authorization/patient.py:25-48,95-112` |
| حقول JSON واسعة للبيانات السريرية (FHIR-like: Coding/CodeableConcept/Quantity/Period) | `care/emr/resources/common/*`, `care/emr/fhir` |
| Feature flags على Facility/User | `FacilityFlag`, `UserFlag` |
| تدقيق: `AuditLogMiddleware` + receivers | `base.py:206`, `care/audit_log` (427 سطراً) |

## 3. الكيانات الأساسية (≈ 30 من ~89) والـ ERD
الملف: `diagrams/database_erd.mmd` (69 علاقة مستخرجة من `ForeignKey/OneToOneField/ManyToManyField`). **تغطيته جزئية** ولا تتضمن الحقول ولا القيود/الفهارس. أبرز العلاقات:

- **الهوية والتفويض:** `User` ↔ `OrganizationUser`/`FacilityOrganizationUser` ↔ `RoleModel` ↔ `RolePermission` ↔ `PermissionModel`.
- **الجغرافيا والمنشآت:** `Facility`→`Organization` (geo)، `FacilityOrganization`→`Facility`، `User.home_facility`.
- **السريري:** `Patient` ← `PatientOrganization`/`PatientUser`؛ `Encounter` (→Patient، Facility، `EncounterOrganization`)؛ ثم Condition، Observation، MedicationRequest/Administration/Dispense، ServiceRequest، DiagnosticReport، Consent، FileUpload… مرتبطة بـ Encounter/Patient.
- **المالي:** `Account`, `Invoice`, `ChargeItem(+Definition)`, `PaymentReconciliation`.
- **الجدولة:** `Schedule`, `Availability`, `TokenSlot`, `TokenBooking`, `TokenQueue/SubQueue/Category`, `Token`.
- **المخزون:** `Product(+Knowledge)`, `InventoryItem`, `SupplyRequest/Delivery`, `DeliveryOrder`, `RequestOrder`, `DispenseOrder`.
- **المحتوى:** `Questionnaire(+Response, +Template, Organization links)`, `ValueSet`, `Template`, `TagConfig`.

## 4. تقييم
| الجانب | الحكم |
|---|---|
| تطابق مع عقود FE | قوي: الواجهة تحاكي هذه النطاقات (emr، billing، inventory، scheduling…) ⇒ الكيانات المستنتجة في `analysis/05` صحيحة في معظمها (Medium→High) |
| سلامة البيانات | `ATOMIC_REQUESTS`، قيود فريدة مشروطة بـ `deleted=False` (مثال `UserFlag.unique_user_flag`)؛ لم تُراجع بقية القيود |
| قابلية التوسع | أعمدة cache مصفوفية ومرتبطة بتحديث متزامن (signals/tasks) ⇒ خطر عدم اتساق؛ فهارس GIN لهذه المصفوفات لم تُتحقَّق |
| تاريخ هجرات | 172 migration في `facility` لقاعدة قديمة، ما زالت `facility` مستثناة من ruff والتغطية (`pyproject.toml`) ⇒ دين تقني |
| الاحتفاظ/الأرشفة | لم يُرصد retention policy؛ soft delete فقط (Unverified) |
