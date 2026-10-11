# auth Module — Simple Explanation

## 1. What this module does

Who is using the system and what they're allowed to do.

## 2. Where the code is

- Backend: `backend/app/modules/auth/`
- Frontend: `frontend/lib/modules/auth/`

## 3. Backend files

- `backend/app/modules/auth/models.py`
- `backend/app/modules/auth/router.py`
- `backend/app/modules/auth/schemas.py`
- `backend/app/modules/auth/service.py`

## 4. Frontend files

- `frontend/lib/modules/auth/data/people_api.dart`
- `frontend/lib/modules/auth/screens/admin_people_screen.dart`
- `frontend/lib/modules/auth/screens/login_screen.dart`

## 5. What the existing project documentation says

## Responsibilities
- User accounts with one role each: `student`, `driver`, `admin`.
- Student profile (roll no, department, year) and driver profile (licence).
- JWT login: short-lived access token (`ACCESS_TOKEN_MINUTES`) + refresh token (`REFRESH_TOKEN_DAYS`).
- Admin-only user management. There's no self sign-up: the transport office creates accounts.
## Data model
## API
## Events
## Public service API
## Security notes
- Tokens carry `typ` (`access` / `refresh` / `board`), so one kind can't be used as another.
- `Principal` is decoded from the token without a DB lookup. A deactivated user keeps REST access
- **Session revocation:** every token carries the user's `token_version` (`ver`). Changing the
- **Password hashing runs in a worker thread** (`security.hash_password_async` /
- **Login rate limit** per IP + email (not per IP alone, so a campus behind one NAT address isn't
- `BCRYPT_ROUNDS` defaults to 12 (tests use 4 for speed).
## Frontend screens
## How to test
## Extension notes
- SSO/OTP can replace `/auth/login` without touching other modules (they only see `Principal`).

## 6. Backend functions — what each one means


### `router.py`
- `async login(body, request, session)` — Handles user sign-in and creates/restores an authenticated session.
- `async refresh(body, session)` — Refreshes authentication state or renews an access token.
- `async me(p, session)` — Internal helper used by this module.
- `async list_users(role, q, active, limit, offset, _, session)` — Returns a list of records for the requested scope.
- `async create_user(body, p, session)` — Creates a new record/resource.
- `async get_user(user_id, _, session)` — Fetches one record or a computed view.
- `async update_user(user_id, body, p, session)` — Updates an existing record/resource.

### `service.py`
- `_tokens(user)` — Internal helper used by this module.
- `async login(session, email, password)` — Handles user sign-in and creates/restores an authenticated session.
- `async refresh(session, refresh_token)` — Refreshes authentication state or renews an access token.
- `async session_valid(session, p)` — Internal helper used by this module.
  - Code comment/docstring: The account behind an access token is still active and the token wasn't revoked.
- `async get_user(session, user_id)` — Fetches one record or a computed view.
- `async ensure_role(session, user_id, role)` — Internal helper used by this module.
- `async get_users(session, user_ids)` — Fetches one record or a computed view.
- `brief(user)` — Internal helper used by this module.
- `async briefs(session, user_ids)` — Internal helper used by this module.
- `async admin_ids(session)` — Builds the admin-facing data/view.
- `async find_student_by_roll_no(session, roll_no)` — Builds or handles the student-facing workflow.
- `async list_users(session)` — Returns a list of records for the requested scope.
- `async create_user(session, data)` — Creates a new record/resource.
- `async update_user(session, user_id, data)` — Updates an existing record/resource.
- `async _active_admin_count(session)` — Builds the admin-facing data/view.

## 7. Frontend functions/screens — what each one means


### `frontend/lib/modules/auth/data/people_api.dart`
- `Person()` — Internal helper used by this module.
- `asList()` — Returns a list of records for the requested scope.
- `create()` — Creates a new record/resource.
- `setActive()` — Internal helper used by this module.

### `frontend/lib/modules/auth/screens/admin_people_screen.dart`
- `AdminPeopleScreen()` — Builds the admin-facing data/view.
- `_Chip()` — Internal helper used by this module.
- `_PersonRow()` — Internal helper used by this module.
- `_AddPersonDialog()` — Internal helper used by this module.
- `_save()` — Internal helper used by this module.
- `SignHeader()` — Internal helper used by this module.
- `setState()` — Internal helper used by this module.

### `frontend/lib/modules/auth/screens/login_screen.dart`
- `LoginScreen()` — Handles user sign-in and creates/restores an authenticated session.
- `dispose()` — Internal helper used by this module.
- `_submit()` — Internal helper used by this module.
- `RouteBadge()` — Works with route/network information.
- `setState()` — Internal helper used by this module.

## 8. Module flow

```text
Login screen → /auth/login → verify password → JWT tokens
→ SessionController stores session → role-based home screen
```

## 9. Main code lines and their meaning


### `backend/app/modules/auth/router.py`
- **Line 1:** `from fastapi import APIRouter, Depends, Query, Request`
  - Meaning: Imports a library/module used by this file.
- **Line 2:** `from sqlalchemy.ext.asyncio import AsyncSession`
  - Meaning: Imports a library/module used by this file.
- **Line 4:** `from app.core.config import settings`
  - Meaning: Imports a library/module used by this file.
- **Line 5:** `from app.core.db import get_session`
  - Meaning: Imports a library/module used by this file.
- **Line 6:** `from app.core.deps import Principal, current_principal, require_roles`
  - Meaning: Imports a library/module used by this file.
- **Line 7:** `from app.core.ratelimit import RateLimiter`
  - Meaning: Imports a library/module used by this file.
- **Line 8:** `from app.core.roles import Role`
  - Meaning: Imports a library/module used by this file.

### `backend/app/modules/auth/service.py`
- **Line 4:** `from sqlalchemy import func, or_, select`
  - Meaning: Imports a library/module used by this file.
- **Line 5:** `from sqlalchemy.ext.asyncio import AsyncSession`
  - Meaning: Imports a library/module used by this file.
- **Line 7:** `from app.core import events`
  - Meaning: Imports a library/module used by this file.
- **Line 8:** `from app.core.db import flush_or_conflict`
  - Meaning: Imports a library/module used by this file.
- **Line 9:** `from app.core.deps import Principal`
  - Meaning: Imports a library/module used by this file.
- **Line 10:** `from app.core.errors import Conflict, InvalidState, NotFound, Unauthorized`
  - Meaning: Imports a library/module used by this file.
- **Line 11:** `from app.core.roles import Role`
  - Meaning: Imports a library/module used by this file.

### `backend/app/modules/auth/__init__.py`
- **Line 3:** `from fastapi import FastAPI`
  - Meaning: Imports a library/module used by this file.
- **Line 5:** `from app.core import events, realtime`
  - Meaning: Imports a library/module used by this file.
- **Line 6:** `from app.core.db import SessionLocal`
  - Meaning: Imports a library/module used by this file.
- **Line 7:** `from app.core.deps import Principal`
  - Meaning: Imports a library/module used by this file.
- **Line 8:** `from app.modules.auth import service`
  - Meaning: Imports a library/module used by this file.
- **Line 9:** `from app.modules.auth.router import router`
  - Meaning: Imports a library/module used by this file.
- **Line 22:** `def register(app: FastAPI) -> None:`
  - Meaning: Connects this module to the event system so other modules can react.

### `frontend/lib/modules/auth/data/people_api.dart`
- **Line 1:** `import 'package:flutter_riverpod/flutter_riverpod.dart';`
  - Meaning: Imports a library/module used by this file.
- **Line 3:** `import '../../../core/api/api_client.dart';`
  - Meaning: Imports a library/module used by this file.
- **Line 4:** `import '../../../core/auth/session.dart';`
  - Meaning: Imports a library/module used by this file.
- **Line 6:** `class Person {`
  - Meaning: Defines a class that groups related data or behavior.
- **Line 7:** `const Person({`
  - Meaning: Important setup or business-logic line used by this module.
- **Line 19:** `final int id;`
  - Meaning: Important setup or business-logic line used by this module.
- **Line 20:** `final String fullName;`
  - Meaning: Important setup or business-logic line used by this module.

## 10. Beginner takeaway

Remember this module as: **auth = the part of the system responsible for its own business job**. The router receives the request, the service does the work, the database stores the result, and domain events let other modules react without tightly coupling everything together.