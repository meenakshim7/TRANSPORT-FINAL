# history Module — Simple Explanation

## 1. What this module does

Read-only views: the event timeline, per-trip reports and attendance reports (with CSV export).

## 2. Where the code is

- Backend: `backend/app/modules/history/`
- Frontend: `frontend/lib/modules/history/`

## 3. Backend files

- `backend/app/modules/history/router.py`
- `backend/app/modules/history/schemas.py`
- `backend/app/modules/history/service.py`

## 4. Frontend files

- `frontend/lib/modules/history/data/history_api.dart`
- `frontend/lib/modules/history/screens/admin_history_screen.dart`

## 5. What the existing project documentation says

## Source of truth
## API
## Frontend screens
## How to test
## Extension notes (Team B)

## 6. Backend functions — what each one means


### `router.py`
- `async events(type, aggregate_type, aggregate_id, trip_id, route_id, date_from, date_to, limit, offset, _, session)` — Reads, filters, or handles an operational event.
- `async timeline(trip_id, _, session)` — Builds the chronological story of a trip.
- `async trips(date_from, date_to, route_id, _, session)` — Internal helper used by this module.
- `async attendance(date_from, date_to, route_id, student_id, format, _, session)` — Ends/completes a trip or workflow.

### `service.py`
- `_day_bounds(d_from, d_to)` — Internal helper used by this module.
- `async _with_actor_names(session, rows)` — Internal helper used by this module.
- `async events(session)` — Reads, filters, or handles an operational event.
- `async trip_timeline(session, trip_id)` — Builds the chronological story of a trip.
- `async trip_reports(session)` — Creates or processes a delay/problem report.
- `async attendance(session)` — Ends/completes a trip or workflow.
- `attendance_csv(rows)` — Ends/completes a trip or workflow.

## 7. Frontend functions/screens — what each one means


### `frontend/lib/modules/history/data/history_api.dart`
- `TripReport()` — Creates or processes a delay/problem report.
- `AttendanceRow()` — Ends/completes a trip or workflow.
- `_d()` — Internal helper used by this module.
- `asList()` — Returns a list of records for the requested scope.

### `frontend/lib/modules/history/screens/admin_history_screen.dart`
- `AdminHistoryScreen()` — Builds the admin-facing data/view.
- `Spacer()` — Internal helper used by this module.
- `_Tab()` — Internal helper used by this module.
- `_Trips()` — Internal helper used by this module.
- `_Attendance()` — Ends/completes a trip or workflow.
- `final()` — Internal helper used by this module.
- `if()` — Internal helper used by this module.

## 8. Module flow

```text
Domain events + trip/boarding/attendance/delay data
→ filter and combine records
→ timeline/report/attendance output
→ admin UI displays it or exports CSV
```

## 9. Main code lines and their meaning


### `backend/app/modules/history/router.py`
- **Line 1:** `from datetime import date`
  - Meaning: Imports a library/module used by this file.
- **Line 3:** `from fastapi import APIRouter, Depends, Query`
  - Meaning: Imports a library/module used by this file.
- **Line 4:** `from fastapi.responses import Response`
  - Meaning: Imports a library/module used by this file.
- **Line 5:** `from sqlalchemy.ext.asyncio import AsyncSession`
  - Meaning: Imports a library/module used by this file.
- **Line 7:** `from app.core.db import get_session`
  - Meaning: Imports a library/module used by this file.
- **Line 8:** `from app.core.deps import Principal, require_roles`
  - Meaning: Imports a library/module used by this file.
- **Line 9:** `from app.core.roles import Role`
  - Meaning: Imports a library/module used by this file.

### `backend/app/modules/history/service.py`
- **Line 3:** `import csv`
  - Meaning: Imports a library/module used by this file.
- **Line 4:** `import io`
  - Meaning: Imports a library/module used by this file.
- **Line 5:** `from datetime import date, datetime, time, timedelta`
  - Meaning: Imports a library/module used by this file.
- **Line 7:** `from sqlalchemy import and_, case, func, or_, select`
  - Meaning: Imports a library/module used by this file.
- **Line 8:** `from sqlalchemy.ext.asyncio import AsyncSession`
  - Meaning: Imports a library/module used by this file.
- **Line 10:** `from app.core.events import DomainEvent, payload_text`
  - Meaning: Imports a library/module used by this file.
- **Line 11:** `from app.core.timeutil import local_tz`
  - Meaning: Imports a library/module used by this file.

### `backend/app/modules/history/__init__.py`
- **Line 3:** `from app.modules.history.router import router`
  - Meaning: Imports a library/module used by this file.

### `frontend/lib/modules/history/data/history_api.dart`
- **Line 1:** `import 'package:flutter_riverpod/flutter_riverpod.dart';`
  - Meaning: Imports a library/module used by this file.
- **Line 3:** `import '../../../core/api/api_client.dart';`
  - Meaning: Imports a library/module used by this file.
- **Line 4:** `import '../../../core/format.dart';`
  - Meaning: Imports a library/module used by this file.
- **Line 5:** `import '../../dashboard/data/dashboard_api.dart';`
  - Meaning: Imports a library/module used by this file.
- **Line 7:** `class TripReport {`
  - Meaning: Defines a class that groups related data or behavior.
- **Line 8:** `const TripReport({`
  - Meaning: Important setup or business-logic line used by this module.
- **Line 27:** `final int tripId;`
  - Meaning: Important setup or business-logic line used by this module.

## 10. Beginner takeaway

Remember this module as: **history = the part of the system responsible for its own business job**. The router receives the request, the service does the work, the database stores the result, and domain events let other modules react without tightly coupling everything together.