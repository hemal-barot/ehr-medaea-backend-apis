# API Reference — Encounters (SOAP Notes)
## Base path: `/api/v1/encounters`
## Updated: April 2026

Clinical encounters are the core documentation mechanism. Each encounter represents a patient visit with a SOAP note format.

---

## SOAP Format

| Component | Field | Description |
|---|---|---|
| **S** — Subjective | `chief_complaint`, `subjective` | Patient's complaint; history of present illness, ROS, history |
| **O** — Objective | `objective` | Vitals, physical exam findings, lab/imaging results |
| **A** — Assessment | `assessment` | Clinical impressions, differential diagnoses |
| **P** — Plan | `plan` | Treatment, orders, medications, referrals, follow-up |

---

## GET /encounters/patient/{patient_id}

Get all encounters for a patient, ordered by `encounter_date` DESC.

**Auth**: JWT required

**Response**:
```json
[
  {
    "id": "enc-uuid",
    "patient_id": "patient-uuid",
    "provider_id": "provider-uuid",
    "appointment_id": "appt-uuid",
    "encounter_type": "Office Visit",
    "chief_complaint": "Chest pain for 2 days",
    "subjective": "Patient reports sharp chest pain 7/10...",
    "objective": "BP: 145/92, HR: 88, RR: 18, SpO2: 97%...",
    "assessment": "Hypertensive urgency. R/O acute coronary syndrome.",
    "plan": "Order EKG, troponin x3. Start amlodipine 5mg QD.",
    "status": "signed",
    "encounter_date": "2026-04-03T14:30:00Z",
    "created_at": "2026-04-03T14:00:00Z"
  }
]
```

---

## POST /encounters

Create a new encounter (SOAP note).

**Auth**: JWT required

```json
{
  "patient_id": "patient-uuid",
  "appointment_id": "appt-uuid",
  "encounter_type": "Office Visit",
  "chief_complaint": "Patient presents with chest pain and shortness of breath for 2 days.",
  "subjective": "Patient reports sharp chest pain 7/10, worse with exertion. Denies fever, chills. No prior cardiac history. Current medications: Metoprolol 25mg QD.",
  "objective": "BP: 145/92, HR: 88, RR: 18, Temp: 98.6°F, SpO2: 97% on room air. Lungs: clear to auscultation bilaterally. Heart: RRR, no murmurs, rubs, or gallops. Abdomen: soft, non-tender.",
  "assessment": "1. Hypertensive urgency (BP 145/92). 2. Chest pain, unspecified etiology — R/O acute coronary syndrome (ACS).",
  "plan": "1. Order EKG — priority stat. 2. Order troponin I x3 q6h. 3. Order CBC, CMP, D-dimer. 4. Add amlodipine 5mg QD to current medications. 5. Patient to return in 48h or go to ED if symptoms worsen. 6. Cardiology referral placed."
}
```

| Code | Description |
|---|---|
| 201 | Encounter created (`status: "open"`) |
| 401 | Unauthorized |
| 422 | Validation error |

---

## PATCH /encounters/{encounter_id}

Update or sign an encounter note.

**Auth**: JWT required

```json
{
  "plan": "EKG: NSR, no ischemic changes. Troponin x3 negative. Discharged with amlodipine 5mg. Follow-up in 1 week or sooner if symptoms recur. Cardiology referral placed — appointment scheduled.",
  "status": "signed"
}
```

| Code | Description |
|---|---|
| 200 | Updated encounter |
| 404 | Not found |

---

## Status State Machine

```
open → signed → amended
```

| Status | Meaning |
|---|---|
| `open` | Draft — can be edited freely |
| `signed` | Finalized — provider has attested to the note |
| `amended` | An addendum was made after signing (HIPAA-compliant correction) |

---

## Encounter Types

| Type | Use Case |
|---|---|
| `Office Visit` | Standard outpatient appointment |
| `Telehealth` | Video or phone visit |
| `Urgent Care` | Acute, unscheduled visit |
| `Annual Wellness` | Preventive wellness exam (AWV) |
| `Procedure` | In-office procedure |
| `Consult` | Specialist consultation |
| `Emergency` | ED or urgent presentation |

---

## HIPAA Notes

- All encounter access is logged in `audit_logs` with `phi_accessed=true`
- Signed encounters should not be deleted — set `status: "amended"` with addendum
- Provider who signed is identified by `provider_id` (their user UUID)
- `encounter_date` represents the actual date of service (not creation date)
