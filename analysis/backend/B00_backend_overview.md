# B00 — Backend Overview (المرحلة الثانية)

**المستودع المفحوص:** `husam713/care` (نسخة من `ohcnetwork/care`) · الفرع `develop` · Commit `67a46185eb98ee4b79b38ecee84d8a69d173306e` (2026-11-01) · **clone سطحي (depth 1)** ⇒ لا تاريخ.
**الرخصة:** MIT، Copyright (c) 2026 Open Healthcare Network Foundation (`LICENSE`).
**الحدود:** فحص ساكن فقط. لم يُشغَّل الخادم ولا الاختبارات (تتطلب PostgreSQL وRedis). لم يُفحص `docs/` ولا `locale/` ولا `plugs/` بعمق.

## 1. الحجم
| المقياس | القيمة | الدليل |
|---|---:|---|
| الملفات المتتبعة | 1125 (1013 `.py`) | `git ls-files` |
| الأسطر (كل الملفات) | ≈ 136,000 | `wc -l` |
| التوزيع | `care/emr` 578 ملفاً · `care/facility` 179 · `care/security` 98 · `care/utils` 59 · `care/users` 47 | `git ls-files care` |
| Migrations | 295 (facility 172، emr 86، users 27، security 6، sites 4) | `find care -path '*/migrations/*.py'` |
| نماذج Django الفعلية | ≈ 89 (heuristic، ‏`/tmp` استخراج آلي) | `care/emr/models`, `care/facility/models`, `care/users/models.py`, `care/security/models` |
| ViewSets مسجلة في الراوتر | 85 تسجيلاً | `config/api_router.py` (518 سطراً) |
| ملفات الاختبار | 80 `test_*.py` | `find care -name 'test_*.py'` |
| Workflows | 8 (`test-pull-request`, `test-merge-queue`, `reusable-test`, `linter`, `deploy`, `release`, `docs`, `validate-pr-title`) | `.github/workflows` |

## 2. المكدس (Verified من `Pipfile`/`pyproject.toml`)
Python 3.13 (`requires-python ==3.13.*`) · **Django 6.0** · DRF 3.16.1 · simplejwt 5.5.1 · drf-nested-routers · drf-spectacular (OpenAPI) · Pydantic 2.12.5 (مخططات الطلب/الاستجابة) · Celery 5.6 · PostgreSQL (psycopg 3) · Redis (django-redis) · gunicorn 23 · whitenoise · boto3 (S3) · Sentry · pyotp (TOTP) · authlib · weasyprint (PDF) · evalidate · django-ratelimit · django-import-export · django-anymail (SES) · pywebpush.
Docker compose: `postgres:17-alpine`، `redis:8-alpine`، `minio` (`docker-compose.yaml`).

## 3. البنية
```
config/        settings (base/local/deployment/production/staging/test)، api_router، auth، ratelimit، celery، exception_handler
care/emr/      النواة السريرية: models (≈75) · resources (Pydantic specs) · api/viewsets · api/otp_viewsets · tasks · fhir · reports · registries
care/facility/ نماذج Facility (+ 172 migration تاريخية)
care/security/ authorization (handlers لكل مورد) · permissions (ثوابت) · models (Role/Permission/RolePermission)
care/users/    User, PlugConfig, Skills, flags, reset-password, OTP
care/audit_log/ middleware + receivers (427 سطراً)
plugs/         نظام plugs الخلفي (manager.py, plug.py)
```
نمط الموارد: **Pydantic spec ↔ Django model ↔ EMRBaseViewSet mixins** (`care/emr/api/viewsets/base.py`)، مع `EMRBaseModel` (حقول `history`, `meta` JSON، `created_by`, `updated_by`، soft delete عبر `deleted`).

## 4. علاقته بالـ Frontend (يغلق أسئلة)
| سؤال FE | الجواب |
|---|---|
| أين الـ Backend وقاعدة البيانات؟ | Django 6 + PostgreSQL + Redis (Verified) |
| من يحق له تعديل `plug_config`؟ | `IsAdminUser` (Django `is_staff`) للكتابة؛ القراءة بدون مصادقة (انظر B01-§4) |
| كيف تُفرض الصلاحيات؟ | داخل الخادم عبر RBAC على شجرة Organizations (انظر B01) — الواجهة تجميلية فعلاً |
| أعمار التوكنات؟ | access 10 دقائق، refresh 30 دقيقة، مع rotation (`base.py:403-411`) — يتوافق مع تجديد FE كل 5 دقائق |
| مصدر العقد؟ | الـ Backend يولّد OpenAPI عبر drf-spectacular ⇒ يمكن توليد أنواع FE منه (يعالج فجوة 06) |
