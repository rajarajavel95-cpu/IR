# EMIMS Android App — Starter Codebase

Generated from `EMIMS_SRS_Specification.pdf` (v1.1). This is a **foundational skeleton**,
not a finished production app — see "What's not built yet" below before assuming feature
parity with the SRS.

## Stack
- Kotlin + Jetpack Compose (Material 3), MVVM
- Hilt for dependency injection
- Retrofit + kotlinx.serialization for the REST API (matches SRS 5.2 exactly)
- Room for the offline-first material cache (FR-02)
- DataStore for session/token persistence (JWT Bearer auth, 30-min session timeout)
- WorkManager for background sync of offline-created requisitions
- CameraX + ML Kit dependencies wired in Gradle for barcode/QR scanning (screen not yet built)

## What's implemented
- Login screen + ViewModel (Company ID / User ID / Password, show/hide password)
- Dashboard with clickable quick-stat cards (total materials, low/zero stock, pending/approved/rejected requests)
- Smart Search screen: debounced query, offline-first (Room cache fallback when the API call fails)
- Requisition list + approval workflow: Approve / Reject, **reject requires a reason** (enforced in `RequisitionRepository`, not just the UI)
- RBAC matrix (`util/RoleAccess.kt`) mirroring SRS 3.1 roles: System Admin, Warehouse Manager, Inventory Operator, Auditor
- REST client wired to every endpoint listed in SRS 5.2

## What's NOT built yet (next iterations)
- Barcode/QR scanning screen itself (CameraX + ML Kit are in Gradle, but no scan UI/logic yet)
- Excel import/export screens (API calls exist in `ApiService`, no UI)
- Material detail screen (multi-location breakdown)
- Reports & Analytics screens (PDF/Excel export)
- Notifications (push/email/SMS/WhatsApp) — not started
- User Management screens (System Admin only)
- Forgot Password / Forgot User ID flows (buttons present, no logic)
- Room migrations beyond `fallbackToDestructiveMigration()` — replace before shipping
- Unit/instrumentation tests

## Running this project
1. Open in Android Studio (Koala or newer).
2. It expects `minSdk 26`, `compileSdk 34`, Kotlin 1.9.24 — let Android Studio sync Gradle.
3. Point `BuildConfig.BASE_URL` (in `app/build.gradle.kts`) at your actual backend once the
   EMIMS backend (Node/FastAPI, per SRS 5.3) is deployed. It currently points at placeholder
   staging/prod URLs that don't exist yet.
4. This app has no login backend to talk to yet — you'll need the backend built (see SRS 5.3 folder
   structure) before Login will actually succeed end-to-end.

## Design decisions worth knowing about
- Search is **offline-first by design** (SRS FR-02): the ViewModel always tries the network first,
  but silently falls back to the local Room cache on any failure — warehouse Wi-Fi is unreliable,
  so the search screen must never spin forever or show a hard error just because the network dropped.
- Rejecting a requisition without a reason is blocked at the repository layer, not just disabled in
  the dialog — so no future call site can accidentally skip it.
- RBAC checks in `RoleAccess.kt` are UX-only (hide/show buttons). The real enforcement must happen
  server-side; don't treat this file as a security boundary.
