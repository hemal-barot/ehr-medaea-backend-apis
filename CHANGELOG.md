# Changelog — Medaea EHR API

All notable API changes are documented here. Format follows [Keep a Changelog](https://keepachangelog.com/).

---

## [1.0.0] — 2026-04-03 — Phase 1 Complete

### Added — Authentication (`/api/v1/auth`)
- `POST /auth/signup` — Provider registration with email verification support
- `POST /auth/login` — OAuth2 form login returning JWT or MFA challenge
- `POST /auth/verify-email` — Email verification via token
- `POST /auth/resend-verification` — Resend verification email
- `POST /auth/forgot-password` — Initiate password reset (always 200, HIPAA)
- `POST /auth/reset-password` — Set new password with reset token
- `POST /auth/mfa/setup` — Initiate TOTP or SMS MFA enrollment
- `POST /auth/mfa/finalize` — Confirm TOTP enrollment with first code
- `POST /auth/mfa/verify` — Verify MFA code during login, returns JWT

### Added — Users (`/api/v1/users`)
- `GET /users/me` — Get authenticated user profile with org memberships
- `PUT /users/me` — Update profile (name, specialty, NPI, etc.)
- `PATCH /users/me` — Partial profile update
- `POST /users/me/password` — Change password (requires current password)

### Added — Patients (`/api/v1/patients`)
- `GET /patients` — List patients with search, status filter, pagination
- `POST /patients` — Create new patient record
- `GET /patients/recent` — Last N patients by creation date
- `GET /patients/{id}` — Get patient by UUID
- `PUT /patients/{id}` — Update patient demographics
- `DELETE /patients/{id}` — Delete patient (hard delete)

### Added — Appointments (`/api/v1/appointments`)
- `GET /appointments` — List appointments by status/date
- `GET /appointments/me` — My last 50 appointments
- `POST /appointments` — Create appointment
- `GET /appointments/{id}` — Get appointment by UUID
- `PATCH /appointments/{id}/status` — Update appointment status only
- `PUT /appointments/{id}` — Full appointment update
- `DELETE /appointments/{id}` — Cancel/delete appointment

### Added — Calendar (`/api/v1/calendar`)
- `GET /calendar/events` — Calendar events for date range (FullCalendar format)
- `GET /calendar/providers` — Active providers with color assignments
- `GET /calendar/rooms` — Room status + today's bookings
- `GET /calendar/pto` — PTO/leave requests
- `POST /calendar/pto` — Submit PTO request
- `PATCH /calendar/pto/{id}` — Update PTO status (approve/deny)
- `GET /calendar/availability-rules` — Scheduling rules
- `POST /calendar/availability-rules` — Create scheduling rule
- `GET /calendar/schedule-templates` — Reusable weekly schedule templates
- `POST /calendar/schedule-templates` — Create schedule template
- `GET /calendar/staff-schedules` — Weekly shift schedules
- `GET /calendar/on-call` — On-call assignments

### Added — Encounters (`/api/v1/encounters`)
- `GET /encounters/patient/{patient_id}` — All encounters for a patient
- `POST /encounters` — Create SOAP note encounter
- `PATCH /encounters/{id}` — Update encounter / sign note

### Added — Clinical Charting (`/api/v1/patients/{id}/...`)
- `GET/POST /patients/{id}/allergies` — Allergy list management
- `GET/POST /patients/{id}/medications` — Medication list management
- `GET/POST /patients/{id}/problems` — Problem/diagnosis list management
- `GET/POST /patients/{id}/immunizations` — Immunization record management

### Added — Organizations (`/api/v1/organizations`)
- `GET /organizations/my-organizations` — User's organization memberships

### Added — Audit (`/api/v1/audit`)
- `GET /audit/logs` — Paginated audit log (HIPAA required)

### Added — System
- `GET /api/health` — Health check endpoint
- `GET /api/docs` — Swagger UI
- `GET /api/redoc` — ReDoc
- `GET /api/openapi.json` — OpenAPI 3.0 spec

---

## [Planned] — v1.1.0

### Planned — Billing & Claims
- `POST /billing/claims` — Create CMS-1500 / UB-04 claim
- `GET /billing/claims` — List claims with status
- `GET /billing/eligibility/{patient_id}` — Insurance eligibility check
- `POST /billing/remittance` — Post ERA/EOB payment

### Planned — Documents
- `POST /documents/upload` — Upload clinical document
- `GET /patients/{id}/documents` — Patient document list
- `GET /documents/{id}/download` — Signed URL for document download

### Planned — Interoperability (ONC)
- `GET /fhir/r4/Patient` — FHIR R4 Patient resource
- `GET /fhir/r4/Observation` — FHIR R4 Observation
- `GET /fhir/r4/MedicationRequest` — FHIR R4 MedicationRequest
- SMART on FHIR authorization flow
- CCD/C-CDA document export

### Planned — Notifications
- `GET /notifications` — In-app notifications
- `PATCH /notifications/{id}/read` — Mark as read
- WebSocket connection for real-time alerts

### Planned — Reporting
- `GET /reports/appointments` — Appointment volume by provider/date
- `GET /reports/patients` — New patient registration trends
- `GET /reports/quality-measures` — CMS Quality Measures (HEDIS)

### Planned — Admin (Django)
- User management
- Organization onboarding
- Feature flag control
- Audit log viewer
