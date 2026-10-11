# boarding Module — Simple Explanation

## 1. What this module does

The driver's phone shows a trip QR; students scan it to board. Attendance is written when the trip ends.

## 2. Where the code is

- Backend: `backend/app/modules/boarding/`
- Frontend: `frontend/lib/modules/boarding/`

## 3. Backend files

- `backend/app/modules/boarding/models.py`
- `backend/app/modules/boarding/router.py`
- `backend/app/modules/boarding/schemas.py`
- `backend/app/modules/boarding/service.py`

## 4. Frontend files

- `frontend/lib/modules/boarding/data/boarding_api.dart`
- `frontend/lib/modules/boarding/screens/driver_qr_screen.dart`
- `frontend/lib/modules/boarding/screens/driver_roster_screen.dart`
- `frontend/lib/modules/boarding/screens/student_attendance_screen.dart`
- `frontend/lib/modules/boarding/screens/student_scan_screen.dart`

## 5. What the existing project documentation says

## Flow
## Anti-cheat
- The token **expires after `QR_TTL_SECONDS` (30s)**. A photo forwarded to a friend who isn't
- The token is bound to one trip, and check-in is refused unless that trip is `in_progress`.
- Tokens are HMAC-signed with the server secret and carry `typ=board`, so login tokens and forged
- One boarding per student per trip (unique constraint → `already_boarded`).
- A student boarding a bus that isn't their allocated route **is recorded, not refused**
- QR errors return **422, not 401**, so the student's app doesn't think the *login* expired.
- P1 hook: compare the student's device location with the bus position once GPS exists.
## Data model
## API
## Events
## Attendance safety net
## Public service API
## Frontend screens
## How to test

## 6. Backend functions — what each one means


### `router.py`
- `async trip_qr(trip_id, p, session)` — Creates or validates the rotating QR code used for boarding.
  - Code comment/docstring: Driver app polls this every `ttl_seconds` and renders `token` as a QR code.
- `async check_in(body, p, session)` — Validates a condition and returns the corresponding result.
- `async manual_board(trip_id, body, p, session)` — Records that a student has boarded the bus.
- `async roster(trip_id, p, session)` — Internal helper used by this module.
- `async my_attendance(limit, p, session)` — Ends/completes a trip or workflow.

### `service.py`
- `_ensure_operator(trip, p)` — Internal helper used by this module.
- `async boarded_count(session, trip_id)` — Records that a student has boarded the bus.
- `async boarded_counts(session, trip_ids)` — Records that a student has boarded the bus.
  - Code comment/docstring: boarded_count for several trips in one query (trips nobody boarded are left out).
- `async boarded_student_ids(session, trip_id)` — Builds or handles the student-facing workflow.
- `async issue_trip_qr(session, trip_id, p)` — Creates or validates the rotating QR code used for boarding.
- `_decode_qr(token)` — Creates or validates the rotating QR code used for boarding.
- `async _board(session, trip, student, method)` — Records that a student has boarded the bus.
- `async check_in(session, token, p)` — Validates a condition and returns the corresponding result.
- `async manual_board(session, trip_id, p)` — Records that a student has boarded the bus.
- `async roster(session, trip_id, p)` — Internal helper used by this module.
- `async finalize_attendance(session, trip_id)` — Ends/completes a trip or workflow.
  - Code comment/docstring: Write attendance for a finished trip. Idempotent: returns None if already done.
- `_finalized(trip_id)` — Internal helper used by this module.
  - Code comment/docstring: The AttendanceFinalized event exists for the trip (also covers trips with no riders).
- `async reconcile_attendance(session)` — Ends/completes a trip or workflow.
  - Code comment/docstring: Finalise attendance for recent finished trips whose TripEnded/TripCancelled handler never ran
(the server restarted, or the handler failed). Returns the trip ids it finalised.
- `async student_attendance(session, student_id, limit)` — Builds or handles the student-facing workflow.
- `async student_boardings_on(session, student_id, trip_ids)` — Builds or handles the student-facing workflow.

## 7. Frontend functions/screens — what each one means


### `frontend/lib/modules/boarding/data/boarding_api.dart`
- `TripQr()` — Creates or validates the rotating QR code used for boarding.
- `BoardingReceipt()` — Records that a student has boarded the bus.
- `RosterEntry()` — Internal helper used by this module.
- `Roster()` — Internal helper used by this module.
- `AttendanceItem()` — Ends/completes a trip or workflow.
- `asList()` — Returns a list of records for the requested scope.
- `qr()` — Creates or validates the rotating QR code used for boarding.
- `checkIn()` — Validates a condition and returns the corresponding result.
- `manual()` — Internal helper used by this module.

### `frontend/lib/modules/boarding/screens/driver_qr_screen.dart`
- `DriverQrScreen()` — Builds or handles the driver-facing workflow.
- `initState()` — Internal helper used by this module.
- `dispose()` — Internal helper used by this module.
- `_load()` — Internal helper used by this module.
- `Spacer()` — Internal helper used by this module.
- `setState()` — Internal helper used by this module.
- `listenLive()` — Returns a list of records for the requested scope.

