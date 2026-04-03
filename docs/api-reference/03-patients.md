# API Reference — Patients
## Base path: `/api/v1/patients`
## Updated: April 2026

All patient endpoints require **JWT authentication** and return data **scoped to the authenticated user's organization**.

---

## GET /patients

List all patients for the authenticated user's organization.

**Query Parameters**:

| Param | Type | Description |
|---|---|---|
| `search` | string | Search by first name, last name, or email |
| `status` | string | Filter: `active` \| `inactive` \| `deceased` |
| `skip` | int | Pagination offset (default: 0) |
| `limit` | int | Results per page (default: 50, max: 200) |

**Response**: Array of PatientResponse objects.

```json
[
  {
    "id": "uuid",
    "first_name": "John",
    "last_name": "Doe",
    "email": "john.doe@email.com",
    "phone": "+15555550001",
    "date_of_birth": "1985-03-15",
    "gender": "Male",
    "address": "123 Main St",
    "city": "Phoenix",
    "state": "AZ",
    "zip": "85001",
    "race": "White",
    "ethnicity": "Not Hispanic or Latino",
    "preferred_language": "English",
    "marital_status": "Married",
    "mrn": "MRN-00001",
    "status": "active",
    "organization_id": "org-uuid",
    "primary_provider_id": null,
    "created_at": "2026-04-03T10:00:00Z"
  }
]
```

---

## POST /patients

Create a new patient record.

**Required fields**: `first_name`, `last_name`

```json
{
  "first_name": "John",
  "last_name": "Doe",
  "email": "john.doe@email.com",
  "phone": "+15555550001",
  "date_of_birth": "1985-03-15",
  "gender": "Male",
  "address": "123 Main St",
  "city": "Phoenix",
  "state": "AZ",
  "zip": "85001",
  "race": "White",
  "ethnicity": "Not Hispanic or Latino",
  "preferred_language": "English",
  "marital_status": "Married",
  "ssn_last4": "4321",
  "mrn": "MRN-00001",
  "status": "active",
  "organization_id": "org-uuid"
}
```

| Code | Description |
|---|---|
| 201 | Patient created |
| 401 | Unauthorized |
| 422 | Validation error |

---

## GET /patients/recent

Get recently created patients (ordered by `created_at` DESC).

**Query Parameters**:

| Param | Type | Default |
|---|---|---|
| `limit` | int | 10 |

---

## GET /patients/{patient_id}

Get a single patient by UUID.

| Code | Description |
|---|---|
| 200 | Patient data |
| 404 | Patient not found |

---

## PUT /patients/{patient_id}

Update patient demographics. All fields optional.

```json
{
  "phone": "+15555550002",
  "address": "456 New Ave",
  "city": "Scottsdale",
  "status": "active"
}
```

| Code | Description |
|---|---|
| 200 | Updated patient |
| 404 | Patient not found |

---

## DELETE /patients/{patient_id}

Delete a patient record permanently.

> ⚠️ HIPAA Note: Consider soft deletion (`status: "inactive"`) instead to preserve audit trails.

| Code | Description |
|---|---|
| 204 | Deleted (no content) |
| 404 | Not found |

---

## GET /patients/{id}/allergies

List all allergies for a patient.

**Response**:
```json
[
  {
    "id": "uuid",
    "patient_id": "patient-uuid",
    "allergen": "Penicillin",
    "reaction": "Anaphylaxis",
    "severity": "life-threatening",
    "status": "active"
  }
]
```

## POST /patients/{id}/allergies

Add an allergy to a patient's chart.

```json
{
  "allergen": "Penicillin",
  "reaction": "Anaphylaxis",
  "severity": "life-threatening",
  "status": "active"
}
```

---

## GET /patients/{id}/medications

List all medications for a patient.

## POST /patients/{id}/medications

Add a medication to a patient's chart.

```json
{
  "name": "Amlodipine",
  "ndc_code": "00071-0155-23",
  "rxnorm_code": "17767",
  "dosage": "5mg",
  "frequency": "Once daily",
  "route": "Oral",
  "status": "active",
  "start_date": "2026-04-03",
  "refills": 3,
  "instructions": "Take in the morning with or without food."
}
```

---

## GET /patients/{id}/problems

List all problems/diagnoses for a patient.

## POST /patients/{id}/problems

Add a problem to a patient's list.

```json
{
  "description": "Essential hypertension",
  "icd_code": "I10",
  "status": "active",
  "onset_date": "2024-01-15"
}
```

---

## GET /patients/{id}/immunizations

List all immunizations for a patient.

## POST /patients/{id}/immunizations

Record an immunization.

```json
{
  "vaccine_name": "Influenza (Flu Shot)",
  "cvx_code": "141",
  "date_administered": "2026-04-03",
  "dose_number": 1,
  "lot_number": "LOT2026A",
  "site": "Left deltoid",
  "route": "IM"
}
```

---

## Field Reference

### Gender Values
`Male` | `Female` | `Other` | `Prefer not to say` | `Unknown`

### Race Values (ONC USCDI v3)
- American Indian or Alaska Native
- Asian
- Black or African American
- Native Hawaiian or Other Pacific Islander
- White
- Other Race

### Ethnicity Values (ONC USCDI v3)
- Hispanic or Latino
- Not Hispanic or Latino

### Status Values
`active` | `inactive` | `deceased`

### Allergy Severity
`mild` | `moderate` | `severe` | `life-threatening`

### Medication Routes
`Oral` | `IV` | `IM` | `Topical` | `Inhaled` | `Sublingual` | `SQ` | `IN` | `ID`
