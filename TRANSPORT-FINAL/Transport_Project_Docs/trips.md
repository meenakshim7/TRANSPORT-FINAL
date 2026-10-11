# trips Module — Simple Explanation

## 1. What this module does

When buses run, and the driver's **start → arrive at stop → end** workflow.

## 2. Where the code is

- Backend: `backend/app/modules/trips/`
- Frontend: `frontend/lib/modules/trips/`

## 3. Backend files

- `backend/app/modules/trips/models.py`
- `backend/app/modules/trips/router.py`
- `backend/app/modules/trips/schemas.py`
- `backend/app/modules/trips/service.py`

## 4. Frontend files

- `frontend/lib/modules/trips/data/trip_models.dart`
- `frontend/lib/modules/trips/data/trips_api.dart`
- `frontend/lib/modules/trips/screens/admin_schedules_screen.dart`
- `frontend/lib/modules/trips/screens/driver_home_screen.dart`
- `frontend/lib/modules/trips/screens/driver_run_screen.dart`

## 5. What the existing project documentation says

## Concepts
- **Schedule**: recurring service: route + bus + driver + direction + departure time + weekdays.
- **Trip**: one run of a schedule on a service date. Generated from schedules (idempotent).
- **Trip stop event**: planned vs actual time at each stop, in the order the bus visits them.
### Stop times
### State machine
- **start**: assigned driver (or admin). Bus must be `active`; neither driver nor bus may have
- **arrive** (`/stops/{sequence}/arrive`, or automatically by the tracking module when GPS puts the bus
- **end**: marks the terminus reached (if not already) and completes the trip.
- **auto-close**: a trip still `in_progress` after its service day, with no start/stop activity for
## Concurrency
## Data model
## API
## Events
## Background jobs
## Public service API
## Frontend screens
## How to test
## Extension notes (Team B)
- **GPS:** done in the tracking module, which calls `service.arrive_at_stop(observed_at=…)`.
- **Temporary route reassignment:** `PATCH /schedules/{id}` changes bus/driver for future trips.

## 6. Backend functions — what each one means


### `router.py`
- `async list_schedules(route_id, driver_id, _, session)` — Returns a list of records for the requested scope.
- `async create_schedule(body, p, session)` — Creates a new record/resource.
- `async update_schedule(schedule_id, body, p, session)` — Updates an existing record/resource.
- `async generate(body, p, session)` — Internal helper used by this module.
- `async list_trips(service_date, status, route_id, driver_id, _, session)` — Returns a list of records for the requested scope.
- `async my_trips(service_date, p, session)` — Internal helper used by this module.
- `async get_trip(trip_id, p, session)` — Fetches one record or a computed view.
- `async start_trip(trip_id, body, p, session)` — Starts a trip/workflow after validating the current state.
- `async arrive(trip_id, sequence, body, p, session)` — Records arrival at a stop and updates trip timing.
- `async end_trip(trip_id, p, session)` — Ends/completes a trip or workflow.
- `async cancel_trip(trip_id, body, p, session)` — Cancels a trip/workflow and records the cancellation.

### `service.py`
- `async list_schedules(session)` — Returns a list of records for the requested scope.
- `async get_schedule(session, schedule_id)` — Fetches one record or a computed view.
- `async _validate_schedule_refs(session)` — Internal helper used by this module.
- `async create_schedule(session, data)` — Creates a new record/resource.
- `async update_schedule(session, schedule_id, data)` — Updates an existing record/resource.
- `async _apply_schedule_to_trips(session, s, changes)` — Internal helper used by this module.
  - Code comment/docstring: Carry a schedule edit over to its trips that haven't started yet (today onwards), so the
day's run, its stop times and the delay watcher follow the new schedule.

Deactivating a schedule only stops future trips being generated: a trip already created
still runs (that is how one-off runs are made), and an admin cancels it explicitly if not.
- `async reassign_bus_driver(session, bus_id, driver_id)` — Builds or handles the driver-facing workflow.
  - Code comment/docstring: Hand a bus's active schedules and its not-yet-started trips (today onwards) to a new driver.
Called when an admin assigns the bus's driver. Running and finished trips keep their driver.
- `_stop_plan(route, direction, departure)` — Internal helper used by this module.
  - Code comment/docstring: Build the ordered stop events for one trip. Sequence 1 is the departure point.
- `async generate_trips(session, service_date)` — Internal helper used by this module.
  - Code comment/docstring: Idempotently create the day's trips from active schedules.

A schedule whose bus is in maintenance or retired gets no trip (it could never start, and the
delay watcher would report it all morning). The transport office is told once per day via
`ScheduleSkipped`; once the bus is back or the schedule gets another bus, the next pass
creates the trip.
- `async _report_skipped(session, s, route, bus, service_date)` — Creates or processes a delay/problem report.
  - Code comment/docstring: Publish ScheduleSkipped once per schedule and day (the generator runs every 15 minutes).
- `async get_trip(session, trip_id)` — Fetches one record or a computed view.
- `async get_trip_for_update(session, trip_id)` — Updates an existing record/resource.
  - Code comment/docstring: The trip, freshly read and locked (SELECT ... FOR UPDATE) until the transaction ends.

Every state change of a trip (start, arrive, end, cancel, GPS ingest) goes through this, so a
double tap or a tap racing a GPS arrival runs one after the other and the second sees the
first's result instead of repeating it.
- `async can_view_trip(session, p, trip)` — Internal helper used by this module.
  - Code comment/docstring: Admins see every trip, drivers their own, students the trips on their allocated route.
- `async ensure_can_view(session, p, trip)` — Internal helper used by this module.
- `async list_trips(session)` — Returns a list of records for the requested scope.
- `async active_trips(session)` — Internal helper used by this module.
- `async trips_for_route_on(session, route_id, service_date)` — Works with route/network information.
- `async driver_trips(session, driver_id, service_date)` — Builds or handles the driver-facing workflow.
- `next_stop(trip)` — Internal helper used by this module.
  - Code comment/docstring: First stop the bus hasn't reached yet (skipped stops before a reached one don't count).
