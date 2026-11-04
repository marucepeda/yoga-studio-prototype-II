# Feature Specification — Class Booking Management

## 1. Feature Overview

**Product:** Yoga Studio
**Feature:** Class Booking Management
**Delivery target:** Local academic MVP
**Status:** Draft for review

The MVP turns the existing visual prototype into a working, single-browser demonstration of class availability, reservations, waitlists, cancellations, attendance and credit history.

## 2. Scope and Technology

Implement with vanilla HTML, CSS and JavaScript, using `localStorage` for persistence and fictional seed data. Do not add a backend, external database, authentication service, paid dependency, package purchasing, online payments, automated messaging or unrelated studio-management features.

Preserve the existing prototype's screens and warm, minimalist design direction. Integrate behavior into the prototype rather than replacing it with an unrelated visual design.

The application is a local demo, not a production booking service:

- The user selects a simulated role/persona (student, instructor or administrator); this is not authentication.
- All users and data are fictional and stored in the current browser profile.
- A reset-to-seed action is available so every scenario can be repeated.
- A clearly labeled test-only demo clock override is available independently of the selected persona; it affects cancellation/attendance time checks but never changes the device clock.
- Roles control which demo actions are shown: students view classes and their own credits/reservations and request or cancel their reservations; instructors view their scheduled classes and record attendance, mark absence or cancel reservations for those classes; administrators manage classes/capacities, configure policy and can cancel any reservation. These client-side checks are demonstration behavior, not security boundaries.

## 3. Domain and State Rules

### 3.1 Classes and capacity

Each class has a date/time, class name, instructor and positive integer capacity. A newly created class defaults to 12 places; an administrator may configure its capacity. Capacity cannot be reduced below its current number of confirmed reservations.

Available places equal configured capacity minus confirmed reservations. Waiting-list entries do not occupy places.

### 3.2 Credit states and ledger

For each student, credits are in exactly one of these states:

- **Available:** may be used for a new confirmed reservation.
- **Reserved:** held for one confirmed reservation that has not been attended or cancelled.
- **Consumed:** permanently spent by recorded attendance, or by a configured late-cancellation/absence outcome.

Credit state is derived from credit movements, not edited as an independent balance. A reservation moves one credit from available to reserved. An on-time cancellation returns the reserved credit to available. Attendance moves the reserved credit to consumed once. Every movement records a unique ID, student, related class or operation, movement type (`grant`, `reserve`, `release` or `consume`), amount, and timestamp. For a `release` or `consume`, the related operation identifies whether it resulted from attendance, cancellation or absence. The history must make each state transition traceable.

The reset-to-seed fixture is deterministic and entirely fictional:

| ID | Demo role/name | Initial credit state |
|---|---|---|
| `admin-andrea` | Administrator / Andrea | Not a student |
| `instructor-laura` | Instructor / Laura | Not a student |
| `student-alma` | Student / Alma | 2 available |
| `student-brisa` | Student / Brisa | 1 available |
| `student-celia` | Student / Celia | 0 available |
| `student-dalia` | Student / Dalia | 1 available |
| `student-01` | Student / Estudiante 01 | 1 reserved for `class-small` |
| `student-02` | Student / Estudiante 02 | 1 reserved for `class-small` |
| `student-03` | Student / Estudiante 03 | 1 reserved for `class-small` |

Seed `class-open` is a future Hatha Flow class with Laura, capacity 12 and no reservations. Seed `class-small` is a future Vinyasa Flow class with Laura, capacity 3 and three confirmed reservations, one each for `student-01`, `student-02` and `student-03`. Both classes are on June 4, 2030, at 18:00 local browser time. Initial grants and the three reserved credits are represented by ledger movements. Set the test-only demo clock to June 1, 2030, before scenarios that require an upcoming class. New-class capacity defaults are tested by creating a class without entering a capacity.

### 3.3 Reservations and duplicate prevention

A confirmed reservation requires an available credit and a free place. Reject it without changing state if either requirement fails.

A student may have at most one active booking per class, where active means confirmed or waiting. Reject a second request if either active state already exists. A past, attended or cancelled booking cannot be reactivated by submitting the same request; a new request is allowed only for a future class and only if no active booking exists.

### 3.4 Waiting list and eligibility

If a class is full, a student may join its waitlist even if they currently have no available credit. Joining the waitlist never reserves or consumes a credit. One student can have at most one active waitlist entry per class.

