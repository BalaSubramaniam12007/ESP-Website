# Onsite Admin Research & Analysis — Plan 02

**Last Updated:** March 17, 2026  
**Scope:** Deep-dive investigation into 8 critical onsite admin research questions  
**Format:** Structured, easy-to-read documentation with clear sections, tables, and diagrams

---

## Executive Summary

This document consolidates research findings across all 8 onsite admin investigation questions:

| Question | Finding | Status |
|----------|---------|--------|
| **Q1: What is onsite?** | General-purpose Splash-day operational interface, not mobile-specific | ✓ Answered |
| **Q2: Current admin features** | 11 major admin modules covering check-in, scheduling, attendance, checkout | ✓ Answered |
| **Q3: Needed features for Splash day** | Real-time analytics, advanced filtering, mobile optimization, bulk operations | ✓ Answered |
| **Q4: Notification system** | Email & SMS exists; needs improvements in real-time alerts & UI feedback | ✓ Answered |
| **Q5: Mobile/desktop responsiveness** | Current design is browser-based, lacks mobile-first CSS framework | ✓ Answered |
| **Q6: Onsite-specific themes** | Multiple themes available; onsite uses generic theme system | ✓ Answered |
| **Q7: Comprehensive plan document** | Consolidated from Q1–Q6 findings | → This document |
| **Q8: Flow & role diagrams** | ASCII diagrams provided below | → Below |

---

## QUESTION 1: What is Onsite?

### Definition

**Onsite** is a **general-purpose Splash-day operational interface** designed for on-the-ground staff (registration desk, check-in crew, teachers, admins) to manage the program in real time.

It is **NOT** mobile-first or tablet-specific. It is a **browser-based web application** meant to run on:
- Desktop/laptop computers at admin stations
- Shared check-in kiosks
- Printer workstations
- Teacher/moderator tablets (secondary use)

### Key Configuration Tags

| Tag | Default | Purpose |
|-----|---------|---------|
| `program_center` | `37.427490, -122.170267` | Geographic center for onsite map (student/teacher webapps) |
| `program_center_zoom` | `17` | Initial zoom level for onsite maps |
| `student_self_checkin` | `none` | Enable student self-check-in (button/code mode) |
| `student_onsite_checkin_note` | "Note: You will not be able to see your classrooms until after check-in" | Message shown to unchecked-in students |
| `teacher_onsite_checkin_note` | "Note: Please make sure to check in before your first class" | Message shown to unchecked-in teachers |
| `switch_time_program_attendance` | (blank) | Time threshold for switching from enrollment to program attendance counts |
| `switch_lag_class_attendance` | (blank) | Minutes into class to switch to class attendance counts |

### Entry Points

**Admin Access:**
```
URL Pattern: /onsite/[program]/[instance]/
Example:    /onsite/Splash/2026/
```

**Core Module:** `OnsiteCore` (`esp.program.modules.handlers.onsitecore.py`)
- Serves as the central landing page
- Collects all enabled onsite modules for the program
- Builds dashboard with links to check-in, scheduling, attendance, and admin tools

**Decorators:**
- `@needs_onsite` — Restricts access to users with `Onsite Admin` permission
- `@onsite_call` — Routes request through the onsite module dispatch system
- `@main_call` — Designates the main view for a module

---

## QUESTION 2: Current Onsite Admin Features

### Overview

The onsite admin interface consists of **11 major handler modules** plus **shared utilities**. Below is a feature matrix of what each handles:

### Admin Modules & Features

#### 1. **OnSiteCore** (`onsitecore.py`)
- **Purpose:** Central landing page and module aggregator
- **Features:**
  - Displays links to all enabled onsite modules
  - Collects module context and renders dashboard
  - Quick navigation to all Splash-day operations

#### 2. **OnSiteCheckinModule** (`onsitecheckinmodule.py`)
- **Purpose:** Rapid student intake and check-in processing
- **Features:**
  - ✅ Single-student check-in by name/grade filter
  - ✅ Barcode/batch check-in with camera scanning
  - ✅ Payment status updates
  - ✅ Medical/liability form verification
  - ✅ Advanced manual check-in editor (undo capability)
  - ✅ AJAX-based status snippets

#### 3. **OnSiteClassSchedule** (`onsiteclassschedule.py`)
- **Purpose:** Real-time schedule changes and class management
- **Features:**
  - ✅ Grid-based class-change UI with filters
  - ✅ Live student/class lookup
  - ✅ Schedule update with auto-check-in option
  - ✅ Printer queue submission for schedules
  - ✅ Open-class scrolling view (alternative UI)
  - ✅ AJAX endpoints for dynamic data refresh

#### 4. **OnSiteAttendance** (`onsiteattendance.py`)
- **Purpose:** Attendance tracking and real-time dashboard
- **Features:**
  - ✅ Time-series attendance graph
  - ✅ Per-timeslot attendance summary tables
  - ✅ Categorized counts (checked in, attending, not attending)
  - ✅ Section-level attendance drill-down
  - ✅ Cached attendance helpers for performance

