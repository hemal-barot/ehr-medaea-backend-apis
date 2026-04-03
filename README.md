# Medaea EHR — Backend API Documentation
## `ehr-medaea-backend-apis`

> **Live API Docs**: http://localhost:8000/api/docs  
> **ReDoc**: http://localhost:8000/api/redoc  
> **OpenAPI JSON**: http://localhost:8000/api/openapi.json  
> **Health Check**: http://localhost:8000/api/health

---

## Overview

This repository contains the complete API documentation, Postman collections, testing guides, and flow diagrams for the **Medaea EHR** FastAPI backend.

The backend is built on **FastAPI** (Python 3.11), backed by **PostgreSQL** via SQLAlchemy, and follows HIPAA/ONC-compliant patterns throughout.

---

## Repository Structure

```
ehr-medaea-backend-apis/
├── postman/
│   ├── Medaea_EHR_API.postman_collection.json   # Full collection — all endpoints
│   └── Medaea_EHR.postman_environment.json      # Dev environment variables
│
├── docs/
│   ├── getting-started.md        # Server setup, prerequisites
│   ├── authentication-guide.md   # Full auth flow with examples
│   ├── testing-guide.md          # How QA testers should run the collection
│   ├── api-reference/
│   │   ├── 00-health.md          # System / health endpoints
│   │   ├── 01-authentication.md  # Signup, Login, MFA, Password Reset
│   │   ├── 02-users.md           # User profile management
│   │   ├── 03-patients.md        # Patient CRUD
│   │   ├── 04-appointments.md    # Appointment scheduling
│   │   ├── 05-calendar.md        # Calendar views, rooms, PTO, on-call
│   │   ├── 06-encounters.md      # SOAP note encounters
│   │   ├── 07-charting.md        # Allergies, Medications, Problems, Immunizations
│   │   ├── 08-organizations.md   # Organization membership
│   │   └── 09-audit.md           # Audit log (HIPAA)
│   └── flows/
│       ├── auth-flow.md          # Full auth sequence diagram
│       ├── patient-encounter-flow.md  # Patient → Appointment → Encounter flow
│       └── scheduling-flow.md    # Scheduling and calendar flow
│
└── CHANGELOG.md                  # What changed in each version
```

---

## Quick Start

### 1. Run the Backend

```bash
# Install dependencies
pip install -r backend/requirements.txt

# Start the FastAPI server
PYTHONPATH=/home/runner/workspace uvicorn backend.fastapi_app.main:app --host 0.0.0.0 --port 8000 --reload
```

### 2. Test with Postman

1. Open Postman → **Import** → select `postman/Medaea_EHR_API.postman_collection.json`
2. Import `postman/Medaea_EHR.postman_environment.json` as the environment
3. Select **Medaea EHR — Local Dev** as the active environment
4. Run **System → Health Check** (no auth needed) — confirm `{"status": "ok"}`
5. Run **Authentication → Signup** or **Login** — token auto-saves
6. All other requests will use the saved token automatically

---

## API Summary

| Domain | Prefix | Endpoints | Auth Required |
|---|---|---|---|
| System | `/api` | 1 | No |
| Authentication | `/api/v1/auth` | 8 | Mixed |
| Users | `/api/v1/users` | 3 | Yes |
| Patients | `/api/v1/patients` | 6 | Yes |
| Appointments | `/api/v1/appointments` | 7 | Yes |
| Calendar | `/api/v1/calendar` | 10 | Yes |
| Encounters | `/api/v1/encounters` | 3 | Yes |
| Charting | `/api/v1/patients/{id}/...` | 8 | Yes |
| Organizations | `/api/v1/organizations` | 1 | Yes |
| Audit | `/api/v1/audit` | 1 | Yes |
| **Total** | | **48** | |

---

## Authentication Flow

```
POST /api/v1/auth/signup
    │
    ▼ (if REQUIRE_EMAIL_VERIFICATION=true)
POST /api/v1/auth/verify-email?token=...
    │
    ▼
POST /api/v1/auth/login  (form: username, password)
    │
    ├── MFA disabled ──────────────────────▶ { access_token: "eyJ..." }
    │
    └── MFA enabled ──▶ { mfa_required: true }
                              │
                              ▼
                   POST /api/v1/auth/mfa/verify
                              │
                              ▼
                        { access_token: "eyJ..." }
```

All subsequent requests must include:
```
Authorization: Bearer eyJ...
```

---

## Base URL

| Environment | URL |
|---|---|
| Local Dev | `http://localhost:8000` |
| Docker | `http://localhost:8000` |
| Staging | `https://api-staging.medaea.ai` |
| Production | `https://api.medaea.ai` |

---

## Tech Stack

| Component | Technology |
|---|---|
| Framework | FastAPI 0.110+ |
| Language | Python 3.11 |
| Database | PostgreSQL 15 (SQLAlchemy ORM) |
| Auth | JWT (python-jose) + bcrypt |
| MFA | TOTP (pyotp) + SMS (Twilio) |
| Email | Gmail SMTP (aiosmtplib) |
| Migrations | SQLAlchemy `create_all` (Alembic planned) |
| API Docs | Swagger UI + ReDoc (auto-generated) |

---

## HIPAA / ONC Compliance Notes

- All PHI access is logged in `audit_logs` table
- Passwords are hashed with bcrypt (cost factor 12)
- JWT tokens expire (configurable, default 8 hours)
- MFA support: TOTP authenticator app + SMS
- HIPAA consent timestamp stored per user
- Email verification enforced in production
- Password reset tokens expire in 1 hour

---

## Changelog

See [CHANGELOG.md](./CHANGELOG.md) for version history.

---

## Related Repositories

| Repo | Description |
|---|---|
| [ehr-medaea-backend](https://github.com/hemal-barot/ehr-medaea-backend) | FastAPI + Django source code |
| [ehr-medaea-frontend](https://github.com/hemal-barot/ehr-medaea-frontend) | React + Vite + TypeScript EHR app |
| [ehr-medaea-website](https://github.com/hemal-barot/ehr-medaea-website) | Marketing website |
| [ehr-medaea-dbmodel](https://github.com/hemal-barot/ehr-medaea-dbmodel) | Database schema + ERD |
| [ehr-medaea-documentations](https://github.com/hemal-barot/ehr-medaea-documentations) | Compliance + progress docs |
