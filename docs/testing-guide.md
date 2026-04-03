# API Testing Guide — Medaea EHR
## For QA Engineers & Backend Testers

---

## Prerequisites

| Tool | Version | Download |
|---|---|---|
| Postman | Desktop v10+ | https://postman.com/downloads |
| Python | 3.11+ | For running server locally |
| PostgreSQL | 15+ | Or use Replit managed DB |

---

## Step 1 — Start the Backend

```bash
# From the project root
PYTHONPATH=/home/runner/workspace \
uvicorn backend.fastapi_app.main:app \
  --host 0.0.0.0 \
  --port 8000 \
  --reload
```

Confirm it's running:
```bash
curl http://localhost:8000/api/health
# Expected: {"status": "ok", "service": "medaea-fastapi"}
```

Interactive API docs are at: http://localhost:8000/api/docs

---

## Step 2 — Import into Postman

1. Open Postman
2. Click **Import** (top left)
3. Import both files:
   - `postman/Medaea_EHR_API.postman_collection.json`
   - `postman/Medaea_EHR.postman_environment.json`
4. In the top-right dropdown, select **Medaea EHR — Local Dev**

---

## Step 3 — Run the Full Test Flow

Run requests in this order — each one sets variables for the next:

### Flow A — New User + Patient + Encounter

```
1. Authentication > Signup
   → Creates account, auto-saves token if no email verification

2. Authentication > Login
   → Saves access_token to environment

3. Users > Get My Profile
   → Confirms token works, see user details

4. Patients > Create Patient
   → Saves patient_id to environment

5. Patients > Get Patient by ID
   → Verifies patient was created

6. Appointments > Create Appointment
   → Uses patient_id, saves appointment_id

7. Appointments > Update Appointment Status (status: "in_progress")
   → Simulates patient check-in

8. Encounters > Create Encounter
   → SOAP note, uses patient_id + appointment_id, saves encounter_id

9. Clinical Charting > Allergies — Add
   → Add allergy to patient chart

10. Clinical Charting > Medications — Add
    → Add medication to patient chart

11. Clinical Charting > Problems — Add
    → Add ICD-10 diagnosis

12. Appointments > Update Appointment Status (status: "completed")
    → Close the visit

13. Audit > Get Audit Logs
    → Verify all actions were logged (HIPAA)
```

---

## Step 4 — Run Automated Tests

The collection includes **Postman Test scripts** on key requests that:
- Assert HTTP status codes
- Auto-save response IDs to environment variables
- Chain requests without manual copy-paste

### Using Collection Runner

1. Click the **Medaea EHR API** collection name
2. Click **Run** button (top right of collection)
3. Select all requests in order
4. Set **Delay** to 200ms between requests
5. Click **Run Medaea EHR API**
6. Review the test results panel

---

## Test Cases by Domain

### Authentication Tests

| Test Case | Method | Endpoint | Expected |
|---|---|---|---|
| Register new user | POST | /auth/signup | 201 |
| Duplicate email | POST | /auth/signup | 409 |
| Valid login | POST | /auth/login | 200 + token |
| Wrong password | POST | /auth/login | 401 |
| Invalid token | GET | /users/me | 401 |
| Expired token | GET | /users/me | 401 |
| Short password | POST | /auth/signup | 422 |
| Missing required fields | POST | /auth/signup | 422 |

### Patient Tests

| Test Case | Method | Endpoint | Expected |
|---|---|---|---|
| Create patient | POST | /patients | 201 |
| List patients | GET | /patients | 200 + array |
| Search by name | GET | /patients?search=John | 200 + filtered |
| Filter by status | GET | /patients?status=active | 200 + filtered |
| Invalid patient ID | GET | /patients/bad-uuid | 404 |
| No auth | GET | /patients | 401 |

### Appointment Tests

| Test Case | Method | Endpoint | Expected |
|---|---|---|---|
| Create appointment | POST | /appointments | 201 |
| List by date | GET | /appointments?date=2026-04-03 | 200 |
| Invalid status value | PATCH | /appointments/{id}/status | 422 |
| Get nonexistent | GET | /appointments/bad-id | 404 |

### Encounter Tests

| Test Case | Method | Endpoint | Expected |
|---|---|---|---|
| Create SOAP note | POST | /encounters | 201 |
| Get patient encounters | GET | /encounters/patient/{id} | 200 |
| Patch encounter | PATCH | /encounters/{id} | 200 |

### Clinical Charting Tests

| Test Case | Endpoint | Expected |
|---|---|---|
| Add allergy | POST /patients/{id}/allergies | 201 |
| List allergies | GET /patients/{id}/allergies | 200 |
| Add medication with NDC | POST /patients/{id}/medications | 201 |
| Add ICD-10 problem | POST /patients/{id}/problems | 201 |
| Record immunization with CVX | POST /patients/{id}/immunizations | 201 |

---

## Environment Variables Reference

| Variable | Set By | Description |
|---|---|---|
| `base_url` | Manual | Server base URL |
| `access_token` | Login / Signup / MFA Verify | JWT Bearer token |
| `patient_id` | Create Patient / List Patients | Last patient UUID |
| `appointment_id` | Create Appointment | Last appointment UUID |
| `encounter_id` | Create Encounter | Last encounter UUID |
| `email_verification_token` | Manual | From verification email |
| `password_reset_token` | Manual | From reset email |

---

## Common Errors

| Status | Message | Fix |
|---|---|---|
| 401 | Not authenticated | Add `Authorization: Bearer {{access_token}}` |
| 401 | Token expired | Run Login again |
| 403 | Email not verified | Run Verify Email first |
| 409 | Email already exists | Use different email in Signup |
| 422 | Validation error | Check request body fields |
| 404 | Not found | Check UUID is correct |
| 500 | Internal error | Check server logs |

---

## MFA Testing

### Testing TOTP (Authenticator App)

1. Run **MFA Setup — Initiate** with `method: "authenticator"`
2. Copy the `qr_url` from the response
3. Scan with Google Authenticator or Authy
4. Get the 6-digit code from the app
5. Run **MFA Setup — Finalize** with the code
6. Log out, log in again → you'll get `mfa_required: true`
7. Enter 6-digit code from app in **MFA Verify — During Login**

### Testing SMS MFA (requires Twilio)

1. Run **MFA Setup — Initiate** with `method: "sms"` and a phone number
2. Check your phone for the SMS code
3. Run **MFA Setup — Finalize**
4. On next login: check phone → run **MFA Verify**

---

## Swagger UI Testing

The auto-generated Swagger docs at http://localhost:8000/api/docs allow:
1. Click **Authorize** button → enter `Bearer <your_token>`
2. Expand any endpoint → **Try it out**
3. Fill in params → **Execute**

Great for quick exploration without Postman.
