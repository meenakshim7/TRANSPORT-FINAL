# notifications Module — Simple Explanation

## 1. What this module does

Decides **who** hears about **what**, stores it in their inbox and pushes it over WebSocket.

## 2. Where the code is

- Backend: `backend/app/modules/notifications/`
- Frontend: `frontend/lib/modules/notifications/`

## 3. Backend files

- `backend/app/modules/notifications/models.py`
- `backend/app/modules/notifications/router.py`
- `backend/app/modules/notifications/schemas.py`
- `backend/app/modules/notifications/service.py`

## 4. Frontend files

- `frontend/lib/modules/notifications/data/notifications_api.dart`
- `frontend/lib/modules/notifications/screens/inbox_screen.dart`
- `frontend/lib/modules/notifications/state/notifications_controller.dart`

## 5. What the existing project documentation says

## Who gets what
## Data model
## API
## Consumes
## Public service API
## Frontend screens
## How to test
## Extension notes (Team B: "intelligent notifications")
- Add channels by extending `deliver()` (FCM push, email) while keeping `messages_for` as the
- Quiet hours / preferences: filter in `deliver()` per user.

## 6. Backend functions — what each one means


### `router.py`
- `async inbox(unread_only, limit, before_id, p, session)` — Internal helper used by this module.
- `async unread(p, session)` — Internal helper used by this module.
- `async mark_read(notification_id, p, session)` — Internal helper used by this module.
- `async mark_all(p, session)` — Internal helper used by this module.

### `service.py`
- `_hm(value)` — Internal helper used by this module.
- `async create(session, msg)` — Creates a new record/resource.
- `async push(rows)` — Pushes an event/message to connected clients.
- `async deliver(session, messages)` — Internal helper used by this module.
  - Code comment/docstring: Persist all messages, commit, then push over WebSocket.
- `async list_for_user(session, user_id)` — Returns a list of records for the requested scope.
- `async unread_count(session, user_id)` — Internal helper used by this module.
- `async mark_read(session, user_id, notification_id)` — Internal helper used by this module.
- `async delete_old_read(session, older_than_days)` — Deletes or removes a record/resource.
  - Code comment/docstring: Housekeeping: read notifications older than this are deleted. Unread ones are kept.
- `async _student_stops(session, route_id)` — Builds or handles the student-facing workflow.
  - Code comment/docstring: student_id -> allocated stop_id, for active allocations on the route.
- `async delay_student_targets(session, payload)` — Fetches one record or a computed view.
  - Code comment/docstring: Students affected by a delay, each with the affected stop entry relevant to them.

Pickup: students waiting at a stop the bus hasn't reached (and who haven't boarded).
Drop:   students on board (trip running) or all allocated riders (not started yet).
- `async approach_student_targets(session, payload)` — Fetches one record or a computed view.
  - Code comment/docstring: Riders of the stop the bus is nearing.

Pickup: allocated to that stop and not boarded yet. Drop: on board and getting off there.
- `async messages_for(session, ev)` — Internal helper used by this module.
- `_cut(text, n)` — Internal helper used by this module.

## 7. Frontend functions/screens — what each one means


### `frontend/lib/modules/notifications/data/notifications_api.dart`
- `AppNotification()` — Internal helper used by this module.
- `asList()` — Returns a list of records for the requested scope.
- `markRead()` — Internal helper used by this module.
- `markAllRead()` — Internal helper used by this module.

### `frontend/lib/modules/notifications/screens/inbox_screen.dart`
- `InboxScreen()` — Internal helper used by this module.
- `_AlertRow()` — Internal helper used by this module.
- `SignHeader()` — Internal helper used by this module.

### `frontend/lib/modules/notifications/state/notifications_controller.dart`
- `LiveNotificationListener()` — Returns a list of records for the requested scope.
- `listenLive()` — Returns a list of records for the requested scope.
- `if()` — Internal helper used by this module.

## 8. Module flow

```text
Domain event
→ identify affected users
→ create inbox notification
→ push through WebSocket
→ user can mark one/all as read
```

## 9. Main code lines and their meaning


### `backend/app/modules/notifications/router.py`
- **Line 1:** `from fastapi import APIRouter, Depends, Query`
  - Meaning: Imports a library/module used by this file.
- **Line 2:** `from sqlalchemy.ext.asyncio import AsyncSession`
  - Meaning: Imports a library/module used by this file.
- **Line 4:** `from app.core.db import get_session`
  - Meaning: Imports a library/module used by this file.
- **Line 5:** `from app.core.deps import Principal, current_principal`
  - Meaning: Imports a library/module used by this file.
- **Line 6:** `from app.core.errors import NotFound`
  - Meaning: Imports a library/module used by this file.
- **Line 7:** `from app.modules.notifications import service`
  - Meaning: Imports a library/module used by this file.
- **Line 8:** `from app.modules.notifications.schemas import NotificationOut, UnreadCount`
  - Meaning: Imports a library/module used by this file.

### `backend/app/modules/notifications/service.py`
- **Line 7:** `from datetime import datetime, timedelta`
  - Meaning: Imports a library/module used by this file.
- **Line 9:** `from sqlalchemy import delete, func, select, update`
  - Meaning: Imports a library/module used by this file.
- **Line 10:** `from sqlalchemy.ext.asyncio import AsyncSession`
  - Meaning: Imports a library/module used by this file.
- **Line 12:** `from app.core.events import Event, jsonable`
  - Meaning: Imports a library/module used by this file.
- **Line 13:** `from app.core.realtime import hub`
  - Meaning: Imports a library/module used by this file.
- **Line 14:** `from app.core.timeutil import local_tz, now_utc`
  - Meaning: Imports a library/module used by this file.
- **Line 15:** `from app.modules.allocation import service as alloc_service`
  - Meaning: Imports a library/module used by this file.

### `backend/app/modules/notifications/__init__.py`
- **Line 1:** `"""Notifications module: in-app inbox + WebSocket push for operational events."""`
  - Meaning: Important setup or business-logic line used by this module.
- **Line 3:** `from fastapi import FastAPI`
  - Meaning: Imports a library/module used by this file.
- **Line 5:** `from app.core import events, tasks`
  - Meaning: Imports a library/module used by this file.
- **Line 6:** `from app.core.config import settings`
  - Meaning: Imports a library/module used by this file.
- **Line 7:** `from app.core.db import SessionLocal`
  - Meaning: Imports a library/module used by this file.
- **Line 8:** `from app.modules.notifications import service`
  - Meaning: Imports a library/module used by this file.
- **Line 9:** `from app.modules.notifications.router import router`
  - Meaning: Imports a library/module used by this file.

### `frontend/lib/modules/notifications/data/notifications_api.dart`
- **Line 1:** `import 'package:flutter_riverpod/flutter_riverpod.dart';`
  - Meaning: Imports a library/module used by this file.
- **Line 3:** `import '../../../core/api/api_client.dart';`
  - Meaning: Imports a library/module used by this file.
- **Line 4:** `import '../../../core/format.dart';`
  - Meaning: Imports a library/module used by this file.
- **Line 6:** `class AppNotification {`
  - Meaning: Defines a class that groups related data or behavior.
- **Line 7:** `const AppNotification({`
  - Meaning: Important setup or business-logic line used by this module.
- **Line 18:** `final int id;`
  - Meaning: Important setup or business-logic line used by this module.
- **Line 19:** `final String type;`
  - Meaning: Important setup or business-logic line used by this module.

## 10. Beginner takeaway

Remember this module as: **notifications = the part of the system responsible for its own business job**. The router receives the request, the service does the work, the database stores the result, and domain events let other modules react without tightly coupling everything together.