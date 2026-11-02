# 01 — Provenance & Originality

## الحكم الموجز
`care_fe` **ليس مشروعاً أصلياً لهذا الحساب**؛ هو نسخة (fork/mirror) من مشروع Open Healthcare Network (OHC). الدليل على الأصل قوي، لكن **التحقق التاريخي مستحيل حالياً** لأن الـ clone مبتور.

| Finding | Evidence | Candidate Upstream | Confidence | Explanation |
|---|---|---|---|---|
| المالك الأصلي هو Open Healthcare Network | `package.json:8,10-13,27` (repo URL، bugs، author)؛ `LICENSE:3`؛ README (شارات `ohcnetwork/care_fe`)؛ 26 إشارة نصية لـ `ohcnetwork/care_fe` | `ohcnetwork/care_fe` | **Verified** (من metadata) | بيانات الحزمة تعلن المصدر صراحة |
| الـ origin الحالي نسخة شخصية | `git remote -v` → `husam713/care_fe` | `ohcnetwork/care_fe` | **Strongly supported** | علم "fork" في GitHub لم يُتحقَّق منه (خارج النطاق) |
| لا تعديلات whitelabel جوهرية بعد | `.env` يحمل قيم CARE/OHC الأصلية (`REACT_APP_TITLE=CARE`, `REACT_PUBLIC_URL=https://care.ohc.network`) | — | Medium | لم تُقارَن الملفات بنسخة upstream |
| Commits حديثة بتوقيعات ومساهمين OHC | أكثر المؤلفين: Rithvik Nishad (13)، Abhimanyu Rajeesh (11)، Bodhish Thomas (6)، Yaswanth Kumar Bethu (5)، Copilot (3)، Claude (2) في آخر 50 commit | — | Medium | عينة 50 commit فقط |
| الرخصة MIT متوافقة مع إعادة الاستخدام والتعديل | `LICENSE` | — | Verified | شرط الإبقاء على إشعار الحقوق |
| Copyright 2024 بينما أقدم commit محلي = 2026-09-07 | `git log --reverse` | — | **Not established** | تفسير محتمل: History مبتور، لا تناقض |

## تاريخ المستودع (المتاح)
- 50 commit، من 2026-09-07 إلى 2026-11-01. أحدث commit: `96d7e93 [ENG-1174] Fix dispenses not listed ...(#17033)`.
- ترقيم PRs حتى #17033 يدل على مشروع نشط جداً (مؤشر، لا دليل).
- ملاحظة: التاريخ في هذه البيئة قد يكون مصطنعاً/معاداً؛ لا يعتمد عليه إلا بعد `git fetch --unshallow` ومقارنته بـ upstream.

## المنظومة المرتبطة (من الوثائق فقط — غير متحقق منها)
| المستودع | المسؤولية (حسب الوثائق) | الدليل | الحالة |
|---|---|---|---|
| `ohcnetwork/care` | Backend (Django/Python 3.13، PostgreSQL، Redis) والـ API على المنفذ 9000 | `CLAUDE.md`, `docs/local-development.md`, `.env.docker` | **Unverified** — غير متاح |
| `ohcnetwork/care_scribe_fe` | Plugin (Module Federation) | `.env` تعليقات `REACT_ENABLED_APPS` | Unverified |
| `ohcnetwork/care_hello_fe` | Plugin نموذجي | `docs/care-apps-*.md` | Unverified |
| `ohcnetwork/care_teleicu_devices_fe` | Plugin أجهزة TeleICU | 1 إشارة | Unverified |
| `ohcnetwork/care_apps_registry` | فهرس التطبيقات `index.json` | `.example.env` `REACT_APP_STORE_INDEX_URL` | Unverified |
| `ohcnetwork/care-fe-code-reviewers` | فريق المراجعة | README, CODEOWNERS | — |
| `ohcnetwork/branding` | أصول الهوية | README | — |

## حدود وإجراءات
1. لم تُجرَ مقارنة ملفية/دالّية مع upstream (يتطلب وصولاً لم يُمنح في هذه الجلسة).
2. **التوصية:** `git fetch --unshallow`, ثم `git merge-base` مع `ohcnetwork/care_fe:develop` وقياس الانحراف (`git diff --stat upstream/develop`).
3. لا تُستنتج أي علاقة بمشاريع أخرى (مثل مشاريع EMR أخرى) — لا دليل عليها.