#### 5. **TeacherCheckinModule** (`teachercheckinmodule.py`)
- **Purpose:** Teacher/moderator presence tracking and reminders
- **Features:**
  - ✅ Check-in by timeslot or day
  - ✅ Missing-teacher detection and alerts
  - ✅ SMS text reminders via Twilio (if configured)
  - ✅ Class flag display (resource issues, missing items)
  - ✅ Barcode scanning support
  - ✅ Detailed class roster and section drill-down

#### 6. **OnSiteCheckoutModule** (`onsitecheckoutmodule.py`)
- **Purpose:** Student departure and unenrollment
- **Features:**
  - ✅ Individual student checkout with selective unenrollment
  - ✅ Mass checkout of all checked-in students
  - ✅ Future-class unenrollment options (return today vs. return forever)

#### 7. **OnSiteRegister** (`onsiteregister.py`)
- **Purpose:** On-the-fly student account creation
- **Features:**
  - ✅ Rapid account creation without pre-registration
  - ✅ Contact info entry
  - ✅ Student info forms
  - ✅ Group membership assignment
  - ✅ Payment/form record initialization
  - ✅ Direct transition to schedule assignment

#### 8. **OnsitePaidItemsModule** (`onsitepaiditemsmodule.py`)
- **Purpose:** Financial tracking and inventory queries
- **Features:**
  - ✅ Student item lookup (meals, shirts, extras)
  - ✅ Required/optional item display
  - ✅ Amount-due calculation
  - ✅ Quick-access for payment desk operations

#### 9. **OnsitePrintSchedules** (`onsiteprintschedules.py`)
- **Purpose:** Unattended printer workstation support
- **Features:**
  - ✅ Polling-based print job queue
  - ✅ Schedule PDF generation
  - ✅ Auto-print loops with image payloads
  - ✅ Print job logging and completion tracking

#### 10. **OnsiteClassList** (`onsiteclasslist.py`)
- **Purpose:** Class roster and unenrollment management
- **Features:**
  - ✅ Bulk unenrollment interface
  - ✅ Student/section selection matrix
  - ✅ No-show cleanup after class start
  - ✅ Cached data endpoints for UI
  - ✅ Stale data warnings

#### 11. **StudentWebapp / TeacherWebapp** (`studentonsite.py`, `teacheronsite.py`)
- **Purpose:** Student/teacher self-service during Splash day
- **Features (Student):**
  - ✅ Schedule view with room locations
  - ✅ Live class catalog with availability (by timeslot)
  - ✅ Section details and survey integration
  - ✅ Map display of classroom locations
  - ✅ Check-in status notification
  
- **Features (Teacher):**
  - ✅ Schedule view with class details
  - ✅ Attendance status (checked in, taught, missing)
  - ✅ Room/resource flags visible
  - ✅ "Ready to teach" confirmation flow

---

### Current Admin Capabilities Matrix

```
┌─────────────────────────────┬─────────┬────────────────────────────────┐
│ Workflow                    │ Enabled │ Primary Module                 │
├─────────────────────────────┼─────────┼────────────────────────────────┤
│ Check-in students           │ ✓ Full  │ OnSiteCheckinModule            │
│ Update payment status       │ ✓ Full  │ OnSiteCheckinModule            │
│ Verify medical/liability    │ ✓ Full  │ OnSiteCheckinModule            │
│ Change student schedules    │ ✓ Full  │ OnSiteClassSchedule            │
│ Track attendance by time    │ ✓ Full  │ OnSiteAttendance               │
│ Check in teachers           │ ✓ Full  │ TeacherCheckinModule           │
│ Text missing teachers       │ ✓ Full  │ TeacherCheckinModule (SMS)     │
│ Checkout students early     │ ✓ Full  │ OnSiteCheckoutModule           │
│ Create onsite accounts      │ ✓ Full  │ OnSiteRegister                 │
│ View item inventory/payment │ ✓ Full  │ OnsitePaidItemsModule          │
│ Print schedules (workstation)│ ✓ Full │ OnsitePrintSchedules           │
│ Bulk unenroll no-shows      │ ✓ Full  │ OnsiteClassList                │
│ View class rosters          │ ✓ Partial│TeacherCheckinModule (drill-down)│
│ Real-time class capacity    │ ✓ Full  │ OnSiteClassSchedule (AJAX)     │
└─────────────────────────────┴─────────┴────────────────────────────────┘
```

---

## QUESTION 3: Needed Features for Splash Day

### Gaps & Opportunities

Based on investigation of current modules and typical Splash-day operations, the following features are **missing or underdeveloped**:

### High Priority

#### 1. **Real-Time Analytics & Dashboards**
- **Current State:** Attendance module shows historical graph + timeslot tables
- **Gap:** No live KPI dashboard (e.g., % checked-in, % in class, enrollment vs. attendance trends)
- **Recommendation:**
  - Add a `/onsite/analytics` or summary card on the main page
  - Show key metrics: total enrolled, checked in, currently attending, no-shows
  - Update via AJAX every 30–60 seconds

#### 2. **Advanced Student Search & Filtering**
- **Current State:** Name/grade filter in check-in module
- **Gap:** No cross-module search (e.g., find student across check-in, scheduling, inventory pages)
- **Recommendation:**
  - Add global search bar on onsite landing page
  - Filter by grade, enrollment status, payment status, check-in status
  - Quick-jump to relevant module for that student

