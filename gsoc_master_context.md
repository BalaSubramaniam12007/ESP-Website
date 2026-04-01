# GSoC 2026 — ESP-Website Admin Onsite Webapp
## Master Context Document — All Ideas, Analysis, Architecture & Prompts
> Use as memory context for Claude Opus, GitHub Copilot, or any AI agent session.
> Do not skip any section — every section feeds into the proposal.

---

## SECTION 1: PROJECT IDENTITY

| Field | Value |
|---|---|
| Issue | #2672 — Admin Onsite Webapp |
| Repo | https://github.com/learning-unlimited/ESP-Website |
| Mentor | @willgearty |
| Program | GSoC 2026 |
| Milestone | Stable Release 17 (due August 1, 2026) |
| Labels | Onsite, UserInterface, Webapp |

### Related Issues (ALL Must Be Read)
| Issue | Title | Status | Relevance |
|---|---|---|---|
| #2507 | Student Onsite Webapp | MERGED | Reference implementation — follow this pattern |
| #2671 | Teacher Onsite Webapp | CLOSED via #2866 | Reference implementation |
| #2895 | Module opening/closing times | OPEN | Architectural opportunity — admin webapp is its runtime layer |
| #3810 | Theming overhaul / Bootstrap upgrade | OPEN | Same milestone — build on this stack |
| #3854 | New module management UI | OPEN | Same milestone — admin webapp is day-of counterpart |
| #142 | iPhone/Android app | CLOSED | 'Searchable admin binder' request lives here |

---

## SECTION 2: ISSUE DISCUSSION — FULL SUMMARY

### Mentor's Original Proposal (Feb 2019)
- Live 'big board' style registration view — list of classes with progress bars (enrollments + check-ins), auto-refreshing
- Per-class toggle to open/close registration on the fly
- On-the-fly class cap changes
- Change overenrollment settings on the fly (overenrollment on/off, use enrollment vs check-in numbers)
- UI structure: items 1+2 on same page, cap changes via click-through mini class page, settings on separate tab

