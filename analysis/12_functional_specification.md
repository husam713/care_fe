# 12 — Functional Specification (مستوى Module)

> **حدود:** المتطلبات أدناه مستخلصة من الواجهة وعقود الـ API (جهة العميل). قواعد العمل الخادمية **غير متحقق منها**. لكل متطلب وسم: **EV** (Existing & verified في FE)، **EP** (Existing partially verified)، **INF** (Inferred)، **MIS** (Missing)، **PE** (Proposed enhancement — منفصل).

## FR-AUTH — المصادقة
| ID | المتطلب | الوسم | الدليل |
|---|---|---|---|
| FR-AUTH-001 | دخول موظف باسم مستخدم/كلمة مرور وحفظ JWT | EV (FE) | `AuthUserProvider.tsx:95-136` |
| FR-AUTH-002 | دعم 2FA عند وجود `temp_token` وتحويل إلى `/2fa` | EV | `AuthUserProvider.tsx:31-38,97-106` |
| FR-AUTH-003 | تجديد دوري للتوكن وإسقاط الجلسة عند فشله | EV | `:70-92` |
| FR-AUTH-004 | إعادة تعيين كلمة المرور (بريد/هاتف) | EP | `ForgotPassword*`, `ResetPassword.tsx` (لم تُقرأ كاملة) |
| FR-AUTH-005 | دخول مريض عبر OTP + مسار `/patient/*` | EP | `PatientRouter`, `types/otp` |
| FR-AUTH-006 | إعادة توجيه آمنة بعد الدخول (same-origin) | EV | `:224-238` |
| FR-AUTH-007 | إنهاء الجلسة عبر `/session-expired` عند `token_not_valid` | EV | `errorHandler.ts` |
| FR-AUTH-008 | تخزين JWT في ‏HttpOnly cookie | PE | — |

## FR-PATIENT / FR-ENCOUNTER / FR-APPOINTMENT (مستوى Route/API فقط)
| ID | المتطلب | الوسم |
|---|---|---|
| FR-PATIENT-001 | تسجيل/تعديل مريض (`PatientRegistration.tsx` 1198 سطراً) | EP |
| FR-PATIENT-002 | بحث وقوائم المرضى والهويات والوسوم | EP |
| FR-PATIENT-003 | ملف مريض بتبويبات (`/facility/:id/patient/:pid/*`، 26 مساراً) | EP |
| FR-ENCOUNTER-001 | إنشاء/عرض Encounter وإدخال بيانات سريرية عبر Questionnaires | EP |
| FR-ENCOUNTER-002 | وصفات/صرف/إعطاء أدوية وطباعة | EP |
| FR-ENCOUNTER-003 | طلبات خدمة (مختبر/تصوير) وتقارير تشخيصية متعددة | EP (`ENG-500` في السجل) |
| FR-APPOINTMENT-001 | جداول وفترات وحجز موعد داخلي | EP |
| FR-APPOINTMENT-002 | حجز موعد عام بدون حساب موظف (OTP) | EP |
| FR-QUEUE-001 | قوائم انتظار وTokens | EP |
| FR-BILLING-001 | حساب/فاتورة/بنود شحن/مدفوعات/مطابقة | EP (TODO: مقاييس BE) |
| FR-INVENTORY-001 | مخزون، أوامر شراء، تحويلات داخلية، توصيلات | EP |
| FR-ADMIN-001 | إدارة المؤسسات والأدوار والـ ValueSets والوسوم | EP |
| FR-PLUGIN-001 | تحميل إضافات وتوسيع المسارات/المكونات واستبدالها | EV (FE) |
| FR-I18N-001 | تعدد لغات (en, hi, kn, ml, mr, ta) | EV |
| FR-I18N-002 | دعم العربية وRTL | **MIS** / PE |

## قبول (نموذج)
FR-AUTH-003: *Given* مستخدم مسجل، *When* ينقضي `tokenRefreshInterval`، *Then* يُستدعى `authApi.tokenRefresh` وتُحدَّث المفاتيح؛ *And When* يفشل الاستدعاء، *Then* تُحذف المفاتيح وتنتقل الجلسة إلى منتهية. (قابل للتحقق عبر Playwright مع Backend — لم يُنفَّذ.)

## ملاحظة
بقية المتطلبات التفصيلية (Actors، Preconditions، Alternative flows لكل module) **لم تُنتَج** لأنها تتطلب قراءة الشاشات وقواعد الخادم؛ مؤجلة للدفعة التالية.
