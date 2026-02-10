# KYP (Know Your Patient) API Specification

## Overview

The KYP (Know Your Patient) API provides comprehensive patient health data management for doctors. It enables doctors to view, record, and manage patient vitals, essentials (medications/allergies), medical history, and chief complaints with full audit trails.

**Base Path:** `/api/v1/doctors/{doctor_id}/patients/{patient_user_id}/kyp`

**Authentication:** Bearer token with `@doctor_authorize` decorator (doctor role required)

**Authorization:** Doctor can only access their own patient records

---

## Data Model

### Vitals
- **vital_id** (UUID): Unique identifier for the vital type
- **vital_name** (string): Name of the vital (e.g., "Blood Pressure", "Heart Rate")
- **unit** (string): Measurement unit (e.g., "mmHg", "bpm")
- **readings** (array): Array of vital readings
  - **id** (UUID): Reading record ID
  - **value** (number): Numerical reading value
  - **systolic** (number, optional): For blood pressure
  - **diastolic** (number, optional): For blood pressure
  - **recorded_at** (ISO8601 datetime): When reading was taken
  - **recorded_by_id** (UUID): Doctor who recorded the reading
  - **recorded_by_name** (string): Name of recording doctor

### Essentials
- **current_medications** (array)
  - **id** (UUID): Medication record ID
  - **medicine_id** (UUID): Reference to master medicines table
  - **medicine_name** (string): Denormalized medicine name
  - **having_since** (string, optional): When patient started taking the medicine
  - **is_active** (boolean): Whether medication is currently active
  
- **allergies** (array)
  - **id** (UUID): Allergy record ID
  - **allergy_id** (UUID): Reference to master allergies table
  - **allergy_name** (string): Denormalized allergy name

- **lifestyle_history** (object, optional)
  - **steps** (number, optional): Daily steps
  - **activity** (string, optional): Physical activity level/description
  - **sleep** (number, optional): Hours of sleep
  - **food** (string, optional): Diet/food preferences
  - **alcohol** (string, optional): Alcohol consumption
  - **smoking** (string, optional): Smoking status

### Medical History
- **known_concerns** (array): Conditions/diseases patient has
  - **id** (UUID): Concern record ID
  - **disease_id** (UUID): Reference to master diseases table
  - **disease_name** (string): Denormalized disease name

- **family_history** (array): Diseases that run in the family
  - **id** (UUID): Family history record ID
  - **family_member** (string): Relation (e.g., "Father", "Sister")
  - **disease_id** (UUID): Reference to master diseases table
  - **disease_name** (string): Denormalized disease name
  - **details** (string, optional): Additional details about the condition

### Chief Complaints
- **id** (UUID): Complaint record ID
- **diagnosis_id** (UUID): Reference to master diagnoses table
- **name** (string): Name of complaint/diagnosis
- **since** (object, optional): Duration information
  - **value** (number): Numeric value
  - **unit** (string): Time unit (e.g., "days", "weeks", "months", "years")
- **severity** (string, optional): Severity level (MILD, MODERATE, SEVERE)
- **note** (string, optional): Additional clinical notes

---

## Endpoints

### 1. Get Patient KYP Data
**GET** `/{doctor_id}/patients/{patient_user_id}/kyp`

Retrieves all KYP sections for a patient in a single request. Returns the latest vital reading per vital type for performance optimization. Use `/vital-trends` for full vital history.

#### Request Parameters
| Parameter | Type | Location | Required | Description |
|-----------|------|----------|----------|-------------|
| doctor_id | UUID | Path | Yes | UUID of the requesting doctor |
| patient_user_id | string | Path | Yes | OHID of the patient |

#### Response
**Status:** 200 OK

```json
{
  "patient_user_id": "string",
  "vitals": [
    {
      "vital_id": "uuid",
      "vital_name": "string",
      "unit": "string",
      "readings": [
        {
          "id": "uuid",
          "value": 120,
          "systolic": 120,
          "diastolic": 80,
          "recorded_at": "2026-02-04T10:30:00Z",
          "recorded_by_id": "uuid",
          "recorded_by_name": "string"
        }
      ]
    }
  ],
  "essentials": {
    "current_medications": [...],
    "allergies": [...],
    "lifestyle_history": { ... }
  },
  "medical_history": {
    "known_concerns": [...],
    "family_history": [...]
  },
  "chief_complaints": [...],
  "chief_complaints_overall_note": "string",
  "last_updated_at": "2026-02-04T10:30:00Z",
  "last_updated_by_id": "uuid"
}
```

