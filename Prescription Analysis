# Prescription Analysis API - Technical Specification

## Overview

The Prescription Analysis API provides endpoints for managing prescription analysis data during the doctor consultation workflow. This includes managing diagnosis, diagnostic tests, medications, advisory notes, and follow-up appointments.

**Base Path**: `/api/v1/doctors/{doctor_id}/prescriptions/{prescription_id}/analysis`

**Controller**: `PrescriptionAnalysisController` (`src/web/controllers/prescription_analysis_controller.py`)

**Authorization**: All endpoints require `@doctor_authorize` decorator

**Architecture Pattern**: CQRS (Command Query Responsibility Segregation)

## Authentication & Authorization

### Doctor Context Enforcement

All endpoints enforce doctor context through the `ensure_doctor_context()` method:

- Retrieves `current_doctor_id` from the authenticated session via `get_current_doctor_id()`
- Validates that `current_doctor_id` matches the `doctor_id` path parameter
- Raises `ForbiddenException` if:
  - Doctor context is not present in the session
  - Authenticated doctor ID does not match the requested doctor ID

This ensures doctors can only access and modify their own prescription data.

## API Endpoints

### 1. Get Prescription Review

**Endpoint**: `GET /api/v1/doctors/{doctor_id}/prescriptions/{prescription_id}/analysis/review`

**Description**: Retrieves complete prescription review data, combining patient KYP (Know Your Patient) data with prescription analysis. This endpoint is used to display the review page before finalizing the prescription.

#### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `doctor_id` | UUID | Yes | UUID of the authenticated doctor |
| `prescription_id` | UUID | Yes | UUID of the prescription |

#### Response: `GetPrescriptionReviewResponse`

**Status Code**: `200 OK`

**Response Structure**:

```json
{
  "prescription_id": "uuid",
  "user_id": "uuid",
  "vitals": [
    {
      "vital_id": "uuid",
      "vital_name": "string",
      "unit": "string",
      "readings": [
        {
          "id": "uuid",
          "value": "float | null",
          "systolic": "float | null",
          "diastolic": "float | null",
          "recorded_at": "datetime",
          "recorded_by_id": "uuid",
          "recorded_by_name": "string"
        }
      ]
    }
  ],
  "essentials": {
    "current_medications": [
      {
        "id": "uuid",
        "medicine_id": "uuid",
        "medicine_name": "string",
        "having_since": "string | null",
        "is_active": "boolean"
      }
    ],
    "allergies": [
      {
        "id": "uuid",
        "allergy_id": "uuid",
        "allergy_name": "string"
      }
    ],
    "lifestyle_history": {
      "steps": "string | null",
      "activity": "string | null",
      "sleep": "string | null",
      "food": "string | null",
      "alcohol": "string | null",
      "smoking": "string | null"
    } | null
  } | null,
  "medical_history": {
    "known_concerns": [
      {
        "id": "uuid",
        "disease_id": "uuid",
        "disease_name": "string"
      }
    ],
    "family_history": [
      {
        "id": "uuid",
        "family_member": "string",
        "disease_id": "uuid",
        "disease_name": "string",
        "details": "string | null"
      }
    ]
  } | null,
  "chief_complaints": [
    {
      "id": "uuid",
      "diagnosis_id": "uuid",
      "name": "string",
      "since": {
        "value": "integer",
        "unit": "string"
      } | null,
      "severity": "string | null",
      "note": "string | null"
    }
  ],
  "chief_complaints_overall_note": "string | null",
  "prescription_analysis": {
    "diagnosis": [
      {
        "id": "uuid",
        "diagnosis_id": "uuid",
        "diagnosis_name": "string",
        "diagnosis_type": "string | null",
        "note": "string | null",
        "is_from_past": "boolean"
      }
    ],
    "diagnostic_tests": [
      {
        "id": "uuid",
        "test_id": "uuid",
        "test_name": "string",
        "test_type_id": "uuid | null",
        "test_type_name": "string | null",
        "note": "string | null"
      }
    ],
    "diagnostic_tests_note": "string | null",
    "medications": [
      {
        "id": "uuid",
        "medicine_id": "uuid",
        "medicine_name": "string",
        "medicine_type": "string",
        "strength": "float | null",
        "strength_unit": "string | null",
        "dose": {
          "value": "float",
          "unit": "string"
        } | null,
        "repeat_medicine": "string | null",
        "time_of_day": ["string"] | null,
        "start_date": "date | null",
        "end_date": "date | null",
        "to_be_taken": "string | null",
        "note": "string | null"
      }
    ],
    "advisory": {
      "note": "string"
    } | null,
    "follow_up": {
      "follow_up_date": "date",
      "follow_up_time": "time | null",
      "details": "string | null",
      "overall_note": "string"
    } | null
  } | null,
  "kyp_last_updated_at": "datetime | null",
  "analysis_updated_at": "datetime | null"
}
```

