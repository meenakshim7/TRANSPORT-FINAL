# master_data Module — Simple Explanation

## 1. What this module does

The fleet and the network: what runs, where it stops, and in what order.

## 2. Where the code is

- Backend: `backend/app/modules/master_data/`
- Frontend: `frontend/lib/modules/master_data/`

## 3. Backend files

- `backend/app/modules/master_data/models.py`
- `backend/app/modules/master_data/router.py`
- `backend/app/modules/master_data/schemas.py`
- `backend/app/modules/master_data/service.py`

## 4. Frontend files

- `frontend/lib/modules/master_data/data/master_data_api.dart`
- `frontend/lib/modules/master_data/screens/admin_fleet_screen.dart`
- `frontend/lib/modules/master_data/screens/admin_network_screen.dart`
- `frontend/lib/modules/master_data/screens/route_editor_screen.dart`

## 5. What the existing project documentation says

## Responsibilities
- Buses (registration, seat capacity, status `active / maintenance / retired`).
- Stops (name, landmark, optional lat/lng, ready for GPS in P1).
- Routes (code shown on the route badge, name, **line colour**) and their ordered stops.
## Route convention (important for everyone)
- `route_stops.sequence` 1 = first pickup point. **The last stop is the campus.**
- `offset_min` = minutes after departure from stop 1, in the pickup direction. It must not decrease.
- Drop trips run the list in reverse. The trips module computes those times.
## Data model
## API
## Events
## Public service API
## Frontend screens
## How to test
## Extension notes
- P1 GPS: stops already have lat/lng; route polylines can live in a new `route_shapes` table here.
- P2 route optimisation writes new `route_stops` through `set_route_stops`, which already preserves

## 6. Backend functions — what each one means


### `router.py`
- `async list_buses(_, session)` — Returns a list of records for the requested scope.
- `async create_bus(body, p, session)` — Creates a new record/resource.
- `async get_bus(bus_id, _, session)` — Fetches one record or a computed view.
- `async update_bus(bus_id, body, p, session)` — Updates an existing record/resource.
- `async assign_driver(bus_id, body, p, session)` — Builds or handles the driver-facing workflow.
  - Code comment/docstring: Assign the bus's regular driver (null removes). 409 `driver_taken` if they drive another
bus, unless `move: true`.
- `async delete_bus(bus_id, _, session)` — Deletes or removes a record/resource.
- `async list_stops(q, _, session)` — Returns a list of records for the requested scope.
- `async create_stop(body, _, session)` — Creates a new record/resource.
- `async update_stop(stop_id, body, _, session)` — Updates an existing record/resource.
- `async delete_stop(stop_id, _, session)` — Deletes or removes a record/resource.
- `async list_routes(active_only, _, session)` — Returns a list of records for the requested scope.
- `async create_route(body, p, session)` — Creates a new record/resource.
- `async get_route(route_id, _, session)` — Fetches one record or a computed view.
- `async update_route(route_id, body, p, session)` — Updates an existing record/resource.
- `async set_route_stops(route_id, body, p, session)` — Works with route/network information.

### `service.py`
- `async list_buses(session)` — Returns a list of records for the requested scope.
- `async get_bus(session, bus_id)` — Fetches one record or a computed view.
- `async create_bus(session, data)` — Creates a new record/resource.
- `async update_bus(session, bus_id, data)` — Updates an existing record/resource.
- `async assign_driver(session, bus_id, driver_id)` — Builds or handles the driver-facing workflow.
  - Code comment/docstring: Set (or clear) a bus's regular driver. The trips module reacts to `BusDriverAssigned`
by handing the bus's active schedules and not-yet-started trips to the new driver.
- `async delete_bus(session, bus_id)` — Deletes or removes a record/resource.
- `async list_stops(session, q)` — Returns a list of records for the requested scope.
- `async get_stop(session, stop_id)` — Fetches one record or a computed view.
- `async create_stop(session, data)` — Creates a new record/resource.
- `async update_stop(session, stop_id, data)` — Updates an existing record/resource.
- `async delete_stop(session, stop_id)` — Deletes or removes a record/resource.
- `async list_routes(session)` — Returns a list of records for the requested scope.
- `async get_route(session, route_id)` — Fetches one record or a computed view.
- `async get_route_stop(session, route_stop_id)` — Fetches one record or a computed view.
- `async find_route_stop(session, route_id, stop_id)` — Works with route/network information.
- `async create_route(session, data)` — Creates a new record/resource.
- `async update_route(session, route_id, data)` — Updates an existing record/resource.
- `async set_route_stops(session, route_id, items)` — Works with route/network information.
  - Code comment/docstring: Replace the ordered stop list. Existing RouteStop rows are kept (and re-sequenced)
for stops still present, so student allocations pointing at them survive a reorder.
Removing a stop that students are allocated to is refused (FK RESTRICT).

## 7. Frontend functions/screens — what each one means


### `frontend/lib/modules/master_data/data/master_data_api.dart`
- `Stop()` — Internal helper used by this module.
- `RouteStop()` — Works with route/network information.
- `TransitRoute()` — Works with route/network information.
- `Bus()` — Works with bus/fleet information.
- `asList()` — Returns a list of records for the requested scope.
- `createRoute()` — Creates a new record/resource.
- `updateRoute()` — Updates an existing record/resource.
- `setStops()` — Internal helper used by this module.
- `createStop()` — Creates a new record/resource.
- `createBus()` — Creates a new record/resource.
- `updateBus()` — Updates an existing record/resource.
- `assignDriver()` — Builds or handles the driver-facing workflow.
- `for()` — Internal helper used by this module.

