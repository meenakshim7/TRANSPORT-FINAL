# How to Read This Project's Code

## Start here

Read these in this order:

1. `backend/app/main.py`
2. `backend/app/core/security.py`
3. `backend/app/core/events.py`
4. `backend/app/core/realtime.py`
5. One module's `router.py`
6. The same module's `service.py`
7. The same module's `schemas.py`
8. Its Flutter `data/*_api.dart`
9. Its Flutter `screens/*`
10. Its tests

## What to look for

### Router

If you see:

```python
@router.get(...)
@router.post(...)
@router.patch(...)
```

it means **"this function is an API endpoint."**

### Service

A function in `service.py` normally contains the real business rule.

Example idea:

```python
async def check_in(...):
    # validate
    # read database
    # apply rules
    # write database
    # publish event
```

### Schema

A schema defines the shape of request/response data.

Think:

```text
JSON coming in
     ↓
schema validation
     ↓
service
```

### Model

A model maps Python objects to database tables.

### Event

When one module finishes an important operation, it can publish an event.

Example:

```text
StudentBoarded
TripDelayed
StopArrived
TripStarted
```

Another module can subscribe to that event.

### Flutter API file

Files such as `boarding_api.dart` or `trips_api.dart` are the bridge between the screen and backend.

```text
Screen
 ↓
API class/function
 ↓
ApiClient
 ↓
HTTP
 ↓
FastAPI router
```

### Flutter screen

A screen displays the state and collects user actions.

It should not contain the whole backend business rule.

## Most important classes/concepts

- `FastAPI` — creates the backend web application.
- `APIRouter` — groups endpoints.
- `AsyncSession` — asynchronous database session.
- `select()` — database query construction.
- `commit()` — save transaction.
- `JWT` — signed authentication token.
- `Dio` — Flutter HTTP client.
- `Riverpod` — Flutter state management.
- `GoRouter` — Flutter navigation.
- `WebSocket` — live server-to-client updates.
- `Alembic` — database schema migrations.

## One request from beginning to end

```text
User taps button
   ↓
Flutter screen method
   ↓
module API function
   ↓
ApiClient.post/get/patch
   ↓
HTTP request
   ↓
FastAPI router endpoint
   ↓
schema validation
   ↓
service function
   ↓
database / other services
   ↓
commit
   ↓
domain event
   ↓
notifications / dashboard / history / subscribers
   ↓
JSON response
   ↓
Flutter updates screen
```

## How to study it without getting confused

Do not try to understand all 12 modules at once.

Recommended order:

1. auth
2. master_data
3. allocation
4. trips
5. boarding
6. capacity
7. delay_monitor
8. notifications
9. dashboard
10. history
11. tracking
12. reports

For every module ask only four questions:

1. **What data does it manage?**
2. **What API endpoints does it expose?**
3. **What does its service actually do?**
4. **Which events does it consume/publish?**
