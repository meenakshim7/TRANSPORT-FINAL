# tracking Module — Simple Explanation

## 1. What this module does

Where each bus is right now. Checks the bus in at stops by GPS and tells riders when it's 2 km away.

## 2. Where the code is

- Backend: `backend/app/modules/tracking/`
- Frontend: `frontend/lib/modules/tracking/`

## 3. Backend files

- `backend/app/modules/tracking/geo.py`
- `backend/app/modules/tracking/models.py`
- `backend/app/modules/tracking/router.py`
- `backend/app/modules/tracking/schemas.py`
- `backend/app/modules/tracking/service.py`

## 4. Frontend files

- `frontend/lib/modules/tracking/data/tracking_api.dart`
- `frontend/lib/modules/tracking/screens/admin_live_map_screen.dart`
- `frontend/lib/modules/tracking/state/position_reporter.dart`
- `frontend/lib/modules/tracking/widgets/live_map.dart`
- `frontend/lib/modules/tracking/widgets/sharing_strip.dart`

## 5. What the existing project documentation says

## How it works
- **Every source goes through `ingest()`.** A Traccar bridge or a hardware feed later only needs
- **Fixes with `accuracy_m` above `MAX_FIX_ACCURACY_M` (100)** are stored but never trigger arrival
- **Look-ahead of 2 stops.** Auto-arrival considers only the next `ARRIVAL_LOOKAHEAD_STOPS`
- **Time of arrival** is the fix's `recorded_at` (clamped between trip start and now), so fixes
- **The driver's ARRIVED button stays.** It covers stops without coordinates and GPS trouble.
- **One ingest per trip at a time.** The trip row is locked for the request, so two batches
- **Distance is straight-line (haversine).** It's cheap and needs no PostGIS or routing
## Who hears `BusApproaching`
## Data model
## API
## Settings
## Events
## Public service API
## Frontend
## How to test
## Next
- **FCM push** so alerts reach phones with the app closed: extend `notifications.deliver()`.
- **Traccar / hardware trackers:** a small bridge that reads Traccar's WebSocket and calls `ingest()`.
- **ETA from speed and position** instead of timetable + delay, fed to `delay_monitor.evaluate(source=gps)`.
- **Retention:** prune `bus_positions` older than N days (~720 rows per bus per hour).

## 6. Backend functions — what each one means


### `geo.py`
- `distance_m(lat1, lon1, lat2, lon2)` — Internal helper used by this module.
  - Code comment/docstring: Haversine distance in metres between two WGS84 points.

### `router.py`
- `async report_positions(trip_id, body, p, session)` — Creates or processes a delay/problem report.
- `async trip_live(trip_id, p, session)` — Internal helper used by this module.
- `async all_live(_, session)` — Internal helper used by this module.

### `service.py`
- `_ensure_reporter(trip, p)` — Creates or processes a delay/problem report.
- `_last_reached(trip)` — Internal helper used by this module.
- `_usable(fix)` — Internal helper used by this module.
- `async _stop_coords(session, stop_ids)` — Internal helper used by this module.
  - Code comment/docstring: stop_id -> (lat, lng), for stops that have coordinates.
- `_arrival_candidate(trip, coords, lat, lng)` — Internal helper used by this module.
  - Code comment/docstring: The earliest of the next few unreached stops the bus is standing at, if any.

Looking only a couple of stops ahead stops a road that passes near a much later stop from
checking the bus in there and skipping everything in between.
- `async ingest(session, trip_id, fixes, p)` — Accepts a position/event from an external source and applies business rules.
- `async broadcast(position, trip)` — Publishes live data to connected clients.
  - Code comment/docstring: Push a fix to everyone watching the route, and to the transport office. Call after commit.
- `async delete_old_positions(session, older_than_days)` — Deletes or removes a record/resource.
  - Code comment/docstring: Housekeeping: GPS fixes are kept this long (a bus reports ~720 an hour). Trip history,
stop arrivals and the domain event log are not affected.
- `async latest_positions(session, trip_ids)` — Internal helper used by this module.
- `async live_trips(session, trips)` — Internal helper used by this module.
- `async live_trip(session, trip_id)` — Internal helper used by this module.

## 7. Frontend functions/screens — what each one means


### `frontend/lib/modules/tracking/data/tracking_api.dart`
- `BusFix()` — Works with bus/fleet information.
- `LiveStop()` — Internal helper used by this module.
- `LiveTrip()` — Internal helper used by this module.
- `withPosition()` — Internal helper used by this module.
- `withNewerPosition()` — Internal helper used by this module.
- `fetch()` — Internal helper used by this module.
- `for()` — Internal helper used by this module.
- `asList()` — Returns a list of records for the requested scope.
- `if()` — Internal helper used by this module.

### `frontend/lib/modules/tracking/screens/admin_live_map_screen.dart`
- `AdminLiveMapScreen()` — Builds the admin-facing data/view.
- `initState()` — Internal helper used by this module.
- `dispose()` — Internal helper used by this module.
- `SignNotice()` — Internal helper used by this module.
- `LayoutBuilder()` — Internal helper used by this module.
- `_BusList()` — Returns a list of records for the requested scope.
- `_BusRow()` — Works with bus/fleet information.
- `if()` — Internal helper used by this module.
- `final()` — Internal helper used by this module.

