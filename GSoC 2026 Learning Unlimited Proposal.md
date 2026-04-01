# GSoC 2026 Learning Unlimited Proposal
## Admin On-site Webapp

**Personal Details**

| Field | Value |
|---|---|
| Full Name | Balasubramaniam L |
| Mobile | +91 9345238008 |
| Email | balasubramaniam12007@gmail.com |
| Discord | balasubrmaniam |
| GitHub | [Balasubramaniam](https://github.com/BalaSubramaniam12007) |
| LinkedIn | [Balasubramaniam](http://www.linkedin.com/in/balasubramaniam2007) |
| Twitter | [Balasubramaniam](https://x.com/BALASUBRAMAN1AM) |
| Time Zone | GMT+5:30 |

**University Info**

| Field | Value |
|---|---|
| University | Saveetha University, Chennai |
| Institution | Saveetha Engineering College |
| Degree & Major | Bachelor of Technology |
| Year | 2nd Year |
| Graduation | 2028 |

## Motivation & Code Contribution

**Why Learning Unlimited captures my interest**

[To fill before submission.]

**Code Contribution**

| S.No | Title | Summary | Link | Status |
|---|---|---|---|---|
| 1 |  |  |  |  |
| 2 |  |  |  |  |
| 3 |  |  |  |  |
| 4 |  |  |  |  |

##       

## **Project Proposal:**

### **Proposal Title: Admin on-site webapp**

### Abstract

During Splash and other events, administrators need real-time visibility and practical control over class registration and attendance. Today, there is no dedicated mobile-friendly interface for admins to manage onsite operations, and the most useful controls are spread across several separate pages that require full reloads. This project proposes an admin on-site webapp that gives staff a live class dashboard, per-class registration controls, quick access to search and attendance actions, and a small settings area for live overenrollment control.

The current `onsitecore` portal is useful as a desktop navigation hub, but it is not designed for rapid event-day use on a phone or tablet. This gap has been tracked in issue #2672 for years and is still open in the Stable Release 17 milestone. The goal of this project is to close that gap with a focused admin experience that reuses existing onsite logic instead of rebuilding it.

### Solution Overview

This project will add a dedicated `AdminOnsite` module following the same general pattern as the existing student and teacher onsite webapps. The main dashboard will be powered by a single aggregated snapshot endpoint, grouped by timeslot and refreshed through lightweight polling.

Core ideas:

- A new mobile-first admin webapp under the existing onsite module space
- One snapshot helper that gathers the data needed for the dashboard
- Polling with client-side diffing instead of heavier infrastructure
- Reuse of existing registration, search, and onsite flows wherever possible
- A drill-down page for each class, where admins can make the most common event-day decisions

### Key Features

**1. Live Class Registration Dashboard**

- Classes grouped by timeslot rather than shown as one long flat list
- Current timeslot expanded by default; past and future timeslots collapsed
- Each row shows the class name, room, teacher status, enrollment pressure, and registration state
- Important classes can be surfaced first, such as nearly full classes or classes with staffing problems
- Dashboard refreshes every 30 seconds and shows a freshness indicator

**2. Per-Class Registration Toggle**

- MVP states: **Open** and **Closed**
- Closing registration includes a confirmation step to reduce accidental changes
- Updates happen without a full page reload
- Soft Closed remains a stretch goal and will only be explored if the core scope is stable

**3. Drill-Down View and Cap Editing**

- Tapping a class row opens a drill-down page with roster, registration controls, and current cap information
- My proposal is that any quick event-day cap adjustment should happen through this drill-down flow
- The exact cap-editing semantics should be finalized during community bonding, including whether the final write should behave at class level or section level
- If cap editing is included in final scope, the UI should warn admins before saving values that conflict with current enrollment

**4. Global Search and Admin Peek Mode**

- One search box for classes, students, teachers, and barcode-based lookup
- Class results show room, fill status, and registration state
- Student detail view supports quick actions such as check-in, check-out, and schedule printing
- Teacher detail view supports quick access to assignment and check-in context

**5. Settings and Graceful Degradation**

- A small settings tab for live overenrollment control using the existing registration settings
- Clear stale/disconnected states instead of failing hard when polling breaks
- Write controls lock automatically when data is stale, then recover after refresh

## Project Deliverables

By the end of 12 weeks, I plan to deliver:

- A new `AdminOnsite` module integrated into the existing onsite admin flow
- A snapshot helper that consolidates dashboard data for sections, staffing, registration state, and capacity
- A live dashboard with timeslot grouping, fill indicators, and graceful degradation
- Per-class registration open/close controls
- Drill-down pages with roster, registration information, and cap information
- A cap-adjustment workflow proposed through the drill-down page, with final semantics confirmed during bonding
- Global search and admin detail views for classes, students, and teachers
- A small settings tab for live overenrollment control
- Unit tests for the main endpoints, permissions, and write paths in final scope
- Admin documentation for usage and maintenance

### Admin On-site Webapp Design

Prototype: https://onsite-admin.netlify.app/

## Weekly Timeline

### Community Bonding

- Confirm final scope with mentors, especially Soft Closed and cap-editing semantics
- Review the existing student and teacher onsite webapps in depth
- Finalize UI direction, test setup, and implementation order
- Submit at least one small contribution before coding begins

### Phase 1 — Foundation

**Week 1 — Module scaffold**

- Create the `AdminOnsite` module, main page, and permission checks
- Add the entry point from `onsitecore`

*Deliverable: the new admin onsite page loads for admins and is blocked for non-admins.*

**Week 2 — Snapshot endpoint**

- Implement the snapshot helper and polling endpoint
- Group classes by timeslot and return the data needed for the dashboard

*Deliverable: dashboard data loads correctly for a seeded multi-timeslot program.*

**Week 3 — Polling and failure handling**

- Implement the polling loop and targeted client-side updates
- Add Live, Stale, and Disconnected states

*Deliverable: dashboard stays current without full page reloads and behaves safely under network problems.*

**Week 4 — Dashboard UI**

- Build the main mobile dashboard layout and class rows
- Verify the layout on common mobile widths

*Deliverable: the dashboard is usable on mobile and ready for mentor feedback.*

### Phase 2 — Core Features

**Week 5 — Registration controls**

- Implement the per-class registration toggle
- Add the confirmation flow and write tests

*Deliverable: admins can open and close class registration directly from the webapp.*

**Week 6 — Stabilization buffer**

- Review mentor feedback from the first half of the project
- Tighten queries and harden dashboard and toggle flows

*Deliverable: the MVP foundation is stable enough to build on safely.*

### Midterm Evaluation

**Week 7 — Search and student detail**

- Build the search tab
- Add the student detail view and quick actions

*Deliverable: class and student lookup works end to end.*

**Week 8 — Teacher detail and settings**

- Add the teacher detail view
- Build the first version of the settings tab

*Deliverable: teacher detail and live settings control work end to end.*

### Phase 3 — Polish and Delivery

**Week 9 — Drill-down and cap display**

- Complete the class drill-down page
- Show current cap information and roster state clearly

*Deliverable: the drill-down page is fully usable for day-of admin work.*

**Week 10 — Cap adjustment flow**

- Implement the agreed cap-adjustment workflow
- Add validation, safe write behavior, and tests

*Deliverable: cap adjustment support works in the final agreed form.*

**Week 11 — Mobile audit and documentation**

- Audit all key screens on target mobile sizes
- Finish admin documentation and final test coverage

*Deliverable: the project is documented, tested, and polished for review.*

**Week 12 — Final review and submission**

- Address remaining review feedback and overflow work
- Prepare final documentation and evaluation material

*Deliverable: the full admin onsite workflow is complete and ready for final review.*

## Future Enhancements

- Program-wide stats header for the current timeslot
- Integration with issue #2895 for time-controlled module state
- Follow-up work related to PR #4911 and live teacher check-in refresh

*Proposal grounded in verified codebase research: `base.py`, `class_.py`, `onsiteclasslist.py`, `onsitecore.py`, PRs #2507, #2866, #3473, and issues #2672, #2671, #2895, #3854, #1840.*