#### Query Handler

- **Handler**: `GetPrescriptionReviewQueryHandler`
- **Location**: `src/core/features/prescription_analysis/get_prescription_review/`

#### Error Responses

| Status Code | Exception | Description |
|-------------|-----------|-------------|
| 403 | `ForbiddenException` | Doctor context mismatch or not authorized |
| 404 | `NotFoundException` | Doctor or prescription not found |

---

### 2. Update Diagnosis

**Endpoint**: `PUT /api/v1/doctors/{doctor_id}/prescriptions/{prescription_id}/analysis/diagnosis`

**Description**: Updates the diagnosis section of a prescription. This is a **full replacement** operation - the entire diagnosis list is replaced with the provided data. Pass an empty array to clear all diagnosis entries.

#### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `doctor_id` | UUID | Yes | UUID of the authenticated doctor |
| `prescription_id` | UUID | Yes | UUID of the prescription |

#### Request Body: `UpdateDiagnosisRequest`

```json
{
  "diagnosis": [
    {
      "diagnosis_id": "uuid",
      "diagnosis_type": "string | null",
      "note": "string | null",
      "is_from_past": "boolean"
    }
  ]
}
```

**Field Specifications**:

| Field | Type | Required | Constraints | Description |
|-------|------|----------|-------------|-------------|
| `diagnosis` | array | Yes | - | List of diagnosis items (replaces existing) |
| `diagnosis[].diagnosis_id` | UUID | Yes | Valid UUID from master diagnosis table | Reference to diagnosis from master table |
| `diagnosis[].diagnosis_type` | string | No | Max 256 chars, trimmed | Specific diagnosis type (free text) |
| `diagnosis[].note` | string | No | Max 1000 chars, trimmed | Additional notes for this diagnosis |
| `diagnosis[].is_from_past` | boolean | No | Default: false | Whether this diagnosis is from past prescriptions |

**Validation Rules**:

- `diagnosis_type`: Text field validation, whitespace trimmed, max 256 characters
- `note`: Text field validation, whitespace trimmed, max 1000 characters
- Empty array is allowed (clears all diagnosis)

#### Response: `UpdateDiagnosisResponse`

**Status Code**: `200 OK`

```json
{
  "prescription_id": "uuid",
  "diagnosis": [
    {
      "id": "uuid",
      "diagnosis_id": "uuid",
      "diagnosis_name": "string",
      "diagnosis_type": "string | null",
      "note": "string | null",
      "is_from_past": "boolean"
    }
  ],
  "updated_at": "datetime"
}
```

**Response Fields**:

| Field | Type | Description |
|-------|------|-------------|
| `prescription_id` | UUID | ID of the updated prescription |
| `diagnosis` | array | Updated list of diagnosis items |
| `diagnosis[].id` | UUID | Generated ID for this diagnosis entry |
| `diagnosis[].diagnosis_id` | UUID | Reference to master diagnosis table |
| `diagnosis[].diagnosis_name` | string | Name from master diagnosis table (joined) |
| `diagnosis[].diagnosis_type` | string\|null | Specific diagnosis type if provided |
| `diagnosis[].note` | string\|null | Additional notes if provided |
| `diagnosis[].is_from_past` | boolean | Whether loaded from past prescriptions |
| `updated_at` | datetime | Timestamp of the update |

#### Command Handler

- **Command**: `UpdateDiagnosisCommand`
- **Handler**: `UpdateDiagnosisCommandHandler`
- **Location**: `src/core/features/prescription_analysis/update_diagnosis/`

#### Business Logic

1. Validates doctor context and authorization
2. Validates prescription exists and belongs to doctor
3. Ensures prescription is in `DRAFT` status
4. Validates all `diagnosis_id` references exist in master table
5. Deletes all existing diagnosis entries for the prescription
6. Creates new diagnosis entries from the request
7. Returns updated diagnosis list with joined master table data

#### Error Responses

| Status Code | Exception | Description |
|-------------|-----------|-------------|
| 400 | `ValidationException` | Invalid input data or prescription not in DRAFT status |
| 403 | `ForbiddenException` | Doctor context mismatch or not authorized |
| 404 | `NotFoundException` | Doctor, prescription, or diagnosis_id not found |
| 422 | `ValidationException` | Business validation failure |