### Mentor's SplashCon 2021 Additions
- Dashboard with quick stats about reg/attendance/etc
- Check student attendance (who's playing hooky?)
- Check in teachers
- Check in students
- Easy AJAX search for individual users (teacher/student info, schedule, record, attendance, check-in)

### Mentor's Explicit Positions on Key Topics
| Topic | Mentor's Position |
|---|---|
| Transactional safety | Cap changes should be immediate for all users. Handle concurrent edits gracefully. |
| Audit logging | NOT important. Admin teams are small and communicate well. |
| DB load | Reducing DB load during large programs is VERY important. |
| Priority | Usability of interface + smooth integration with day-of activities |
| Feature bloat | Prefer reliability, stability, up-to-date data over extra features |
| Task management | Don't reinvent the wheel — existing apps handle this |
| UI design | No fixed design. Similar to student/teacher webapps. Open to proposals. |
| Cap editing | Editing raw class_size_max / max_class_capacity values is fine |
| Cache reuse | Fine to use same cache as OnSiteAttendance |
| Changelog | Use AJAXChangeLogEntry in module_ext.py (line 257) as reference |
| Scope | MVP must cover ENTIRE program regardless of length (not just one day) |
| Coding before GSoC | Do NOT write code before internship begins |
| Freshness indicator | 'Last refreshed X seconds ago' is a good strategy |
| Graceful degradation | Handle polling failures gracefully — non-blocking warning |
| Refactoring | Everything within limits. Goal: avoid duplicating existing logic. |

### Contributor Ideas (Summary)
- **@jacklat**: Task management system, green/yellow/red class status → MENTOR REJECTED
- **@kkbrum**: 'Online instant searchable admin binder' (from closed #142)
- **@vanshaj2023** (best technical contributor):
  - Cap editing complexity: class_cap_multiplier and class_cap_offset affect capacity separately
  - OnSiteAttendance already has ~105s TTL cache — reuse instead of new queries
  - Group classes by timeslot using AdminClass.timeslot_counts() — flat list won't scale for 200+ classes
- **@arpit10128**: Freshness indicator, graceful degradation → MENTOR ENDORSED BOTH
- **@sameer0297**: Transactional safety, aggregated queries → MENTOR ENDORSED DB optimization
- **@rafiya618**: Asked for UI prototype → MENTOR: no fixed design, open to proposals

---

## SECTION 3: CODEBASE ANALYSIS

### Key Files to Study
| File | What It Contains |
|---|---|
| `esp/esp/program/modules/module_ext.py` | AJAXChangeLogEntry (line 257), ProgramModuleObj — module registration |
| `esp/esp/program/modules/handlers/onsitecore.py` | OnSiteAttendance — times_checked_in(), times_attending_class(), ~105s TTL cache |
| `esp/esp/program/modules/handlers/studentwebapp.py` | Student webapp — reference implementation |
| `esp/esp/program/modules/handlers/teacherwebapp.py` | Teacher webapp — reference implementation |
| `esp/esp/program/models/class_.py` | ClassSection, ClassSubject — class_size_max, max_class_capacity |
| `esp/esp/program/modules/handlers/studentreg.py` | StudentClassRegModuleInfo — class_cap_multiplier, class_cap_offset |
| `esp/esp/program/modules/handlers/adminclass.py` | AdminClass utility — timeslot_counts() method |

### Key Models
| Model | What It Represents |
|---|---|
| ClassSection | A specific section of a class. Has max_class_capacity (section level). |
| ClassSubject | The parent class/subject. Has class_size_max (subject level). |
| StudentRegistration | A student's relationship to a section (Enrolled, Attended, etc.). |
| RegistrationType | The type — Enrolled vs Attended (checked-in). |
| StudentClassRegModuleInfo | Holds class_cap_multiplier and class_cap_offset. |
| AJAXChangeLogEntry | Tracks changes. Located at module_ext.py line 257. |
| ProgramModuleObj | Base class for all program modules. Admin webapp extends this. |

### Cap Math (CRITICAL — Only @vanshaj2023 caught this)
```
effective_cap = class_size_max * class_cap_multiplier + class_cap_offset
```
- `class_size_max` → subject level (ClassSubject)
- `max_class_capacity` → section level (ClassSection)
- **Edit strategy**: Edit raw class_size_max / max_class_capacity directly (mentor confirmed)
- **Display strategy**: Show computed effective cap to admin
- **WARNING**: Never let admin edit effective cap directly — breaks multiplier/offset

### Existing Cache Layer
- Location: OnSiteAttendance in onsitecore.py
- Methods: times_checked_in(), times_attending_class()
- TTL: ~105 seconds
- Scope: **Per-student** — NOT per-class aggregate
- Strategy: Reuse for dashboard. Don't create new queries.
- Bonus: BigBoard also has aggregation path — share same cache key to halve DB hits

---

## SECTION 4: CREATIVE & DIFFERENTIATED IDEAS

### BIGGEST DIFFERENTIATOR: Shared Service Layer

**Instead of** building admin webapp as another self-contained module duplicating query logic:

**Propose** extracting `esp/program/services/onsite_data.py` — a shared service layer all three webapps draw from:

```python
# esp/program/services/onsite_data.py

def get_program_status(program):
    """Used by: Admin webapp dashboard. Returns all classes with counts, grouped by timeslot."""
    pass

def get_student_schedule(user, program):
    """Used by: Student webapp. Returns student's classes, times, rooms."""
    pass

def get_teacher_schedule(user, program):
    """Used by: Teacher webapp. Returns teacher's classes and students."""
    pass

def get_class_detail(section_id):
    """Used by: Admin webapp click-through. Returns full detail with roster, cap info."""
    pass
```

**Why it wins:**
- Directly addresses mentor's goal: "avoid duplicating existing logic"
- Refactors existing student + teacher webapps while building the third
- Lasting contribution — one place to maintain
- Nobody else proposed this

---

### SECOND DIFFERENTIATOR: Delta-Based Polling

**Every other applicant**: polling returns full class list every N seconds → heavy DB load at peak

**Your proposal**: delta-based polling
```
First poll:  GET /admin_webapp/status              → full dataset
Next polls:  GET /admin_webapp/status?since=<ts>   → only changes
```
- Reduces payload by ~90% after first load
- Nobody in the thread proposed this
- Directly answers mentor's DB load concern better than anyone

---

### THIRD DIFFERENTIATOR: Cross-Issue Connection

**Nobody else drew this map:**
```
#2895 — Module opening/closing times
   ↓ feeds into
#3854 — New Module Management UI
   ↓ missing the runtime layer ← YOUR PROJECT
#2672 — Admin Onsite Webapp (completes the system)
   ↓ builds on
#2507/#2866 — Student + Teacher Onsite Webapps
   ↓ shares theming with
#3810 — Theming Overhaul (Bootstrap upgrade)
```

All five issues are in the **same Stable Release 17 milestone**. The admin webapp is the runtime control layer that #2895 and #3854 lack during live events.

---

### UI/UX Creative Ideas

#### 1. Traffic Light Heat Map
Color band on left edge of each class row:
- 🟢 Green: < 70% full
- 🟡 Yellow: 70–90% full
- 🔴 Red: > 90% or over cap
**Cost:** One CSS class per row. Zero extra DB work.

#### 2. "Needs Attention" Swimlane
Auto-populated panel at top of dashboard:
- Teacher hasn't checked in but class has started
- Enrollment is 0 with 15+ min until start
- Cap was just exceeded
**Cost:** Pure client-side filtering on data already in polling response. No new queries.

#### 3. Sticky Current Timeslot
Current timeslot's class group stays pinned/sticky at viewport top as time passes.
Past timeslots collapse automatically. Future timeslots remain collapsed but visible.
**Cost:** Pure CSS/JS — `position: sticky` with time-comparison on timeslot start/end fields.

#### 4. Quick-Search Overlay
Floating button (bottom-right corner, mobile-friendly) → opens modal → type name → get schedule + check-in status instantly via AJAX.
**Origin:** 'Searchable admin binder' from closed issue #142 + willgearty's 2021 SplashCon notes.
**Cost:** One endpoint, one modal.

#### 5. Overenrollment Settings as Live Toggle Panel
Settings tab with 2-3 sliders/toggles mapping directly to StudentClassRegModuleInfo fields.
**Origin:** Directly addresses willgearty's original 2019 request: 'change overenrollment settings on the fly.'

---

### Optimization Ideas

#### Single Annotated Queryset
```python
from django.db.models import Count, Q

sections = ClassSection.objects.filter(
    parent_class__parent_program=program,
    status=10  # approved/active only
).select_related(
    'parent_class',
    'parent_class__parent_program'
).prefetch_related(
    'meeting_times',
    'classroomassignment_set__room'
).annotate(
    enrollment_count=Count(
        'studentregistration',
        filter=Q(studentregistration__relationship__name='Enrolled',
                 studentregistration__expired=False)
    ),
    checkin_count=Count(
        'studentregistration',
        filter=Q(studentregistration__relationship__name='Attended')
    )
).order_by('meeting_times__start')
```

| Approach | Queries for 200 classes |
|---|---|
| Naive (loop + count per class) | 801 queries per refresh |
| Single annotated queryset | 3 queries total |

- `select_related` = SQL JOIN for ForeignKey fields. One query instead of N.
- `prefetch_related` = Optimized separate query for ManyToMany/reverse FK. 2 queries instead of 200.
- `annotate` = Adds computed COUNT columns — database computes, not Python.
- Result: every section object has `.enrollment_count` and `.checkin_count` as attributes. No extra queries in loop.

#### Debounced Cap Edits
- Debounce AJAX write 800ms after last keystroke
- Prevents firing DB write for every digit typed
- Smoother than button-confirm flow
- Nobody else proposed inline debounced edit

#### Progressive Hydration
- Initial page load: pre-rendered HTML shell with skeleton-loaded class rows
- First AJAX poll: hydrates actual data
- Admin sees page structure instantly on slow event-day WiFi

---

## SECTION 5: NAVIGATION STRUCTURE (PROPOSED)

```
Tab 1: Dashboard
  ├── Program-level stats (total enrolled, total checked in, classes with issues)
  ├── "Needs Attention" swimlane (auto-filtered from live data)
  ├── Current timeslot classes (sticky, open by default, traffic light colors)
  └── Other timeslot groups (collapsed)

Tab 2: Classes (click-through from dashboard)
  ├── Class detail: full enrollment list, check-in status per student
  ├── Cap editor (raw value, debounced, displays effective cap)
  ├── Open/close registration toggle
  └── Teacher check-in status

Tab 3: Students
  ├── AJAX search by name
  └── Student info: schedule, check-in record, attendance history

Tab 4: Teachers
  ├── AJAX search by name
  └── Teacher info: classes being taught, check-in status

Tab 5: Settings
  ├── Overenrollment on/off toggle
  ├── Overenrollment percentage buffer
  └── Count by enrollment vs check-ins toggle
```

---

## SECTION 6: WHAT NOT TO BUILD (MENTOR EXPLICITLY REJECTED)

- WebSockets in MVP — polling is sufficient
- Audit logging — mentor said not important
- Task/issue management system — existing tools handle this
- Chat feature — feature bloat
- Note-taking — feature bloat
- Coding before GSoC begins — mentor explicitly told contributors NOT to do this

---

## SECTION 7: PROPOSAL WRITING STRUCTURE

### Section 1 — Problem Statement
- Angle: Operational pain, not technical problem
- Open with what a Splash admin experiences today without this tool
- Missing teachers, overenrolled classes, students in wrong rooms, multi-click admin panel
- Closing: "The goal isn't just to build a dashboard — it's to reduce cognitive load during high-pressure moments."

### Section 2 — Cross-Issue Context
- Connect #2672 to #2895, #3854, #3810, #2507/#2866
- Admin webapp is the runtime control layer those issues lack

### Section 3 — Architecture
- Shared onsite service layer
- Delta-based polling
- Single annotated queryset (include actual code)
- AJAXChangeLogEntry integration
- Name specific classes, methods, file paths

### Section 4 — UI Design
- 5-tab navigation structure
- 5 creative UX ideas with which are MVP vs stretch goals

### Section 5 — What I'm Not Building and Why
- WebSockets, audit logs, task management — explain why each is excluded

### Section 6 — Timeline
| Week | Deliverable |
|---|---|
| 1 | Study student/teacher webapps, document shared query patterns, write test scaffold |
| 2 | Understand module_ext.py, AJAXChangeLogEntry, ProgramModuleObj registration |
| 3–4 | Shared service layer (onsite_data.py) + aggregated status endpoint + caching |
| 5–6 | Dashboard tab: timeslot grouping, progress bars, polling, freshness indicator, failure handling |
| 7–8 | Class detail view: cap editor, open/close toggle, concurrency handling |
| 9–10 | Students tab: AJAX user search, schedule + check-in history |
| 11–12 | Teachers tab + Settings tab (overenrollment toggles) |
| 13 | Mobile responsiveness, Bootstrap/theming alignment (#3810), documentation, tests |

### Section 7 — Risk Analysis
| Risk | Mitigation |
|---|---|
| DB load under peak traffic | Delta polling + single annotated queryset + BigBoard cache sharing |
| Concurrent cap edits | select_for_update() + AJAXChangeLogEntry + debounced writes |
| Bootstrap theming alignment | Build on upgraded Bootstrap stack from #3810 from day one |

### Winning Closing Line
> "This project doesn't just add a third webapp — it creates the shared data service layer that makes all three onsite webapps maintainable as one cohesive system, while giving admins a reliable, mobile-first command center for the most chaotic hours of a Splash program."

---

## SECTION 8: COPILOT / OPUS PROMPTS

### Phase 1 — Codebase Exploration

**1.1 Module System**
```
Read esp/esp/program/modules/module_ext.py and explain:
1. How ProgramModuleObj works and what a new module must implement
2. How AJAXChangeLogEntry is structured — what fields it stores, what it tracks (line 257+)
3. How an admin webapp module would register itself into this system
Be specific about class names, field names, and method signatures.
```

**1.2 OnSiteAttendance Cache**
```
Read esp/esp/program/modules/handlers/onsitecore.py and find OnSiteAttendance.
Explain:
1. What times_checked_in() and times_attending_class() return
2. What the ~105s TTL cache covers — per class, per student, or per program?
3. How many DB queries triggered if called for 200 classes
4. Whether I can reuse this cache for a polling dashboard
Show relevant code with line numbers.
```

**1.3 Cap Math**
```
Search across studentreg.py, class_.py, module_ext.py.
Find and explain:
1. What class_cap_multiplier and class_cap_offset do
2. What class_size_max (subject) vs max_class_capacity (section) control
3. The exact formula for effective cap
4. What ClassManageForm.save_data() writes and where
5. Whether editing class_size_max directly breaks multiplier/offset logic
Show worked example: base=20, multiplier=1.5, offset=5
```

**1.4 Webapp Pattern**
```
Read the student onsite webapp files. Answer:
1. How does it register its URLs?
2. How does it serve initial page vs AJAX data endpoints?
3. What does its JSON response structure look like?
4. What JavaScript pattern does it use for polling/refresh?
5. What can I reuse vs what needs to be written fresh for admin webapp?
```

**1.5 AdminClass Timeslot**
```
Find AdminClass in ESP-Website and locate timeslot_counts().
Explain:
1. What timeslot_counts() returns
2. How classes are associated with timeslots in DB schema
3. Whether I can group 200+ classes by timeslot in a single query
4. What select_related/prefetch_related/annotate to use for all class data in 1-2 DB hits
```

### Phase 2 — Request Lifecycle Tracing

**2.1 Cap Change End-to-End**
```
Trace the full request lifecycle for an admin changing a class cap.
Follow: URL → view → form → model fields written → cache invalidation → signals → HTTP response.
Show each step with file path and line numbers.
```

**2.2 Student Check-in**
```
Trace the full request lifecycle for checking in a student.
I want to understand:
1. Which model/field records a check-in
2. Whether check-in is atomic/transactional
3. What happens if check-in and cap change happen simultaneously
4. Whether OnSiteAttendance cache is invalidated on check-in or just TTL-expires
```

**2.3 Race Condition**
```
Two admins simultaneously change the cap of the same class section.
Look at current cap-editing code and tell me:
1. Is there any optimistic locking or select_for_update() in place?
2. What is the actual race condition risk?
3. How does AJAXChangeLogEntry get written?
4. What is the minimal change needed to handle this gracefully?
```

### Phase 3 — Endpoint Design

**3.1 Aggregated Status Endpoint**
```
Design a single aggregated AJAX polling endpoint for admin onsite dashboard.
Must return all class data for a program in one response.
Design: URL pattern, Django view, queryset using select_related/prefetch_related/annotate,
JSON schema (timeslot grouping, class data, enrollment/cap/checkin counts, teacher status),
cache integration.
Output as actual Python/Django code with optimization comments.
```

**3.2 Cap Edit Endpoint**
```
Design a Django AJAX view for editing class cap from admin webapp.
Requirements: write to raw values, immediate visibility, AJAXChangeLogEntry hook,
handle concurrent edits gracefully.
Write actual Django view code with auth check, validation, transaction, JSON response.
```

**3.3 Registration Toggle Endpoint**
```
Find where registration open/closed per-class is controlled in ESP-Website.
Design the toggle endpoint: permission check, atomic write, JSON response with new state.
Note any side effects.
```

**3.4 Delta Polling**
```
Design a delta-based polling endpoint:
- First call: GET /admin_webapp/status → full dataset with timestamp
- Next calls: GET /admin_webapp/status?since=<timestamp> → only changes
Implement in Django with timestamp comparison logic, delta data structure,
client-side merge strategy, fallback to full refresh if delta too large.
```

**3.5 Shared Service Layer**
```
Create esp/program/services/onsite_data.py as shared service for all three onsite webapps.
Implement:
1. get_program_status(program) — all classes with counts, grouped by timeslot
2. get_student_schedule(user, program) — student's classes, times, rooms
3. get_teacher_schedule(user, program) — teacher's classes, students
4. get_class_detail(section_id) — full detail, roster, cap info
Use select_related, prefetch_related, annotate throughout.
Add docstrings with return type and query count per function.
```

### Phase 4 — Edge Cases

**4.1 Multi-Section Cap**
```
ClassSubject can have multiple ClassSections.
Walk through every edge case for the cap editor:
1. Does editing class_size_max affect all sections equally?
2. If sections have different max_class_capacity, what does dashboard display?
3. What happens if cap is lowered below current enrollment?
4. What is the correct behavior?
```

**4.2 Polling Failure State Machine**
```
Design client-side failure handling for polling dashboard:
1. Single poll times out
2. Three consecutive polls fail
3. Server returns 500
4. Stale data (timestamp unchanged)
5. Brief network loss then reconnect

For each: what UI shows, whether last state is preserved, whether polling continues.
Write as JavaScript state machine with states: LIVE, STALE, DEGRADED, FAILED.
```

**4.3 DB Load Model**
```
Model DB load for 200-class program with dashboard polling:
1. Requests per minute across N admin devices
2. Query cost: aggregated endpoint vs naive per-class
3. At what point does polling frequency need to drop?
4. Recommended poll interval and cache TTL for MVP
Give concrete numbers.
```

### Phase 5 — Proposal Writing

**5.1 Validate Timeline** — Paste your week-by-week plan, ask for scoring and feedback.

**5.2 Stress Test Architecture** — Paste your architecture, ask a skeptical senior Django engineer to find holes.

**5.3 Technical Section** — Write the 400-600 word technical section grounded in real ESP codebase classes and file names.

**5.4 Problem Statement** — Write 150-word operational problem statement, no tech jargon.

**5.5 Final Review** — Paste full proposal, get scored 1-5 per section with prioritized change list.

---

## SECTION 9: PRE-PROPOSAL SELF-CHECK CHECKLIST

- [ ] I can describe how a new webapp module gets registered in the ESP module system
- [ ] I know the difference between class_size_max, max_class_capacity, class_cap_multiplier, and class_cap_offset
- [ ] I know what AJAXChangeLogEntry stores and how to write a new entry
- [ ] I know what OnSiteAttendance's cache covers and what TTL it uses
- [ ] I know how student webapp and teacher webapp differ structurally
- [ ] I can describe the single annotated queryset that avoids N+1 queries
- [ ] I know what happens to enrollment when a cap is lowered below current enrollment count
- [ ] I have a concrete navigation structure with named tabs and content
- [ ] I know whether 'program scope' means one day or all days (ANSWER: all days per willgearty)
- [ ] I have a week-by-week timeline where each week has a specific testable deliverable
- [ ] I have connected #2672 to at least three other Stable Release 17 issues
- [ ] I understand the delta-based polling pattern and why it beats full-refresh polling
- [ ] I can explain the shared service layer and why it benefits existing webapps too

---

## SECTION 10: META-PRINCIPLES

1. **Let the code answer your questions.** Every other applicant asks willgearty questions answerable by reading the code. Ask willgearty only about genuine design ambiguities.

2. **Open with operational pain.** Most proposals are feature lists. Yours should open with what running a Splash program actually feels like without this tool.

3. **The trilogy story.** No other applicant connected student webapp + teacher webapp + admin webapp into a single architectural story with a shared service layer.

4. **Delta polling.** Proposed by nobody. Directly answers the DB load concern better than any other answer in the thread.

5. **Cross-issue connection.** Nobody connected #2895, #3854, #3810, #2507, #2866 to #2672. Shows you understand willgearty's bigger vision for Stable Release 17.

6. **Show the code.** Include the annotated queryset as actual Python/Django code in your proposal — not described, coded. Shows advanced ORM knowledge.

7. **Before submitting:** Fix at least one small bug from open issues (Onsite or UserInterface label). One merged PR puts you above 90% of applicants who only commented.

8. **The winning line:** "This project doesn't just add a third webapp — it creates the shared data service layer that makes all three onsite webapps maintainable as one cohesive system, while giving admins a reliable, mobile-first command center for the most chaotic hours of a Splash program."
