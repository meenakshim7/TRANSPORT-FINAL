# reports Module — Simple Explanation

## 1. What this module does

Students report problems with their bus; an agent checks each report against the trip records

## 2. Where the code is

- Backend: `backend/app/modules/reports/`
- Frontend: `frontend/lib/modules/reports/`

## 3. Backend files

- `backend/app/modules/reports/agent.py`
- `backend/app/modules/reports/evidence.py`
- `backend/app/modules/reports/llm.py`
- `backend/app/modules/reports/models.py`
- `backend/app/modules/reports/router.py`
- `backend/app/modules/reports/schemas.py`
- `backend/app/modules/reports/service.py`

## 4. Frontend files

- `frontend/lib/modules/reports/data/reports_api.dart`
- `frontend/lib/modules/reports/screens/admin_issue_screen.dart`
- `frontend/lib/modules/reports/screens/admin_issues_screen.dart`
- `frontend/lib/modules/reports/screens/my_reports_screen.dart`
- `frontend/lib/modules/reports/screens/report_form_screen.dart`
- `frontend/lib/modules/reports/widgets/choice.dart`
- `frontend/lib/modules/reports/widgets/log_found_item_sheet.dart`

## 5. What the existing project documentation says

## Responsibilities
- Students file a report: what kind (late or skipped stop, overcrowding, safety or conduct, lost
- The **agent** analyses every report in the background (it never slows the student's request):
- Admins reply (editing the draft), match lost items to found items, re-run the check, and close.
- Drivers and admins log found items; the agent suggests matches for lost-item reports.
### Evidence checks
### When the model isn't there
### Numbers in the draft are checked
### Prompt injection
## Anonymous reports
## Guard rails on the text
- Urgent (critical) words are matched as whole words and phrases (`llm.CRITICAL_PATTERNS`):
- Delays are read in minutes only ("800m" is a distance) and capped at 600 min.
- A model draft reply that mentions the bus staff, makes promises or claims causes
## Data model
## API
## Events
## Public service API (for other modules)
## Settings
## Frontend screens
## How to test
## Extension notes
- **Clustering and patterns** (next step): group open reports by route, kind and day; a weekly
- **Photos:** add an upload endpoint and a vision model (e.g. `qwen2.5vl:3b`, already installed).
- **Tamil:** switch `OLLAMA_MODEL` to a larger model and add the language to the write prompt.

## 6. Backend functions — what each one means


### `agent.py`
- `_max(a, b)` — Internal helper used by this module.
- `severity_floor(kind, text, claims, findings)` — Internal helper used by this module.
  - Code comment/docstring: The lowest urgency the rules allow. The model can raise it (up to high), never lower it.
- `async analyse(session, report_id)` — Runs report analysis using rules/evidence and optionally an AI model.

### `evidence.py`
- `_hm(dt)` — Internal helper used by this module.
- `plan_checks(kind, claims, trip)` — Validates a condition and returns the corresponding result.
- `_stop_event(trip, report, claims)` — Reads, filters, or handles an operational event.
  - Code comment/docstring: The stop the report is about: one the student names, else their allocated stop.

An anonymous report only ever uses a stop the student named: their allocated stop would
narrow down who sent it.
- `async gather(session, report, claims)` — Internal helper used by this module.
- `async _on_this_trip(session, report, claims, trip)` — Internal helper used by this module.
- `async _trip_ran(session, report, claims, trip)` — Internal helper used by this module.
- `async _lateness(session, report, claims, trip)` — Internal helper used by this module.
- `_late_text(minutes)` — Internal helper used by this module.
- `async _stop_reached(session, report, claims, trip)` — Internal helper used by this module.
- `async _crowding(session, report, claims, trip)` — Internal helper used by this module.
- `async _speed(session, report, claims, trip)` — Internal helper used by this module.
- `_words(text)` — Internal helper used by this module.
- `_cosine(a, b)` — Internal helper used by this module.
- `async lost_item_candidates(session, report, trip)` — Internal helper used by this module.
- `_lost_item_finding(candidates)` — Internal helper used by this module.

### `llm.py`
- `enabled()` — Internal helper used by this module.
- `async _post(path, body)` — Internal helper used by this module.
- `async _chat_json(system, user, schema, max_tokens)` — Internal helper used by this module.
- `async embed(text)` — Internal helper used by this module.
  - Code comment/docstring: Embedding for lost-and-found matching, or None when the model is off or unavailable.
- `_read_schema()` — Internal helper used by this module.
- `async read_claims(kind, text)` — Internal helper used by this module.
  - Code comment/docstring: (claims, engine). Uses the model when enabled, otherwise or on failure the rules.
- `merge_with_rules(model, rules, kind)` — Internal helper used by this module.
  - Code comment/docstring: Keyword rules are blunt but never miss an explicit phrase ("drove past", "touching").
When they found something more specific than the model's reading, theirs wins, so the matching
evidence check always runs. Extra checks are combined; the higher severity hint is kept.
- `has_critical_words(text)` — Internal helper used by this module.
- `rules_claims(kind, text)` — Internal helper used by this module.
- `_clip(s, n)` — Internal helper used by this module.
- `async write_up(kind, text, claims, findings)` — Internal helper used by this module.
- `unsupported_numbers(written, sources)` — Internal helper used by this module.
  - Code comment/docstring: Facts in the model's text that don't match the report or the findings: numbers and times
that appear nowhere in them, and place names cut short ("Perung:" for Perungudi). A small model
sometimes garbles these; a draft with a wrong fact in it is worse than the plainer template.
- `reply_breaks_rules(reply)` — Internal helper used by this module.
- `rules_writeup(kind, claims, findings)` — Internal helper used by this module.

### `router.py`
- `async trip_options(p, session)` — Internal helper used by this module.
- `async create_report(body, p, session)` — Creates a new record/resource.
- `async my_reports(p, session)` — Creates or processes a delay/problem report.
- `async follow_up(report_id, body, p, session)` — Internal helper used by this module.
- `async get_report(report_id, p, session)` — Fetches one record or a computed view.
- `async list_reports(status, kind, severity, _, session)` — Returns a list of records for the requested scope.
- `async reply(report_id, body, p, session)` — Internal helper used by this module.
- `async close(report_id, body, p, session)` — Internal helper used by this module.
- `async reanalyse(report_id, p, session)` — Runs report analysis using rules/evidence and optionally an AI model.
- `async match(report_id, found_item_id, p, session)` — Matches a report/lost item against another record.
- `async log_found_item(body, p, session)` — Creates or manages a found-item record.
- `async found_items(status, p, session)` — Creates or manages a found-item record.
- `async update_found_item(item_id, body, p, session)` — Updates an existing record/resource.

### `service.py`
- `async get_report(session, report_id)` — Fetches one record or a computed view.
- `async recipient_id(session, report_id)` — Internal helper used by this module.
  - Code comment/docstring: Who to notify about staff activity on a report (for the notifications module).
- `ensure_can_read(report, p)` — Internal helper used by this module.
- `async trip_options(session, student_id)` — Internal helper used by this module.
  - Code comment/docstring: The student's trips from the last few days that have departed (or should have):
trips they boarded, plus every trip on their allocated route (covers "the bus never came").
- `async create_report(session, p, data)` — Creates a new record/resource.
- `async add_student_message(session, report_id, p, body)` — Builds or handles the student-facing workflow.
- `async list_mine(session, student_id)` — Returns a list of records for the requested scope.
- `async list_reports(session)` — Returns a list of records for the requested scope.
  - Code comment/docstring: Newest first, with open critical and high reports pinned to the top.
- `async reply(session, report_id, p, body)` — Internal helper used by this module.
- `async close(session, report_id, p, note)` — Internal helper used by this module.
- `async request_analysis(session, report_id, p)` — Internal helper used by this module.
- `async match_found_item(session, report_id, item_id, p)` — Matches a report/lost item against another record.
- `async pending_analysis(session, older_than_seconds)` — Ends/completes a trip or workflow.
  - Code comment/docstring: Reports whose analysis never finished (e.g. the server restarted mid-way).
- `async _messages(session, report_id)` — Internal helper used by this module.
- `async _base(session, r, lk, with_messages)` — Internal helper used by this module.
- `async student_views(session, reports)` — Builds or handles the student-facing workflow.
- `async student_view(session, r)` — Builds or handles the student-facing workflow.
- `async admin_views(session, reports)` — Builds the admin-facing data/view.
- `async admin_view(session, r)` — Builds the admin-facing data/view.
- `async get_found_item(session, item_id)` — Fetches one record or a computed view.
- `async log_found_item(session, p, trip_id, description)` — Creates or manages a found-item record.
- `async list_found_items(session, p, status)` — Returns a list of records for the requested scope.
- `async set_found_item_status(session, item_id, status, p)` — Creates or manages a found-item record.
- `async found_item_view(session, item)` — Creates or manages a found-item record.

## 7. Frontend functions/screens — what each one means


### `frontend/lib/modules/reports/data/reports_api.dart`
- `ReportKind()` — Creates or processes a delay/problem report.
- `ReportMessage()` — Creates or processes a delay/problem report.
- `Finding()` — Internal helper used by this module.
- `MatchCandidate()` — Matches a report/lost item against another record.
- `Analysis()` — Internal helper used by this module.
- `Report()` — Creates or processes a delay/problem report.
- `TripOption()` — Internal helper used by this module.
- `FoundItem()` — Creates or manages a found-item record.
- `asList()` — Returns a list of records for the requested scope.
- `submit()` — Internal helper used by this module.
- `followUp()` — Internal helper used by this module.
- `reply()` — Internal helper used by this module.
- `close()` — Internal helper used by this module.
- `reanalyse()` — Runs report analysis using rules/evidence and optionally an AI model.
- `match()` — Matches a report/lost item against another record.
- `logFoundItem()` — Creates or manages a found-item record.
- `markReturned()` — Internal helper used by this module.

### `frontend/lib/modules/reports/screens/admin_issue_screen.dart`
- `AdminIssueScreen()` — Builds the admin-facing data/view.
- `dispose()` — Internal helper used by this module.
- `Function()` — Internal helper used by this module.
- `action()` — Internal helper used by this module.
- `_close()` — Internal helper used by this module.
- `_run()` — Internal helper used by this module.
- `SignNotice()` — Internal helper used by this module.
- `_Facts()` — Internal helper used by this module.
- `_AgentPanel()` — Internal helper used by this module.
- `if()` — Internal helper used by this module.
- `TextSpan()` — Internal helper used by this module.
- `setState()` — Internal helper used by this module.
- `listenLive()` — Returns a list of records for the requested scope.
- `SignHeader()` — Internal helper used by this module.

### `frontend/lib/modules/reports/screens/admin_issues_screen.dart`
- `AdminIssuesScreen()` — Builds the admin-facing data/view.
- `_reports()` — Creates or processes a delay/problem report.
- `severityColor()` — Internal helper used by this module.
- `_IssueRow()` — Internal helper used by this module.
- `_FoundItems()` — Creates or manages a found-item record.
- `_returned()` — Internal helper used by this module.
- `showLogFoundItemDialog()` — Creates or manages a found-item record.
- `_Tab()` — Internal helper used by this module.
- `listenLive()` — Returns a list of records for the requested scope.
- `final()` — Internal helper used by this module.
- `TextButton()` — Internal helper used by this module.

### `frontend/lib/modules/reports/screens/my_reports_screen.dart`
- `MyReportsScreen()` — Creates or processes a delay/problem report.
- `_Row()` — Internal helper used by this module.
- `StudentReportScreen()` — Builds or handles the student-facing workflow.
- `dispose()` — Internal helper used by this module.
- `_send()` — Ends/completes a trip or workflow.
- `_Bubble()` — Internal helper used by this module.
- `setState()` — Internal helper used by this module.
- `SignHeader()` — Internal helper used by this module.

### `frontend/lib/modules/reports/screens/report_form_screen.dart`
- `ReportFormScreen()` — Creates or processes a delay/problem report.
- `dispose()` — Internal helper used by this module.
- `_send()` — Ends/completes a trip or workflow.
- `SnackBar()` — Internal helper used by this module.
- `SignNotice()` — Internal helper used by this module.
- `setState()` — Internal helper used by this module.
- `if()` — Internal helper used by this module.
- `SignHeader()` — Internal helper used by this module.

### `frontend/lib/modules/reports/widgets/choice.dart`
- `Choice()` — Internal helper used by this module.

### `frontend/lib/modules/reports/widgets/log_found_item_sheet.dart`
- `showLogFoundItemSheet()` — Creates or manages a found-item record.
- `_LogFoundItemSheet()` — Creates or manages a found-item record.
- `dispose()` — Internal helper used by this module.
- `_save()` — Internal helper used by this module.
- `setState()` — Internal helper used by this module.
- `TextField()` — Internal helper used by this module.

## 8. Module flow

```text
Student submits report
→ save report
→ background analysis
→ model/rules extract claim
→ evidence.py checks real records
→ severity floor applied
→ draft returned to admin
→ admin replies/closes/matches found items
```

## 9. Main code lines and their meaning


### `backend/app/modules/reports/router.py`
- **Line 1:** `from fastapi import APIRouter, Depends`
  - Meaning: Imports a library/module used by this file.
- **Line 2:** `from sqlalchemy.ext.asyncio import AsyncSession`
  - Meaning: Imports a library/module used by this file.
- **Line 4:** `from app.core.config import settings`
  - Meaning: Imports a library/module used by this file.
- **Line 5:** `from app.core.db import get_session`
  - Meaning: Imports a library/module used by this file.
- **Line 6:** `from app.core.deps import Principal, require_roles`
  - Meaning: Imports a library/module used by this file.
- **Line 7:** `from app.core.ratelimit import RateLimiter`
  - Meaning: Imports a library/module used by this file.
- **Line 8:** `from app.core.roles import Role`
  - Meaning: Imports a library/module used by this file.

### `backend/app/modules/reports/service.py`
- **Line 12:** `from datetime import timedelta`
  - Meaning: Imports a library/module used by this file.
- **Line 14:** `from sqlalchemy import case, select`
  - Meaning: Imports a library/module used by this file.
- **Line 15:** `from sqlalchemy.ext.asyncio import AsyncSession`
  - Meaning: Imports a library/module used by this file.
- **Line 17:** `from app.core import events`
  - Meaning: Imports a library/module used by this file.
- **Line 18:** `from app.core.config import settings`
  - Meaning: Imports a library/module used by this file.
- **Line 19:** `from app.core.deps import Principal`
  - Meaning: Imports a library/module used by this file.
- **Line 20:** `from app.core.errors import Forbidden, InvalidState, NotFound`
  - Meaning: Imports a library/module used by this file.

### `backend/app/modules/reports/__init__.py`
- **Line 3:** `import asyncio`
  - Meaning: Imports a library/module used by this file.
- **Line 5:** `from fastapi import FastAPI`
  - Meaning: Imports a library/module used by this file.
- **Line 7:** `from app.core import events, tasks`
  - Meaning: Imports a library/module used by this file.
- **Line 8:** `from app.core.db import SessionLocal`
  - Meaning: Imports a library/module used by this file.
- **Line 9:** `from app.core.realtime import hub`
  - Meaning: Imports a library/module used by this file.
- **Line 10:** `from app.core.roles import Role`
  - Meaning: Imports a library/module used by this file.
- **Line 11:** `from app.modules.reports import agent, llm, service`
  - Meaning: Imports a library/module used by this file.

### `backend/app/modules/reports/agent.py`
- **Line 8:** `import logging`
  - Meaning: Imports a library/module used by this file.
- **Line 10:** `from sqlalchemy.ext.asyncio import AsyncSession`
  - Meaning: Imports a library/module used by this file.
- **Line 12:** `from app.core import events`
  - Meaning: Imports a library/module used by this file.
- **Line 13:** `from app.core.timeutil import now_utc`
  - Meaning: Imports a library/module used by this file.
- **Line 14:** `from app.modules.reports import evidence, llm`
  - Meaning: Imports a library/module used by this file.
- **Line 15:** `from app.modules.reports.models import AnalysisStatus, Report, ReportKind, ReportSeverity`
  - Meaning: Imports a library/module used by this file.
- **Line 16:** `from app.modules.reports.schemas import Analysis, Claims, Finding`
  - Meaning: Imports a library/module used by this file.

### `backend/app/modules/reports/evidence.py`
- **Line 7:** `import math`
  - Meaning: Imports a library/module used by this file.
- **Line 8:** `import re`
  - Meaning: Imports a library/module used by this file.
- **Line 9:** `from datetime import timedelta`
  - Meaning: Imports a library/module used by this file.
- **Line 11:** `from sqlalchemy import func, or_, select`
  - Meaning: Imports a library/module used by this file.
- **Line 12:** `from sqlalchemy.ext.asyncio import AsyncSession`
  - Meaning: Imports a library/module used by this file.
- **Line 14:** `from app.core.config import settings`
  - Meaning: Imports a library/module used by this file.
- **Line 15:** `from app.core.timeutil import local_tz, minutes_between, now_utc`
  - Meaning: Imports a library/module used by this file.

### `frontend/lib/modules/reports/data/reports_api.dart`
- **Line 1:** `import 'package:flutter_riverpod/flutter_riverpod.dart';`
  - Meaning: Imports a library/module used by this file.
- **Line 3:** `import '../../../core/api/api_client.dart';`
  - Meaning: Imports a library/module used by this file.
- **Line 4:** `import '../../../core/format.dart';`
  - Meaning: Imports a library/module used by this file.
- **Line 7:** `enum ReportKind {`
  - Meaning: Important setup or business-logic line used by this module.
- **Line 14:** `const ReportKind(this.label);`
  - Meaning: Important setup or business-logic line used by this module.
- **Line 16:** `final String label;`
  - Meaning: Important setup or business-logic line used by this module.
- **Line 23:** `class ReportMessage {`
  - Meaning: Defines a class that groups related data or behavior.

## 10. Beginner takeaway

Remember this module as: **reports = the part of the system responsible for its own business job**. The router receives the request, the service does the work, the database stores the result, and domain events let other modules react without tightly coupling everything together.