---

### 3. Update Diagnostic Tests

**Endpoint**: `PUT /api/v1/doctors/{doctor_id}/prescriptions/{prescription_id}/analysis/diagnostic-tests`

**Description**: Updates the diagnostic tests section of a prescription. This is a **full replacement** operation - the entire diagnostic tests list is replaced with the provided data. Pass an empty array to clear all diagnostic test entries.

#### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `doctor_id` | UUID | Yes | UUID of the authenticated doctor |
| `prescription_id` | UUID | Yes | UUID of the prescription |

#### Request Body: `UpdateDiagnosticTestsRequest`

```json
{
  "diagnostic_tests": [
    {
      "test_id": "uuid",
      "test_type_id": "uuid | null",
      "note": "string | null"
    }
  ],
  "overall_note": "string | null"
}
```

**Field Specifications**:

| Field | Type | Required | Constraints | Description |
|-------|------|----------|-------------|-------------|
| `diagnostic_tests` | array | Yes | - | List of diagnostic test items (replaces existing) |
| `diagnostic_tests[].test_id` | UUID | Yes | Valid UUID from master diagnostic_test table | Reference to test from master table |
| `diagnostic_tests[].test_type_id` | UUID | No | Valid UUID from test types | Optional test type/subtype |
| `diagnostic_tests[].note` | string | No | Max length varies, trimmed | Optional notes for this test |
| `overall_note` | string | No | Trimmed | Overall note for all diagnostic tests |

#### Response: `UpdateDiagnosticTestsResponse`

**Status Code**: `200 OK`

```json
{
  "prescription_id": "uuid",
  "diagnostic_tests": [
    {
      "id": "uuid",
      "test_id": "uuid",
      "test_name": "string",
      "test_type_id": "uuid | null",
      "test_type_name": "string | null",
      "note": "string | null"
    }
  ],
  "overall_note": "string | null",
  "updated_at": "datetime"
}
```

#### Command Handler

- **Command**: `UpdateDiagnosticTestsCommand`
- **Handler**: `UpdateDiagnosticTestsCommandHandler`
- **Location**: `src/core/features/prescription_analysis/update_diagnostic_tests/`

#### Error Responses

| Status Code | Exception | Description |
|-------------|-----------|-------------|
| 400 | `ValidationException` | Invalid input data or prescription not in DRAFT status |
| 403 | `ForbiddenException` | Doctor context mismatch or not authorized |
| 404 | `NotFoundException` | Doctor, prescription, or test_id not found |

---

### 4. Update Medications

**Endpoint**: `PUT /api/v1/doctors/{doctor_id}/prescriptions/{prescription_id}/analysis/medications`

**Description**: Updates the medications section of a prescription. This is a **full replacement** operation - the entire medications list is replaced with the provided data. Pass an empty array to clear all medication entries.

#### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `doctor_id` | UUID | Yes | UUID of the authenticated doctor |
| `prescription_id` | UUID | Yes | UUID of the prescription |

#### Request Body: `UpdateMedicationsRequest`

```json
{
  "medications": [
    {
      "medicine_name": "string",
      "medicine_type": "enum:DosageForm",
      "strength": "float | null",
      "strength_unit": "string | null",
      "dose": {
        "value": "float",
        "unit": "string"
      } | null,
      "repeat_medicine": {
        "type": "enum:RepeatMedicineType",
        "interval_days": "integer | null",
        "days_of_week": ["enum:DayOfWeek"] | null,
        "days_of_month": ["integer"] | null,
        "interval_hours": "integer | null"
      } | null,
      "time_of_day": ["enum:TimeOfDay"] | null,
      "start_date": "date | null",
      "end_date": "date | null",
      "to_be_taken": "enum:MedicationTiming | null",
      "note": "string | null"
    }
  ]
}
```

**Field Specifications**:

| Field | Type | Required | Constraints | Description |
|-------|------|----------|-------------|-------------|
| `medications` | array | Yes | - | List of medications (replaces existing) |
| `medications[].medicine_name` | string | Yes | 1-256 chars, trimmed | Medicine name from third-party API |
| `medications[].medicine_type` | DosageForm | Yes | Valid enum value | Medicine dosage form (TABLET, CAPSULE, SYRUP, etc.) |
| `medications[].strength` | float | No | > 0, ≤ 10000 | Strength value |
| `medications[].strength_unit` | string | No | Max 20 chars, must be in allowed list | Strength unit (mg, ml, mcg, g, etc.) |
| `medications[].dose` | object | No | - | Dosage information |
| `medications[].dose.value` | float | Yes* | > 0, ≤ 100 | Dose value |
| `medications[].dose.unit` | string | Yes* | 1-50 chars, must be in allowed list | Dose unit (Tablet, ml, etc.) |
| `medications[].repeat_medicine` | object | No | - | Structured repeat pattern |
| `medications[].repeat_medicine.type` | RepeatMedicineType | Yes* | EVERYDAY, EVERY_X_DAYS, EVERY_WEEK, EVERY_MONTH, EVERY_X_HOURS | Type of repeat pattern |
| `medications[].repeat_medicine.interval_days` | integer | Conditional | 1-365, required for EVERY_X_DAYS | Days interval |
| `medications[].repeat_medicine.days_of_week` | array[DayOfWeek] | Conditional | Required for EVERY_WEEK | Days of week (MONDAY, TUESDAY, etc.) |
| `medications[].repeat_medicine.days_of_month` | array[integer] | Conditional | 1-31, required for EVERY_MONTH | Days of month (1-31) |
| `medications[].repeat_medicine.interval_hours` | integer | Conditional | 1-24, required for EVERY_X_HOURS | Hours interval |
| `medications[].time_of_day` | array[TimeOfDay] | No | No duplicates | Times to take (MORNING, NOON, EVENING, NIGHT) |
| `medications[].start_date` | date | No | Cannot be in past, YYYY-MM-DD | Start date |
| `medications[].end_date` | date | No | Must be ≥ start_date, YYYY-MM-DD | End date |
| `medications[].to_be_taken` | MedicationTiming | No | Valid enum | When to take (BEFORE_FOOD, AFTER_FOOD, etc.) |
| `medications[].note` | string | No | Max 250 chars, trimmed | Additional notes |

**Enum Values**:

- **DosageForm**: TABLET, CAPSULE, SYRUP, INJECTION, CREAM, OINTMENT, DROPS, INHALER, SPRAY, etc.
- **RepeatMedicineType**: EVERYDAY, EVERY_X_DAYS, EVERY_WEEK, EVERY_MONTH, EVERY_X_HOURS
- **DayOfWeek**: MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY
- **TimeOfDay**: MORNING, NOON, EVENING, NIGHT
- **MedicationTiming**: BEFORE_FOOD, AFTER_FOOD, WITH_FOOD, EMPTY_STOMACH, etc.

**Allowed Strength Units**: mg, ml, mcg, g, IU, etc. (validated against `STRENGTH_UNITS` constant)

**Allowed Dose Units**: Tablet, Capsule, ml, Drop, Puff, etc. (validated against `DOSE_UNITS` constant)

**Validation Rules**:

1. `medicine_name`: Required, 1-256 characters, trimmed
2. `medicine_type`: Required, valid DosageForm enum
3. `strength` and `strength_unit`: If strength is provided, strength_unit is required
4. `strength_unit`: Must be in allowed STRENGTH_UNITS list
5. `dose.unit`: Must be in allowed DOSE_UNITS list
6. `repeat_medicine`: Conditional field requirements based on type:
   - EVERYDAY: No additional fields required
   - EVERY_X_DAYS: `interval_days` required (1-365)
   - EVERY_WEEK: `days_of_week` required (non-empty array)
   - EVERY_MONTH: `days_of_month` required (1-31, no duplicates, sorted)
   - EVERY_X_HOURS: `interval_hours` required (1-24)
7. `time_of_day`: No duplicates allowed (duplicates removed automatically)
8. `start_date`: Cannot be in the past
9. `end_date`: Must be on or after `start_date`
10. `note`: Max 250 characters, trimmed

#### Response: `UpdateMedicationsResponse`

**Status Code**: `200 OK`

```json
{
  "prescription_id": "uuid",
  "medications": [
    {
      "id": "uuid",
      "medicine_name": "string",
      "medicine_type": "string",
      "strength": "float | null",
      "strength_unit": "string | null",
      "dose": {
        "value": "float",
        "unit": "string"
      } | null,
      "repeat_medicine": {
        "type": "string",
        "interval_days": "integer | null",
        "days_of_week": ["string"] | null,
        "days_of_month": ["integer"] | null,
        "interval_hours": "integer | null"
      } | null,
      "time_of_day": ["string"] | null,
      "start_date": "date | null",
      "end_date": "date | null",
      "to_be_taken": "string | null",
      "note": "string | null"
    }
  ],
  "updated_at": "datetime"
}
```

#### Command Handler

