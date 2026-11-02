# Source Inventory & Coverage

## الأرقام (Commit 96d7e93)
| المقياس | القيمة | أمر |
|---|---:|---|
| ملفات متتبعة | 1397 | `git ls-files \| wc -l` |
| `src/` | 1082 ملف / 233,599 سطر | `wc -l` |
| tsx / ts | 796 / 427 | امتدادات `git ls-files` |
| `tests/` | 150 (109 spec) | |
| `.github/` | 55 (32 workflow) | |
| Routes فريدة | 144 | `evidence/routes_all.txt` |
| ملفات `*Api.ts` | 65 → 422 تعريفاً | `evidence/api_inventory_raw.tsv` |

## حالة الفحص حسب المجلد
| المجلد | ملفات | الأسطر | نوع الفحص |
|---|---:|---:|---|
| `src/pages` | 402 | 106,044 | آلي (grep) + Routers يدوياً؛ لا قراءة شاشات |
| `src/components` | 395 | 94,338 | آلي؛ `ui/markdown.tsx`، `Auth/AuthHero.tsx`(جزئي)، `Users/BackupCodesDialog.tsx`(جزئي) يدوياً |
| `src/types` | 153 | 12,278 | آلي (استخراج API)؛ ملفان يدوياً (TODO) |
| `src/Utils` | 34 | 4,229 | `request/query.ts`, `utils.ts`, `errorHandler.ts`, `queryClient.ts` يدوياً |
| `src/hooks` | 30 | 4,481 | آلي فقط |
| `src/CAREUI` | 13 | 6,458 | غير مفحوص (أيقونات) |
| `src/Routers` | 11 | 1,103 | `AppRouter`, `PublicRouter` (جزئي) |
| `src/Providers` | 2 | 308 | `AuthUserProvider.tsx` يدوياً |
| `src/context` + `common` | 7 | 1,173 | `PermissionContext.tsx`، `Permissions.tsx` (أول 60 سطراً) |
| `src/PluginEngine.tsx` (+ pluginTypes) | 2 | 452 | `PluginEngine.tsx` أول 140 سطراً |
| `tests/` | 150 | — | توزيع فقط |
| `.github/workflows` | 32 | — | عناوين/خطوات 6 ملفات |
| إعداد الجذر | ~25 | — | `package.json`, `.env`, `Dockerfile`, `nginx`, `vite.config` (مقاطع), `care.config` (مقاطع) |

**تقدير القراءة اليدوية:** ≈ 35 ملفاً (≈ 2.5% من الملفات)؛ ‏ملفات `src` المفتوحة يدوياً ≈ 20.

## ملفات مُولَّدة/مُتجاهَلة
`src/supportedBrowsers.ts` (مُولَّد، gitignore)، `build/`، `node_modules/` (مثبت للتحقق فقط، غير متتبع).
