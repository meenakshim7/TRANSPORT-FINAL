# allocation Module — Simple Explanation

## 1. What this module does

Which bus route each student rides, and where they get on.

## 2. Where the code is

- Backend: `backend/app/modules/allocation/`
- Frontend: `frontend/lib/modules/allocation/`

## 3. Backend files

- `backend/app/modules/allocation/models.py`
- `backend/app/modules/allocation/router.py`
- `backend/app/modules/allocation/schemas.py`
- `backend/app/modules/allocation/service.py`

## 4. Frontend files

- `frontend/lib/modules/allocation/data/allocation_api.dart`
- `frontend/lib/modules/allocation/screens/admin_allocation_screen.dart`

## 5. What the existing project documentation says

## Rules
- **One active allocation per student** (partial unique index). Re-assigning ends the old one
- The stop must be on the route (`stop_not_on_route`), the user must be an active student
- **Seat guard:** route capacity = smallest bus among the route's active schedules
- Assigning a student to the allocation they already have is a no-op (`changed: false`, no event).
## Live channel policy
## Data model
## API
## Events
## Public service API
## Frontend screens
## How to test
## Extension notes (Team B)
- **Smart reallocation / consolidation (P2):** call `service.assign` (or `bulk_assign`) from the
- The capacity recommendation can read `capacity_service.route_utilization()`.

## 6. Backend functions — what each one means


### `router.py`
- `async list_allocations(route_id, stop_id, student_id, include_ended, _, session)` — Returns a list of records for the requested scope.
- `async assign(body, p, session)` — Internal helper used by this module.
- `async bulk_assign(body, p, session)` — Internal helper used by this module.
- `async unassign(student_id, p, session)` — Internal helper used by this module.
- `async my_allocation(p, session)` — Assigns or manages a student's route and stop allocation.

### `service.py`
- `async get_active(session, student_id)` — Fetches one record or a computed view.
- `async active_on_route(session, route_id)` — Works with route/network information.
- `async students_at_stops(session, route_id, stop_ids)` — Builds or handles the student-facing workflow.
- `async count_active_by_route(session)` — Works with route/network information.
- `async to_out(session, allocations)` — Internal helper used by this module.
- `_end(a, reason)` — Ends/completes a trip or workflow.
- `async assign(session, data)` — Internal helper used by this module.
- `async bulk_assign(session, items)` — Internal helper used by this module.
  - Code comment/docstring: Each item runs in its own savepoint: one bad row doesn't sink the batch.
- `async unassign(session, student_id)` — Internal helper used by this module.
- `async list_allocations(session)` — Returns a list of records for the requested scope.
- `async end_for_deactivated(session, student_id)` — Ends/completes a trip or workflow.
  - Code comment/docstring: A deactivated student gives their seat back. Returns whether they had one.
- `async may_follow_route(session, p, route_id)` — Works with route/network information.
  - Code comment/docstring: WebSocket "route:<id>" topics: the route's allocated students and today's drivers on it.
- `async allocation_valid_on(session, student_id, route_id, on)` — Assigns or manages a student's route and stop allocation.

## 7. Frontend functions/screens — what each one means


### `frontend/lib/modules/allocation/data/allocation_api.dart`
- `Allocation()` — Assigns or manages a student's route and stop allocation.
- `asList()` — Returns a list of records for the requested scope.
- `assign()` — Internal helper used by this module.
- `unassign()` — Internal helper used by this module.

### `frontend/lib/modules/allocation/screens/admin_allocation_screen.dart`
- `AdminAllocationScreen()` — Builds the admin-facing data/view.
- `SignHeader()` — Internal helper used by this module.
- `_RouteAllocations()` — Works with route/network information.
- `SignNotice()` — Internal helper used by this module.
- `_assign()` — Internal helper used by this module.
- `_AssignDialog()` — Internal helper used by this module.
- `_submit()` — Internal helper used by this module.
- `for()` — Internal helper used by this module.
- `LineStop()` — Internal helper used by this module.
- `showDialog()` — Internal helper used by this module.
- `if()` — Internal helper used by this module.
- `setState()` — Internal helper used by this module.

