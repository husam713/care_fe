# B03 — API Contract, Security Hardening, Quality & Operations (Backend)

## 1. API
- **85 تسجيل viewset** في `config/api_router.py` (Router: `DefaultRouter` عند DEBUG وإلا `SimpleRouter`) مع routers متداخلة (`NestedSimpleRouter`) تحت `/api/v1/`.
- ‏Pagination: `CareLimitOffsetPagination` (PAGE_SIZE 14)؛ بحث: `search_text`؛ OpenAPI عبر drf-spectacular (`SPECTACULAR_SETTINGS`)؛ استثناءات موحدة `config/exception_handler.py`.
- **Batch endpoint** (`BatchRequestView`) يقابل `src/Utils/request/batch.ts` في FE.
- **تطابق صلاحيات FE/BE (فحص آلي):** من 67 ثابتاً في `src/common/Permissions.tsx` يوجد 65 مطابقاً لـ slug في الـ Backend (113 صلاحية معرّفة في `care/security/permissions/*.py`). المختلفان: `can_manage_template` (الـ BE فيه `can_write_template`/`can_read_template`) و`can_manage_locked_invoice_in_facility` (الـ BE slug هو `can_manage_locked_invoice`؛ أما `..._in_facility` فاسم دالة handler في `authorization/invoice.py:28`). كلا الثابتين **غير مستخدمين** في بقية `src/` ⇒ أثر معدوم حالياً، لكنه دليل على انحراف ثوابت مكتوبة يدوياً (يعزز توصية توليد الأنواع من OpenAPI). 48 صلاحية Backend لا مقابل لها في FE (طبيعي إن لم تكن لها واجهة).

## 2. تحصين أمني (Backend) — ما هو جيد
- JWT قصير العمر (10 د) + rotation + إبطال عند logout.
- rate limit + captcha للدخول وإعادة التعيين وMFA؛ حدود OTP (`OTP_MAX_SENDS_PER_WINDOW=10`, `OTP_MAX_VERIFY_ATTEMPTS=3`, `OTP_MAX_FAILURES=5`).
- TOTP 2FA + رموز احتياطية.
- `X_FRAME_OPTIONS="DENY"`، `SESSION_COOKIE_HTTPONLY/CSRF_COOKIE_HTTPONLY`، في deployment: `SECURE_SSL_REDIRECT` (افتراضي True)، `SESSION_COOKIE_SECURE`، `nosniff`، HSTS (لكن 60 ث فقط).
- CORS مغلق افتراضياً في deployment (`CORS_ALLOWED_ORIGINS` افتراضي `[]`)؛ مفتوح فقط في `local.py:12`.
- رفع الملفات عبر Bucket خارجي (S3/MinIO) مع فحص نوع بـ `python-magic` وتفويض لكل نوع ملف (`file_authorizer` في `viewsets/file_upload.py:1-80`: patient/encounter/consent/diagnostic_report…).
- Audit log middleware (`care/audit_log`).

## 3. النتائج (متممة لـ B01)
| ID | الخطورة | الوصف | الدليل |
|---|---|---|---|
| B-08 | Low | `CustomBasicAuthentication` و`SessionAuthentication` مفعلتان خارج production | `base.py:366-373`, `staging.py` |
| B-09 | Info | `local.py` يفتح `CORS_ORIGIN_ALLOW_ALL=True` (للتطوير فقط) | `local.py:12` |
| B-10 | Info | `facility` (172 migration) مستثناة من ruff والتغطية ⇒ جودة أقل في الكود القديم | `pyproject.toml` |
| B-11 | Info | `get_role_from_permissions` بلا cache (TODO) ⇒ كلفة DB لكل فحص | `authorization/base.py` |

## 4. جودة وعمليات
| الجانب | الحالة | الدليل |
|---|---|---|
| الاختبارات | 80 ملف `test_*.py`، ‏CI على كل PR وmerge queue، تغطية Codecov | `.github/workflows/reusable-test.yml` وأخواتها |
| Lint | Ruff بقواعد واسعة تشمل `S` (bandit) | `pyproject.toml` |
| نشر | Docker (dev/prod)، ECS scripts (`scripts/*-ecs.sh`)، Coolify compose، Nix flake | `docker/`, `scripts/` |
| تشغيل | gunicorn، Celery worker + beat، health checks (`healthy-django`, `scripts/healthcheck.sh`)، backup script | `config/gunicorn.py`, `scripts/` |
| المراقبة | Sentry، تسجيل زمن الطلبات (`RequestTimeLoggingMiddleware`) | `deployment.py`, `config/middlewares.py` |
| نتائج الاختبارات الفعلية | 2005 اختبار: 1991 نجح، 14 فشل (كلها رفع ملفات، سبب بيئي: لا bucket/MinIO) في 56.6 ثانية | تشغيل محلي، انظر `09` §7 |
| ما لم يُنفَّذ | `ruff` و`pip-audit` | — |

## 5. خلاصة
الـ Backend أنضج من حيث نموذج الصلاحيات والتوثيق والاختبارات، وأهم مخاطره **تصميمية** (التفويض opt-in، تجاوز superuser، `SECRET_KEY` افتراضي، staging متساهل) وليست ثغرات مستغلة مثبتة. لا ينبغي نشره دون: ضبط `DJANGO_SECRET_KEY` إلزامياً، ورفع HSTS، وتقييد `ALLOWED_HOSTS`، واختبار آلي يمنع viewset بلا تغطية صلاحيات.