#### Error Responses
| Status | Error | Description |
|--------|-------|-------------|
| 403 | ForbiddenException | Doctor context required or mismatched doctor_id |
| 404 | NotFoundException | Doctor not found |

---

### 2. Get Vital Trends (Full History)
**GET** `/{doctor_id}/patients/{patient_user_id}/kyp/vital-trends`

Returns full reading history for patient vitals with pagination support. Results are sorted by `recorded_at` descending (latest first). Used for trend analysis and graphs.

#### Request Parameters
| Parameter | Type | Location | Required | Description |
|-----------|------|----------|----------|-------------|
| doctor_id | UUID | Path | Yes | UUID of the requesting doctor |
| patient_user_id | string | Path | Yes | OHID of the patient |
| vital_id | UUID | Query | No | Filter to specific vital type |
| limit | integer | Query | No | Max readings per vital (default: 50) |
| offset | integer | Query | No | Pagination offset (default: 0) |

#### Response
**Status:** 200 OK

```json
{
  "patient_user_id": "string",
  "vitals": [
    {
      "vital_id": "uuid",
      "vital_name": "string",
      "unit": "string",
      "readings": [
        {
          "id": "uuid",
          "value": 120,
          "systolic": 120,
          "diastolic": 80,
          "recorded_at": "2026-02-04T10:30:00Z",
          "recorded_by_id": "uuid",
          "recorded_by_name": "string"
        }
      ],
      "total_readings": 42
    }
  ]
}
```

#### Error Responses
| Status | Error | Description |
|--------|-------|-------------|
| 403 | ForbiddenException | Doctor context required or mismatched doctor_id |
| 404 | NotFoundException | Doctor not found |

---

### 3. Update Vitals
**PUT** `/{doctor_id}/patients/{patient_user_id}/kyp/vitals`

Adds new vital readings to the patient's KYP data. Each reading is timestamped and associated with the recording doctor. Multiple readings for the same vital are stored as history for trend analysis.

#### Request Body
```json
{
  "vitals": [
    {
      "vital_id": "uuid",
      "readings": [
        {
          "value": 120,
          "systolic": 120,
          "diastolic": 80
        }
      ]
    }
  ]
}
```

#### Response
**Status:** 200 OK

```json
{
  "patient_user_id": "string",
  "vitals": [
    {
      "vital_id": "uuid",
      "vital_name": "string",
      "unit": "string",
      "readings": [...]
    }
  ],
  "last_updated_at": "2026-02-04T10:30:00Z"
}
```

#### Error Responses
| Status | Error | Description |
|--------|-------|-------------|
| 403 | ForbiddenException | Doctor context required or mismatched doctor_id |
| 404 | NotFoundException | Doctor or vital ID not found |
| 400 | ValidationException | Invalid vital data |

---

### 4. Update Essentials
**PUT** `/{doctor_id}/patients/{patient_user_id}/kyp/essentials`

Replaces the entire essentials section. Pass `null` or empty arrays to clear specific sections.

#### Request Body
```json
{
  "current_medications": [
    {
      "medicine_id": "uuid",
      "having_since": "2025-06-15",
      "is_active": true
    }
  ],
  "allergies": [
    {
      "allergy_id": "uuid"
    }
  ],
  "lifestyle_history": {
    "steps": 8000,
    "activity": "Moderate exercise",
    "sleep": 7,
    "food": "Vegetarian",
    "alcohol": "Occasional",
    "smoking": "Never"
  }
}
```

#### Response
**Status:** 200 OK

```json
{
  "patient_user_id": "string",
  "essentials": {
    "current_medications": [...],
    "allergies": [...],
    "lifestyle_history": {...}
  },
  "last_updated_at": "2026-02-04T10:30:00Z"
}
```

