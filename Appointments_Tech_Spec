# OHA Doctor API - QA Technical Specification

## Appointments, Calendar Pauses, and Availability Endpoints

**Version:** 1.0
**Last Updated:** January 2026
**Base URL:** `/api/v1/doctors/{doctor_id}`

---

## Table of Contents

1. [Authentication & Authorization](#authentication--authorization)
2. [Enumerations Reference](#enumerations-reference)
3. [Appointments API](#appointments-api)
   - [List Appointments](#1-list-appointments)
   - [Get Appointment](#2-get-appointment)
   - [Create Appointment](#3-create-appointment)
   - [Cancel Appointment](#4-cancel-appointment)
   - [Complete Appointment](#5-complete-appointment)
   - [Reschedule Appointment](#6-reschedule-appointment)
4. [Calendar Pauses API](#calendar-pauses-api)
   - [List Calendar Pauses](#7-list-calendar-pauses)
   - [Get Calendar Pause](#8-get-calendar-pause)
   - [Create Calendar Pause](#9-create-calendar-pause)
   - [Cancel Calendar Pause](#10-cancel-calendar-pause)
5. [Available Slots API](#available-slots-api)
   - [Get Available Slots](#11-get-available-slots)
6. [Error Responses](#error-responses)
7. [Test Scenarios](#test-scenarios)

---

## Authentication & Authorization

All endpoints require:
- **Authentication:** Bearer token (JWT)
- **Authorization:** `@doctor_authorize` decorator with feature-based access control
- **Doctor Context:** The `doctor_id` in the path must match the authenticated doctor's ID

### Feature Access Control
| Feature | Access Level |
|---------|--------------|
| `Features.Doctor.APPOINTMENTS` | READ / WRITE |
| `Features.Doctor.AVAILABILITY` | READ / WRITE |

### Error Responses for Authorization
- **401 Unauthorized:** Missing or invalid token
- **403 Forbidden:** Doctor context mismatch ("You are not authorized to access this doctor's resources")

---

## Enumerations Reference

### ChannelType (Appointment Type)
```
"online"       - Online/Video consultation
"in_clinic"    - In-clinic visit
"home_visit"   - Home visit
```

### AppointmentStatus
```
"SCHEDULED"    - Initial status for regular bookings
"PRIORITY"     - Initial status for priority bookings
"RESCHEDULED"  - Appointment was rescheduled (old appointment)
"IN_PROGRESS"  - Consultation in progress
"COMPLETED"    - Appointment completed successfully
"CANCELLED"    - Appointment was cancelled
"NO_SHOW"      - Patient did not attend
"BLOCKED"      - Slot is blocked
"PAUSE"        - Calendar pause period
```

### CompletionStatus (for Complete Appointment)
```
"COMPLETED"    - Patient attended the appointment
"NO_SHOW"      - Patient did not attend
```

### PauseReasonCategory
```
"CONFERENCE"         - Attending a conference
"STAFF_SHORTAGE"     - Staff shortage
"PERSONAL_EMERGENCY" - Personal emergency
"OTHER"              - Other reason (requires details)
```

### PauseStatus
```
"ACTIVE"    - Pause is currently active
"CANCELLED" - Pause was cancelled
"EXPIRED"   - Pause period has passed
```

### DayOfWeek
```
"MONDAY", "TUESDAY", "WEDNESDAY", "THURSDAY", "FRIDAY", "SATURDAY", "SUNDAY"
```

### AdvanceBookingPeriod
```
"WEEKLY"  - Value represents weeks (max 52)
"MONTHLY" - Value represents months (max 24)
"YEARLY"  - Value represents years (max 5)
```

### UpdateStrategy (for Schedule Updates)
```
"IMMEDIATE" - Update the schedule immediately
"VERSION"   - Create a new version with effective_from_date
```

### CurrencyCode
```
"INR" - Indian Rupee (default)
```

---

## Appointments API

### 1. List Appointments

**Endpoint:** `GET /api/v1/doctors/{doctor_id}/appointments`
**Access:** READ

#### Query Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `date_from` | date | No | Start date filter (inclusive), format: `YYYY-MM-DD` |
| `date_to` | date | No | End date filter (inclusive), format: `YYYY-MM-DD` |
| `appointment_status` | array | No | Filter by status(es), can provide multiple |
| `appointment_type` | array | No | Filter by type(s), can provide multiple |
| `limit` | int | No | Max results (1-100, default: 20) |
| `offset` | int | No | Skip results (default: 0) |

#### Response (200 OK)
```json
{
  "items": [
    {
      "id": "uuid",
      "channel_id": "uuid | null",
      "appointment_date": "2025-01-30",
      "slot_start_time": "09:00:00",
      "slot_end_time": "09:30:00",
      "appointment_type": "online",
      "appointment_status": "SCHEDULED",
      "category": "General Consultation | null",
      "token_number": "T001 | P001 | null",
      "is_priority_booking": false,
      "total_amount": "500.00",
      "currency_code": "INR",
      "booked_by": {
        "user_id": "OHID123",
        "full_name": "John Doe",
        "profile_picture_url": "https://... | null"
      },
      "booked_for": {
        "user_id": "OHID456",
        "full_name": "Jane Doe",
        "profile_picture_url": "https://... | null"
      },
      "clinic_id": "uuid",
      "clinic_name": "ABC Clinic",
      "doctor_id": "uuid",
      "doctor_name": "Dr. Smith",
      "created_at_utc": "2025-01-30T10:00:00Z"
    }
  ],
  "total": 100,
  "limit": 20,
  "offset": 0
}
```

---

### 2. Get Appointment

**Endpoint:** `GET /api/v1/doctors/{doctor_id}/appointments/{appointment_id}`
**Access:** READ

#### Path Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `doctor_id` | UUID | Yes | Doctor identifier |
| `appointment_id` | UUID | Yes | Appointment identifier |

#### Response (200 OK)
```json
{
  "id": "uuid",
  "channel_id": "uuid | null",
  "appointment_date": "2025-01-30",
  "slot_start_time": "09:00:00",
  "slot_end_time": "09:30:00",
  "appointment_type": "online",
  "appointment_status": "SCHEDULED",
  "category": "General Consultation | null",
  "staff_notes": "Patient requested morning slot | null",
  "token_number": "T001 | null",
  "is_priority_booking": false,
  "total_amount": "500.00",
  "currency_code": "INR",
  "booked_by": {
    "user_id": "OHID123",
    "full_name": "John Doe",
    "email": "john@example.com",
    "phone": "+91-9876543210 | null",
    "date_of_birth": "1990-01-15 | null",
    "gender": "Male | null",
    "profile_picture_url": "https://... | null"
  },
  "booked_for": {
    "user_id": "OHID456",
    "full_name": "Jane Doe",
    "email": "jane@example.com",
    "phone": "+91-9876543211 | null",
    "date_of_birth": "1985-05-20 | null",
    "gender": "Female | null",
    "profile_picture_url": "https://... | null"
  },
  "clinic": {
    "id": "uuid",
    "name": "ABC Clinic",
    "contact_number": "+91-1234567890 | null",
    "email": "clinic@example.com | null"
  },
  "doctor": {
    "id": "uuid",
    "full_name": "Dr. Smith"
  },
  "procedure": {
    "id": "uuid",
    "name": "Root Canal",
    "base_price": "5000.00 | null"
  } | null,
  "created_at_utc": "2025-01-30T10:00:00Z",
  "updated_at_utc": "2025-01-30T10:00:00Z | null",
  "completed_at_utc": "2025-01-30 | null",
  "cancelled_at_utc": "2025-01-30 | null"
}
```

#### Error Responses
- **404 Not Found:** Appointment not found

---

### 3. Create Appointment

**Endpoint:** `POST /api/v1/doctors/{doctor_id}/appointments`
**Access:** WRITE

#### Request Body
```json
{
  "clinic_id": "uuid",
  "doctor_id": "uuid",
  "booked_by_user_id": "OHID123",
  "booked_for_user_id": "OHID456",
  "appointment_date": "2025-02-15",
  "slot_start_time": "09:00:00",
  "slot_end_time": "09:30:00",
  "appointment_type": "online",
  "category": "General Consultation",
  "procedure_id": "uuid | null",
  "staff_notes": "Patient prefers morning | null",
  "is_priority_booking": false
}
```

#### Field Validations
| Field | Validation |
|-------|------------|
| `booked_by_user_id` | Required, 1-64 chars, non-empty |
| `booked_for_user_id` | Required, 1-64 chars, non-empty |
| `category` | Optional, max 100 chars |
| `is_priority_booking` | Default: false |

#### Response (201 Created)
```json
{
  "id": "uuid",
  "channel_id": "uuid",
  "clinic_id": "uuid",
  "doctor_id": "uuid",
  "booked_by_user_id": "OHID123",
  "booked_for_user_id": "OHID456",
  "appointment_date": "2025-02-15",
  "slot_start_time": "09:00:00",
  "slot_end_time": "09:30:00",
  "appointment_type": "online",
  "appointment_status": "SCHEDULED",
  "is_priority_booking": false,
  "total_amount": "500.00",
  "currency_code": "INR",
  "category": "General Consultation | null",
  "procedure_id": "uuid | null",
  "token_number": "T001"
}
```

#### Error Responses
- **400 Validation Exception:** Invalid input data
- **403 Forbidden:** User not authorized to book for patient
- **404 Not Found:** Clinic, doctor, or user not found

---

### 4. Cancel Appointment

**Endpoint:** `PATCH /api/v1/doctors/{doctor_id}/appointments/{appointment_id}/cancel`
**Access:** WRITE

#### Request Body
```json
{
  "cancelled_by": "doctor",
  "cancellation_reason": "Doctor unavailable due to emergency | null"
}
```

#### Field Validations
| Field | Validation |
|-------|------------|
| `cancelled_by` | Required, 1-50 chars (e.g., "doctor", "patient", "system") |
| `cancellation_reason` | Optional, max 1000 chars |

#### Response (200 OK)
```json
{
  "id": "uuid",
  "appointment_status": "CANCELLED",
  "cancelled_at_utc": "2025-01-30",
  "cancelled_by": "doctor",
  "cancellation_reason": "Doctor unavailable due to emergency | null"
}
```

#### Error Responses
- **400 Validation Exception:** Appointment cannot be cancelled (wrong status)
- **404 Not Found:** Appointment not found

---

### 5. Complete Appointment

**Endpoint:** `PATCH /api/v1/doctors/{doctor_id}/appointments/{appointment_id}/complete`
**Access:** WRITE

#### Request Body
```json
{
  "status": "COMPLETED",
  "completed_by": "dr.smith@example.com",
  "staff_notes": "Follow-up in 2 weeks | null"
}
```

#### Field Validations
| Field | Validation |
|-------|------------|
| `status` | Required, enum: `"COMPLETED"` or `"NO_SHOW"` |
| `completed_by` | Required, 1-100 chars |
| `staff_notes` | Optional, max 5000 chars |

#### Response (200 OK)
```json
{
  "id": "uuid",
  "appointment_status": "COMPLETED",
  "completed_at_utc": "2025-01-30",
  "completed_by": "dr.smith@example.com",
  "staff_notes": "Follow-up in 2 weeks | null"
}
```

#### Error Responses
- **400 Validation Exception:** Appointment cannot be completed (wrong status)
- **404 Not Found:** Appointment not found

---

### 6. Reschedule Appointment

**Endpoint:** `PATCH /api/v1/doctors/{doctor_id}/appointments/{appointment_id}/reschedule`
**Access:** WRITE

#### Request Body
```json
{
  "new_appointment_date": "2025-02-20",
  "new_slot_start_time": "10:00:00",
  "new_slot_end_time": "10:30:00",
  "new_appointment_type": "in_clinic",
  "category": "Follow-up | null",
  "procedure_id": "uuid | null",
  "staff_notes": "Rescheduled per patient request | null",
  "is_priority_booking": false
}
```

#### Field Validations
| Field | Validation |
|-------|------------|
| `new_appointment_date` | Required |
| `new_slot_start_time` | Required |
| `new_slot_end_time` | Required |
| `new_appointment_type` | Required |
| `category` | Optional, max 100 chars |
| `is_priority_booking` | Default: false |

#### Response (200 OK)
```json
{
  "new_appointment_id": "uuid",
  "old_appointment_id": "uuid",
  "old_appointment_status": "RESCHEDULED",
  "new_appointment_status": "SCHEDULED",
  "new_appointment_date": "2025-02-20",
  "new_slot_start_time": "10:00:00",
  "new_slot_end_time": "10:30:00"
}
```

#### Error Responses
- **400 Validation Exception:** Appointment cannot be rescheduled or new slot invalid
- **404 Not Found:** Appointment not found

---

## Calendar Pauses API

### 7. List Calendar Pauses

**Endpoint:** `GET /api/v1/doctors/{doctor_id}/appointments/calendar-pauses`
**Access:** READ

#### Query Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `clinic_id` | UUID | No | Filter by specific clinic |
| `pause_statuses` | array | No | Filter by status(es) |
| `date_from` | date | No | Pauses overlapping from this date |
| `date_to` | date | No | Pauses overlapping until this date |
| `limit` | int | No | Max results (1-100, default: 50) |
| `offset` | int | No | Skip results (default: 0) |

#### Response (200 OK)
```json
{
  "items": [
    {
      "id": "uuid",
      "doctor_id": "uuid",
      "clinic_id": "uuid | null",
      "pause_start_date": "2025-02-01",
      "pause_end_date": "2025-02-03",
      "pause_start_time": "08:00:00",
      "pause_end_time": "18:00:00",
      "pause_reason_category": "CONFERENCE",
      "pause_reason_details": "Annual medical conference | null",
      "pause_status": "ACTIVE",
      "had_conflicting_appointments": true,
      "conflicting_appointments_count": 5,
      "created_at_utc": "2025-01-25T10:00:00Z"
    }
  ],
  "total": 10,
  "limit": 50,
  "offset": 0
}
```

---

### 8. Get Calendar Pause

**Endpoint:** `GET /api/v1/doctors/{doctor_id}/appointments/calendar-pauses/{pause_id}`
**Access:** READ

#### Response (200 OK)
```json
{
  "id": "uuid",
  "doctor_id": "uuid",
  "clinic_id": "uuid | null",
  "pause_start_date": "2025-02-01",
  "pause_end_date": "2025-02-03",
  "pause_start_time": "08:00:00",
  "pause_end_time": "18:00:00",
  "pause_reason_category": "CONFERENCE",
  "pause_reason_details": "Annual medical conference | null",
  "pause_status": "ACTIVE",
  "had_conflicting_appointments": true,
  "conflicting_appointments_count": 5,
  "created_at_utc": "2025-01-25T10:00:00Z",
  "updated_at_utc": "2025-01-25T10:00:00Z",
  "cancelled_at_utc": "2025-01-26T15:00:00Z | null",
  "cancelled_by": "dr.smith@example.com | null",
  "cancellation_reason": "Conference cancelled | null"
}
```

---

### 9. Create Calendar Pause

**Endpoint:** `POST /api/v1/doctors/{doctor_id}/appointments/calendar-pauses`
**Access:** WRITE

#### Request Body
```json
{
  "clinic_id": "uuid | null",
  "pause_start_date": "2025-02-01",
  "pause_end_date": "2025-02-03",
  "pause_start_time": "08:00:00",
  "pause_end_time": "18:00:00",
  "pause_reason_category": "CONFERENCE",
  "pause_reason_details": "Attending annual medical conference | null"
}
```

#### Field Validations
| Field | Validation |
|-------|------------|
| `clinic_id` | Optional (null = all clinics for doctor) |
| `pause_start_date` | Required |
| `pause_end_date` | Required |
| `pause_start_time` | Required (use 00:00:00 for full day start) |
| `pause_end_time` | Required (use 23:59:59 for full day end) |
| `pause_reason_category` | Required, enum |
| `pause_reason_details` | Required when category is "OTHER", max 1000 chars |

#### Response (201 Created)
```json
{
  "id": "uuid",
  "doctor_id": "uuid",
  "clinic_id": "uuid | null",
  "pause_start_date": "2025-02-01",
  "pause_end_date": "2025-02-03",
  "pause_start_time": "08:00:00",
  "pause_end_time": "18:00:00",
  "pause_reason_category": "CONFERENCE",
  "pause_reason_details": "Attending annual medical conference | null",
  "pause_status": "ACTIVE",
  "had_conflicting_appointments": true,
  "conflicting_appointments_count": 5,
  "created_at_utc": "2025-01-25T10:00:00Z"
}
```

#### Error Responses
- **400 Validation Exception:** Overlapping pause exists
- **404 Not Found:** Doctor or clinic not found

---

### 10. Cancel Calendar Pause

**Endpoint:** `PATCH /api/v1/doctors/{doctor_id}/appointments/calendar-pauses/{pause_id}/cancel`
**Access:** WRITE

#### Request Body
```json
{
  "cancelled_by": "dr.smith@example.com",
  "cancellation_reason": "Conference cancelled | null"
}
```

#### Field Validations
| Field | Validation |
|-------|------------|
| `cancelled_by` | Required, 1-64 chars |
| `cancellation_reason` | Optional, max 1000 chars |

#### Response (200 OK)
```json
{
  "id": "uuid",
  "pause_status": "CANCELLED",
  "cancelled_at_utc": "2025-01-26T15:00:00Z",
  "cancelled_by": "dr.smith@example.com",
  "cancellation_reason": "Conference cancelled | null"
}
```

#### Error Responses
- **400 Validation Exception:** Pause cannot be cancelled (not ACTIVE)
- **404 Not Found:** Pause not found

---

## Available Slots API

### 11. Get Available Slots

**Endpoint:** `GET /api/v1/doctors/{doctor_id}/available-slots`
**Access:** READ

This is the patient-facing API for checking slot availability.

#### Query Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `clinic_id` | UUID | **Yes** | Clinic for the appointment |
| `date` | string | **Yes** | Date for slots (format: `YYYY-MM-DD`) |
| `appointment_type` | enum | No | Filter by specific type |

#### Response (200 OK)
```json
{
  "doctor_id": "uuid",
  "clinic_id": "uuid",
  "clinic_name": "ABC Clinic",
  "date_range": {
    "start_date": "2025-02-15",
    "end_date": "2025-02-15"
  },
  "slots_by_date": {
    "2025-02-15": {
      "slot_date": "2025-02-15",
      "day_of_week": "SATURDAY",
      "slots": [
        {
          "start_time": "09:00:00",
          "end_time": "09:30:00",
          "appointment_type": "online",
          "duration_minutes": 30,
          "max_appointments": 3,
          "scheduled_appointments": 1,
          "is_available": true,
          "unavailable_reason": null
        },
        {
          "start_time": "09:30:00",
          "end_time": "10:00:00",
          "appointment_type": "online",
          "duration_minutes": 30,
          "max_appointments": 3,
          "scheduled_appointments": 3,
          "is_available": false,
          "unavailable_reason": "fully_booked"
        }
      ],
      "total_slots": 16,
      "available_slots": 12,
      "is_blocked": false,
      "blocked_reason": null
    }
  },
  "summary": {
    "total_days": 1,
    "days_with_availability": 1,
    "total_slots": 16,
    "available_slots": 12
  }
}
```

#### Unavailable Reasons
| Reason | Description |
|--------|-------------|
| `fully_booked` | Slot has reached max appointments |
| `minimum_notice` | Booking too close to slot time |
| `outside_window` | Date outside advance booking window |
| `no_schedule` | No schedule defined for this day/time |
| `paused` | Doctor has calendar pause active |

#### Error Responses
- **400 Validation Exception:** Invalid date format

---

## Error Responses

### Standard Error Format
```json
{
  "detail": "Error message description"
}
```

### HTTP Status Codes
| Code | Description |
|------|-------------|
| 200 | OK - Request successful |
| 201 | Created - Resource created |
| 400 | Bad Request - Validation error |
| 401 | Unauthorized - Missing/invalid token |
| 403 | Forbidden - Not authorized for resource |
| 404 | Not Found - Resource not found |
| 500 | Internal Server Error |

---

## Test Scenarios

### Appointments

#### Create Appointment
1. **Happy Path:** Create appointment with all required fields
2. **Priority Booking:** Create with `is_priority_booking: true`, verify different token prefix (P001)
3. **Self Booking:** `booked_by_user_id` same as `booked_for_user_id`
4. **Family Booking:** `booked_by_user_id` different from `booked_for_user_id`
5. **Invalid Clinic:** Non-existent clinic_id -> 404
6. **Past Date:** appointment_date in past -> 400
7. **Slot Conflict:** Fully booked slot -> 400
8. **Invalid User ID:** Empty or whitespace user ID -> 400

#### Cancel Appointment
1. **Happy Path:** Cancel SCHEDULED appointment
2. **Already Cancelled:** Cancel CANCELLED appointment -> 400
3. **Already Completed:** Cancel COMPLETED appointment -> 400
4. **With Reason:** Include cancellation_reason
5. **Without Reason:** cancellation_reason null

#### Complete Appointment
1. **Completed Status:** Mark as COMPLETED
2. **No Show Status:** Mark as NO_SHOW
3. **Wrong Status:** Complete CANCELLED appointment -> 400
4. **With Notes:** Include staff_notes
5. **Notes Length:** staff_notes at max 5000 chars

#### Reschedule Appointment
1. **Happy Path:** Reschedule to valid future slot
2. **Change Type:** Reschedule from online to in_clinic
3. **Same Slot:** Reschedule to same date/time -> 400 (or allowed?)
4. **Past Date:** Reschedule to past date -> 400
5. **Verify Old Status:** Old appointment becomes RESCHEDULED
6. **Verify New Status:** New appointment is SCHEDULED

### Calendar Pauses

#### Create Pause
1. **Single Clinic:** Pause specific clinic
2. **All Clinics:** clinic_id null, pauses all
3. **Full Day:** 00:00:00 to 23:59:59
4. **Partial Day:** 08:00:00 to 13:00:00
5. **Multi-Day:** Different start and end dates
6. **Category OTHER:** Requires pause_reason_details
7. **Overlapping Pause:** Create overlapping pause -> 400
8. **Conflicting Appointments:** Verify count returned

#### Cancel Pause
1. **Active Pause:** Cancel ACTIVE pause
2. **Already Cancelled:** Cancel CANCELLED pause -> 400
3. **Expired Pause:** Cancel EXPIRED pause -> 400

### Available Slots

1. **Valid Date:** Get slots for future date
2. **No Schedule:** Date with no schedule defined
3. **Fully Booked:** All slots at max capacity
4. **Partial Availability:** Some slots available
5. **Calendar Pause:** Date with active pause -> is_blocked: true
6. **Filter by Type:** Filter only online slots
7. **Minimum Notice:** Slots within notice period unavailable
8. **Outside Window:** Date beyond advance booking period

### Authorization Tests

1. **Missing Token:** All endpoints -> 401
2. **Invalid Token:** All endpoints -> 401
3. **Wrong Doctor ID:** Access another doctor's resources -> 403
4. **Read vs Write:** READ access trying WRITE operation -> 403

---

## Notes for QA

1. **Token Numbers:** Regular appointments get T001, T002, etc. Priority get P001, P002, etc. Resets daily.

2. **Time Format:** All times use 24-hour format HH:MM:SS

3. **Date Format:** All dates use ISO format YYYY-MM-DD

4. **UUIDs:** All IDs are UUID v4 format

5. **Amounts:** All monetary amounts are decimals with 2 decimal places

6. **User IDs (OHID):** User identifiers are string format, not UUIDs

7. **Soft Deletes:** Entities are soft-deleted; check `is_deleted` flags

8. **Channel ID:** Created for appointments to support video/chat features

9. **Clinic Scope:** Calendar pauses with null clinic_id affect ALL doctor's clinics