### `frontend/lib/modules/master_data/screens/admin_fleet_screen.dart`
- `AdminFleetScreen()` — Builds the admin-facing data/view.
- `_BusRow()` — Works with bus/fleet information.
- `_AssignDriverDialog()` — Builds or handles the driver-facing workflow.
- `_save()` — Internal helper used by this module.
- `_AddBusDialog()` — Works with bus/fleet information.
- `SignHeader()` — Internal helper used by this module.
- `TextButton()` — Internal helper used by this module.
- `PopupMenuItem()` — Internal helper used by this module.
- `setState()` — Internal helper used by this module.
- `if()` — Internal helper used by this module.
- `NumberPlate()` — Internal helper used by this module.
- `TextField()` — Internal helper used by this module.

### `frontend/lib/modules/master_data/screens/admin_network_screen.dart`
- `AdminNetworkScreen()` — Builds the admin-facing data/view.
- `_newRoute()` — Works with route/network information.
- `ColorPicker()` — Internal helper used by this module.
- `_RouteStrip()` — Works with route/network information.
- `_MiniLine()` — Internal helper used by this module.
- `LayoutBuilder()` — Internal helper used by this module.
- `TextField()` — Internal helper used by this module.
- `if()` — Internal helper used by this module.

### `frontend/lib/modules/master_data/screens/route_editor_screen.dart`
- `RouteEditorScreen()` — Works with route/network information.
- `_init()` — Internal helper used by this module.
- `_move()` — Internal helper used by this module.
- `_save()` — Internal helper used by this module.
- `_addStop()` — Internal helper used by this module.
- `_editColor()` — Internal helper used by this module.
- `_StopPicker()` — Internal helper used by this module.
- `setState()` — Internal helper used by this module.
- `if()` — Internal helper used by this module.
- `IconButton()` — Internal helper used by this module.
- `SignHeader()` — Internal helper used by this module.
- `static()` — Internal helper used by this module.
- `TextField()` — Internal helper used by this module.

## 8. Module flow

```text
Admin enters bus/stop/route data
→ router validates input
→ service applies business rules
→ PostgreSQL stores it
→ other modules use the route/bus/stop data
```

## 9. Main code lines and their meaning


### `backend/app/modules/master_data/router.py`
- **Line 1:** `from fastapi import APIRouter, Depends, Response`
  - Meaning: Imports a library/module used by this file.
- **Line 2:** `from sqlalchemy.ext.asyncio import AsyncSession`
  - Meaning: Imports a library/module used by this file.
- **Line 4:** `from app.core.db import get_session`
  - Meaning: Imports a library/module used by this file.
- **Line 5:** `from app.core.deps import Principal, current_principal, require_roles`
  - Meaning: Imports a library/module used by this file.
- **Line 6:** `from app.core.roles import Role`
  - Meaning: Imports a library/module used by this file.
- **Line 7:** `from app.modules.master_data import service`
  - Meaning: Imports a library/module used by this file.
- **Line 8:** `from app.modules.master_data.schemas import (`
  - Meaning: Imports a library/module used by this file.

### `backend/app/modules/master_data/service.py`
- **Line 6:** `from sqlalchemy import select`
  - Meaning: Imports a library/module used by this file.
- **Line 7:** `from sqlalchemy.ext.asyncio import AsyncSession`
  - Meaning: Imports a library/module used by this file.
- **Line 9:** `from app.core import events`
  - Meaning: Imports a library/module used by this file.
- **Line 10:** `from app.core.db import flush_or_conflict`
  - Meaning: Imports a library/module used by this file.
- **Line 11:** `from app.core.errors import Conflict, InvalidState, NotFound`
  - Meaning: Imports a library/module used by this file.
- **Line 12:** `from app.core.roles import Role`
  - Meaning: Imports a library/module used by this file.
- **Line 13:** `from app.modules.auth import service as auth_service`
  - Meaning: Imports a library/module used by this file.

### `backend/app/modules/master_data/__init__.py`
- **Line 3:** `from app.modules.master_data.router import router`
  - Meaning: Imports a library/module used by this file.

### `frontend/lib/modules/master_data/data/master_data_api.dart`
- **Line 1:** `import 'dart:ui';`
  - Meaning: Imports a library/module used by this file.
- **Line 3:** `import 'package:flutter_riverpod/flutter_riverpod.dart';`
  - Meaning: Imports a library/module used by this file.
- **Line 5:** `import '../../../core/api/api_client.dart';`
  - Meaning: Imports a library/module used by this file.
- **Line 6:** `import '../../../design/tokens.dart';`
  - Meaning: Imports a library/module used by this file.
- **Line 8:** `class Stop {`
  - Meaning: Defines a class that groups related data or behavior.
- **Line 9:** `const Stop({required this.id, required this.name, this.landmark});`
  - Meaning: Important setup or business-logic line used by this module.
- **Line 11:** `final int id;`
  - Meaning: Important setup or business-logic line used by this module.

## 10. Beginner takeaway

Remember this module as: **master_data = the part of the system responsible for its own business job**. The router receives the request, the service does the work, the database stores the result, and domain events let other modules react without tightly coupling everything together.