#### 3. **Mobile-Optimized UI**
- **Current State:** Desktop-browser-first HTML/CSS
- **Gap:** No responsive design; tablets/phones get cramped layouts
- **Recommendation:**
  - Add `<meta viewport>` tags
  - Use CSS media queries for < 768px width
  - Touch-friendly button sizing (min 44x44px)
  - See **Question 5** for detailed audit

#### 4. **Notification & Alert System (Onsite Staff)**
- **Current State:** Text messages to teachers; email/SMS only pre-program
- **Gap:** No in-app alerts or staff notifications (e.g., "Critical: 10 students unpaid," "Missing 3 teachers in Timeslot 2")
- **Recommendation:**
  - Add a notification center on onsite home page
  - Use WebSocket or polling for live alerts
  - Categories: critical (missing teachers), warnings (unpaid students), info (class full)
  - See **Question 4** for notification system deep-dive

#### 5. **Bulk Operations & Workflows**
- **Current State:** Unenroll module is slow; no bulk payment/form updates
- **Gap:** Admin must click through multiple students for common mass ops
- **Recommendation:**
  - Bulk "mark as paid" for students checked in at payment desk
  - Bulk "verify liability forms" for a group of students
  - Batch SMS to unchecked-in teachers

#### 6. **Improved Teacher Coordination**
- **Current State:** TeacherCheckinModule shows missing teachers; SMS text support
- **Gap:** No "which teacher is in which room right now" tracker; no resource request integration
- **Recommendation:**
  - Teacher location tracking (optional) or at least "currently teaching" vs. "between classes"
  - Resource request status (e.g., "waiting for whiteboard"; flag alert)
  - Summary of no-show classes per timeslot

#### 7. **Class Capacity & Demand Forecasting**
- **Current State:** Grid shows open classes; no predictive warnings
- **Gap:** No "classes will fill quickly" or "classes likely to be empty" alerts
- **Recommendation:**
  - Highlight high-demand classes (many waitlists pre-program)
  - Show predicted vs. actual enrollment trends in real-time

#### 8. **Offline Support**
- **Current State:** All modules require server connection
- **Gap:** Network outage = total onsite system failure
- **Recommendation:**
  - Implement local caching for critical data (check-in list, class roster)
  - Sync queue when network returns
  - At minimum, allow check-in on cached list

---

### Medium Priority

- **Undo/Audit Trail:** Better logging of admin actions (who changed what, when)
- **Export Reports:** CSV export of check-in status, attendance, payment records
- **Customizable Dashboards:** Admin can choose which widgets appear on home page
- **Timer/Clock:** Prominent clock on admin pages to prevent misclicks on timeslot boundaries
- **Accessibility (A11Y):** WCAG 2.1 AA compliance for color contrast, keyboard nav, screen readers

---

### Low Priority (Nice-to-Have)

- **QR Code Integration:** Scan student ID → auto-check-in (vs. barcode scanning)
- **Photo ID Capture:** Optional student photo at check-in for verification
- **Voice Alerts:** "Check-in desk is backed up" announcement for staff
- **Student Queue Management:** Formal line queue display on kiosk screens

---

## QUESTION 4: Notification System Analysis

### Current Notification Channels

#### 1. **Email (via `dbmail`)**
- **Used by:** All registration modules, reminders, surveys
- **Onsite Relevance:** Pre-program notifications (class confirmations, reminders)
- **Limitation:** Not real-time; students must check inbox

#### 2. **SMS Text Messages (via Twilio)**
- **Used by:** `TeacherCheckinModule` for missing-teacher reminders
- **Feature:** `GroupTextModule` orchestrates Twilio API calls
- **Onsite Flow:**
  ```
  Admin clicks "Text all unchecked-in teachers"
       ↓
  Template rendered: esp/templates/program/modules/teachercheckin/teachertext.txt
       ↓
  GroupTextModule.sendMessages(teacher, message) called
       ↓
  Twilio SMS sent
  ```

#### 3. **In-App Messages (Limited)**
- **Used by:** Temporary form errors, success messages
- **Limitation:** Only per-page; no persistent inbox

### Notification System Strengths

| Feature | Status | Notes |
|---------|--------|-------|
| Email delivery | ✅ Robust | Django email backend; batch support |
| SMS integration | ✅ Functional | Twilio API configured in `grouptextmodule.py` |
| Permission-based routing | ✅ Built-in | Teachers only receive teacher reminders |
| Template-based messages | ✅ Flexible | Easy to customize SMS/email templates |

---

### Notification System Weaknesses

#### 1. **No Real-Time Alert Channel**
- **Issue:** Admins have no fast way to broadcast alerts (e.g., "Class 5 has 0 teachers!")
- **Current Workaround:** Manual email or call staff
- **Impact:** Slow response to Splash-day emergencies

#### 2. **SMS Batching Issues**
- **Issue:** Twilio API has rate limits; texting 50 teachers simultaneously may fail or be throttled
- **Current:** Single-threaded; no retry logic visible
- **Impact:** Some messages may not deliver; no clear feedback to admin