- **Command**: `UpdateMedicationsCommand`
- **Handler**: `UpdateMedicationsCommandHandler`
- **Location**: `src/core/features/prescription_analysis/update_medications/`

#### Error Responses

| Status Code | Exception | Description |
|-------------|-----------|-------------|
| 400 | `ValidationException` | Invalid input data, validation failures, or prescription not in DRAFT status |
| 403 | `ForbiddenException` | Doctor context mismatch or not authorized |
| 404 | `NotFoundException` | Doctor or prescription not found |
| 422 | `ValidationException` | Business validation failure (e.g., invalid strength_unit, invalid dose_unit) |

---

### 5. Update Advisory

**Endpoint**: `PUT /api/v1/doctors/{doctor_id}/prescriptions/{prescription_id}/analysis/advisory`

**Description**: Updates the advisory section of a prescription. This is a **full replacement** operation. Advisory is a text-only field containing overall recommendations and notes. Supports bullet points and numbered lists for readability.

#### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `doctor_id` | UUID | Yes | UUID of the authenticated doctor |
| `prescription_id` | UUID | Yes | UUID of the prescription |

#### Request Body: `UpdateAdvisoryRequest`

```json
{
  "note": "string"
}
```

**Field Specifications**:

| Field | Type | Required | Constraints | Description |
|-------|------|----------|-------------|-------------|
| `note` | string | Yes | 50-5000 chars, supports formatting | Overall advisory note with recommendations |

**Validation Rules**:

- `note`: Required, 50-5000 characters
- Supports formatting (bullet points, numbered lists)
- Whitespace trimmed

#### Response: `UpdateAdvisoryResponse`

**Status Code**: `200 OK`

```json
{
  "prescription_id": "uuid",
  "note": "string",
  "updated_at": "datetime"
}
```

#### Command Handler

- **Command**: `UpdateAdvisoryCommand`
- **Handler**: `UpdateAdvisoryCommandHandler`
- **Location**: `src/core/features/prescription_analysis/update_advisory/`

#### Error Responses

| Status Code | Exception | Description |
|-------------|-----------|-------------|
| 400 | `ValidationException` | Invalid input data (note too short/long) or prescription not in DRAFT status |
| 403 | `ForbiddenException` | Doctor context mismatch or not authorized |
| 404 | `NotFoundException` | Doctor or prescription not found |

---

### 6. Update Follow-Up

**Endpoint**: `PUT /api/v1/doctors/{doctor_id}/prescriptions/{prescription_id}/analysis/follow-up`

**Description**: Updates the follow-up section of a prescription. This is a **full replacement** operation. The system validates that the doctor has availability schedules for the follow-up date and that the date is not paused.

#### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `doctor_id` | UUID | Yes | UUID of the authenticated doctor |
| `prescription_id` | UUID | Yes | UUID of the prescription |

#### Request Body: `UpdateFollowUpRequest`

```json
{
  "follow_up_date": "date",
  "follow_up_time": "time | null",
  "details": "string | null",
  "overall_note": "string"
}
```

**Field Specifications**:

| Field | Type | Required | Constraints | Description |
|-------|------|----------|-------------|-------------|
| `follow_up_date` | date | Yes | 1-180 days in future, YYYY-MM-DD | Follow-up appointment date |
| `follow_up_time` | time | No | HH:MM:SS or HH:MM format | Follow-up appointment time |
| `details` | string | No | Max 500 chars (~100 words), trimmed | Follow-up details |
| `overall_note` | string | Yes | 20-2000 chars, trimmed | Overall consultation note |

**Validation Rules**:

1. `follow_up_date`:
   - Must be in YYYY-MM-DD format
   - Must be at least 1 day in the future (minimum: tomorrow)
   - Cannot be more than 180 days in the future (maximum: 6 months ahead)
   - Doctor must have availability schedules for this date
   - Date must not be paused in doctor's schedule

2. `follow_up_time`:
   - Optional
   - Format: HH:MM:SS or HH:MM
   - Example: "14:30:00" or "14:30"

3. `details`:
   - Optional
   - Max 500 characters (~100 words)
   - Whitespace trimmed

4. `overall_note`:
   - Required
   - 20-2000 characters
   - Whitespace trimmed

#### Response: `UpdateFollowUpResponse`

**Status Code**: `200 OK`

```json
{
  "prescription_id": "uuid",
  "follow_up_date": "date",
  "follow_up_time": "time | null",
  "details": "string | null",
  "overall_note": "string",
  "updated_at": "datetime"
}
```

#### Command Handler

