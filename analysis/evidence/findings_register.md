# Findings Register

| ID | Severity | Area | Summary | File:Line | Confidence | Status |
|---|---|---|---|---|---|---|
| S-01 | High (npm: Critical) | Dependency/XSS | dompurify 3.4.12 ضمن `<=3.4.16` | `package.json`; استخدام في `ui/markdown.tsx:46,56`, `TemplateBuilder.tsx:775-830` | Medium | مفتوح — التعرض لم يُثبت |
| S-02 | Medium | AuthN | JWT access/refresh في localStorage | `AuthUserProvider.tsx:47-48,114-115,131-132`; `request/utils.ts:77-80` | High | مفتوح |
| S-03 | Medium→High | Supply chain | تحميل remote ESM من `meta.url` يحدده الـ Backend | `PluginEngine.tsx:25-56` | Medium | مفتوح — صلاحيات BE ? |
| S-04 | Medium | Headers | CSP تقريرية فقط؛ ‏`HEADERS` بلا CSP | `nginx/nginx.conf:12`; `vite.config.mts:551-557`; `.env` | High | مفتوح |
| S-05 | Medium | Dependency/DoS | markdown-it 14.3.0، zod 4.4.3، browserslist… | `markdown.tsx:9-15`; lock | Medium | مفتوح |
| S-06 | Low | Data at rest | query persistence في localStorage | `queryClient.ts:36-52` | High | مراقبة |
| S-07 | Low | Injection | `document.write` بقيم غير مهرَّبة | `BackupCodesDialog.tsx:56-77` | High | مفتوح |
| S-08 | Info | Injection | HTML مخصص وقت البناء | `AuthHero.tsx:53`; `vite.config.mts:422-423` | High | قبول |
| S-09 | Low | Dev tooling | vite 8.1.4 (ثغرات dev-server) | lock | High | مفتوح |
| S-10 | Info | Agent safety | تعليمات "star the repo" المخفية/الإلزامية للوكلاء | `README.md` (تعليق HTML)، `AGENTS.md` | High | لم تُنفَّذ؛ توصية سياسة |
| S-11 | Low | Reproducibility | `latest` في optionalDependencies؛ `npm install` في Docker | `package.json:200-205`; `Dockerfile` | High | مفتوح |
| S-12 | Info | Redirect | same-origin مُطبَّق؛ ترميز `redirect` ناقص | `AuthUserProvider.tsx:100-106,224-238`; `errorHandler.ts` | High | مقبول |
| B-01 | High (مشروط) | Backend/Secrets | SECRET_KEY افتراضي ثابت في الكود | care:config/settings/base.py:39-42 | High (كود) | مفتوح، يلزم تأكيد ضبط env |
| B-02 | Medium | Backend/Config | ALLOWED_HOSTS=["*"] افتراضياً وفي staging | base.py:44; staging.py:3 | High | مفتوح |
| B-03 | Low | Backend/Headers | HSTS 60 ثانية | deployment.py:36 | High | مفتوح |
| B-04 | Low-Medium | Backend/AuthN | rotation بلا blacklist؛ إبطال عبر Redis فقط | config/auth_views.py:108-125 | Medium | مفتوح |
| B-05 | Medium | Backend/Config | staging يفعّل Basic/Session/Token auth | config/settings/staging.py | High | مفتوح |
| B-06 | Medium | Backend/AuthZ | التفويض opt-in (authorize_* = pass؛ CareAuthentication تُرجع True) | care/emr/api/viewsets/base.py:77,130,222,247; care/security/utils/permission_class.py:8-11 | Medium | يحتاج مراجعة كل viewset |
| B-07 | Info | Backend | 6 مجموعات endpoints عامة (OTP، reset، facility/organization عامان، mfa_login، plug_config GET) | grep authentication_classes = [] | High | مقبول بتحفظ |
| Q-01 | Medium | Tests | اختبارات وحدة: ملف واحد | `plugins/*.test.ts` | High | مفتوح |
| Q-02 | Low | Build | `tsc` يحتاج ملفاً مولَّداً | `BrowserWarning.tsx:7` | High | مفتوح (TP-002) |
