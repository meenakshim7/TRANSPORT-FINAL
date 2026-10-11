# dashboard Module — Simple Explanation

## 1. What this module does

One call per role that returns everything its home screen shows, plus live "refresh" pushes.

## 2. Where the code is

- Backend: `backend/app/modules/dashboard/`
- Frontend: `frontend/lib/modules/dashboard/`

## 3. Backend files

- `backend/app/modules/dashboard/router.py`
- `backend/app/modules/dashboard/schemas.py`
- `backend/app/modules/dashboard/service.py`

## 4. Frontend files

- `frontend/lib/modules/dashboard/data/dashboard_api.dart`
- `frontend/lib/modules/dashboard/screens/admin_board_screen.dart`
- `frontend/lib/modules/dashboard/screens/student_home_screen.dart`

## 5. What the existing project documentation says

## API
## Live updates
- to all **admin** sockets
- to topic **`route:{id}`** (student apps subscribe to their route), **without** `student_id`,
- to the trip's **driver**
## Frontend screens
## How to test

## 6. Backend functions — what each one means


### `router.py`
- `async admin(_, session)` — Builds the admin-facing data/view.
- `async driver(p, session)` — Builds or handles the driver-facing workflow.
- `async student(p, session)` — Builds or handles the student-facing workflow.

### `service.py`
- `async _delays(session, trips)` — Calculates, records, or exposes delay information.
  - Code comment/docstring: Effective delay per trip: the worse of the last check-in and the last alert.
- `_stops_done(d)` — Internal helper used by this module.
- `async admin(session)` — Builds the admin-facing data/view.
- `async driver(session, driver_id)` — Builds or handles the driver-facing workflow.
- `async student(session, student_id)` — Builds or handles the student-facing workflow.

## 7. Frontend functions/screens — what each one means


### `frontend/lib/modules/dashboard/data/dashboard_api.dart`
- `BoardRow()` — Records that a student has boarded the bus.
- `AlertEvent()` — Reads, filters, or handles an operational event.
- `RouteUtil()` — Works with route/network information.
- `AdminDashboard()` — Builds the admin-facing data/view.
- `DriverTrip()` — Builds or handles the driver-facing workflow.
- `DriverDashboard()` — Builds or handles the driver-facing workflow.
- `StudentTrip()` — Builds or handles the student-facing workflow.
- `StudentDashboard()` — Builds or handles the student-facing workflow.

### `frontend/lib/modules/dashboard/screens/admin_board_screen.dart`
- `refreshesBoard()` — Refreshes authentication state or renews an access token.
- `AdminBoardScreen()` — Builds the admin-facing data/view.
- `_scheduleRefresh()` — Refreshes authentication state or renews an access token.
- `initState()` — Internal helper used by this module.
- `dispose()` — Internal helper used by this module.
- `_Board()` — Records that a student has boarded the bus.
- `_row()` — Internal helper used by this module.
- `DepartureRow()` — Internal helper used by this module.
- `_Count()` — Internal helper used by this module.
- `_SidePanel()` — Internal helper used by this module.
- `describeEvent()` — Reads, filters, or handles an operational event.
- `switch()` — Internal helper used by this module.
- `_AlertLine()` — Internal helper used by this module.
- `_TripDialog()` — Internal helper used by this module.
- `_cancel()` — Cancels a trip/workflow and records the cancellation.
- `listenLive()` — Returns a list of records for the requested scope.
- `if()` — Internal helper used by this module.
- `final()` — Internal helper used by this module.

### `frontend/lib/modules/dashboard/screens/student_home_screen.dart`
- `StudentHomeScreen()` — Builds or handles the student-facing workflow.
- `_NotAllocated()` — Internal helper used by this module.
- `SignNotice()` — Internal helper used by this module.
- `_SignOut()` — Internal helper used by this module.
- `_MyLine()` — Internal helper used by this module.
- `_NextBusPanel()` — Works with bus/fleet information.
- `_BusMap()` — Works with bus/fleet information.
- `_LiveLine()` — Internal helper used by this module.
- `_OtherTripRow()` — Internal helper used by this module.
- `listenLive()` — Returns a list of records for the requested scope.
- `final()` — Internal helper used by this module.