- **Command**: `UpdateFollowUpCommand`
- **Handler**: `UpdateFollowUpCommandHandler`
- **Location**: `src/core/features/prescription_analysis/update_follow_up/`

#### Business Logic

1. Validates doctor context and authorization
2. Fetches the prescription to extract `clinic_id` from associated appointment
3. Validates prescription exists and belongs to doctor
4. Ensures prescription is in `DRAFT` status
5. Validates follow-up date is within 1-180 days range
6. Validates doctor has availability schedules for the clinic on the follow-up date
7. Validates the follow-up date is not paused
8. Creates/updates the follow-up record
9. Returns updated follow-up data

#### Special Implementation Notes

This endpoint requires additional repository access to fetch the prescription and extract the `clinic_id`:

```python
from src.data.repositories.prescription_repository import PrescriptionRepository
from src.data.specifications.prescription_specifications import (
    GetPrescriptionByIdAndDoctorSpecification,
)

prescription_repository = await self.account_context_accessor.resolve(PrescriptionRepository)
prescription = await prescription_repository.find_one_async(prescription_spec)
clinic_id = prescription.appointment.clinic_id
```

The `clinic_id` is required for the availability validation logic in the command handler.

#### Error Responses

| Status Code | Exception | Description |
|-------------|-----------|-------------|
| 400 | `ValidationException` | Invalid date format, date out of range, note too short/long, or prescription not in DRAFT status |
| 403 | `ForbiddenException` | Doctor context mismatch or not authorized |
| 404 | `NotFoundException` | Doctor or prescription not found |
| 422 | `ValidationException` | Doctor not available on follow-up date or date is paused |

---

## Common Response Patterns

### Success Responses

All successful mutation operations (PUT endpoints) return:

- `prescription_id`: UUID of the updated prescription
- Updated data for the modified section
- `updated_at`: Timestamp of the update (ISO 8601 format)

### Error Responses

All endpoints follow consistent error response structure from `oha_shared`:

```json
{
  "error": {
    "code": "string",
    "message": "string",
    "details": {} | null
  }
}
```

**Common HTTP Status Codes**:

| Status Code | Exception Type | Description |
|-------------|----------------|-------------|
| 200 | - | Success |
| 400 | `ValidationException` | Invalid request data or business validation failure |
| 403 | `ForbiddenException` | Authorization failure (doctor context mismatch) |
| 404 | `NotFoundException` | Resource not found (doctor, prescription, reference data) |
| 422 | `ValidationException` | Business logic validation failure |
| 500 | `InternalServerException` | Unexpected server error |

---

## Data Flow & Architecture

### CQRS Pattern

The controller follows CQRS pattern:

**Queries** (Read Operations):
- `GetPrescriptionReviewQuery` → `GetPrescriptionReviewQueryHandler`
- Returns combined view data (KYP + Analysis)
- No side effects

**Commands** (Write Operations):
- `UpdateDiagnosisCommand` → `UpdateDiagnosisCommandHandler`
- `UpdateDiagnosticTestsCommand` → `UpdateDiagnosticTestsCommandHandler`
- `UpdateMedicationsCommand` → `UpdateMedicationsCommandHandler`
- `UpdateAdvisoryCommand` → `UpdateAdvisoryCommandHandler`
- `UpdateFollowUpCommand` → `UpdateFollowUpCommandHandler`
- Each command encapsulates a state change
- Returns updated state after modification

### Request Flow

1. **Controller Layer** (`src/web/controllers/`)
   - Receives HTTP request
   - Validates doctor context via `ensure_doctor_context()`
   - Converts DTOs to domain models via `request.to_model()`
   - Dispatches command/query to mediator
   - Maps domain results to response DTOs
   - Returns HTTP response

2. **Core Layer** (`src/core/features/prescription_analysis/`)
   - Command/Query handlers contain business logic
   - Validates business rules (prescription status, date ranges, etc.)
   - Coordinates with repositories via specifications
   - Validates reference data (diagnosis_id, test_id, etc.)
   - Performs data persistence
   - Returns domain models

3. **Data Layer** (`src/data/`)
   - Repositories handle data access
   - Specifications encapsulate query logic
   - Entities define database schema
   - Enums define constants

### Dependency Injection

The controller uses dependency injection for:

- `Mediator`: Dispatches commands and queries to handlers
- `AccountContextAccessor`: Resolves scoped dependencies (repositories)

### DTO Conversion

**Request Flow**:
```
HTTP Request → Pydantic DTO → Domain Model → Command/Query → Handler
```