#### 3. **No Push Notifications**
- **Issue:** Students/teachers on mobile don't get desktop browser notifications
- **Current:** Only SMS (teachers) and email (all)
- **Recommendation:** Integrate Firebase Cloud Messaging or similar

#### 4. **Limited Onsite Staff Notifications**
- **Issue:** Admins don't receive critical alerts (e.g., database error, printer down)
- **Current:** Reliant on manual monitoring
- **Recommendation:** Add staff dashboard alerts for operational events

#### 5. **No Notification Preference UI**
- **Issue:** Students/teachers cannot configure notification channels (SMS vs. email vs. push)
- **Current:** Hardcoded per-module (SMS for teacher check-in only)
- **Impact:** Inflexible; some users may not want SMS

#### 6. **No Audit Trail**
- **Issue:** No record of which messages were sent, failed, or bounced
- **Current:** Twilio logs exist, but not integrated into ESP
- **Recommendation:** Store message metadata in local `SentMessage` model

---

### Notification Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                    Onsite Notification Flow                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Trigger Event                 Channel Selection                │
│  ─────────────────────────────────────────────────────────────  │
│                                                                  │
│  ├─ Teacher not checked in    ┌─ Email (dbmail)               │
│  │                             │                                │
│  ├─ Student payment due       ├─ SMS (Twilio)                 │
│  │                             │                                │
│  ├─ Class missing teacher     ├─ Push (Firebase) - NOT IMPL   │
│  │                             │                                │
│  └─ Resource request          └─ In-App Alert - LIMITED        │
│                                                                  │
│     Template Render             Delivery Queue                  │
│     ─────────────────────────────────────────────────────────── │
│                                                                  │
│     esp/templates/program/modules/teachercheckin/teachertext.txt│
│              ↓                                                   │
│     GroupTextModule.sendMessages()                              │
│              ↓                                                   │
│     Twilio API / Django Email Backend                           │
│              ↓                                                  │
│     [Success/Failure Logged]    [No persistent record]         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

### Pros & Cons Summary

| Aspect | Pro | Con |
|--------|-----|-----|
| **Email** | Reliable, built-in | Slow (minutes) |
| **SMS** | Immediate, opt-in | Rate-limited, cost, no rich format |
| **Push (none)** | – | No mobile app, no web push |
| **In-App** | Real-time, persistent | Limited reach, browser-dependent |

---

### Recommended Improvements (Priority)

1. **[HIGH]** Add WebSocket-based alerts for onsite staff (real-time dashboard alerts)
2. **[HIGH]** Implement SMS retry logic with exponential backoff
3. **[MEDIUM]** Add push notification support (Firebase Cloud Messaging)
4. **[MEDIUM]** Create message audit log in database
5. **[MEDIUM]** Add notification preference UI for students/teachers
6. **[LOW]** Integrate Slack/Discord webhook for staff notifications

---

## QUESTION 5: Mobile & Desktop Responsiveness

### Current State Assessment

#### Design Approach
- **Primary Target:** Desktop/laptop browsers (1024px+ width)
- **Secondary:** Tablet browsers (but not optimized)
- **Tertiary:** Mobile phones (severely limited)

#### Evidence from Codebase

1. **No Viewport Meta Tag** (in most onsite templates)
   - File: `esp/templates/program/modules/onsitecore/index.html`
   - Issue: Mobile browsers zoom out to fit 1024px-wide page on small screens

2. **No CSS Media Queries** (for onsite-specific styles)
   - File: `esp/templates/program/modules/classreg/classchange.css`
   - Issue: Fixed-width layouts; sidebar doesn't collapse on small screens

3. **Large Tables & Grids**
   - File: `esp/templates/program/modules/onsiteattendance/index.html`
   - Issue: No horizontal scroll; content overflow on mobile

4. **Touch-Unfriendly Buttons**
   - Issue: Button sizes < 44x44px; hard to tap on mobile devices

#### Current Template Structure

```
esp/templates/program/modules/
  ├─ onsitecore/
  │  └─ index.html              [No responsive CSS]
  ├─ onsitecheckinmodule/
  │  ├─ checkin.html            [Fixed-width form]
  │  ├─ barcode.html            [Camera input OK; layout tight]
  │  └─ advanced.html           [Table-heavy; not responsive]
  ├─ classreg/
  │  ├─ classchange.html        [Grid layout; NOT responsive]
  │  ├─ classchange.css         [No media queries]
  │  └─ open_classes.html       [Scroller; fixed width]
  ├─ onsiteattendance/
  │  └─ index.html              [Large tables; overflow issues]
  ├─ teachercheckin/
  │  ├─ teachercheckin.html     [Moderate responsiveness]
  │  └─ classlist.html          [Keyboard-friendly; small screens hard]
  └─ studentonsite/
     ├─ schedule.html           [React-based; likely responsive]
     ├─ catalog.html            [Mobile-friendly list]
     └─ map.html                [Map tile; responsive]
```

#### Mobile Challenges

| Screen Size | Affected Modules | Impact |
|-------------|------------------|--------|
| **< 480px (phone)** | All | Text runs off screen, buttons overlap, tables overflow |
| **480–768px (tablet)** | ClassChange, Attendance | Sidebar too wide; grid hard to navigate |
| **> 768px (desktop)** | None | Works as designed |