### `frontend/lib/modules/boarding/screens/driver_roster_screen.dart`
- `DriverRosterScreen()` — Builds or handles the driver-facing workflow.
- `_RiderRow()` — Internal helper used by this module.
- `_board()` — Records that a student has boarded the bus.
- `listenLive()` — Returns a list of records for the requested scope.

### `frontend/lib/modules/boarding/screens/student_attendance_screen.dart`
- `StudentAttendanceScreen()` — Builds or handles the student-facing workflow.
- `_Row()` — Internal helper used by this module.
- `final()` — Internal helper used by this module.

### `frontend/lib/modules/boarding/screens/student_scan_screen.dart`
- `StudentScanScreen()` — Builds or handles the student-facing workflow.
- `dispose()` — Internal helper used by this module.
- `_submit()` — Internal helper used by this module.
- `_retry()` — Internal helper used by this module.
- `_Viewfinder()` — Internal helper used by this module.
- `_Success()` — Internal helper used by this module.
- `setState()` — Internal helper used by this module.
- `SignHeader()` — Internal helper used by this module.

## 8. Module flow

```text
Driver starts trip
→ rotating QR is generated
→ student scans
→ token is verified
→ allocation is checked/flagged
→ boarding record is created
→ StudentBoarded event
```

## 9. Main code lines and their meaning


### `backend/app/modules/boarding/router.py`
- **Line 1:** `from fastapi import APIRouter, Depends, Query`
  - Meaning: Imports a library/module used by this file.
- **Line 2:** `from sqlalchemy.ext.asyncio import AsyncSession`
  - Meaning: Imports a library/module used by this file.
- **Line 4:** `from app.core.db import get_session`
  - Meaning: Imports a library/module used by this file.
- **Line 5:** `from app.core.deps import Principal, require_roles`
  - Meaning: Imports a library/module used by this file.
- **Line 6:** `from app.core.roles import Role`
  - Meaning: Imports a library/module used by this file.
- **Line 7:** `from app.modules.boarding import service`
  - Meaning: Imports a library/module used by this file.
- **Line 8:** `from app.modules.boarding.schemas import (`
  - Meaning: Imports a library/module used by this file.

### `backend/app/modules/boarding/service.py`
- **Line 11:** `import secrets`
  - Meaning: Imports a library/module used by this file.
- **Line 12:** `from datetime import date, timedelta`
  - Meaning: Imports a library/module used by this file.
- **Line 14:** `from sqlalchemy import exists, func, or_, select`
  - Meaning: Imports a library/module used by this file.
- **Line 15:** `from sqlalchemy.exc import IntegrityError`
  - Meaning: Imports a library/module used by this file.
- **Line 16:** `from sqlalchemy.ext.asyncio import AsyncSession`
  - Meaning: Imports a library/module used by this file.
- **Line 18:** `from app.core import events`
  - Meaning: Imports a library/module used by this file.
- **Line 19:** `from app.core.config import settings`
  - Meaning: Imports a library/module used by this file.

### `backend/app/modules/boarding/__init__.py`
- **Line 3:** `from fastapi import FastAPI`
  - Meaning: Imports a library/module used by this file.
- **Line 5:** `from app.core import events, tasks`
  - Meaning: Imports a library/module used by this file.
- **Line 6:** `from app.core.config import settings`
  - Meaning: Imports a library/module used by this file.
- **Line 7:** `from app.core.db import SessionLocal`
  - Meaning: Imports a library/module used by this file.
- **Line 8:** `from app.core.realtime import hub`
  - Meaning: Imports a library/module used by this file.
- **Line 9:** `from app.modules.boarding import service`
  - Meaning: Imports a library/module used by this file.
- **Line 10:** `from app.modules.boarding.router import router`
  - Meaning: Imports a library/module used by this file.

### `frontend/lib/modules/boarding/data/boarding_api.dart`
- **Line 1:** `import 'package:flutter_riverpod/flutter_riverpod.dart';`
  - Meaning: Imports a library/module used by this file.
- **Line 3:** `import '../../../core/api/api_client.dart';`
  - Meaning: Imports a library/module used by this file.
- **Line 4:** `import '../../../core/format.dart';`
  - Meaning: Imports a library/module used by this file.
- **Line 6:** `class TripQr {`
  - Meaning: Defines a class that groups related data or behavior.
- **Line 7:** `const TripQr({`
  - Meaning: Important setup or business-logic line used by this module.
- **Line 17:** `final String token;`
  - Meaning: Important setup or business-logic line used by this module.
- **Line 18:** `final DateTime issuedAt;`
  - Meaning: Important setup or business-logic line used by this module.

## 10. Beginner takeaway

Remember this module as: **boarding = the part of the system responsible for its own business job**. The router receives the request, the service does the work, the database stores the result, and domain events let other modules react without tightly coupling everything together.