- `stops_after(trip, sequence)` — Internal helper used by this module.
- `async route_seat_capacity(session, route_id)` — Works with route/network information.
  - Code comment/docstring: Seats every trip on the route can offer = smallest bus among its active schedules.
None when the route has no active schedule yet (capacity unknown).
- `async route_seat_capacities(session)` — Works with route/network information.
  - Code comment/docstring: route_seat_capacity for every route that has an active schedule, in one query.
- `async trip_details(session, trips)` — Internal helper used by this module.
- `async trip_detail(session, trip)` — Internal helper used by this module.
- `_ensure_operator(trip, p)` — Internal helper used by this module.
- `_effective_time(requested, p)` — Internal helper used by this module.
- `_payload(trip)` — Internal helper used by this module.
- `async start_trip(session, trip_id, p)` — Starts a trip/workflow after validating the current state.
- `async arrive_at_stop(session, trip_id, sequence, p)` — Records arrival at a stop and updates trip timing.
  - Code comment/docstring: Check the bus in at a stop.

`arrived_at` is the simulation override (admins only). `observed_at` is for in-process callers
that detected the arrival themselves (tracking's GPS geofence): the time of the fix, clamped
between the trip start and now. The router never passes it.
- `async end_trip(session, trip_id, p)` — Ends/completes a trip or workflow.
- `_last_activity(trip)` — Internal helper used by this module.
- `async close_stale_trips(session)` — Internal helper used by this module.
  - Code comment/docstring: Complete trips left in progress after their service day (the driver never tapped End).

A trip is stale once its service date is past and it has had no start/stop activity for
`stale_trip_grace_hours`, so a late run that crosses midnight is left alone. It ends at its
last activity, and unreached stops (the terminus included) stay unreached rather than
being marked arrived.
- `async cancel_trip(session, trip_id, reason, p)` — Cancels a trip/workflow and records the cancellation.
- `async _cancel(session, trip, reason)` — Cancels a trip/workflow and records the cancellation.

## 7. Frontend functions/screens — what each one means


### `frontend/lib/modules/trips/data/trip_models.dart`
- `RouteRef()` — Works with route/network information.
- `StopEvent()` — Reads, filters, or handles an operational event.
- `TripDetail()` — Internal helper used by this module.

### `frontend/lib/modules/trips/data/trips_api.dart`
- `Schedule()` — Internal helper used by this module.
- `asList()` — Returns a list of records for the requested scope.
- `start()` — Starts a trip/workflow after validating the current state.
- `arrive()` — Records arrival at a stop and updates trip timing.
- `end()` — Ends/completes a trip or workflow.
- `cancel()` — Cancels a trip/workflow and records the cancellation.
- `generateToday()` — Internal helper used by this module.
- `createSchedule()` — Creates a new record/resource.
- `updateSchedule()` — Updates an existing record/resource.

### `frontend/lib/modules/trips/screens/admin_schedules_screen.dart`
- `daysLabel()` — Internal helper used by this module.
- `AdminSchedulesScreen()` — Builds the admin-facing data/view.
- `_ScheduleDialog()` — Internal helper used by this module.
- `_save()` — Internal helper used by this module.
- `two()` — Internal helper used by this module.
- `SignHeader()` — Internal helper used by this module.
- `if()` — Internal helper used by this module.
- `setState()` — Internal helper used by this module.
- `for()` — Internal helper used by this module.

### `frontend/lib/modules/trips/screens/driver_home_screen.dart`
- `DriverHomeScreen()` — Builds or handles the driver-facing workflow.
- `_RunBlock()` — Internal helper used by this module.
- `_start()` — Starts a trip/workflow after validating the current state.
- `SignButton()` — Internal helper used by this module.
- `SignHeader()` — Internal helper used by this module.

### `frontend/lib/modules/trips/screens/driver_run_screen.dart`
- `DriverRunScreen()` — Builds or handles the driver-facing workflow.
- `_refresh()` — Refreshes authentication state or renews an access token.
- `_arrive()` — Records arrival at a stop and updates trip timing.
- `_end()` — Ends/completes a trip or workflow.
- `_toast()` — Internal helper used by this module.
- `SignButton()` — Internal helper used by this module.
- `_StatusStrip()` — Internal helper used by this module.
- `_DriverMap()` — Builds or handles the driver-facing workflow.
- `listenLive()` — Returns a list of records for the requested scope.
- `SignHeader()` — Internal helper used by this module.

## 8. Module flow

```text
Schedule → generate today's trip
→ driver starts
→ stop arrivals update actual times
→ delay module reacts
→ driver ends/cancels
→ attendance is finalised
```

## 9. Main code lines and their meaning


### `backend/app/modules/trips/router.py`
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
- **Line 9:** `from app.modules.trips import service`
  - Meaning: Imports a library/module used by this file.

### `backend/app/modules/trips/service.py`
- **Line 8:** `from datetime import date, datetime, timedelta`
  - Meaning: Imports a library/module used by this file.
- **Line 10:** `from sqlalchemy import func, select`
  - Meaning: Imports a library/module used by this file.
- **Line 11:** `from sqlalchemy.exc import IntegrityError`
  - Meaning: Imports a library/module used by this file.
- **Line 12:** `from sqlalchemy.ext.asyncio import AsyncSession`
  - Meaning: Imports a library/module used by this file.
- **Line 14:** `from app.core import events`
  - Meaning: Imports a library/module used by this file.
- **Line 15:** `from app.core.config import settings`
  - Meaning: Imports a library/module used by this file.
- **Line 16:** `from app.core.db import flush_or_conflict`
  - Meaning: Imports a library/module used by this file.

### `backend/app/modules/trips/__init__.py`
- **Line 3:** `from fastapi import FastAPI`
  - Meaning: Imports a library/module used by this file.
- **Line 5:** `from app.core import events, realtime, tasks`
  - Meaning: Imports a library/module used by this file.
- **Line 6:** `from app.core.config import settings`
  - Meaning: Imports a library/module used by this file.
- **Line 7:** `from app.core.db import SessionLocal`
  - Meaning: Imports a library/module used by this file.
- **Line 8:** `from app.core.deps import Principal`
  - Meaning: Imports a library/module used by this file.
- **Line 9:** `from app.core.errors import NotFound`
  - Meaning: Imports a library/module used by this file.
- **Line 10:** `from app.modules.trips.router import router`
  - Meaning: Imports a library/module used by this file.

### `frontend/lib/modules/trips/data/trip_models.dart`
- **Line 1:** `import 'dart:ui';`
  - Meaning: Imports a library/module used by this file.
- **Line 3:** `import '../../../core/api/api_client.dart';`
  - Meaning: Imports a library/module used by this file.
- **Line 4:** `import '../../../core/format.dart';`
  - Meaning: Imports a library/module used by this file.
- **Line 5:** `import '../../../design/tokens.dart';`
  - Meaning: Imports a library/module used by this file.
- **Line 7:** `class RouteRef {`
  - Meaning: Defines a class that groups related data or behavior.
- **Line 8:** `const RouteRef({required this.id, required this.code, required this.name, required this.color});`
  - Meaning: Important setup or business-logic line used by this module.
- **Line 10:** `final int id;`
  - Meaning: Important setup or business-logic line used by this module.

## 10. Beginner takeaway

Remember this module as: **trips = the part of the system responsible for its own business job**. The router receives the request, the service does the work, the database stores the result, and domain events let other modules react without tightly coupling everything together.