## 8. Module flow

```text
Admin/driver/student request
→ combine current information from other services
→ return one role-specific dashboard response
→ WebSocket ops event tells client to refresh
```

## 9. Main code lines and their meaning


### `backend/app/modules/dashboard/router.py`
- **Line 1:** `from fastapi import APIRouter, Depends`
  - Meaning: Imports a library/module used by this file.
- **Line 2:** `from sqlalchemy.ext.asyncio import AsyncSession`
  - Meaning: Imports a library/module used by this file.
- **Line 4:** `from app.core.db import get_session`
  - Meaning: Imports a library/module used by this file.
- **Line 5:** `from app.core.deps import Principal, require_roles`
  - Meaning: Imports a library/module used by this file.
- **Line 6:** `from app.core.roles import Role`
  - Meaning: Imports a library/module used by this file.
- **Line 7:** `from app.modules.dashboard import service`
  - Meaning: Imports a library/module used by this file.
- **Line 8:** `from app.modules.dashboard.schemas import AdminDashboard, DriverDashboard, StudentDashboard`
  - Meaning: Imports a library/module used by this file.

### `backend/app/modules/dashboard/service.py`
- **Line 3:** `from datetime import timedelta`
  - Meaning: Imports a library/module used by this file.
- **Line 5:** `from sqlalchemy.ext.asyncio import AsyncSession`
  - Meaning: Imports a library/module used by this file.
- **Line 7:** `from app.core.config import settings`
  - Meaning: Imports a library/module used by this file.
- **Line 8:** `from app.core.timeutil import today_local`
  - Meaning: Imports a library/module used by this file.
- **Line 9:** `from app.modules.allocation import service as alloc_service`
  - Meaning: Imports a library/module used by this file.
- **Line 10:** `from app.modules.allocation.schemas import MyAllocation`
  - Meaning: Imports a library/module used by this file.
- **Line 11:** `from app.modules.boarding import service as boarding_service`
  - Meaning: Imports a library/module used by this file.

### `backend/app/modules/dashboard/__init__.py`
- **Line 3:** `from fastapi import FastAPI`
  - Meaning: Imports a library/module used by this file.
- **Line 5:** `from app.core import events`
  - Meaning: Imports a library/module used by this file.
- **Line 6:** `from app.core.realtime import hub`
  - Meaning: Imports a library/module used by this file.
- **Line 7:** `from app.core.roles import Role`
  - Meaning: Imports a library/module used by this file.
- **Line 8:** `from app.modules.dashboard.router import router`
  - Meaning: Imports a library/module used by this file.
- **Line 32:** `def register(app: FastAPI) -> None:`
  - Meaning: Connects this module to the event system so other modules can react.

### `frontend/lib/modules/dashboard/data/dashboard_api.dart`
- **Line 1:** `import 'package:flutter_riverpod/flutter_riverpod.dart';`
  - Meaning: Imports a library/module used by this file.
- **Line 3:** `import '../../../core/api/api_client.dart';`
  - Meaning: Imports a library/module used by this file.
- **Line 4:** `import '../../../core/format.dart';`
  - Meaning: Imports a library/module used by this file.
- **Line 5:** `import '../../../design/tokens.dart';`
  - Meaning: Imports a library/module used by this file.
- **Line 6:** `import '../../master_data/data/master_data_api.dart';`
  - Meaning: Imports a library/module used by this file.
- **Line 7:** `import '../../trips/data/trip_models.dart';`
  - Meaning: Imports a library/module used by this file.
- **Line 10:** `class BoardRow {`
  - Meaning: Defines a class that groups related data or behavior.

## 10. Beginner takeaway

Remember this module as: **dashboard = the part of the system responsible for its own business job**. The router receives the request, the service does the work, the database stores the result, and domain events let other modules react without tightly coupling everything together.