# B01 — Authentication, Authorization & Tenant Isolation

## 1. المصادقة (Verified)
| الجانب | التفصيل | الدليل |
|---|---|---|
| الفئات الافتراضية | `CustomJWTAuthentication`, `CustomBasicAuthentication`, `SessionAuthentication`, `TokenAuthentication` | `config/settings/base.py:366-373` |
| في `production` | **JWT فقط** (يُستبدل `DEFAULT_AUTHENTICATION_CLASSES`) | `config/settings/production.py:3-5` |
| في `staging` | يرث `deployment` بلا حصر (أي Basic/Session/Token نشطة) و`ALLOWED_HOSTS=["*"]` | `config/settings/staging.py:1-10` |
| JWT | access 10 د، refresh 30 د، `ROTATE_REFRESH_TOKENS=True`، `USER_ID_FIELD=external_id` | `base.py:403-411` |
| إبطال | عند logout تُخزَّن إبطالات في الكاش (Redis) بمفاتيح `ACCESS_TOKEN_INVALIDATE:` و`REFRESH_TOKEN_INVALIDATE:` | `config/auth_views.py:22-23,195-200`، `authentication.py:21-24` |
| Blacklist | تطبيق `token_blacklist` **غير مثبت** ⇒ لا إبطال عند الـ rotation | grep `token_blacklist` في `base.py` = لا شيء |
| تسجيل الدخول | rate limit عبر `django-ratelimit` ثم reCAPTCHA (HTTP 429 `captchaRequired`) | `auth_views.py:54-66`، `config/ratelimit.py` |
| 2FA | TOTP (`pyotp`) + `mfa_login` + رموز احتياطية | `care/emr/api/viewsets/totp.py`, `mfa_login.py`, `care/emr/utils/mfa.py` |
| مريض | JWT منفصل `PatientToken` + OTP (إعدادات `OTP_MAX_*`) | `config/patient_otp_*.py`, `base.py:511-529` |
| كلمات المرور | Argon2 هو الـ hasher الأول (ثم PBKDF2 وBCrypt للتوافق) | `base.py:173-179` |

## 2. التفويض — النموذج (Verified)
1. **RBAC على شجرة Organizations:** `User → OrganizationUser / FacilityOrganizationUser → RoleModel → RolePermission → PermissionModel(slug)`. الصلاحيات نصية (`can_list_patients`، `can_write_patient`…) وتطابق ثوابت FE `src/common/Permissions.tsx` (68 ثابتاً).
2. **Handlers لكل مورد** (≈35 ملفاً في `care/security/authorization/`) تُسجَّل في `AuthorizationController` وتستدعى بـ `AuthorizationController.call("can_view_clinical_data", user, patient)`؛ ويمكن للـ plugs استبدالها (`override_authz_controllers`).
3. **وصول المريض** (`care/security/authorization/patient.py:17-48`): الأدوار تُستنتج من (أ) Encounters غير المكتملة ومواقعها (`facility_organization_cache`)، (ب) Organizations المرتبطة بالمريض (`organization_cache`)، (ج) ربط مباشر `PatientUser`.
4. **تصفية القوائم:** `get_filtered_patients` عبر تقاطع `organization_cache`/`users_cache` (`patient.py:95-112`).
5. **Superuser يتجاوز كل شيء:** `if user.is_superuser: return True` في كل handler (55 موضع `is_superuser/is_staff` في `care/emr/api` + `care/security`).
6. **علم ميزة:** `PATIENT_GLOBAL_EDIT_ACCESS_ENABLED` (افتراضي `False`) يسمح بالتعديل/الاطلاع الشامل عند تفعيله (`config/settings/config.py:301`، `patient.py:61`، `viewsets/patient.py:100`) — خطر إن فُعِّل خطأً.

## 3. عزل المستأجرين (Tenant isolation)
**ليس عزلاً على مستوى قاعدة البيانات/Schema:** قاعدة واحدة، والعزل منطقي عبر شجرة Organizations/Facilities وأعمدة cache (`*_organization_cache`) وفلاتر queryset. ‏**الاستنتاج:** سلامة العزل تعتمد على أن **كل** viewset يطبّق فلتراً صحيحاً — انظر النتيجة B-06 أدناه.