**Response Flow**:
```
Handler → Domain Model → Response DTO → HTTP Response
```

**Key Methods**:
- `request.to_model(prescription_id, doctor_id)`: Converts request DTO to domain model
- Manual mapping in controller: Converts domain result to response DTO

---

## Database Schema

### Related Tables

**Core Tables**:
- `doctor.prescriptions`: Main prescription record
- `doctor.prescription_diagnosis`: Diagnosis items
- `doctor.prescription_diagnostic_tests`: Diagnostic test items
- `doctor.prescription_medications`: Medication items
- `doctor.prescription_advisory`: Advisory notes
- `doctor.prescription_follow_up`: Follow-up appointment data

**Reference Tables**:
- `doctor.diagnosis`: Master diagnosis table (ICD-10 codes)
- `doctor.diagnostic_test`: Master diagnostic tests table
- `doctor.diagnostic_test_types`: Test type/subtype reference

**Related Tables**:
- `doctor.appointments`: Links prescriptions to appointments (provides clinic_id)
- `doctor.doctor_schedules`: Availability validation for follow-ups

### Prescription Status

Prescriptions must be in `DRAFT` status for all update operations. The status is validated in command handlers before allowing modifications.

**Prescription Status Values**:
- `DRAFT`: Editable, analysis can be updated
- `FINALIZED`: Read-only, cannot be modified
- Other statuses may exist but are not editable

---

## Security Considerations

### Authentication

- All endpoints require doctor authentication via `@doctor_authorize` decorator
- JWT token must be present in request headers
- Token must contain valid doctor identity claims

### Authorization

- Doctor context enforcement prevents cross-doctor access
- Each request validates that the authenticated doctor matches the `doctor_id` path parameter
- Prevents horizontal privilege escalation

### Input Validation

- All string inputs are trimmed to prevent whitespace injection
- Length constraints prevent buffer overflow and database issues
- Enum validation prevents invalid state values
- Date validation prevents past dates for start_date and follow_up_date
- Reference ID validation ensures foreign key integrity

### Data Integrity

- Full replacement semantics ensure consistent state
- Transactional updates maintain database consistency
- Cascading deletes handled at application layer
- Audit fields (`created_at`, `updated_at`, `created_by`) populated automatically

---

## Performance Considerations

### Database Queries

- Use specifications for complex queries
- Join operations for reference data (diagnosis_name, test_name)
- Batch operations for array replacements (delete all + insert many)

### Response Payload Size

- `GetPrescriptionReviewResponse` can be large (includes full KYP + analysis data)
- Consider pagination or filtering for large datasets in future iterations

### Caching

- Master reference data (diagnosis, tests) are good candidates for caching
- Prescription data should not be cached (frequently updated)

---

## Testing Recommendations

### Unit Tests

- Test DTO validation logic (field validators, model validators)
- Test domain model conversion (`to_model()` methods)
- Test handler business logic in isolation
- Test specification query logic

### Integration Tests

- Test full request/response cycle via FastAPI TestClient
- Test doctor context enforcement
- Test authorization failures
- Test reference data validation (non-existent IDs)
- Test prescription status validation
- Test date range validations
- Test database transactions and rollback scenarios

### Test Coverage Areas

1. **Authentication & Authorization**
   - Valid doctor token
   - Invalid/expired token
   - Doctor ID mismatch (forbidden access)

2. **Validation**
   - Required fields missing
   - Field length constraints
   - Enum value validation
   - Date format and range validation
   - Reference ID existence
   - Conditional field requirements (repeat_medicine patterns)

3. **Business Logic**
   - Prescription status validation (DRAFT only)
   - Doctor availability validation (follow-up)
   - Full replacement semantics (empty arrays clear data)
   - Date range validations (start_date < end_date)

4. **Error Handling**
   - 404 for non-existent resources
   - 403 for authorization failures
   - 400 for validation errors
   - 422 for business logic failures

---

## Future Enhancements

### Potential Improvements

1. **Partial Updates**: Support PATCH operations for incremental updates instead of full replacement

2. **Batch Operations**: Support updating multiple sections in a single request

3. **Validation Extensions**:
   - Drug interaction checks for medications
   - Contraindication checks based on patient allergies
   - Dosage validation based on patient age/weight

4. **Audit Trail**: Track detailed change history for compliance and review

5. **Templates**: Support saving and loading prescription templates for common patterns

6. **AI Assistance**: Integration with AI for medication recommendations based on diagnosis

7. **Webhooks**: Notify external systems when prescription analysis is updated