### `frontend/lib/modules/tracking/state/position_reporter.dart`
- `ReporterState()` — Creates or processes a delay/problem report.
- `copyWith()` — Internal helper used by this module.
- `start()` — Starts a trip/workflow after validating the current state.
- `stop()` — Internal helper used by this module.
- `fixSettings()` — Internal helper used by this module.
- `_settings()` — Internal helper used by this module.
- `AndroidSettings()` — Internal helper used by this module.
- `_onFix()` — Internal helper used by this module.
- `_send()` — Ends/completes a trip or workflow.
- `_teardown()` — Internal helper used by this module.
- `syncTripReporting()` — Creates or processes a delay/problem report.
- `if()` — Internal helper used by this module.
- `while()` — Internal helper used by this module.

### `frontend/lib/modules/tracking/widgets/live_map.dart`
- `LiveMap()` — Internal helper used by this module.
- `didUpdateWidget()` — Updates an existing record/resource.
- `_fitAll()` — Internal helper used by this module.
- `_recentre()` — Internal helper used by this module.
- `_busMarker()` — Works with bus/fleet information.
- `_StopMark()` — Internal helper used by this module.
- `Tooltip()` — Internal helper used by this module.
- `_BusMark()` — Works with bus/fleet information.
- `_Overlay()` — Internal helper used by this module.
- `distanceLabel()` — Internal helper used by this module.
- `if()` — Internal helper used by this module.
- `FlutterMap()` — Internal helper used by this module.

### `frontend/lib/modules/tracking/widgets/sharing_strip.dart`
- `LocationSharingStrip()` — Internal helper used by this module.
- `Semantics()` — Internal helper used by this module.
- `final()` — Internal helper used by this module.

## 8. Module flow

```text
Driver phone GPS
→ POST positions
→ validate source and accuracy
→ store fix
→ detect nearby stop
→ auto-arrive / BusApproaching when applicable
→ broadcast live position
```

## 9. Main code lines and their meaning


### `backend/app/modules/tracking/router.py`
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
- **Line 7:** `from app.modules.tracking import service`
  - Meaning: Imports a library/module used by this file.
- **Line 8:** `from app.modules.tracking.schemas import IngestOut, LiveTrip, PositionBatchIn, PositionOut`
  - Meaning: Imports a library/module used by this file.

### `backend/app/modules/tracking/service.py`
- **Line 16:** `from dataclasses import dataclass, field`
  - Meaning: Imports a library/module used by this file.
- **Line 17:** `from datetime import timedelta`
  - Meaning: Imports a library/module used by this file.
- **Line 19:** `from sqlalchemy import delete, select`
  - Meaning: Imports a library/module used by this file.
- **Line 20:** `from sqlalchemy.ext.asyncio import AsyncSession`
  - Meaning: Imports a library/module used by this file.
- **Line 22:** `from app.core import events`
  - Meaning: Imports a library/module used by this file.
- **Line 23:** `from app.core.config import settings`
  - Meaning: Imports a library/module used by this file.
- **Line 24:** `from app.core.deps import Principal`
  - Meaning: Imports a library/module used by this file.

### `backend/app/modules/tracking/__init__.py`
- **Line 3:** `from fastapi import FastAPI`
  - Meaning: Imports a library/module used by this file.
- **Line 5:** `from app.core import tasks`
  - Meaning: Imports a library/module used by this file.
- **Line 6:** `from app.core.config import settings`
  - Meaning: Imports a library/module used by this file.
- **Line 7:** `from app.core.db import SessionLocal`
  - Meaning: Imports a library/module used by this file.
- **Line 8:** `from app.modules.tracking import service`
  - Meaning: Imports a library/module used by this file.
- **Line 9:** `from app.modules.tracking.router import router`
  - Meaning: Imports a library/module used by this file.
- **Line 15:** `await session.commit()`
  - Meaning: Uses the database session asynchronously.

### `frontend/lib/modules/tracking/data/tracking_api.dart`
- **Line 1:** `import 'package:flutter_riverpod/flutter_riverpod.dart';`
  - Meaning: Imports a library/module used by this file.
- **Line 2:** `import 'package:latlong2/latlong.dart';`
  - Meaning: Imports a library/module used by this file.
- **Line 4:** `import '../../../core/api/api_client.dart';`
  - Meaning: Imports a library/module used by this file.
- **Line 5:** `import '../../../core/format.dart';`
  - Meaning: Imports a library/module used by this file.
- **Line 6:** `import '../../../core/realtime/realtime.dart';`
  - Meaning: Imports a library/module used by this file.
- **Line 7:** `import '../../trips/data/trip_models.dart';`
  - Meaning: Imports a library/module used by this file.
- **Line 9:** `/// One GPS fix of a bus, as the API and the `position` WebSocket message send it.`
  - Meaning: Important setup or business-logic line used by this module.

## 10. Beginner takeaway

Remember this module as: **tracking = the part of the system responsible for its own business job**. The router receives the request, the service does the work, the database stores the result, and domain events let other modules react without tightly coupling everything together.