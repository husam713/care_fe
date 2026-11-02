# 00 — Repository Inventory

## 1. هوية المستودع
| الحقل | القيمة | الدليل |
|---|---|---|
| الاسم/الإصدار | `care_fe` v3.0.5 (private: true) | `package.json:2-4` |
| الوصف | "Digital Public Good enabling TeleICU & Decentralised Administration of Healthcare Capacity" | `package.json:4` |
| Upstream المعلن | `git@github.com:ohcnetwork/care_fe.git` | `package.json:8` |
| Origin الفعلي | `https://github.com/husam713/care_fe` | `git remote -v` |
| الرخصة | MIT، Copyright (c) 2024 Open Healthcare Network | `LICENSE:1-3` |
| Default branch | `develop` | `AGENTS.md` (Git Workflow) |
| Submodules / LFS | لا يوجد `.gitmodules`؛ `.gitattributes` موجود (لم يُفحص LFS) | `ls` |
| Node | 22 (Dockerfile)، `.node-version` موجود | `Dockerfile:2` |

## 2. جدول الجرد
| Category | Status | Evidence | Notes |
|---|---|---|---|
| Source files | Inspected (آلياً)/ جزئياً يدوياً | `src/` 1082 ملف: tsx 796 + ts 427 (إجمالي المستودع) | أكبر: `components` 395 ملف/94K سطر، `pages` 402 ملف/106K سطر |
| Configuration | Inspected | `vite.config.mts`, `tsconfig.json`, `eslint.config.mjs`, `care.config.ts`, `tailwind.config.js`, `knip.json`, `components.json` | |
| Tests | Executed (E2E) | `tests/` 150 ملف، 109 `*.spec.ts` + 1 unit test | 379/413 نجحت محلياً (انظر 09 §6) |
| Database definitions | **غير موجودة في هذا المستودع** | لا migrations/ORM/SQL | موجودة في `husam713/care` (انظر `backend/B02`) |
| API definitions | Inspected (آلياً) | 65 ملف `src/**/*Api.ts` = 422 تعريف | العقد على جهة العميل فقط |
| Deployment files | Inspected | `Dockerfile`, `nginx/nginx.conf`, `.github/workflows/*` (32 ملف) | لا docker-compose ولا IaC |
| Documentation | Inspected (جزئياً) | `README.md`, `docs/*.md` (7), `AGENTS.md`, `CLAUDE.md`, `UPGRADING.md` | |
| Git history | **Incomplete (shallow)** | 50 commits | يمنع Provenance تاريخي |
| External dependencies | Inspected | 90 dependencies + 42 devDependencies + 4 optional (`package.json:64-205`) | Backend خارجي |
| i18n | Inspected | `public/locale/{en,hi,kn,ml,mr,ta}.json`، en = 6856 مفتاحاً | لا ar |
| AI-agent config | Identified | `.cursor/`, `.codex/`, `.opencode/`, `.entire/`, `.github/agents|aw|hooks|instructions`, `.cursorrules` | يحتوي workflows وكلاء بصلاحيات (غير مراجعة) |

## 3. توزيع الملفات (git ls-files)
`src` 1082 · `tests` 150 · `.github` 55 · `public` 33 · `.entire` 9 · `scripts` 8 · `docs` 7 · `.cursor` 6 · `plugins` 5 · ملفات الجذر ≈ 35.

## 4. أكبر الملفات (أسطر)
| الملف | الأسطر |
|---|---:|
| `src/CAREUI/icons/UniconPaths.json` | 4857 |
| `src/components/Questionnaire/QuestionnaireEditor.tsx` | 3258 |
| `src/components/Questionnaire/QuestionTypes/MedicationRequestQuestion.tsx` | 2403 |
| `src/types/emr/medicationRequest/medicationRequest.ts` | 1773 |
| `src/pages/Facility/billing/account/components/PrintChargeItems.tsx` | 1677 |
| `src/pages/Facility/billing/invoice/InvoiceShow.tsx` | 1608 |

## 5. تصنيف الملفات
- **Inspected**: انظر `ANALYSIS_MASTER_INDEX.md` §4.
- **Generated**: `src/supportedBrowsers.ts` (gitignore:35)، `build/`, `package-lock.json`.
- **Externally referenced**: `ohcnetwork/care` (API)، `care_scribe_fe`, `care_hello_fe`, `care_teleicu_devices_fe`, `care_apps_registry`, `care-fe-code-reviewers`, `branding`.
- **Inaccessible في هذه الجلسة**: كل ما سبق (نطاق GitHub محصور في `husam713/care_fe`).

## 6. قيد الاكتمال
**لا يُدّعى تغطية شاملة.** التغطية اليدوية ≈ 3% من ملفات `src`؛ الباقي بفحص آلي نمطي (patterns). انظر `evidence/source_inventory.md`.
