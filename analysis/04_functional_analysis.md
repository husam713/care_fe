# 04 — Functional & Domain Analysis

> **تحذير منهجي:** الجرد أدناه مبني على (أ) مجموعات Routes، (ب) نطاقات API في `src/types`، (ج) أحجام مجلدات `pages/components`، (د) أسماء الملفات. **لم تُقرأ شاشات كل module سطراً بسطر**. لذلك لا يُوسَم أي module بـ "Fully Implemented"؛ أعلى وسم ممكن هنا: **UI + API client verified; Backend = External Dependency**.

## 1. جرد Modules
| ID | Module | Routes (≈) | API defs (`src/types/<dom>`) | حجم التنفيذ في FE | الوسم |
|---|---|---:|---:|---|---|
| M01 | Authentication & Session (login, 2FA, reset, session-expired, patient OTP) | 8 عامة | auth 12 + otp 4 | `components/Auth` 7 ملفات/1.9K سطر | UI+API client verified |
| M02 | Patient (تسجيل، بحث، ملف المريض، هويات، وسوم) | 26 (`/facility/:id/patient/*`) + 9 `/patient/*` | patient 5 + emr 111 (مشتركة) | `components/Patient` 38/8.1K؛ `pages/Patient` 11/1.3K | UI+API client |
| M03 | Encounter / Clinical EMR (ملاحظات، أعراض، تشخيصات، حساسية، أدوية، ملاحظات طبية، قوالب تقارير) | 6 encounters + ضمن patient | emr 111، notes 4، careTeam 1 | `pages/Encounters` 53/10.1K؛ `components/Encounter` | UI+API client |
| M04 | Questionnaire Engine (بناء/تعبئة/أنواع أسئلة، FHIRPath) | داخل admin/facility | questionnaire 25 | `components/Questionnaire` **47 ملف/21.3K** (`QuestionnaireEditor.tsx` 3258 سطراً) | UI+API client |
| M05 | Medication / Prescription / Dispense / Administration | ضمن encounter + `medication_dispense` | emr (medication*) | `components/Medicine` 20/6.4K؛ `MedicationRequestQuestion.tsx` 2403 سطراً | UI+API client |
| M06 | Diagnostics (ServiceRequest, Specimen, DiagnosticReport, ActivityDefinition) | `service_requests`, `report/*` | emr (serviceRequest, specimen, diagnosticReport, activityDefinition) | `components/ServiceRequest` | UI+API client |
| M07 | Billing (Account, Invoice, Charge Items, Payments, Reconciliation) | 14 `/billing` | billing 37 | `pages/Facility/billing`، `PrintChargeItems.tsx` 1677 / `InvoiceShow.tsx` 1608 | UI+API client؛ ‏TODO: مقاييس charge-item من BE غير جاهزة (`chargeItemDefinitionApi.ts:34`) → **Partial** |
| M08 | Inventory & Supply (Product, Delivery orders, PO, internal transfers, usage) | داخل `locations/:id/*` | inventory 38 | `pages/Facility/services/inventory` (ملفات 1000+ سطر) | UI+API client؛ `productKnowledge.ts:69` TODO أنواع BE → Partial |
| M09 | Scheduling & Appointments (جداول، فترات، حجز، حجز عام بدون دخول) | 8 appointments + ScheduleRoutes + PublicAppointments | scheduling 30 | `pages/Appointments` 15/5.9K؛ `pages/Scheduling`؛ `PublicAppointments` | UI+API client |
| M10 | Queues & Tokens | `queues`, `queue` | tokens 23 | `components/Tokens`, `pages/Facility/queues` | UI+API client |
| M11 | Facility / Location / Organization / Users | facility، organization 19، users، locations | facility 15, facilityOrganization 12, location 15, organization 15, user 12 | `pages/Facility` **272 ملف/76K**، `pages/Organization`, `components/Users`, `components/Location` | UI+API client |
| M12 | Devices | settings/devices | device 16 | `pages/Facility/settings/devices` (لم تُقرأ) | UI+API client (6 اختبارات E2E) |
| M13 | Consent | داخل patient | consent 8 | `components/Consent` 1 ملف/594 | UI+API client |
| M14 | Files / Attachments | — | files 7, file 1 | `components/Files` 11/2.6K | UI+API client |
| M15 | Resource Requests (تحويلات/طلبات موارد) | 5 `/resource` | resourceRequest 6 | `components/Resource` | UI+API client |
| M16 | Admin (مؤسسات، أدوار، value sets، tags، هويات) | 16 `/admin` | valueSet 15, emr/role, permission, tagConfig | `pages/Admin` 12/3.4K؛ `components/ValueSet` | UI+API client |
| M17 | Plugins / Apps store / Extensions | `/apps` + مسارات إضافات | extensions، plugConfig | `PluginEngine.tsx`, `components/Extensions`, `pages/Apps` | Implemented FE-side؛ أمن/حوكمة = Unverified |
| M18 | Licenses/SBOM, Landing, Dashboard | `/licenses`, `/`, | — | `pages/Licenses`, `Landing`, `UserDashboard.tsx` | UI |

## 2. تتبع الميزات (Traceability) — عينات مُتحقَّقة
| الميزة | UI | State | API client | Backend |
|---|---|---|---|---|
| تسجيل الدخول | `components/Auth/Login.tsx` | `AuthUserProvider` (useMutation) | `authApi` (`src/types/auth`) | **External/Unverified** |
| تجديد التوكن | — | `useQuery` refetchInterval (`AuthUserProvider.tsx:70-77`) | `authApi.tokenRefresh` | External |
| تحميل الإضافات | `PluginEngine` | React Query `["enabled-plugins"]` | `plugConfigApi.list` | External |
| قراءة الصلاحيات | `PermissionProvider` في `AppRouter` | Context | `user.permissions` من `currentUser` | External |
تتبع بقية الميزات: `evidence/feature_traceability.md`.

## 3. مؤشرات "غير مكتمل/مؤقت" (من grep، 31 TODO في `src/`)
- `CloneQuestionnaireSheet.tsx:98` و`QuestionnaireEditor.tsx:725`: حقل `version` مؤقت "حتى يتولاه BE" ⇒ اقتران مع Backend.
- `PatientRegistration.tsx:426,443`: "add progress bar".
- `QuestionnaireForm.tsx:574`: حجب التنقل ينتظر الانتقال إلى TanStack Router.
- لا Mock/placeholder بيانات مزيفة مرصودة في هذه العينة (لم يُبحث بشمول).

## 4. ما لا يمكن استنتاجه
قواعد العمل (business rules)، التحقق من جهة الخادم، التقارير الخلفية، الإشعارات، المعالجة الخلفية: **Unverified** (Backend).

## 5. Modules بانتظار قراءة معمّقة (الدفعة التالية)
M04، M05، M07، M08 (الأكبر والأعلى مخاطرة).