---

### Responsive Design Audit

#### What's Missing

1. **Viewport Meta Tag**
   ```html
   <!-- MISSING in all onsite templates -->
   <meta name="viewport" content="width=device-width, initial-scale=1.0">
   ```

2. **CSS Media Queries**
   ```css
   /* MISSING: No mobile-first breakpoints */
   @media (max-width: 768px) {
     /* tablet layout */
   }
   @media (max-width: 480px) {
     /* mobile layout */
   }
   ```

3. **Flexible Grid Layout**
   ```html
   <!-- Current: Fixed-width tables -->
   <table style="width: 1200px;">
   
   <!-- Needed: Flexible grid -->
   <div class="grid-container responsive">
   ```

---

### Recommendations for Mobile Optimization

#### Phase 1: Immediate (Low Effort, High Impact)

1. **Add Viewport Meta Tag** to [esp/templates/program/base.html](esp/templates/program/base.html)
   ```html
   <head>
     <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=yes">
   </head>
   ```

2. **Add Touch-Friendly Button Sizing**
   ```css
   button, .btn { min-height: 44px; min-width: 44px; }
   ```

3. **Hide Low-Priority Content on Mobile**
   ```css
   @media (max-width: 768px) {
     .sidebar { display: none; }
     .main-content { width: 100%; }
   }
   ```

#### Phase 2: Medium Effort (Medium Impact)

1. **Convert Fixed Layouts to Flexbox/Grid**
   - Refactor [esp/templates/program/modules/classreg/classchange.html](esp/templates/program/modules/classreg/classchange.html)
   - Add breakpoints for tablet (768px) and mobile (480px)

2. **Implement Responsive Tables**
   - Convert [esp/templates/program/modules/onsiteattendance/index.html](esp/templates/program/modules/onsiteattendance/index.html) to card-based layout on mobile

3. **Add Hamburger Menu** for onsite module navigation

#### Phase 3: Long-Term (High Effort, Strategic)

1. **Adopt a CSS Framework** (Bootstrap 5, Tailwind) for consistency
2. **Test on Real Devices** (iOS Safari, Android Chrome)
3. **Implement PWA** (Progressive Web App) for offline support
4. **A/B Test** mobile-optimized check-in flow with users

---

### Mobile-First Checklist

- [ ] All templates include `<meta viewport>`
- [ ] All buttons ≥ 44x44px
- [ ] All forms are single-column on mobile
- [ ] All tables have horizontal scroll on mobile (or card layout)
- [ ] Navigation collapses to hamburger on < 768px
- [ ] Touch-friendly spacing (min 8px padding)
- [ ] No fixed-width containers (use max-width instead)
- [ ] Font sizes are readable (min 16px base)
- [ ] Links are 24px+ wide

---

## QUESTION 6: Onsite-Specific Themes

### Theme System Overview

#### Theme Architecture

ESP uses a **modular theme system** in [esp/esp/themes/](esp/esp/themes/):

```
esp/esp/themes/
  ├─ __init__.py                    [Theme base/config]
  ├─ theme_data/
  │  ├─ barebones/                  [Minimal theme]
  │  ├─ bigpicture/                 [Default theme]
  │  ├─ circles/                     [Circle-based design]
  │  ├─ droplets/                    [Droplet shapes]
  │  ├─ floaty/                      [Floating layouts]
  │  └─ fruitsalad/                  [Colorful theme]
  ├─ settings.py                     [Theme configuration]
  ├─ forms.py                        [Theme admin form]
  ├─ models.py                       [Theme metadata]
  ├─ views.py                        [Theme upload/switch]
  ├─ controllers.py                  [Theme compilation]
  ├─ urls.py                         [Theme URL routing]
  └─ migrations/                     [Database migrations]
```

---

### Onsite Theme Usage

#### Default Behavior
- **Onsite modules inherit:** The program's selected theme (configured in admin)
- **No Onsite-Specific Overrides:** All onsite templates use the same theme CSS as student/teacher registration
- **Theme Applied Via:** Django template inheritance from `esp/templates/program/base.html`

#### Theme Configuration

| Tag | Default | Onsite Impact |
|-----|---------|----------------|
| `current_theme_version` | `8daf9a` | Cache buster for theme CSS |
| `current_logo_version` | `8daf9a` | Cache buster for logo |
| `current_header_version` | `8daf9a` | Cache buster for header |
| `current_favicon_version` | `8daf9a` | Cache buster for favicon |

---

### Onsite-Specific Template CSS Files

Only **ONE** dedicated onsite CSS file exists:

| File | Purpose |
|------|---------|
| `esp/templates/program/modules/classreg/classchange.css` | Class-change grid styling (onsite-specific) |

All other onsite modules rely on:
- Base theme CSS
- Module-specific `<style>` tags inline
- Shared utilities from [esp/public/static/css/](esp/public/static/css/)

---

### Theme Customization for Onsite

#### Available Theme Options

1. **barebones** — Minimal CSS; good for testing
2. **bigpicture** — Default; balanced design
3. **circles** — Rounded, modern look
4. **droplets** — Organic shapes
5. **floaty** — Floating/card-based layouts
6. **fruitsalad** — High-contrast, colorful