8. **Bulk Import**: Import prescription data from external sources

---

## Constants & Enums Reference

### DosageForm Enum
Tablet, Capsule, Syrup, Injection, Cream, Ointment, Drops, Inhaler, Spray, Powder, Gel, Patch, Suppository, etc.

### RepeatMedicineType Enum
- `EVERYDAY`: Take every day
- `EVERY_X_DAYS`: Take every X days (requires interval_days)
- `EVERY_WEEK`: Take on specific days of week (requires days_of_week)
- `EVERY_MONTH`: Take on specific days of month (requires days_of_month)
- `EVERY_X_HOURS`: Take every X hours (requires interval_hours)

### DayOfWeek Enum
MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY

### TimeOfDay Enum
MORNING, NOON, EVENING, NIGHT

### MedicationTiming Enum
BEFORE_FOOD, AFTER_FOOD, WITH_FOOD, EMPTY_STOMACH, etc.

### Strength Units (STRENGTH_UNITS constant)
mg, ml, mcg, g, IU, %, etc.

### Dose Units (DOSE_UNITS constant)
Tablet, Capsule, ml, Drop, Puff, Teaspoon, Tablespoon, etc.

---

## Related Documentation

- **CQRS Pattern**: See `oha_shared` documentation for mediator usage
- **Repository Pattern**: See `oha_shared` documentation for repository and specification patterns
- **Authentication**: See `oha_shared.fastapi.auth` for authorization decorators
- **Validation Helpers**: See `src/web/validators.py` for text field validation utilities
- **Enums**: See `src/data/enums/` for all enumeration definitions
- **Constants**: See `src/data/constants/medication_constants.py` for allowed units

---

## API Client Examples

### Example: Get Prescription Review

```bash
curl -X GET \
  "https://api.example.com/api/v1/doctors/123e4567-e89b-12d3-a456-426614174000/prescriptions/987fcdeb-51a2-43d8-9c71-0242ac130003/analysis/review" \
  -H "Authorization: Bearer <jwt_token>" \
  -H "Content-Type: application/json"
```

### Example: Update Diagnosis

```bash
curl -X PUT \
  "https://api.example.com/api/v1/doctors/123e4567-e89b-12d3-a456-426614174000/prescriptions/987fcdeb-51a2-43d8-9c71-0242ac130003/analysis/diagnosis" \
  -H "Authorization: Bearer <jwt_token>" \
  -H "Content-Type: application/json" \
  -d '{
    "diagnosis": [
      {
        "diagnosis_id": "550e8400-e29b-41d4-a716-446655440000",
        "diagnosis_type": "Type 2",
        "note": "Patient shows symptoms of diabetes mellitus type 2",
        "is_from_past": false
      }
    ]
  }'
```

### Example: Update Medications

```bash
curl -X PUT \
  "https://api.example.com/api/v1/doctors/123e4567-e89b-12d3-a456-426614174000/prescriptions/987fcdeb-51a2-43d8-9c71-0242ac130003/analysis/medications" \
  -H "Authorization: Bearer <jwt_token>" \
  -H "Content-Type: application/json" \
  -d '{
    "medications": [
      {
        "medicine_name": "Metformin",
        "medicine_type": "TABLET",
        "strength": 500,
        "strength_unit": "mg",
        "dose": {
          "value": 1,
          "unit": "Tablet"
        },
        "repeat_medicine": {
          "type": "EVERYDAY"
        },
        "time_of_day": ["MORNING", "EVENING"],
        "start_date": "2026-03-12",
        "end_date": "2026-06-12",
        "to_be_taken": "AFTER_FOOD",
        "note": "Take with plenty of water"
      }
    ]
  }'
```

### Example: Update Follow-Up

```bash
curl -X PUT \
  "https://api.example.com/api/v1/doctors/123e4567-e89b-12d3-a456-426614174000/prescriptions/987fcdeb-51a2-43d8-9c71-0242ac130003/analysis/follow-up" \
  -H "Authorization: Bearer <jwt_token>" \
  -H "Content-Type: application/json" \
  -d '{
    "follow_up_date": "2026-04-15",
    "follow_up_time": "10:30:00",
    "details": "Check blood sugar levels and review medication effectiveness",
    "overall_note": "Patient is responding well to treatment. Continue current medication and monitor blood sugar daily. Follow up in 5 weeks to assess progress and adjust dosage if needed."
  }'
```

---

## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-03-11 | Initial technical specification |

---

## Contact & Support

For questions or issues regarding this API specification, please contact the development team or refer to the project's issue tracker.
