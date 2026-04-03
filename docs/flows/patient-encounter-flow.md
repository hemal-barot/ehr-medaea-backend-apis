# Patient → Appointment → Encounter Flow
## Medaea EHR — Clinical Workflow

---

## Overview

This document describes the complete clinical workflow from patient registration through encounter documentation, the core value flow of the EHR system.

---

## Full Sequence Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                  COMPLETE PATIENT VISIT WORKFLOW                    │
└─────────────────────────────────────────────────────────────────────┘

Staff/Provider             Frontend              FastAPI             Database
      │                        │                    │                    │
      │                        │                    │                    │
  ╔══════════════════════════════════════════════════════════════════╗
  ║  STEP 1 — PATIENT REGISTRATION                                   ║
  ╚══════════════════════════════════════════════════════════════════╝
      │  Search existing patient  │                    │                    │
      │──────────────────────────▶│  GET /patients     │                    │
      │                           │  ?search=John Doe  │                    │
      │                           │───────────────────▶│                    │
      │                           │                    │  Query by name     │
      │                           │◀───────────────────│  [] (not found)    │
      │  New patient — fill form  │                    │                    │
      │──────────────────────────▶│                    │                    │
      │                           │  POST /patients    │                    │
      │                           │  {first_name, dob, │                    │
      │                           │   gender, address, │                    │
      │                           │   insurance, mrn}  │                    │
      │                           │───────────────────▶│                    │
      │                           │                    │  INSERT patients   │
      │                           │                    │  (org scoped)      │
      │                           │                    │───────────────────▶│
      │                           │◀───────────────────│  201 {patient_id}  │
      │  Patient created          │                    │                    │
      │◀──────────────────────────│                    │                    │
      │                           │                    │                    │

  ╔══════════════════════════════════════════════════════════════════╗
  ║  STEP 2 — SCHEDULE APPOINTMENT                                   ║
  ╚══════════════════════════════════════════════════════════════════╝
      │  Book appointment         │                    │                    │
      │──────────────────────────▶│                    │                    │
      │                           │  POST /appointments│                    │
      │                           │  {patient_id,      │                    │
      │                           │   start_time,      │                    │
      │                           │   visit_type,      │                    │
      │                           │   room, reason}    │                    │
      │                           │───────────────────▶│                    │
      │                           │                    │  INSERT appointments│
      │                           │                    │  provider_id = me  │
      │                           │                    │───────────────────▶│
      │                           │◀───────────────────│  201 {appt_id}     │
      │  Appointment confirmed    │                    │                    │
      │◀──────────────────────────│                    │                    │
      │                           │                    │                    │

  ╔══════════════════════════════════════════════════════════════════╗
  ║  STEP 3 — APPOINTMENT DAY — CHECK-IN                            ║
  ╚══════════════════════════════════════════════════════════════════╝
      │  Patient arrives          │                    │                    │
      │──────────────────────────▶│                    │                    │
      │                           │  PATCH /appointments/                   │
      │                           │  {id}/status       │                    │
      │                           │  {status:"in_progress"}                 │
      │                           │───────────────────▶│                    │
      │                           │                    │  UPDATE status     │
      │                           │◀───────────────────│  200               │
      │  Patient shown as checked-in                   │                    │
      │◀──────────────────────────│                    │                    │
      │                           │                    │                    │

  ╔══════════════════════════════════════════════════════════════════╗
  ║  STEP 4 — REVIEW CHART (Pre-visit)                               ║
  ╚══════════════════════════════════════════════════════════════════╝
      │  Open patient chart       │                    │                    │
      │──────────────────────────▶│                    │                    │
      │                           │  GET /patients/{id}│                    │
      │                           │  GET /patients/{id}/allergies            │
      │                           │  GET /patients/{id}/medications          │
      │                           │  GET /patients/{id}/problems             │
      │                           │  GET /patients/{id}/immunizations        │
      │                           │  GET /encounters/patient/{id}            │
      │                           │───────────────────▶│  (parallel calls)  │
      │                           │◀───────────────────│  All chart data    │
      │  Full chart displayed     │                    │                    │
      │◀──────────────────────────│                    │                    │
      │                           │                    │                    │

  ╔══════════════════════════════════════════════════════════════════╗
  ║  STEP 5 — CREATE ENCOUNTER (SOAP Note)                           ║
  ╚══════════════════════════════════════════════════════════════════╝
      │  Start encounter note     │                    │                    │
      │──────────────────────────▶│                    │                    │
      │                           │  POST /encounters  │                    │
      │                           │  {patient_id,      │                    │
      │                           │   appointment_id,  │                    │
      │                           │   chief_complaint, │                    │
      │                           │   subjective,      │                    │
      │                           │   objective,       │                    │
      │                           │   assessment,      │                    │
      │                           │   plan}            │                    │
      │                           │───────────────────▶│                    │
      │                           │                    │  INSERT encounters │
      │                           │                    │  provider_id = me  │
      │                           │                    │  status = "open"   │
      │                           │                    │───────────────────▶│
      │                           │◀───────────────────│  201 {encounter_id}│
      │  Encounter draft saved    │                    │                    │
      │◀──────────────────────────│                    │                    │
      │                           │                    │                    │

  ╔══════════════════════════════════════════════════════════════════╗
  ║  STEP 6 — UPDATE CHART (During/After Visit)                     ║
  ╚══════════════════════════════════════════════════════════════════╝
      │  Add new allergy found    │                    │                    │
      │──────────────────────────▶│  POST /patients/   │                    │
      │                           │  {id}/allergies    │                    │
      │                           │───────────────────▶│                    │
      │                           │◀───────────────────│  201               │
      │                           │                    │                    │
      │  Add new medication Rx    │                    │                    │
      │──────────────────────────▶│  POST /patients/   │                    │
      │                           │  {id}/medications  │                    │
      │                           │───────────────────▶│                    │
      │                           │◀───────────────────│  201               │
      │                           │                    │                    │
      │  Add diagnosis to problem │                    │                    │
      │  list                     │  POST /patients/   │                    │
      │──────────────────────────▶│  {id}/problems     │                    │
      │                           │───────────────────▶│                    │
      │                           │◀───────────────────│  201               │
      │                           │                    │                    │

  ╔══════════════════════════════════════════════════════════════════╗
  ║  STEP 7 — SIGN ENCOUNTER NOTE                                    ║
  ╚══════════════════════════════════════════════════════════════════╝
      │  Sign and finalize note   │                    │                    │
      │──────────────────────────▶│  PATCH /encounters/│                    │
      │                           │  {encounter_id}    │                    │
      │                           │  {status:"signed", │                    │
      │                           │   plan: "updated..."│                   │
      │                           │───────────────────▶│                    │
      │                           │                    │  UPDATE encounters │
      │                           │                    │  status="signed"   │
      │                           │◀───────────────────│  200               │
      │  Visit closed             │                    │                    │
      │◀──────────────────────────│                    │                    │
      │                           │                    │                    │

  ╔══════════════════════════════════════════════════════════════════╗
  ║  STEP 8 — COMPLETE APPOINTMENT                                   ║
  ╚══════════════════════════════════════════════════════════════════╝
      │  Mark visit complete      │                    │                    │
      │──────────────────────────▶│  PATCH /appointments/                   │
      │                           │  {id}/status       │                    │
      │                           │  {status:"completed"}                   │
      │                           │───────────────────▶│                    │
      │                           │◀───────────────────│  200               │
      │  Appointment closed       │                    │                    │
      │◀──────────────────────────│                    │                    │
```

---

## Data Model Relationships

```
Organization
    │
    ├── Users (providers)
    │       │
    │       └── Appointments ──────┐
    │                              │
    └── Patients                   │
            │                      │
            ├── Appointments ◀─────┘
            │       │
            │       └── Encounters (SOAP)
            │
            ├── Allergies
            ├── Medications
            ├── Problems (ICD-10)
            ├── Immunizations (CVX)
            └── Documents
```

---

## API Calls Per Step (Performance Reference)

| Step | API Calls | Parallel? |
|---|---|---|
| Search patient | 1 | — |
| Create patient | 1 | — |
| Schedule appointment | 1 | — |
| Check-in | 1 | — |
| Load full chart | 6 | Yes — all parallel |
| Create encounter | 1 | — |
| Add allergy/meds/problems | 2-4 | Yes |
| Sign encounter | 1 | — |
| Complete appointment | 1 | — |
| **Total** | **15-18** | |

---

## Appointment Status State Machine

```
    ┌───────────┐
    │ scheduled │ ←── Default on creation
    └─────┬─────┘
          │  Patient calls to confirm
          ▼
    ┌───────────┐
    │ confirmed │
    └─────┬─────┘
          │  Patient arrives / check-in
          ▼
    ┌─────────────┐
    │ in_progress │ ←── Visit started
    └─────┬───────┘
          │  Visit finished
          ▼
    ┌───────────┐
    │ completed │ ←── Chart closed
    └───────────┘

    At any point:
    ┌───────────┐     ┌──────────┐
    │ cancelled │     │ no_show  │
    └───────────┘     └──────────┘
```

---

## Encounter Status State Machine

```
    ┌──────┐
    │ open │ ←── Default on creation (draft)
    └──┬───┘
       │  Provider reviews and signs
       ▼
    ┌────────┐
    │ signed │ ←── Finalized clinical note
    └──┬─────┘
       │  Correction needed (HIPAA compliant)
       ▼
    ┌─────────┐
    │ amended │ ←── Addendum added (original preserved)
    └─────────┘
```
