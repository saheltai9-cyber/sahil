# CivicFix

PC par chalane ke liye [START_HERE_HINGLISH.md](START_HERE_HINGLISH.md) padho. Is export mein original application code ke saath local development server diya gaya hai.

Production application for civic reporting with distinct citizen and administrator workspaces.

## Architecture

- Worker-compatible ESM API and frontend assets.
- Firebase Authentication: email/password, Google sign-in, email verification and password reset. The browser SDK obtains tokens; the Worker validates RS256 signatures against Google’s signing keys, issuer, audience, expiry and auth time. Passwords are handled by Firebase and are never sent to the CivicLens API.
- D1: profiles, reports, activity, observations, bookmarks, support, upload metadata and request limits.
- R2: uploaded report photos. Images are resized and re-encoded in the browser; server validates size and magic bytes.
- The approved navy/cyan UI is retained. The admin workspace uses a distinct violet operations theme.

## Development

`npm ci`, `npm run db:generate` after schema changes, `npm run build`, `npm test`.

Migrations live under `drizzle/` and are applied by Sites during publication. Never edit applied migrations. There is no sample-data seeding. Existing static demo data was session-only and has not been imported.

Runtime values:
- `APP_ORIGIN`: exact public origin used to enforce same-origin writes.
- `ADMIN_EMAILS`: a server-only comma-separated allowlist matched against the verified Firebase email claim. Unset means no administrators. The site owner is configured through Sites runtime settings.

Firebase tokens travel over HTTPS in `x-civiclens-token`; raw client user IDs and roles are never trusted. Sites identity headers alone cannot authenticate API requests. During migration only, the dispatcher-provided identity can link an existing profile when its email matches a verified Firebase identity. Do not trust those migration headers on any host other than the Sites dispatcher. Existing records remain in place; linking preserves their existing owner IDs. Accounts without a matching existing dispatcher session receive a new Firebase profile.

## Authorization

Anonymous visitors see only verified, in-progress or resolved reports. A signed-in citizen additionally sees their own pending/rejected reports. Administrators see all reports. The server independently checks every write; ownership and role supplied by a client are ignored. Verified report editing is administrator-only. Citizen edits to rejected submissions re-enter verification. Rejected and resolved updates require an explanation. Optimistic version checks prevent lost updates, and activity records are written in the same D1 batch.

Login and sign-up offer email/password and Google. Email/password users must verify their email before making changes; only verified allowlisted emails receive administrator privileges. Email verification and password reset use Firebase-hosted email actions. Google login uses a popup; the Firebase console must authorize `civiclens-ai.sahiltai005.chatgpt.site`. The public web config is intentionally bundled; no service-account private key is required. Firebase SDKs refresh ID tokens automatically; issued ID tokens remain usable until expiry (up to about one hour), so immediate revocation requires an additional revocation check if needed later.

Government routing and automatic image recognition are not connected; public-facing copy is explicit about both.

## Checks and operational scope

Backend tests run the deployed Worker logic against a real local SQLite schema and an in-memory R2 adapter. Firebase tokens in tests are signed with test-only RSA keys; test network mocks supply their public JWK. Production code has no test identity override. UI tests use a simulated DOM; they do not substitute for browser visual QA or a real hosted identity round trip.

The UI loads the latest 1,000 accessible reports and explicitly signals truncation. Lists, metrics and exports apply to that loaded set. Reports refresh every 90 seconds while the workspace is visible and no modal is open. Report creation is limited to 10 attempts per account per day; photo uploads to 20/day; observations to 30/hour. Unattached photos may remain after abandoned submissions; add a scheduled retention policy before high-volume rollout. External map/search availability depends on OpenStreetMap and Open-Meteo/GeoNames.


## Separate workspaces and account administration
- `/app` is a light citizen portal with personal complaint journeys, notifications, saved reports and a location picker inside the report form. `/admin` is the dark navy operations console with queues, departments, analytics, bulk report actions, user directory and access audit. Public community browsing remains available at `/`.
- Firebase ID tokens are verified on every API request. Owner identities come only from server `ADMIN_EMAILS`; other administrators are granted in D1 `profile_access`. Only owners can grant/revoke roles and suspend/restore accounts. Protected owner accounts cannot be changed in the directory. Every access change requires a reason, optimistic version check and audit event.
- Suspension blocks authenticated CivicLens API access (except session status), including existing sessions. It does not delete or disable the external Firebase account. Admin pages use client routing guards; every admin API independently checks the verified server role. No private data is embedded in the HTML shell.
- Notifications derive from administrator activity on a user's own reports, and read receipts persist in D1. No email/push delivery is implied.
- Profile directory covers accounts that have used CivicLens. Existing profiles without a linked verified login remain visible read-only until that user signs in. Account email and access records are only returned by administrator APIs.
- Migration 0002 adds access records, audit history and notification reads without modifying or removing existing reports/photos. Backend authorization is implemented for the existing D1/R2 architecture; the app does not use client-side Firestore or Firebase Storage for reports.
- Automated checks cover role grants/revocation, owner protection, suspension, cross-user notification privacy, read persistence, separate dashboard navigation and a report-location picker that preserves form drafts. Browser visual QA remains unavailable in this environment.


## Admin Command Center (v13)
Admin-only additions: six database-backed KPI cards, Chart.js monthly/category/department/lifecycle/weekly charts with accessible data tables, collapsible navigation, dated PDF (jsPDF) and CSV exports, operations feed, user-profile complaint activity, editable departments and coordinate density visualization.

The AI Insights page deliberately reports rules-based evidence only: keyword suggestions, same-category text similarity among up to 200 recent complaints, and coordinate-cell concentrations. No external AI service, prediction or confidence value is fabricated.

Aggregate KPI counts use all stored rows. Charts, maps, exports and insights use the latest 1,000 accessible reports; the app displays truncation and scope. Weekly updates count each complaint's latest update, not every historical event. CSV supports Unicode; PDF uses a standard Latin font.

Core department names remain fixed to preserve existing assignments; descriptions are editable. Custom department names and descriptions are editable; renames update assignments transactionally and increment report versions so stale edits fail safely. The new schema-only migration adds departments; no existing records are deleted or migrated to a different database.

Citizen page DOM is regression-tested against version 12 for overview, personal reports, map, notifications, saved reports and account. Admin CSS is scoped under `.admin-mode`. No Firestore collections or Firebase rules files exist in this project; privileged checks remain in the trusted D1/R2 Worker backend, with existing Firebase token verification unchanged.