#### How to Apply Theme to Onsite

1. **Admin selects theme** in `/admin/` for the program
2. **All onsite templates** automatically inherit that theme
3. **To override onsite-specific styles:**
   - Create `esp/templates/program/modules/onsitecore/base_onsite.html`
   - Or add `<style scoped>` tags to individual onsite templates

#### Example: Custom Onsite Theme

To create a "Onsite Day-of-Operations" theme:

```html
<!-- esp/templates/program/modules/onsitecore/base_onsite.html -->

{% extends "base.html" %}

{% block extra_css %}
<style>
  /* Onsite-specific overrides */
  body { font-size: 16px; } /* Larger for kiosk visibility */
  button { min-height: 48px; } /* Touch-friendly */
  .onsite-alert { background: #ff6b6b; color: white; }
</style>
{% endblock %}
```

---

### Theme vs. Onsite Confusion

**Important:** Themes do NOT provide onsite-specific UI/UX. They only control:
- Color scheme
- Typography
- Layout grid
- Logo/header appearance

**Onsite-specific** UX (check-in workflow, class grid, teacher rosters) is driven by:
- Module handlers (Python)
- Templates (HTML/template tags)
- Inline CSS (per-module)

---

### Recommendation: Dedicated Onsite Theme

To improve onsite UX (especially mobile), consider creating a dedicated theme:

```
esp/esp/themes/theme_data/onsite/
  ├─ less/
  │  └─ onsite.less                  [Responsive, kiosk-friendly CSS]
  ├─ images/
  │  └─ [onsite-specific graphics]
  └─ [other theme assets]
```

**This theme would include:**
- Large, touch-friendly buttons (48–64px)
- High contrast (WCAG AAA)
- Larger base font size (16px+)
- Responsive grid for kiosk/tablet/desktop
- Minimal visual noise (optimized for high-stress environments)

---

## Summary Comparison Table

| Aspect | Current State | Needed |
|--------|---------------|--------|
| **Mobile Viewport** | ❌ Missing | ✅ Add |
| **Responsive CSS** | ❌ Minimal | ✅ Framework-based |
| **Touch-Friendly UI** | ⚠️ Partial | ✅ Improve buttons, spacing |
| **Theme System** | ✅ Works | ⚠️ Add onsite-specific theme |
| **Notification System** | ⚠️ Limited | ✅ WebSocket alerts + SMS retry |
| **Real-Time Analytics** | ❌ Missing | ✅ Add dashboard |
| **Offline Support** | ❌ None | ✅ Local caching |
| **Bulk Operations** | ⚠️ Partial | ✅ Improve workflows |

---

## DIAGRAM 1: Onsite Admin Role & Permission Hierarchy

