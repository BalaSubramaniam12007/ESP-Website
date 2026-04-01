# Onsite Admin Project - File Inventory & Documentation

**Last Updated:** March 17, 2026  
**Purpose:** Comprehensive mapping of all onsite admin-related files in the ESP Website codebase, with 2-3 line descriptions of what each file handles.

---

## Table of Contents

1. [Core Onsite Entry Points & Access Control](#core-onsite-entry-points--access-control)
2. [Admin-Only Onsite Handlers](#admin-only-onsite-handlers)
3. [Admin Onsite Templates & Frontend Assets](#admin-onsite-templates--frontend-assets)
4. [Shared Models, Records, Printing & Tag/Config Support](#shared-models-records-printing--tagconfig-support)
5. [Closely Related Shared Student/Teacher Onsite Files](#closely-related-shared-studentteacher-onsite-files)
6. [Docs, Tests, Seeds & Reference Material](#docs-tests-seeds--reference-material)
7. [Ambiguous & Secondary/Historical Files](#ambiguous--secondaryhistorical-files)

---

## Core Onsite Entry Points & Access Control

### `esp/esp/program/modules/handlers/onsite.py`

Defines the `OnSiteModule`, which is the landing module for the entire onsite admin portal. The main `onsite()` view collects enabled onsite modules for the current program and builds the central dashboard used by onsite-admin staff.

### `esp/templates/program/modules/onsite/index.html`

The main UI page for the onsite admin portal. Groups links into check-in, class changes, attendance, and student-info actions, making this the clearest high-level "map" of admin onsite capabilities on Splash day.

### `esp/esp/program/modules/base.py`

Contains `needs_onsite()` and `@onsite_call` decorators, which gate access to onsite pages and define the core program-module dispatch behavior. These decorators control which users can access onsite views and validate the program/module routing.

### `esp/esp/urls.py`

Registers the generic `onsite` URL patterns alongside `manage`, `teach`, and `learn` module routes. It does not implement onsite logic itself but serves as the routing entry point that all onsite module URLs depend on.

---

## Admin-Only Onsite Handlers

### `esp/esp/program/modules/handlers/classchecklist.py`

Main student check-in backend for onsite admins. Covers rapid single-student check-in, barcode/batch check-in, AJAX status snippets, payment/medical/liability record updates, and advanced manual check-in editing with undo capability.

### `esp/esp/program/modules/handlers/classreg.py`

Backend for the grid-based class-change tool and scrolling open-class board. Powers live class availability, student lookups, schedule updates, optional auto-check-in on schedule change, and print-queue submission—critical for managing students' schedules on Splash day.

### `esp/esp/program/modules/handlers/attendance.py`

Builds the admin attendance dashboard and time-series attendance graph. Exposes section-level attendance via AJAX endpoints and contains cached helpers like `getAttendance()` and `getSectionAttendance()` used by other onsite modules.

### `esp/esp/program/modules/handlers/teachercheckin.py`

Handles teacher/moderator check-in, missing-teacher reporting, SMS reminders, and AJAX-loaded class details. One of the richer operational modules, enabling staff to quickly spot and manage missing teachers on Splash day.

### `esp/esp/program/modules/handlers/checkout.py`

Implements student checkout from the program, supporting both individual checkout with selective unenrollment and mass checkout of all currently checked-in students. A critical end-of-day admin workflow.

### `esp/esp/program/modules/handlers/onsiteclass.py`

Creates brand-new student accounts onsite, including contact info, student info, group membership, and payment/form records. Core day-of-operations workflow for the registration desk when students arrive without pre-registration.

### `esp/esp/program/modules/handlers/studentwebapp.py`

Allows onsite admins to morph into a student user and access normal student registration pages for more flexible schedule edits. Also supports printing schedules through onsite workflow, so it remains admin-facing even though it delegates to shared student-reg logic.

### `esp/esp/program/modules/handlers/iteminventory.py`

Searches for a student and displays required/optional purchased items plus amount due. Small but operationally useful for onsite desks handling meals, shirts, or other extras and tracking financial obligations.

### `esp/esp/program/modules/handlers/printmodule.py`

Polling backend for unattended schedule printing at printer workstations. Serves the printer-workstation page, consumes `PrintingRequest` rows, and returns printable schedule output or JSON image payloads for automatic print loops.

### `esp/esp/program/modules/handlers/unenroll.py`

Admin-only onsite tool for expiring student enrollments based on missed first classes and check-in status. The cached `get_unenroll_data()` endpoint computes the student/section/timeslot matrix that the UI uses for bulk drops.

---

## Admin Onsite Templates & Frontend Assets

### `esp/templates/program/modules/classchecklist/checkin.html`

UI for rapid student check-in by name/grade filter. Includes admin autocomplete wiring and links onward to barcode and advanced check-in flows, forming the primary entry point for intake on Splash day.

### `esp/templates/program/modules/classchecklist/barcode.html`

UI for batch/barcode check-in with camera scanning support. Supports combined updates for attendance, payment, medical, and liability status in a single fast flow.

### `esp/templates/program/modules/classchecklist/advanced.html`

Advanced per-student check-in editor showing prior check-in/check-out pairs. Lets staff manually toggle `checkin`, `paid`, `medical`, `liability`, and undo the most recent check-in or checkout.

### `esp/templates/program/modules/classreg/classchange.html`

Main browser UI for the grid-based class-change tool. Exposes filters, student selection, printer selection, live refresh, and the "check in student when making schedule changes" option for live scheduling updates.

### `esp/templates/program/modules/classreg/classchange.js`

Large client-side controller for the class-change grid. Fetches multiple JSON endpoints, assembles the class/schedule model in the browser, filters by student/class metadata, and drives schedule updates via AJAX.

### `esp/templates/program/modules/classreg/classchange.css`

Dedicated stylesheet for the class-change grid and sidebar controls. Provides onsite-admin-specific presentation styling separate from the shared site theme.

### `esp/templates/program/modules/attendance/index.html`

Renders both the attendance summary graph and the per-timeslot operational tables. Exposes admin-facing categories: checked in, not attending, onsite-attending, and no-attendance-recorded sections.

### `esp/templates/program/modules/teachercheckin/dashboard.html`

Landing page for teacher check-in by day or timeslot. Serves as the admin navigation surface for finding, managing, and tracking teacher presence throughout Splash day.

### `esp/templates/program/modules/teachercheckin/classlist.html`

Main operational UI for finding, checking in, texting, and inspecting missing teachers/moderators. Surfaces class flags, missing resources, barcode scanning, and detailed section drill-downs.

### `esp/templates/program/modules/teachercheckin/classlist.js`

Frontend logic for live teacher check-in actions. Handles keyboard shortcuts, live AJAX check-in/undo, "text all unchecked-in teachers," and lazy class-detail loading.

### `esp/templates/program/modules/checkout/checkout.html`

Checkout UI for a single student or all checked-in students. Adds operational logic around return-today vs return-forever behavior by prechecking which future classes should be dropped.

### `esp/templates/program/modules/onsiteclass/register.html`

Form UI for creating a new student account onsite. Simple but key data-entry page for the onsite registration desk when walk-in students arrive.

### `esp/templates/program/modules/onsiteclass/registerconfirm.html`

Confirmation page for onsite registration. Exposes the newly created username and user ID, then sends staff directly into scheduling.

### `esp/templates/program/modules/iteminventory/index.html`

Displays required items, reserved items, totals, financial aid, and amount due for a student. The admin-facing readout tied to onsite purchase/payment questions.

### `esp/templates/program/modules/printmodule/printer_station.html`

Browser page intended to sit unattended on a printer workstation. Polls for pending print jobs, prints them automatically, and logs completed requests.

### `esp/templates/program/modules/unenroll/index.html`

Main UI for selecting which students and sections are affected by bulk unenrollment. Clearly admin onsite operations, especially for no-show cleanup after program start.

### `esp/templates/program/modules/unenroll/index.js`

Client-side logic for the unenroll page. Polls `get_unenroll_data()`, computes affected enrollments from checkbox selections, warns on stale data, and enables submit only when work exists.

---

## Shared Models, Records, Printing & Tag/Config Support

### `esp/esp/program/modules/forms/onsite.py`

Holds `AjaxCheckinForm`, `OnsiteClassCreationForm`, and `OnSiteStudentRegForm`. These form classes define the canonical input contracts for several admin onsite modules.

### `esp/esp/program/models/utils.py`

Important support file because `onsite_added()`, `onsite_removed()`, and `user_checked_in()` define the canonical program-level onsite state. Multiple admin modules use these helpers rather than reimplementing attendance/check-out logic.

### `esp/esp/users/models/__init__.py`

Central support for onsite permissions and record types. `ESPUser`, `Permission`, `Privilege`, `ContactInfo`, and the `Onsite` permission/record constants underpin nearly every onsite admin flow.

### `esp/esp/program/models/printings.py`

Defines `PrintingRequest` and `PrintingLog`, which are the persistent queue objects used by onsite schedule-printing workflows. A narrow but important operational dependency for the printer workstation.

### `esp/esp/tagdict/__init__.py`

Tag definitions relevant to onsite admin and nearby shared onsite behavior. Key onsite tags include `onsite_classlist_min_refresh`, `teacher_onsite_checkin_note`, `student_onsite_checkin_note`, `program_center`, `program_center_zoom`, `student_self_checkin`, and attendance webapp switching tags.

---

## Closely Related Shared Student/Teacher Onsite Files

### `esp/esp/program/modules/handlers/studentwebapp.py`

Not admin-only, but closely related because it consumes the same checked-in state, map tags, survey flow, and class-capacity behavior. Admin onsite work must preserve assumptions made by the student webapp.

### `esp/esp/program/modules/handlers/teacherwebapp.py`

Also not admin-only, but shares teacher check-in state, map tags, and attendance workflows with the admin side. Its roster and details views show how teacher-facing onsite flows overlap with admin check-in and attendance tools.

### `esp/templates/program/modules/studentwebapp/roster.html`

Not in an onsite-specific directory, but onsite modules render it. Real shared dependency of admin onsite attendance and class management views.

### `esp/esp/program/modules/handlers/printschedules.py`

Not an onsite module itself, but onsite printing calls into `printschedules.pdf_from_enrollment()`. The deeper shared implementation behind schedule printing.

---

## Docs, Tests, Seeds & Reference Material

### `docs/admin/program_modules.rst`

Best human-readable overview of both classic onsite admin modules and the student/teacher onsite webapps. Documents intended setup, URLs, and the expected operational role of each module on Splash day.

### `esp/esp/program/tests/test_onsite.py`

Focused regression tests for AJAX barcode check-in and onsite state transitions. Useful as a research anchor because it confirms which `OnSiteRecord` behavior is important enough to protect.

### `seeds/onsite_seed.py`

Dev seed for exploring the onsite admin portal and student onsite webapp together. Not production code, but usefully enumerates expected URLs, required record types, and realistic onsite states.

### `docs/dev/program_modules.rst`

Generic module-system reference; helps explain how onsite routes are auto-generated via `@main_call` and `@aux_call` decorators and the module dispatch chain.

---

## Ambiguous & Secondary/Historical Files

### `esp/templates/program/modules/classreg/open_classes.html`

Projection/scroller view for open classes; admin-relevant but less central than the grid-based class-change UI.

### `esp/templates/program/modules/classreg/open_classes_scroller_setup.html`

Setup screen for the open-classes scroller view. Secondary configuration interface.

### `esp/templates/program/modules/classreg/open_classes_fulllist.html` & `open_classes_by_category.html`

Alternate open-class displays (non-grid and category-based); less commonly used than the main class-change grid.

### `esp/templates/program/modules/teachercheckin/classdetail_ajax.html`

AJAX-loaded class detail partial for the missing-teacher view. Supporting template, not a primary workflow.

### `esp/templates/program/modules/teachercheckin/reminder_sms_template.txt`

SMS template used by teacher check-in reminders. Narrow operational utility for a specific notification path.

### `esp/templates/program/modules/printmodule/printer_station_setup.html`

Printer workstation setup page. Secondary configuration interface for printer queue initialization.

### `esp/templates/program/modules/studentwebapp/onsite_access_denied.html`

Onsite access-denied page tied to permission checks. Narrow error/fallback template.

### `esp/esp/program/modules/handlers/program_chooser.html`

Program chooser for users with access to multiple onsite programs. Narrow routing/disambiguation view.

### `esp/esp/program/modules/handlers/classreg.js` & `classchecklist.js`

Supporting JS used by teacher check-in or classlist UIs, but not exclusive business logic—shared utilities.

### `esp/esp/program/modules/handlers/onsite_export.py`

Appears to be a historical ad hoc export script for onsite/check-in data. Likely not part of active onsite admin flows.

### `docs/admin/onsite_unenroll_walkthrough.md`

Manual testing guide for an onsite admin unenroll enhancement. Useful context but not code.

---

## Key Classes & Functions to Reference

### Core Classes
- `OnSiteModule`
- `ProgramModuleObj` (base class for all onsite handlers)
- `ESPUser`
- `Permission`
- `OnSiteRecord`
- `StudentRegistration`
- `ClassSection`

### Key Decorators
- `@needs_onsite`
- `@main_call`
- `@aux_call`
- `@needs_admin`
- `@needs_teacher`
- `@meets_deadline`

### Important Functions
- `onsite_added(user, program)`
- `onsite_removed(user, program)`
- `user_checked_in(user, program)`
- `getAttendance(section)`
- `getSectionAttendance(section)`
- `get_unenroll_data(program)`
- `pdf_from_enrollment(enrollment)`

### Forms
- `AjaxCheckinForm`
- `OnsiteClassCreationForm`
- `OnSiteStudentRegForm`

### Models
- `PrintingRequest`
- `PrintingLog`
- `Tag`
- `Program`
- `ClassSection`
- `StudentRegistration`

---

## Known Gaps & Uncertainties

1. **Morphing ("become student") Dependencies:** The `studentwebapp.py` module depends heavily on general student-reg code. Not all dependencies are listed here to avoid sprawl beyond onsite-admin-specific files.

2. **Attendance Machinery:** Attendance also depends on shared teacher attendance machinery, especially `teacherwebapp.py` roster views and event-based timeslot helpers. Included as shared dependencies but not exhaustively unpacked.

3. **Legacy/Low-Use Templates:** Some templates under `esp/templates/program/modules/classreg/` (e.g., `open_classes_fulllist.html`, `open_classes_by_category.html`) appear lightly used relative to the active class-change grid. May be candidates for deprecation or consolidation.

4. **Common Assets Not Listed:** Utilities like `barcodescanner.js`, Twilio/group-text helpers, resource/flag support, and generic form handling are used by onsite screens but are not onsite-specific. Treated as supporting infrastructure rather than "onsite admin files."

5. **Indirect Dependencies:** Onsite modules call into small shared support under class-reg, user search, accounting, and resource assignment that are not exhaustively listed here.

6. **Mobile/Responsive Design:** No dedicated mobile templates or CSS framework noted; current design appears general-purpose browser-based rather than mobile-first. Potential gap for true mobile Splash-day workflows.

---

## Next Steps for Implementation

1. **Review & Validate:** Confirm this inventory matches actual Splash-day operational workflows in your environment.
2. **Identify Enhancement Gaps:** Use this list to cross-check against the 6 research questions in the parent onsite admin project.
3. **Plan Mobile/Responsive Improvements:** If mobile devices are used on Splash day, assess templates and CSS for mobile optimization.
4. **Documentation:** Link this inventory to [docs/admin/program_modules.rst](../../docs/admin/program_modules.rst) and create accompanying admin guides for each workflow.
5. **Testing & Seeding:** Ensure [seeds/onsite_seed.py](../../seeds/onsite_seed.py) covers all critical onsite module entry points.

---

**End of Onsite Admin File Inventory**