## 8. Module flow

```text
Admin chooses student + route + stop
→ validate student/route/stop
→ calculate route capacity
→ reject or warn if full
→ save active allocation
→ publish allocation event
```

## 9. Main code lines and their meaning


### `backend/app/modules/allocation/router.py`
- **Line 1:** `from fastapi import APIRouter, Depends, Response`
  - Meaning: Imports a library/module used by this file.
- **Line 2:** `from sqlalchemy.ext.asyncio import AsyncSession`
  - Meaning: Imports a library/module used by this file.
- **Line 4:** `from app.core.db import get_session`
  - Meaning: Imports a library/module used by this file.
- **Line 5:** `from app.core.deps import Principal, require_roles`
  - Meaning: Imports a library/module used by this file.
- **Line 6:** `from app.core.errors import NotFound`
  - Meaning: Imports a library/module used by this file.
- **Line 7:** `from app.core.roles import Role`
  - Meaning: Imports a library/module used by this file.
- **Line 8:** `from app.modules.allocation import service`
  - Meaning: Imports a library/module used by this file.

### `backend/app/modules/allocation/service.py`
- **Line 7:** `from datetime import date`
  - Meaning: Imports a library/module used by this file.
- **Line 9:** `from sqlalchemy import func, select`
  - Meaning: Imports a library/module used by this file.
- **Line 10:** `from sqlalchemy.ext.asyncio import AsyncSession`
  - Meaning: Imports a library/module used by this file.
- **Line 12:** `from app.core import events`
  - Meaning: Imports a library/module used by this file.
- **Line 13:** `from app.core.deps import Principal`
  - Meaning: Imports a library/module used by this file.
- **Line 14:** `from app.core.errors import Conflict, InvalidState, NotFound`
  - Meaning: Imports a library/module used by this file.
- **Line 15:** `from app.core.roles import Role`
  - Meaning: Imports a library/module used by this file.

### `backend/app/modules/allocation/__init__.py`
- **Line 3:** `from fastapi import FastAPI`
  - Meaning: Imports a library/module used by this file.
- **Line 5:** `from app.core import events, realtime`
  - Meaning: Imports a library/module used by this file.
- **Line 6:** `from app.core.db import SessionLocal`
  - Meaning: Imports a library/module used by this file.
- **Line 7:** `from app.core.deps import Principal`
  - Meaning: Imports a library/module used by this file.
- **Line 8:** `from app.modules.allocation import service`
  - Meaning: Imports a library/module used by this file.
- **Line 9:** `from app.modules.allocation.router import router`
  - Meaning: Imports a library/module used by this file.
- **Line 18:** `await session.commit()`
  - Meaning: Uses the database session asynchronously.

### `frontend/lib/modules/allocation/data/allocation_api.dart`
- **Line 1:** `import 'package:flutter_riverpod/flutter_riverpod.dart';`
  - Meaning: Imports a library/module used by this file.
- **Line 3:** `import '../../../core/api/api_client.dart';`
  - Meaning: Imports a library/module used by this file.
- **Line 5:** `class Allocation {`
  - Meaning: Defines a class that groups related data or behavior.
- **Line 6:** `const Allocation({`
  - Meaning: Important setup or business-logic line used by this module.
- **Line 16:** `final int id;`
  - Meaning: Important setup or business-logic line used by this module.
- **Line 17:** `final int studentId;`
  - Meaning: Important setup or business-logic line used by this module.
- **Line 18:** `final String studentName;`
  - Meaning: Important setup or business-logic line used by this module.

## 10. Beginner takeaway

Remember this module as: **allocation = the part of the system responsible for its own business job**. The router receives the request, the service does the work, the database stores the result, and domain events let other modules react without tightly coupling everything together.