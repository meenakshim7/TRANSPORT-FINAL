# Transit Project — Simple Explanation

## 1. What is this project?

**Transit** is an intelligent college transport management platform.

In simple words, it connects three types of users:

- **Student** — sees their route, trip information, attendance, alerts and reports.
- **Driver** — starts a bus trip, shows a QR code, records/handles boarding and sees trip status.
- **Admin / Transport office** — manages buses, routes, students, schedules, allocations, live trips, reports and history.

The project is a full-stack application:

- **Frontend:** Flutter (web + Android)
- **Backend:** FastAPI (Python)
- **Database:** PostgreSQL
- **ORM:** SQLAlchemy async
- **Migrations:** Alembic
- **Realtime:** WebSockets
- **Authentication:** JWT + bcrypt
- **AI/P1 extension:** report analysis using a local LLM (Ollama) with rule-based safety checks

## 2. The main idea

Think of the system as:

```text
Student / Driver / Admin
          |
          v
     Flutter App
          |
          | HTTP REST / WebSocket
          v
     FastAPI Backend
          |
          v
   Module Services
          |
          v
      PostgreSQL
```

The backend is a **modular monolith**. That means it is one backend application, but the business logic is divided into separate modules.

## 3. Main modules

| Module | Simple meaning |
|---|---|
| auth | Login, users and roles |
| master_data | Buses, stops and routes |
| allocation | Which student uses which route/stop |
| trips | Schedules and actual daily trips |
| boarding | QR boarding and attendance |
| capacity | Bus occupancy and capacity alerts |
| delay_monitor | Detects and reports delays |
| notifications | Decides who should receive alerts |
| history | Stores/reads the operational story |
| dashboard | Builds role-specific home screens |
| tracking | Live GPS position and automatic stop arrival |
| reports | Student complaints, AI analysis and lost/found |

## 4. Typical real-life flow

```text
Admin creates route
      ↓
Admin adds bus + driver
      ↓
Admin creates schedule
      ↓
Admin assigns students to route + stop
      ↓
Daily trip is generated
      ↓
Driver starts trip
      ↓
Driver shows rotating QR
      ↓
Student scans QR
      ↓
Boarding is recorded
      ↓
Capacity is recalculated
      ↓
Bus reaches stops
      ↓
Delay / tracking modules evaluate status
      ↓
Notifications are sent to the right users
      ↓
Dashboard updates through WebSocket
      ↓
Trip ends
      ↓
Attendance is finalised
      ↓
History contains the complete timeline
```

## 5. Important architecture rule

Each module generally follows:

```text
router.py
   ↓
service.py
   ↓
other services / database
   ↓
domain event
   ↓
notifications / dashboard / history / other subscribers
```

- **router.py** = API entry point. It receives HTTP requests.
- **schemas.py** = input/output data validation.
- **service.py** = actual business logic.
- **models.py** = database-specific model definitions where needed.
- **__init__.py** = event subscriptions / module registration.
- **tests/** = automated tests for the module.

The frontend follows a similar idea:

```text
screen/widget
    ↓
data/*_api.dart
    ↓
ApiClient
    ↓
FastAPI endpoint
```

## 6. The most important concept: domain events

A module does not need to directly know every other module.

For example:

```text
Driver arrives at stop
        ↓
trips publishes StopArrived
        ↓
delay_monitor receives it
        ↓
delay_monitor detects +12 minutes
        ↓
publishes TripDelayed
        ↓
notifications finds affected students
        ↓
dashboard refresh is pushed
        ↓
history keeps the event
```

This makes the project easier to extend.

## 7. Backend entry point

`backend/app/main.py` creates the FastAPI application and registers all module routers.

The `/health` endpoint checks whether the database is reachable.

## 8. Frontend entry point

`frontend/lib/main.dart` starts Flutter.

`frontend/lib/app.dart` is the composition root. It defines navigation and decides which screens are available for student, driver and admin roles.

## 9. Authentication in one sentence

The user logs in → backend verifies the password → backend returns access + refresh JWTs → frontend stores the session → `ApiClient` sends the access token with API requests → expired access tokens are refreshed once.

## 10. Tracking in one sentence

The driver's phone periodically sends GPS fixes → tracking validates/stores them → a nearby stop can be automatically marked arrived → the system can notify riders when the bus is approaching → live position is pushed over WebSocket.

## 11. Reports/AI in one sentence

A student submits a problem → the report is stored → background analysis gathers evidence from real trip records → rules protect the minimum severity → an optional local LLM helps extract claims and draft a response → admin reviews it.

## 12. Important note

This documentation is a **beginner-friendly guide to the code that is actually present in the uploaded project**, not a replacement for the source code. The module files below explain the important functions, flow and selected code lines.
