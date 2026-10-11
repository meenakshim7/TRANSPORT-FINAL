# capacity Module — Simple Explanation

## 1. What this module does

How full each running bus is, how full each route's allocation is, and alerts when it matters.

## 2. Where the code is

- Backend: `backend/app/modules/capacity/`
- Frontend: `frontend/lib/modules/capacity/`

## 3. Backend files

- `backend/app/modules/capacity/router.py`
- `backend/app/modules/capacity/schemas.py`
- `backend/app/modules/capacity/service.py`

## 4. Frontend files

- `frontend/lib/modules/capacity/widgets/utilization_list.dart`

## 5. What the existing project documentation says

## What it computes
- **Trip occupancy** = boarded students / bus capacity.
- **Route utilisation** = active allocations / route seat capacity (smallest scheduled bus).
## Alerts
## API
## Events
## Public service API
## Frontend
- `SeatBlocks` (`frontend/lib/design/widgets/seat_blocks.dart`) renders occupancy as a top-down
- `UtilizationList` (`frontend/lib/modules/capacity/widgets/utilization_list.dart`): seats allocated
## How to test
## Extension notes (Team B)

## 6. Backend functions — what each one means


### `router.py`
- `async trip_occupancy(trip_id, p, session)` — Internal helper used by this module.
- `async active(_, session)` — Internal helper used by this module.
- `async routes(_, session)` — Works with route/network information.

### `service.py`
- `level_for(count, capacity)` — Internal helper used by this module.
- `_pct(count, capacity)` — Internal helper used by this module.
- `_occupancy(trip, bus, boarded)` — Internal helper used by this module.
- `async trip_occupancy(session, trip)` — Internal helper used by this module.
- `async trip_occupancies(session, trips)` — Internal helper used by this module.
  - Code comment/docstring: trip_occupancy for many trips in two queries (dashboards list every trip of the day).
- `async active_occupancy(session)` — Internal helper used by this module.
- `async route_utilization(session)` — Works with route/network information.
- `async _already_raised(session, event_type, trip_id)` — Internal helper used by this module.
- `async check_trip_capacity(session, trip_id)` — Validates a condition and returns the corresponding result.
  - Code comment/docstring: Raise CapacityWarning / OverCapacity for a trip, at most once each. Returns the event type raised.

The trip row is locked first, so two students boarding at the same moment can't both see
"not raised yet" and raise the alert twice.

## 7. Frontend functions/screens — what each one means


### `frontend/lib/modules/capacity/widgets/utilization_list.dart`
- `UtilizationList()` — Returns a list of records for the requested scope.

## 8. Module flow

```text
StudentBoarded event
→ count boarded students
→ read bus capacity
→ calculate percentage + level
→ raise warning/over-capacity only once per trip
```

## 9. Main code lines and their meaning


### `backend/app/modules/capacity/router.py`
- **Line 1:** `from fastapi import APIRouter, Depends`
  - Meaning: Imports a library/module used by this file.
- **Line 2:** `from sqlalchemy.ext.asyncio import AsyncSession`
  - Meaning: Imports a library/module used by this file.
- **Line 4:** `from app.core.db import get_session`
  - Meaning: Imports a library/module used by this file.
- **Line 5:** `from app.core.deps import Principal, current_principal, require_roles`
  - Meaning: Imports a library/module used by this file.
- **Line 6:** `from app.core.roles import Role`
  - Meaning: Imports a library/module used by this file.
- **Line 7:** `from app.modules.capacity import service`
  - Meaning: Imports a library/module used by this file.
- **Line 8:** `from app.modules.capacity.schemas import RouteUtilization, TripOccupancy`
  - Meaning: Imports a library/module used by this file.

### `backend/app/modules/capacity/service.py`
- **Line 7:** `import math`
  - Meaning: Imports a library/module used by this file.
- **Line 9:** `from sqlalchemy import func, select`
  - Meaning: Imports a library/module used by this file.
- **Line 10:** `from sqlalchemy.ext.asyncio import AsyncSession`
  - Meaning: Imports a library/module used by this file.
- **Line 12:** `from app.core import events`
  - Meaning: Imports a library/module used by this file.
- **Line 13:** `from app.core.config import settings`
  - Meaning: Imports a library/module used by this file.
- **Line 14:** `from app.core.events import DomainEvent`
  - Meaning: Imports a library/module used by this file.
- **Line 15:** `from app.modules.allocation import service as alloc_service`
  - Meaning: Imports a library/module used by this file.

### `backend/app/modules/capacity/__init__.py`
- **Line 3:** `from fastapi import FastAPI`
  - Meaning: Imports a library/module used by this file.
- **Line 5:** `from app.core import events`
  - Meaning: Imports a library/module used by this file.
- **Line 6:** `from app.core.db import SessionLocal`
  - Meaning: Imports a library/module used by this file.
- **Line 7:** `from app.modules.capacity import service`
  - Meaning: Imports a library/module used by this file.
- **Line 8:** `from app.modules.capacity.router import router`
  - Meaning: Imports a library/module used by this file.
- **Line 14:** `await session.commit()`
  - Meaning: Uses the database session asynchronously.
- **Line 17:** `def register(app: FastAPI) -> None:`
  - Meaning: Connects this module to the event system so other modules can react.

## 10. Beginner takeaway

Remember this module as: **capacity = the part of the system responsible for its own business job**. The router receives the request, the service does the work, the database stores the result, and domain events let other modules react without tightly coupling everything together.