Entries receive a monotonically increasing sequence number when added; promotion considers entries in ascending sequence order. When a confirmed reservation is cancelled and a place opens, scan the queue from oldest to newest and promote the first student who has an available credit at that moment. A student without an available credit is ineligible for that promotion: retain their entry at its original sequence and continue scanning. This can allow a later eligible student to be promoted; skipped entries remain ahead of newer entries and can be considered on a later opening. Promotion changes the entry to confirmed and reserves exactly one credit atomically with the cancellation in the local state update.

### 3.5 Cancellation policy

The administrator configures (a) the number of hours before class that distinguishes an on-time from a late cancellation, and (b) whether a late cancellation and an absence respectively return the reserved credit to available or consume it. There is no assumed commercial deadline or default late/absence credit treatment: policy must be explicitly set before those outcomes can be processed. An on-time cancellation always releases its reserved credit.

Every confirmed cancellation before class start, whether on time or late, frees a class place and triggers the waitlist promotion rule above. The configured late-cancellation treatment determines whether the cancelling student's reserved credit is released or consumed; it does not change the order or credit requirement for promoting another student. Waitlist entries may be removed by their student or an instructor/administrator, but removal does not affect credits.

Cancellation is allowed only before the class start time. A cancellation is on time when it is made at or before `class start - configured deadline`; after that point and before class start it is late. After class start, cancellation is unavailable and the instructor records attendance or absence instead. The local demo uses the browser's current date/time unless the user sets a test-only demo clock override. Acceptance tests set a specific policy and clock override so the expected classification is repeatable; the override is visibly identified as test-only and does not change the system clock.

### 3.6 Attendance and absence

An instructor may record attendance only for a confirmed reservation. Attendance changes it to attended, moves exactly one reserved credit to consumed and writes one corresponding history movement. Repeating the operation for an already attended reservation returns the existing result and creates no additional movement.

An instructor may mark a confirmed reservation absent after the class. The configured absence treatment determines whether its reserved credit is released or consumed. Repeating the absence operation must not create another movement. Attendance and absence are mutually exclusive terminal outcomes for a reservation.

### 3.7 Persistence and localStorage limitations

Persist the complete application state, including seed version, classes, reservations, waitlist sequence, policy, demo clock override and credit movements, as JSON in `localStorage`. On load, parse and validate stored state; if it is missing, initialize from fictional seed data. If stored data is malformed or has an unsupported seed version, report the error and offer reset-to-seed; do not silently discard it or report a successful operation. Provide an explicit reset that replaces the demo state with the seed.

`localStorage` persists only in the same browser profile/origin and can be cleared, blocked, edited or lost by the user. It is not shared across browsers or devices, does not identify a real user, does not enforce authorization, and provides no server-side backup or audit protection. It cannot reliably coordinate simultaneous operations across multiple users/tabs or guarantee transactional, race-safe capacity and credit updates. The app must validate each operation against the latest state it reads and write a consistent state for single-browser sequential use; it must not claim cross-tab or production concurrency guarantees. A corrupted or unavailable store must be surfaced to the user rather than silently treated as a successful save.

## 4. Functional Requirements

### FR-01 — Class availability

Show scheduled class date/time, name, instructor, confirmed count, capacity, remaining places and whether it is full. New classes default to capacity 12; administrators can configure capacity subject to the capacity rule.

### FR-02 — Reservation

Confirm a reservation only if a place and an available credit exist and the student has no active reservation for that class. A successful operation creates one confirmed reservation and moves one credit from available to reserved.

### FR-03 — Waiting list

When full, add the requesting student to the FIFO waitlist without credit use, including when their available-credit balance is zero. Prevent duplicate active entries. A waitlisted student is not represented as a confirmed reservation.

### FR-04 — Cancellation and promotion

Apply the configured cancellation policy, free the confirmed place, update the corresponding credit state, and scan the waitlist in FIFO sequence for the first currently eligible student. Promote that student and reserve one available credit. Do not lose or reorder skipped ineligible entries.

### FR-05 — Attendance, absence and history

Instructors can mark attendance or absence for a confirmed reservation. Apply the defined credit transition once only. Show each student's credit state/balance and movement history, including timestamp and related operation.

## 5. Acceptance Criteria

All criteria are reproducible by resetting to fictional seed data before each scenario. Test through the visible demo interface and verify the resulting class, reservation and ledger state. A criterion passes only if all listed assertions hold.

