# delay_monitor Module — Simple Explanation

## 1. What this module does

Notices when a bus is late and announces it (once), so notifications can tell the right people.

## 2. Where the code is

- Backend: `backend/app/modules/delay_monitor/`
- Frontend: `frontend/lib/modules/delay_monitor/`

## 3. Backend files

- `backend/app/modules/delay_monitor/models.py`
- `backend/app/modules/delay_monitor/router.py`
- `backend/app/modules/delay_monitor/schemas.py`
- `backend/app/modules/delay_monitor/service.py`

## 4. Frontend files

- `frontend/lib/modules/delay_monitor/data/delay_api.dart`
- `frontend/lib/modules/delay_monitor/widgets/report_delay_sheet.dart`

## 5. What the existing project documentation says

## Detection sources (P0: no GPS)
## Alerting rules
- Alerts go out in **steps** (`DELAY_ALERT_STEPS`, default 5, 15, 30, 60 min late): `TripDelayed`
- Steps above `DELAY_STUDENT_ALERT_MAX_MIN` (30) go to the transport office only
- Raise `TripDelayResolved` when a stop check-in comes in under the threshold after a delay.
- Manual reports always raise (a human decided it matters).
- The payload includes `affected_stops`: every stop not yet reached, with `scheduled_at` and
## Data model
## API
## Events
## Background job
## Public service API
## Frontend
## How to test
## Extension notes (Team B: this is your main entry point)
- **ETA prediction / GPS:** compute a predicted delay and call `service.evaluate(trip, predicted,
- **Route deviation:** publish a new `RouteDeviation` event in the same style.
- **Agent:** subscribe to `TripDelayed`. It already carries the affected stops, and

## 6. Backend functions — what each one means


### `router.py`
- `async report_delay(trip_id, body, p, session)` — Creates or processes a delay/problem report.
- `async trip_delays(trip_id, p, session)` — Calculates, records, or exposes delay information.
- `async delays(service_date, _, session)` — Calculates, records, or exposes delay information.
- `async run_watcher(_, session)` — Internal helper used by this module.
  - Code comment/docstring: Run one watcher pass now (the background job does this every DELAY_WATCH_INTERVAL_SECONDS).

### `service.py`
- `async latest_report(session, trip_id)` — Creates or processes a delay/problem report.
- `async current_delays(session, trip_ids)` — Calculates, records, or exposes delay information.
  - Code comment/docstring: Latest alerted delay per trip (0 once recovered).
- `affected_stops(trip, delay_min)` — Internal helper used by this module.
  - Code comment/docstring: Stops the bus has not reached yet, with their new expected time.
- `async evaluate(session, trip, observed_delay, source)` — Internal helper used by this module.
  - Code comment/docstring: Apply the alerting rules. Returns the report created, or None if nothing was raised.
- `alert_step(delay_min)` — Internal helper used by this module.
  - Code comment/docstring: The highest alert step this delay has reached (0 = under the first step).
- `async _alerted_delay(session, trip_id)` — Calculates, records, or exposes delay information.
  - Code comment/docstring: The largest delay announced for the trip since it was last back on schedule.
- `async report_manual(session, trip_id, p, delay_min, reason)` — Creates or processes a delay/problem report.
- `async watch_once(session)` — Internal helper used by this module.
  - Code comment/docstring: One pass of the background watcher. Returns how many alerts were raised.
- `async list_reports(session)` — Returns a list of records for the requested scope.

## 7. Frontend functions/screens — what each one means


### `frontend/lib/modules/delay_monitor/data/delay_api.dart`
- `report()` — Creates or processes a delay/problem report.

### `frontend/lib/modules/delay_monitor/widgets/report_delay_sheet.dart`
- `showReportDelaySheet()` — Creates or processes a delay/problem report.
- `_ReportDelaySheet()` — Creates or processes a delay/problem report.
- `_send()` — Ends/completes a trip or workflow.
- `_choice()` — Internal helper used by this module.
- `setState()` — Internal helper used by this module.
- `TextField()` — Internal helper used by this module.

## 8. Module flow

```text
TripStarted / StopArrived / watcher / manual report
→ calculate delay
→ compare against alert steps
→ publish TripDelayed or TripDelayResolved
→ notifications targets the affected users
```

## 9. Main code lines and their meaning


### `backend/app/modules/delay_monitor/router.py`
- **Line 1:** `from datetime import date`
  - Meaning: Imports a library/module used by this file.
- **Line 3:** `from fastapi import APIRouter, Depends`
  - Meaning: Imports a library/module used by this file.
- **Line 4:** `from sqlalchemy.ext.asyncio import AsyncSession`
  - Meaning: Imports a library/module used by this file.
- **Line 6:** `from app.core.db import get_session`
  - Meaning: Imports a library/module used by this file.
- **Line 7:** `from app.core.deps import Principal, current_principal, require_roles`
  - Meaning: Imports a library/module used by this file.
- **Line 8:** `from app.core.roles import Role`
  - Meaning: Imports a library/module used by this file.
- **Line 9:** `from app.modules.delay_monitor import service`
  - Meaning: Imports a library/module used by this file.

### `backend/app/modules/delay_monitor/service.py`
- **Line 25:** `from datetime import date, timedelta`
  - Meaning: Imports a library/module used by this file.
- **Line 27:** `from sqlalchemy import func, select`
  - Meaning: Imports a library/module used by this file.
- **Line 28:** `from sqlalchemy.ext.asyncio import AsyncSession`
  - Meaning: Imports a library/module used by this file.
- **Line 30:** `from app.core import events`
  - Meaning: Imports a library/module used by this file.
- **Line 31:** `from app.core.config import settings`
  - Meaning: Imports a library/module used by this file.
- **Line 32:** `from app.core.deps import Principal`
  - Meaning: Imports a library/module used by this file.
- **Line 33:** `from app.core.errors import Forbidden, InvalidState`
  - Meaning: Imports a library/module used by this file.

### `backend/app/modules/delay_monitor/__init__.py`
- **Line 4:** `from fastapi import FastAPI`
  - Meaning: Imports a library/module used by this file.
- **Line 6:** `from app.core import events, tasks`
  - Meaning: Imports a library/module used by this file.
- **Line 7:** `from app.core.config import settings`
  - Meaning: Imports a library/module used by this file.
- **Line 8:** `from app.core.db import SessionLocal`
  - Meaning: Imports a library/module used by this file.
- **Line 9:** `from app.modules.delay_monitor import service`
  - Meaning: Imports a library/module used by this file.
- **Line 10:** `from app.modules.delay_monitor.models import DelaySource`
  - Meaning: Imports a library/module used by this file.
- **Line 11:** `from app.modules.delay_monitor.router import router`
  - Meaning: Imports a library/module used by this file.

### `frontend/lib/modules/delay_monitor/data/delay_api.dart`
- **Line 1:** `import 'package:flutter_riverpod/flutter_riverpod.dart';`
  - Meaning: Imports a library/module used by this file.
- **Line 3:** `import '../../../core/api/api_client.dart';`
  - Meaning: Imports a library/module used by this file.
- **Line 5:** `class DelayActions {`
  - Meaning: Defines a class that groups related data or behavior.
- **Line 8:** `final ApiClient _api;`
  - Meaning: Important setup or business-logic line used by this module.
- **Line 15:** `final delayActionsProvider = Provider((ref) => DelayActions(ref.read(apiProvider)));`
  - Meaning: Important setup or business-logic line used by this module.

## 10. Beginner takeaway

Remember this module as: **delay_monitor = the part of the system responsible for its own business job**. The router receives the request, the service does the work, the database stores the result, and domain events let other modules react without tightly coupling everything together.