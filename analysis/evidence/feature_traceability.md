# Feature Traceability (عينات متحقق منها)

| Feature | UI | State | API client | Endpoint | Backend/DB | الوسم |
|---|---|---|---|---|---|---|
| Login | `components/Auth/Login.tsx` | `AuthUserProvider` (`useMutation`) | `src/types/auth/authApi.ts` | `POST /api/v1/auth/login/` (`authApi.ts:24`) | External | UI→API verified |
| Token refresh | — | `useQuery` + `refetchInterval` (`AuthUserProvider.tsx:70-77`) | `authApi.tokenRefresh` | `POST /api/v1/auth/token/refresh/` (`authApi.ts:37`) | External | verified (FE) |
| Current user + permissions | `PermissionProvider` في `AppRouter` | `useQuery(["currentUser", accessToken])` (`:58-66`) | `authApi`/`userApi` | — | External | verified (FE) |
| Plugin loading | `PluginEngine` | `useQuery(["enabled-plugins"])`, `useQueries` | `plugConfigApi.list` | — | External | verified (FE) |
| Persisted queries | — | `queryClient.ts` | — | — | n/a | verified |
| Consent create/update/verify | `components/Consent` | — | `consentApi.ts:13-55` | `/api/v1/patient/{patientId}/consent/...` | External | UI→API (UI لم تُقرأ) |
| Care team set | — | — | `careTeamApi.setCareTeam` (`:8`) | `/api/v1/encounter/{encounterId}/set_care_team_members/` | External | API only verified |

> مسارات 2FA: `POST /api/v1/mfa/login/` (`authApi.ts:109`)، `/api/v1/mfa/totp/{setup,verify,regenerate_backup_codes,disable}/`. المرجع الدقيق: `api_inventory_raw.tsv`.
> بقية الميزات لم تُتتبع على كامل المسار — انظر `unresolved_questions.md`.
