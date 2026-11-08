# Commit Report — 74d136f Harden describeMailError: fixed error-name categories, sentinel regression tests

- **Repository:** `vitrixLab/hopkinstalentgroup`
- **Branch:** `feat/inquiry-smtp-email`
- **Reported commit:** `74d136f` (local only, not pushed)
- **PR:** #6 — head remains `a76383b` (deploying READY on Vercel)
- **Report created:** 2026-10-12

## What the commit changed

| File | Change |
| --- | --- |
| `lib/contact/mail.ts` | `describeMailError()` name handling switched from a character pattern (`^[A-Za-z][A-Za-z0-9]{0,49}$`) to a fixed `Set` of known error class names (`Error`, built-ins, `MailConfigError`, `SMTPError`) with `Error` fallback — pattern-validated names were not "fixed safe categories" and alphanumerical injected names could reach logs. `code` stays pattern-allowlisted (`^[A-Z0-9_]{1,32}$`), `responseCode` stays finite-numeric, `MailConfigError` messages retained (all 12 constructor sites verified: fixed text / setting names / validated port only), `try/catch` retained for throwing getters and revoked proxies. HTTP responses unchanged. |
| `lib/contact/mail.test.ts` | +2 tests, 1 extended: unique alnum name sentinel dropped to `Error`; 7 env-injected secrets across 4 real `getMailConfig()` failure paths never appear in descriptions; non-allowlisted `code` sentinels (mixed-case, oversized) dropped. |
| `app/api/contact/route.test.ts` | +2 tests: real-sanitizer `console.error` capture proving sentinels in message/name/code/`response` never reach logs; real `getMailConfig` error built from sentinel env logged with no secrets. Only `sendInquiryMail` is mocked; all spies restored in `finally`. |

## Verification (run after the commit's working state, exit codes)

| Command | Exit | Result |
| --- | --- | --- |
| `npm ls --depth=0` | 0 | All declared deps present (`nodemailer@10.1.0`, `vitest@5.0.4`, `@types/nodemailer@8.0.2`) |
| `npx vitest run lib/contact/mail.test.ts app/api/contact/route.test.ts` | 0 | 65/65 passed |
| `npm test` | 0 | 107/107 passed (6 files) |
| `npm run lint` | 0 | clean |
| `npx tsc --noEmit` | 0 | clean |
| `npm run build` | 0 | Next.js build success, `/api/contact` dynamic |

## Related prior commits (also local-to-remote state at report time)

- `5e4c37f` — Sanitize describeMailError: never copy exception messages into logs (pushed)
- `6e9cd13` — Audit: regression tests for revoked proxies and non-finite response codes (pushed)
- `a76383b` — Resolve lockfile mismatch: remove stale pnpm-lock.yaml (pushed; Vercel preview READY)
- `74d136f` — this report's subject (not pushed)

## Vercel status

- Failed deployments (`047e2db`, `d815a67`): install-phase failure reproduced locally as `CI=true pnpm install --frozen-lockfile` → `ERR_PNPM_OUTDATED_LOCKFILE` (3 deps missing from stale `pnpm-lock.yaml`); exact Vercel log lines not retrievable via CLI/API (ERROR deployments expose no logs; dashboard: https://vercel.com/citrix-lab-phs-projects/dev-hopkinstalentgroup/Fp4zr8goXFfULNf2sQVoQrJDbP16).
- Current head `a76383b` after pnpm-lock removal: **READY** (npm path).
- Package manager of record: **npm** (`package-lock.json`).

## Constraints honored

No push, merge, deployment trigger, or PR review; no live SMTP test; no VPS/WHM/cPanel/DNS/firewall changes; no secrets in this report or in the diff; no dependencies or lockfile changed by `74d136f`.