```
┌─────────────────────────────────────────────────────────────────┐
│                   ONSITE ROLE HIERARCHY                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ESP SuperAdmin                                                  │
│  └─ manage.py permission system                                 │
│                                                                  │
│     ├─ Onsite/All              [Master onsite access]          │
│     │                                                            │
│     ├─ Onsite/CheckIn          [Student check-in only]         │
│     │  ├─ OnSiteCheckinModule                                  │
│     │  └─ OnSiteRegister (create accounts)                     │
│     │                                                            │
│     ├─ Onsite/ClassChange      [Schedule management]           │
│     │  └─ OnSiteClassSchedule                                  │
│     │                                                            │
│     ├─ Onsite/Attendance       [Attendance tracking]           │
│     │  ├─ OnSiteAttendance                                     │
│     │  └─ OnSiteClassList (rosters)                            │
│     │                                                            │
│     ├─ Onsite/TeacherCheckin   [Teacher operations]            │
│     │  └─ TeacherCheckinModule (SMS reminders)                │
│     │                                                            │
│     ├─ Onsite/Checkout         [Student departure]            │
│     │  └─ OnSiteCheckoutModule                                 │
│     │                                                            │
│     ├─ Onsite/Inventory        [Financial items]              │
│     │  └─ OnsitePaidItemsModule                               │
│     │                                                            │
│     └─ Onsite/Printing         [Printer workstations]         │
│        └─ OnsitePrintSchedules                                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## DIAGRAM 2: Splash Day Operational Workflow

```
┌─────────────────────────────────────────────────────────────────┐
│              SPLASH DAY ONSITE ADMIN WORKFLOW                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  8:00 AM — Program Start                                        │
│  ───────────────────────────────────────────────────────────── │
│                                                                  │
│  Desk #1: Student Registration                                 │
│  ├─ OnSiteRegister (create accounts for walk-ins)             │
│  ├─ Collect payment                                            │
│  └─ Assign initial schedule                                    │
│                                                                  │
│  Desk #2: Student Check-In                                     │
│  ├─ OnSiteCheckinModule (barcode/name lookup)                 │
│  ├─ Verify payment                                             │
│  ├─ Verify medical/liability forms                            │
│  └─ Generate nametag + schedule                               │
│                                                                  │
│  Desk #3: Teacher Arrivals                                     │
│  ├─ TeacherCheckinModule (check in teachers)                  │
│  ├─ Collect classroom assignments                              │
│  └─ SMS reminders for missing teachers                        │
│                                                                  │
│  Admin Station: Real-Time Monitoring                           │
│  ├─ OnSiteAttendance (live attendance graph)                  │
│  ├─ OnSiteCore (master dashboard)                             │
│  └─ Teacher/class status alerts                               │
│                                                                  │
│  10:00 AM — Classes Start (Timeslot 1)                         │
│  ───────────────────────────────────────────────────────────── │
│                                                                  │
│  Ongoing: Class Management                                      │
│  ├─ OnSiteClassSchedule (live schedule changes)              │
│  │  ├─ Student switches class                                 │
│  │  ├─ Auto-mark checked-in (optional)                        │
│  │  └─ Update capacity counts                                 │
│  │                                                              │
│  ├─ TeacherCheckinModule (attendance per class)              │
│  │  ├─ Teacher check-in by section                           │
│  │  └─ Flag missing teachers                                  │
│  │                                                              │
│  └─ OnsiteClassList (monitor attendance)                      │
│     ├─ Real-time roster view                                  │
│     └─ Detect no-shows for unenrollment                       │
│                                                                  │
│  12:30 PM — Lunch Break                                        │
│  ───────────────────────────────────────────────────────────── │
│                                                                  │
│  Activities:                                                     │
│  ├─ OnsitePaidItemsModule (meal/shirt inventory)             │
│  ├─ OnSiteAttendance (update attendance records)             │
│  └─ Check print queue, restock nametags                       │
│                                                                  │
│  2:00 PM — Final Classes                                       │
│  ───────────────────────────────────────────────────────────── │
│                                                                  │
│  Continue monitoring:                                            │
│  ├─ OnSiteClassSchedule (last-minute changes)                │
│  ├─ OnSiteAttendance (final attendance count)                │
│  └─ TeacherCheckinModule (teacher status)                    │
│                                                                  │
│  4:30 PM — Program End                                          │
│  ───────────────────────────────────────────────────────────── │
│                                                                  │
│  Final Steps:                                                    │
│  ├─ OnSiteCheckoutModule (mass checkout)                     │
│  ├─ OnsiteClassList (bulk unenroll no-shows)                │
│  ├─ OnsitePrintSchedules (print final rosters)               │
│  └─ OnSiteAttendance (final report)                          │
│                                                                  │
│  4:00 PM — After Program                                        │
│  ───────────────────────────────────────────────────────────── │
│                                                                  │
│  Wrap-Up:                                                        │
│  ├─ Export attendance/payment records                           │
│  ├─ Archive check-in logs                                      │
│  └─ Prepare survey data                                        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## DIAGRAM 3: Feature Matrix — Current vs. Needed

```
┌──────────────────────────┬────────────┬────────────┬──────────────┐
│ Feature                  │ Current    │ Priority   │ Effort       │
├──────────────────────────┼────────────┼────────────┼──────────────┤
│ Student check-in         │ ✓ Full     │ —          │ —            │
│ Schedule changes         │ ✓ Full     │ —          │ —            │
│ Attendance tracking      │ ✓ Full     │ —          │ —            │
│ Teacher check-in         │ ✓ Full     │ —          │ —            │
│ Student checkout         │ ✓ Full     │ —          │ —            │
│ Onsite registration      │ ✓ Full     │ —          │ —            │
│ Item inventory           │ ✓ Full     │ —          │ —            │
│ Schedule printing        │ ✓ Full     │ —          │ —            │
│ Unenroll no-shows        │ ✓ Full     │ —          │ —            │
├──────────────────────────┼────────────┼────────────┼──────────────┤
│ Real-time KPI dashboard  │ ✗ Missing  │ HIGH       │ Medium       │
│ Advanced search          │ ✗ Missing  │ HIGH       │ Medium       │
│ Mobile UI                │ ⚠ Poor     │ HIGH       │ High         │
│ Onsite staff alerts      │ ✗ Missing  │ HIGH       │ High         │
│ Bulk operations          │ ⚠ Partial  │ MEDIUM     │ Medium       │
│ Teacher location tracking│ ✗ Missing  │ MEDIUM     │ High         │
│ Offline support          │ ✗ Missing  │ MEDIUM     │ High         │
│ SMS retry logic          │ ⚠ Limited  │ MEDIUM     │ Low          │
│ Push notifications       │ ✗ Missing  │ MEDIUM     │ High         │
│ Message audit trail      │ ✗ Missing  │ LOW        │ Low          │
│ Accessibility (A11Y)     │ ⚠ Partial  │ LOW        │ Medium       │
│ Voice announcements      │ ✗ Missing  │ LOW        │ High         │
└──────────────────────────┴────────────┴────────────┴──────────────┘
```

---