#### Error Responses
| Status | Error | Description |
|--------|-------|-------------|
| 403 | ForbiddenException | Doctor context required or mismatched doctor_id |
| 404 | NotFoundException | Doctor, medicine ID, or allergy ID not found |
| 400 | ValidationException | Invalid essentials data |

---

### 5. Update Medical History
**PUT** `/{doctor_id}/patients/{patient_user_id}/kyp/medical-history`

Replaces the entire medical history section. Pass `null` or empty arrays to clear specific sections.

#### Request Body
```json
{
  "known_concerns": [
    {
      "disease_id": "uuid"
    }
  ],
  "family_history": [
    {
      "family_member": "Father",
      "disease_id": "uuid",
      "details": "Diagnosed at age 55"
    }
  ]
}
```

#### Response
**Status:** 200 OK

```json
{
  "patient_user_id": "string",
  "medical_history": {
    "known_concerns": [...],
    "family_history": [...]
  },
  "last_updated_at": "2026-02-04T10:30:00Z"
}
```

#### Error Responses
| Status | Error | Description |
|--------|-------|-------------|
| 403 | ForbiddenException | Doctor context required or mismatched doctor_id |
| 404 | NotFoundException | Doctor or disease ID not found |
| 400 | ValidationException | Invalid medical history data |

---

### 6. Update Chief Complaints
**PUT** `/{doctor_id}/patients/{patient_user_id}/kyp/chief-complaints`

Replaces the entire chief complaints section. Pass an empty array to clear all complaints.

#### Request Body
```json
{
  "chief_complaints": [
    {
      "diagnosis_id": "uuid",
      "since": {
        "value": 7,
        "unit": "days"
      },
      "severity": "MODERATE",
      "note": "Patient reports sharp pain"
    }
  ],
  "overall_note": "Follow-up appointment scheduled"
}
```

#### Response
**Status:** 200 OK

```json
{
  "patient_user_id": "string",
  "chief_complaints": [
    {
      "id": "uuid",
      "diagnosis_id": "uuid",
      "name": "string",
      "since": {
        "value": 7,
        "unit": "days"
      },
      "severity": "MODERATE",
      "note": "string"
    }
  ],
  "overall_note": "string",
  "last_updated_at": "2026-02-04T10:30:00Z"
}
```

#### Error Responses
| Status | Error | Description |
|--------|-------|-------------|
| 403 | ForbiddenException | Doctor context required or mismatched doctor_id |
| 404 | NotFoundException | Doctor or diagnosis ID not found |
| 400 | ValidationException | Invalid chief complaints data |

---

## Authentication & Authorization

### Doctor Context Validation
All endpoints validate that the authenticated doctor matches the `doctor_id` in the request path:

```python
async def ensure_doctor_context(self, doctor_id: UUID) -> None:
    current_doctor_id = get_current_doctor_id()
    if current_doctor_id is None:
        raise ForbiddenException("Doctor context is required for this route.")
    if current_doctor_id != doctor_id:
        raise ForbiddenException(
            "You are not authorized to access this doctor's resources"
        )
```

**Key Points:**
- Uses `@doctor_authorize` class-level decorator for role validation
- Method-level decorators can include feature-specific access control:
  ```python
  @doctor_authorize(feature=Features.Doctor.KYP, access=Access.READ)
  ```
- All write operations must include `Access.WRITE` permission
- All read operations use `Access.READ` permission

---

## Data Integrity & Audit Trail

### Automatic Fields
- **id**: Generated by `BaseEntity` listeners (never set manually)
- **created_at**: Automatically set on record creation
- **updated_at**: Automatically updated on modification
- **created_by**: Set via shared SQLAlchemy events
- **updated_by**: Set via shared SQLAlchemy events

### Denormalized Fields
Fields like `medicine_name`, `allergy_name`, `disease_name` are denormalized for performance:
- Copied from master tables at write time
- Not updated if master records change (historical accuracy)
- Used for API responses without additional joins

### Uniqueness Constraints
- Repository uses `exists_async_by_specification()` for checking uniqueness
- No manual validation in handlers
- Constraints enforced at database level through specifications