## 4. PlugConfig (يغلق S-03)
`care/users/api/viewsets/plug_config.py`:
- `list/retrieve`: `get_permissions → []` و`get_authenticators → []` للـ GET ⇒ **بدون مصادقة** (عام). هذا مقصود لأن الـ FE يستدعيه قبل الدخول، ويُخزَّن في الكاش.
- `create/update/destroy`: `IsAdminUser` (Django `is_staff`).
- `meta` حقل `JSONField` بلا تحقق من الـ URL أو allowlist (`serializers/plug_config.py` = `exclude=("id",)` فقط).
⇒ **حدّ الثقة:** من يملك `is_staff` يستطيع جعل جميع المستخدمين يحمّلون JS من أي عنوان. كل ما سبق يعزز توصية TP-006 (allowlist في FE) ويضيف: **قيّد `is_staff` بشدة** وسجّل تغييرات PlugConfig في الـ audit.

## 5. الكاش والأداء في التفويض
`get_role_from_permissions` فيها `# TODO Cache this endpoint` (`authorization/base.py`) ⇒ استعلام DB عند كل فحص؛ مصدر حمل محتمل (لم يُقَس).

## 6. النتائج
| ID | الخطورة | الوصف | الدليل | الثقة |
|---|---|---|---|---|
| B-01 | High (مشروط) | `SECRET_KEY` له قيمة افتراضية ثابتة في الكود؛ لا يوجد فحص يمنع التشغيل بها في production. إن لم يُضبط `DJANGO_SECRET_KEY` يمكن توقيع JWT صالحة لأي مستخدم يُعرف `external_id` الخاص به (UUID، لا يُخمَّن لكنه قد يظهر عبر الـ API). رموز إعادة التعيين تُوقَّع بالمفتاح نفسه لكنها تضم تجزئة كلمة المرور فتبقى أصعب في التزوير | `base.py:39-42`; `care/emr/utils/reset_password.py:17-30` (JWT HS256 بـ `SECRET_KEY`); `.env.example:70` (معلّق) | High (للكود)، التشغيل الفعلي Unverified |
| B-02 | Medium | `ALLOWED_HOSTS` افتراضي `["*"]`، وفي `staging.py` مفروض `*` | `base.py:44`, `staging.py:3` | High |
| B-03 | Low | HSTS = 60 ثانية فقط افتراضياً | `deployment.py:36` | High |
| B-04 | Low–Medium | rotation بلا blacklist: refresh القديم يبقى صالحاً حتى انتهاء مدته (30 د)؛ الإبطال يعتمد على Redis فقط | `auth_views.py:108-125`; غياب `token_blacklist` | Medium |
| B-05 | Medium | `staging` يتيح Basic/Session/Token auth (وصف الـ schema نفسه يقول "Do not use in production") | `staging.py`, `authentication.py:46-53` | High |
| B-06 | Medium | **التفويض opt-in:** هوكات `authorize_create/update/retrieve/destroy` في `EMRBaseViewSet` تعمل `pass` افتراضياً؛ و`CareAuthentication` تُرجع `True` إن لم يعرّف الـ view `permissions_controller` (`permission_class.py:8-11`). ‏أي viewset ينسى تجاوز الهوك يصبح مفتوحاً لأي مستخدم مصادَق عليه. فحص آلي لـ 59 viewset: 27 لا تتجاوز ≥1 هوك، أغلبها `authorize_retrieve` فقط (غالباً يُعوَّض بـ `get_queryset` المفلتر، كما في `PatientViewSet`)، لكن `ResourceRequestViewSet` و`ResourceRequestCommentViewSet` لا تحتوي أي `authorize_*` وتعتمد على `build_queryset` للرؤية فقط، و`ValueSetViewSet` يعتمد على `permissions_controller` | `base.py:77,130,222,247`; `resource_request.py:48-93` | Medium — يحتاج مراجعة يدوية لكل حالة؛ لم يُثبت استغلال |
| B-07 | Info | ‏6 مجموعات endpoints بلا مصادقة (OTP login، reset password، facility عام، organization عام، mfa_login، plug_config GET) — بعضها محمي بـ rate limit/captcha | grep `authentication_classes = []` | High |

**توصية:** جعل الهوكات `raise NotImplementedError`/deny-by-default مع opt-out صريح، واختبار آلي يمر على كل viewset مسجَّل ويتأكد من وجود تغطية صلاحيات.