### AC-01 — Availability, capacity and reservation validation

**Setup:** Reset seed data and set the test-only demo clock to June 1, 2030. Use `student-alma` and `class-open`; use `student-celia` for the no-credit attempt and `class-small` for the full-class attempt.

**Actions:** As `student-alma`, confirm a reservation for `class-open`; attempt to book `class-open` as `student-celia`; attempt to book `class-small` as `student-brisa`; submit a second booking for `student-alma` on `class-open`. As `admin-andrea`, create one class without entering a capacity and another with capacity 2.

**Expected:** The successful request creates exactly one confirmed booking, changes one credit from available to reserved and reduces remaining places by one. The other three booking requests are rejected without changing counts or credit movements. No class has more confirmed bookings than its configured capacity. The first new class has capacity 12; the second has capacity 2 and its availability reflects that value.

### AC-02 — FIFO waitlist without credit use

**Setup:** Reset seed data and use `class-small`, which has capacity 3 and is full. Use `student-brisa`, `student-dalia` and `student-celia`.

**Actions:** As each student, in turn, request a place on `class-small` in the order Brisa, Dalia, Celia; then repeat Brisa's request.

**Expected:** Exactly three entries remain in the original order; the duplicate request is rejected. All are waiting, none is confirmed, and no credit movements or balances change. The zero-credit student can join the queue.

### AC-03 — Cancellation policy and eligible promotion

**Setup:** Reset seed data. Set the cancellation deadline to 24 hours and the late-cancellation treatment to consume (test fixture values only). Use `class-small`, `student-01`, `student-celia` and `student-brisa`. Set the test-only demo clock override to June 3, 2030, at 12:00 local time.

**Actions:** As `admin-andrea`, add Celia and then Brisa to `class-small`'s waitlist. Cancel `student-01`'s confirmed reservation.

**Expected:** Because the cancellation is 30 hours before class, `student-01`'s reserved credit returns to available; the place is freed; Celia remains waiting in her original position; Brisa is promoted to confirmed with exactly one credit moved from available to reserved. Capacity is not exceeded. Reset and repeat with the test-only clock override set to June 4, 2030, at 12:00 local time (6 hours before class). The cancellation is late: `student-01`'s credit is consumed under the configured test policy, and Brisa is still promoted while Celia remains waiting.

### AC-04 — Idempotent attendance, absence and auditable history

**Setup:** Reset seed data. Configure absence treatment to consume (test fixture value only), set the test-only demo clock to June 5, 2030, and select `instructor-laura`. Use the confirmed reservations for `student-02` and `student-03` in `class-small`.

**Actions:** Record attendance twice for `student-02`; mark `student-03` absent; attempt to mark `student-03` attended afterward.

**Expected:** Attendance consumes exactly one reserved credit and creates exactly one movement with the student, class/operation, type, amount and timestamp. The repeated attendance creates no movement. Absence follows the configured treatment exactly once; the contradictory attendance action is rejected and creates no movement. The displayed balances equal the ledger-derived available, reserved and consumed states.

### AC-05 — Persistence and local-demo boundaries

**Setup:** Reset seed data, set the test-only demo clock to June 1, 2030, then book `student-alma` into `class-open` and add `student-brisa` to `class-small`'s waitlist.

**Actions:** Reload the same browser origin; switch among the student, instructor and administrator demo roles; then use reset-to-seed.

**Expected:** Before reset, the booking, waitlist and credit history remain after reload and are visible consistently across simulated roles, subject to role-specific actions. Reset restores the documented fictional seed state. The UI identifies roles as demo-only and does not imply authentication or cross-device synchronization. Any storage failure is visibly reported and is not presented as a successful operation.

## 6. Out of Scope

- Production authentication, authorization or real role security.
- Backend, remote database, cross-device synchronization or transactional concurrency.
- Payments, package sales, financial management, messaging/notifications, marketing, video content, native applications or third-party integrations.
- Commercial cancellation/absence deadlines or credit rules beyond the values explicitly configured in the demo.

## 7. Definition of Done

- The MVP behavior is implemented in vanilla HTML, CSS and JavaScript and persisted in `localStorage`.
- All five acceptance criteria pass from a reset fictional seed using reproducible steps.
- The existing prototype's visual direction is preserved.
- Local storage limitations and simulated roles are communicated accurately.