## DIAGRAM 4: Notification System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│          RECOMMENDED NOTIFICATION SYSTEM (FUTURE)               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Event Trigger                      Channel Router              │
│  ├─ Teacher not checked in     ┌─ Email (dbmail)              │
│  ├─ Student payment due        │   └─ Fast, reliable          │
│  ├─ Class missing teacher      ├─ SMS (Twilio + Retry)        │
│  ├─ Resource request unfilled  │   └─ Real-time (rate-limited)│
│  ├─ Database error             ├─ WebSocket (OnSite Staff)    │
│  └─ Network connectivity       │   └─ Real-time, persistent   │
│                                ├─ Push Notification (FCM)      │
│                                │   └─ Mobile app (future)      │
│                                └─ Slack/Discord (optional)     │
│                                    └─ Staff alerts             │
│                                                                  │
│  Persistent Queue                   Delivery Service           │
│  ├─ Message.objects.create()    ┌─ Email Worker (Celery)     │
│  ├─ SMS retry logic (exp. backoff)├─ SMS Batch Processor      │
│  └─ Delivery status tracking    ├─ WebSocket Server (Django) │
│                                 └─ Push Service (Firebase)    │
│                                                                  │
│  Notification Log                   User Preferences           │
│  ├─ SentMessage model           ├─ Notify via SMS: Y/N       │
│  ├─ Delivery status             ├─ Notify via Email: Y/N     │
│  ├─ Retry count                 ├─ Notify via Push: Y/N      │
│  └─ Error details               └─ Quiet hours (DND)         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## DIAGRAM 5: Responsive Design Breakpoints

```
Mobile-First Responsive Design

Phone              Tablet             Desktop
< 480px            480–768px          > 768px
┌─────────┐        ┌──────────┐       ┌──────────────┐
│ Header  │        │  Header  │       │    Header    │
│────────┤        ├──────────┤       ├──────────────┤
│         │        │Sidebar|  │       │ Sidebar      │
│ Nav     │        │    |  C  │       │ │   Content  │
│────────┤        │    |  o  │       │ │             │
│         │        │    |  n  │       │ │             │
│Content  │        │    |  t  │       │ │             │
│(full)   │        │    |  e  │       │ │             │
│         │        │    |  n  │       │ │             │
│         │        │    |  t  │       │ │             │
│────────┤        ├──────────┤       ├──────────────┤
│ Footer  │        │  Footer  │       │    Footer    │
└─────────┘        └──────────┘       └──────────────┘

CSS Breakpoints:
@media (max-width: 480px) {
  /* phone layout */
  .sidebar { display: none; }
  .content { width: 100%; }
}

@media (min-width: 481px) and (max-width: 768px) {
  /* tablet layout */
  .sidebar { width: 30%; }
  .content { width: 70%; }
}

@media (min-width: 769px) {
  /* desktop layout */
  .sidebar { width: 25%; }
  .content { width: 75%; }
}
```

---

## Key Takeaways

### ✅ Strengths of Current Onsite System

1. **Modular Architecture:** 11+ independent handler modules allow flexibility
2. **Real-Time AJAX:** Class scheduling and attendance use AJAX for live updates
3. **Permission System:** Fine-grained role-based access control
4. **Template Inheritance:** Consistent UI across modules
5. **SMS Integration:** Teacher reminders work well (when configured)

### ⚠️ Weaknesses & Gaps

1. **No Mobile-First Design:** Desktop-only, poor tablet/phone UX
2. **Limited Notifications:** SMS only; no push, in-app, or staff alerts
3. **No Real-Time Analytics:** No live KPI dashboard for admins
4. **No Offline Support:** Complete failure on network outage
5. **No Accessibility:** Likely WCAG 2.1 non-compliant

### 🎯 Recommended Quick Wins

1. **Add viewport meta tag** (10 min, huge mobile impact)
2. **Implement SMS retry logic** (1–2 hours, prevents message loss)
3. **Create admin KPI dashboard** (4–8 hours, valuable for operations)
4. **Add mobile CSS breakpoints** (4–8 hours, significantly better UX)
5. **Implement WebSocket alerts** for onsite staff (16–24 hours, critical for emergencies)

---

## Next Steps

1. **Prioritize improvements** based on your Splash day needs (high volume = mobile/alerts critical)
2. **Create feature tickets** for each item in the matrix above
3. **Allocate dev time** for mobile optimization (Phase 1 = < 1 day; Phase 2 = 1–2 weeks)
4. **Set up testing** on real devices (iOS/Android) before Splash day
5. **Plan documentation** for new features (admin guides, screenshots)

---

**End of Onsite Admin Research & Analysis — Plan 02**

---

### Quick Reference: File Index for Developers

| Topic | Files to Reference |
|-------|-------------------|
| Core Routing | `esp/esp/program/modules/base.py`, `esp/esp/urls.py` |
| Check-In | `esp/esp/program/modules/handlers/onsitecheckinmodule.py`, templates in `onsitecheckinmodule/` |
| Scheduling | `esp/esp/program/modules/handlers/onsiteclassschedule.py`, templates in `classreg/` |
| Attendance | `esp/esp/program/modules/handlers/onsiteattendance.py`, templates in `onsiteattendance/` |
| Teacher Ops | `esp/esp/program/modules/handlers/teachercheckinmodule.py`, templates in `teachercheckin/` |
| SMS Notifications | `esp/esp/program/modules/handlers/grouptextmodule.py` |
| Tags/Config | `esp/esp/tagdict/__init__.py` |
| Themes | `esp/esp/themes/`, `esp/esp/themes/theme_data/` |
| Admin Docs | `docs/admin/program_modules.rst`, `docs/dev/program_modules.rst` |
| Seeds | `seeds/onsite_seed.py` |

