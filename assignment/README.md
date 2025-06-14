# Clinic Task-Ingestion Service – README

This repository contains a toy—but production-style—service that ingests **`PatientTask`** objects from an external system and consolidates them into **`PatientRequest`** objects for easier triage by medical staff.

---

## Background

* **External Source** Periodically (e.g., every two minutes) the external system pushes a **list of `PatientTask` updates** to our service.
* **Why Merge?** Doctors prefer seeing a single, consolidated **`PatientRequest`** per patient rather than dozens of individual tasks.

> 🗒 **Out of scope**: replying to tasks, editing `PatientRequest` objects, or syncing changes back to the external system.

---

## Terminology

| Term                 | Meaning                                                                                        |
| -------------------- | ---------------------------------------------------------------------------------------------- |
| **`PatientTask`**    | A single item from the external system. Examples: “Question for doctor”, “Medication request”. |
| **Status**           | `Open` (new/active) or `Closed` (handled in the external system).                              |
| **`PatientRequest`** | A server-side aggregate of one or more tasks **for the same patient** *and* **department**.    |
| **Department**       | One of `["Dermatology", "Radiology", "Primary"]` stored in the task’s `assigned_to` field.     |

---

## Current Behaviour

* Tasks are merged **only by patient ID**.
* When *any* task within a request is marked **`Closed`**, its *data* is removed (but the task record stays for history).
* Closed requests remain in the database for medical-history purposes.

---

## New Requirement

> **In addition to patient ID, merge by *department***.

### Details

1. **Grouping Key** `(patient_id, department)` instead of just `patient_id`.
2. **Department Reassignment** A task may later move between departments. Your code must:

   * Remove it from the old request (if still open).
   * Insert or merge it into the appropriate new request.
3. **Edge Case** If all open tasks in a department move elsewhere, the now-empty request should be closed or deleted—up to your design, but document the decision.

---

## What You Must Implement

Create or modify **`services/DepartmentPatientRequestService`** so that:

1. `ClinicManager.process_tasks_update()` calls into your service to perform the merge logic.
2. All tests in **`tests/test_clinic_manager_with_departments.py`** pass.
3. The code remains clean, idiomatic, and easy to reason about.

---

## Tests

| File                                            | Purpose                                                              |
| ----------------------------------------------- | -------------------------------------------------------------------- |
| `tests/test_clinic_manager.py`                  | Validates the *original* implementation (group by patient).          |
| `tests/test_clinic_manager_with_departments.py` | Validates the **new** behaviour (group by patient *and* department). |

Run with:

```bash
pytest -q
```

---

## Guidelines

* **Models & Storage**

  * You *may* alter existing TinyDB tables or add new ones.
  * `PatientTask` is considered external input; do **not** change its schema.
* **Readability First** Optimise for clarity; premature micro-optimisation is unnecessary.
* **Document Decisions** Any noteworthy trade-off or assumption goes in `CHANGES.md`.
* **Answer Inline Questions** Look for `# Question` comments in the code and add your answers.

---

## Technical Notes

* Database: TinyDB (JSON file at `tinydb/db.json`). The test suite pre-populates \~100 closed requests to mimic historical data.
* If TinyDB limitations block a clean solution, feel free to add comments noting which MongoDB / SQL feature you would normally rely on.
* Tests use **pytest**. No additional dependencies are required.

---

## Submission Checklist

1. **Code changes** in `services/DepartmentPatientRequestService` (and elsewhere as needed).
2. **All tests pass**—run `pytest`.
3. **`CHANGES.md`** summarising:

   * What you changed and **why**.
   * Any trade-offs or assumptions.
4. **Answered every `# Question`** comment in the code.
5. Provide a short note (in `README` or PR description) explaining how to run the app and the tests.

Good luck!
