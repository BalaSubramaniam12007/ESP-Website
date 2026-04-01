# GSoC 2026 — Admin Onsite Webapp Ideas Catalog

Source basis: [gsoc_master_context.md](gsoc_master_context.md)

This file consolidates all major ideas discussed around Issue #2672 and groups them by priority, originality, and mentor alignment.

---

## 1) Core Mentor-Requested Ideas (Must Have)

1. **Live admin dashboard for classes**
	- Show class-by-class progress bars for:
	  - enrollment
	  - check-ins/attendance

2. **On-the-fly registration control**
	- Per-class open/close toggle
	- Add confirmation friction to prevent accidental taps

3. **On-the-fly cap updates**
	- Edit raw cap values directly
	- Reflect changes immediately for day-of operations

4. **On-the-fly overenrollment settings**
	- Toggle whether overenrollment is allowed
	- Toggle whether counts use enrollment or check-in basis

5. **Dashboard summary information**
	- Quick stats for registration + attendance

6. **Admin searchability / quick lookup**
	- Fast AJAX lookup for student/teacher info
	- Include schedule and check-in related context

---

## 2) Mentor-Endorsed Technical Direction

1. **Polling-first MVP** (not websocket-first)
2. **Reduce DB load aggressively** (critical for large programs)
3. **Graceful degradation **
4. **Show freshness information** (e.g., last refreshed)
5. **Handle concurrent edits gracefully**
6. **Reuse/refactor existing logic** (avoid duplication)
7. **Reuse existing cache paths where possible**
8. **Scope covers whole program duration** (day/weekend/multi-weekend)

---

## 3) High-Value Contributor Ideas from Thread

1. **Timeslot grouping for scale**
	- Better than flat lists for 200+ classes
	- Reuse existing `AdminClass.timeslot_counts()` patterns

2. **Cap math awareness**
	- Distinguish raw values vs effective capacity
	- Avoid editing computed effective cap directly

3. **Staleness + reliability UX**
	- Explicit stale/refresh indicators
	- Non-blocking warning state on update failure

4. **Server-side aggregation over many per-class queries**
	- One optimized query/endpoint per refresh cycle

---

## 4) Differentiated Ideas (Proposal Standout)

1. **Shared service layer for all onsite webapps**
	- Introduce `esp/program/services/onsite_data.py`
	- Shared data backbone for student + teacher + admin webapps
	- Reduces duplication and future maintenance cost

2. **Delta-based polling**
	- First call returns full payload
	- Subsequent calls send only changes since timestamp
	- Strong DB/network efficiency advantage under load

3. **Cross-issue systems story**
	- Position #2672 as runtime layer connecting:
	  - module timing/config issues
	  - module management UI
	  - existing student/teacher webapps
	  - release theming work

---

## 5) UX Ideas for Fast Onsite Operations

1. **Needs Attention swimlane**
	- Auto-highlight classes with urgent conditions

2. **Traffic-light occupancy cues**
	- Green/Yellow/Red fullness thresholds for instant scan

3. **Sticky current timeslot**
	- Keep currently running block visible while scrolling

4. **Quick-search modal (admin binder behavior)**
	- Fast user lookup with schedule/check-in details

5. **Mini class drill-down page**
	- Dashboard for monitor + lightweight click-through for edits

---

## 6) Performance/Robustness Ideas

1. **Single annotated queryset** with `select_related`, `prefetch_related`, `annotate`
2. **Aggregated status endpoint** returning timeslot + class metrics in one response
3. **Adaptive polling cadence** (active tab faster, background slower)
4. **Failure state model** (`LIVE`, `STALE`, `DEGRADED`, `FAILED`)
5. **Conflict-safe write path** for simultaneous admin edits

---

## 7) Ideas to Avoid (Rejected / De-prioritized)

1. Full task-management system
2. Feature-bloat extras (chat, notes, unrelated collaboration tools)
3. Heavy real-time architecture in MVP (websocket-first)
4. Over-indexing on audit logging in initial MVP

---

## 8) Suggested MVP Packaging (Clean and Mentor-Aligned)

### Tab 1: Dashboard
- summary stats
- timeslot grouped classes
- progress bars + attention flags

### Tab 2: Classes
- per-class details
- cap updates (raw values) + registration toggle

### Tab 3: Students
- fast search + schedule/check-in data

### Tab 4: Teachers
- fast search + class/check-in data

### Tab 5: Settings
- overenrollment behavior toggles

---

## 9) Priority Ladder for Proposal Narrative

1. **Reliability + usability under event pressure**
2. **Low-load architecture (aggregation + delta polling)**
3. **Safe mutable operations (caps/status with graceful conflicts)**
4. **Refactor value (shared service layer for all onsite webapps)**
5. **Optional polish (advanced UX cues and overlays)**