---

## Data Specifications & Filtering

### Vital Records Specification
- Fetches all vitals for a patient with their readings
- Supports filtering by vital_id
- Sorts readings by recorded_at descending
- Supports pagination via limit/offset

### Medication Records Specification
- Filters by patient and doctor context
- Denormalizes medicine names from master table
- Returns only active medications or all based on request

### Allergy Records Specification
- Filters by patient
- Includes denormalized allergy names
- No status filtering (all allergies preserved)

### Disease/Diagnosis References
- Always validated against master tables
- Returns 404 if reference doesn't exist
- Denormalized names stored for historical accuracy

---

## Workflow Examples

### Complete Patient Intake
1. **Get existing KYP data**: GET `/kyp`
2. **Update vitals**: PUT `/kyp/vitals`
3. **Update essentials**: PUT `/kyp/essentials`
4. **Update medical history**: PUT `/kyp/medical-history`
5. **Update chief complaints**: PUT `/kyp/chief-complaints`

### Vital Trend Analysis
1. **Get vital trends**: GET `/kyp/vital-trends?vital_id={vital_id}&limit=30`
2. **Process readings** for graphing/analysis

### Medication Management
1. **Get current essentials**: GET `/kyp`
2. **Update medications**: PUT `/kyp/essentials` with new list
3. **Note**: Replaces entire medications section

---

## Performance Considerations

### Endpoint Optimization
- **GET `/kyp`**: Returns only latest vital reading per vital type
- **GET `/vital-trends`**: Returns full history with pagination (default limit: 50)
- **PUT operations**: Entire sections replaced (not partial updates)

### Database Queries
- Uses Specification pattern for composable, optimized queries
- Denormalized fields reduce JOIN complexity
- Pagination prevents large result sets

### Caching Opportunities
- Patient KYP data can be cached with TTL
- Invalidated on any PUT operation
- Master table references (medicines, diseases) have longer TTL

---

## Error Handling

### Common Errors
| Scenario | Status | Error Type | Message |
|----------|--------|-----------|---------|
| Missing doctor context | 403 | ForbiddenException | "Doctor context is required for this route" |
| Wrong doctor_id | 403 | ForbiddenException | "You are not authorized to access this doctor's resources" |
| Doctor not found | 404 | NotFoundException | "Doctor not found" |
| Medicine/disease reference invalid | 404 | NotFoundException | Reference not found in master table |
| Invalid vital data | 400 | ValidationException | Data validation failed |
| Token missing/invalid | 401 | Unauthorized | Authentication failed |

### Response Format
All errors return structured JSON:
```json
{
  "error": "error_type",
  "message": "Human-readable message",
  "timestamp": "2026-02-04T10:30:00Z"
}
```

---

## Implementation Details

### Controller Pattern
- Class-based with `@doctor_authorize` decorator
- Mediator pattern for CQRS command/query dispatch
- Explicit DTOs with `to_model()` conversion helpers
- Response mapping through mapper services

### CQRS Handlers
Located in `src/core/features/kyp/`:
- **GetPatientKypDataQuery**: Retrieves all KYP sections
- **GetVitalTrendsQuery**: Fetches vital reading history
- **UpdateVitalsCommand**: Adds vital readings
- **UpdateEssentialsCommand**: Replaces essentials section
- **UpdateMedicalHistoryCommand**: Replaces medical history
- **UpdateChiefComplaintsCommand**: Replaces chief complaints

### DTOs
Located in `src/web/dtos/kyp/`:
- Request DTOs: `Update*Request` classes
- Response DTOs: `*Response`, `*Dto` classes
- DTO to model conversion via `to_model()` methods

### Repositories
- **VitalRepository**: Manage vital records and readings
- **EssentialsRepository**: Handle medications, allergies, lifestyle
- **MedicalHistoryRepository**: Manage concerns and family history
- **ChiefComplaintRepository**: Handle complaint records
- **MedicineRepository**: Master medicines reference
- **AllergyRepository**: Master allergies reference
- **DiseaseRepository**: Master diseases reference
- **DiagnosisRepository**: Master diagnoses reference

---
