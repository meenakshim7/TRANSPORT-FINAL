# Complete Project Flow

## A. Login flow

```text
Login screen
   ↓
people/auth API call
   ↓
POST /auth/login
   ↓
Backend verifies email + password
   ↓
JWT access token + refresh token
   ↓
Flutter SessionController saves session
   ↓
Router checks role
   ↓
Student / Driver / Admin home
```

### Important code ideas

- `SessionController.login()` sends the credentials.
- `Session.fromTokenPair()` converts JSON into a Dart session object.
- `create_access_token()` and `create_refresh_token()` create backend JWTs.
- `ApiClient` attaches `Authorization: Bearer <token>` to normal requests.

---

## B. Student boarding flow

```text
Driver starts trip
      ↓
Driver opens QR screen
      ↓
GET /boarding/trips/{trip_id}/qr
      ↓
Backend creates short-lived signed boarding token
      ↓
Student scans QR
      ↓
POST /boarding/check-in
      ↓
Backend verifies token + trip + student
      ↓
Boarding record created
      ↓
StudentBoarded event
      ↓
Capacity recalculated
      ↓
Driver/admin can be notified
```

---

## C. Delay flow

```text
Trip starts late OR stop is reached late
             ↓
        delay_monitor
             ↓
       calculate delay
             ↓
     check alert threshold
             ↓
        TripDelayed
             ↓
       notifications
             ↓
affected waiting students + driver + admin
             ↓
       WebSocket push
             ↓
      dashboard refresh
```

---

## D. Live GPS flow

```text
Driver phone GPS
      ↓
Flutter position_reporter
      ↓
POST /trips/{id}/positions
      ↓
tracking.service.ingest()
      ↓
validate driver/trip/fix accuracy
      ↓
save GPS fix
      ↓
check next unreached stops
      ↓
within 100m? → auto-arrive stop
      ↓
within 2km? → BusApproaching
      ↓
broadcast live position
      ↓
Admin live map / student views update
```

---

## E. Trip completion flow

```text
Driver taps End
      ↓
trips.service.end()
      ↓
last/terminal stop is handled
      ↓
trip becomes completed
      ↓
attendance finalisation
      ↓
attendance records written
      ↓
history can report the trip
```

---

## F. Student report + AI flow

```text
Student submits report
       ↓
reports.service.create_report()
       ↓
report saved
       ↓
background analysis requested
       ↓
AI/rules extract claims
       ↓
evidence.py checks actual system records
       ↓
severity rules set minimum urgency
       ↓
LLM may draft admin summary/reply
       ↓
admin reviews
       ↓
reply / follow-up / close
```

---

## G. Why WebSocket is used

Normal HTTP is good for:

```text
"Give me the dashboard now"
```

WebSocket is useful for:

```text
"The bus is delayed — update the screen now."
```

The backend can push an event without waiting for the user to manually refresh.

---

## H. Overall dependency picture

```text
                    ┌──────────────┐
                    │    Flutter   │
                    └──────┬───────┘
                           │
                     REST / WS
                           │
                    ┌──────▼───────┐
                    │   FastAPI    │
                    └──────┬───────┘
                           │
       ┌───────────────────┼───────────────────┐
       ↓                   ↓                   ↓
    auth             master_data             trips
       │                   │                   │
       ↓                   ↓                   ↓
 allocation ───────────────┼──────────── boarding
       │                   │                   │
       └───────────────┬───┴──────────────┬────┘
                       ↓                  ↓
                   capacity          delay_monitor
                       │                  │
                       └────────┬─────────┘
                                ↓
                         notifications
                                ↓
                           WebSocket
                                ↓
                             Flutter

tracking ──► trips + notifications + dashboard
reports  ──► history/trips/boarding data + AI/rules
history  ◄── domain events from the whole system
dashboard ◄── current state from multiple modules
```
