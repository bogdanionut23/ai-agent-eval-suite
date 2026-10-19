# Test Cases: Online Appointment Cancellation (24-Hour Rule)

**Requirement:** A patient can cancel an appointment online free of charge up to 24 hours before its start time. Later cancellations must be made by phone.

## Assumptions (only what the requirement states)
- "Online cancellation" is available to a patient for their own appointment.
- The cutoff is start time minus 24h. Whether the exact 24:00:00 mark is inside the window is unclear (see OQ-1), so boundary results at exactly 24h are marked "per clarification".
- No behaviour is invented for fees, refunds, notifications, or phone handling beyond what is stated.

## Test Cases

### Positive

**TC-01: Cancel well before the cutoff**
- Preconditions: Patient logged in; has a confirmed appointment starting in 7 days.
- Steps: 1. Open "My appointments". 2. Select the appointment. 3. Choose Cancel online. 4. Confirm.
- Expected: Cancellation succeeds; appointment status becomes Cancelled; no charge is applied; the slot no longer appears in the patient's upcoming list.

**TC-02: Cancel just outside the cutoff (24h 1min before)**
- Preconditions: Appointment starts in 24 h 01 min.
- Steps: As TC-01.
- Expected: Online cancellation succeeds, free of charge.

**TC-03: Cancel several hours inside the free window on the same day as booking**
- Preconditions: Appointment booked a moment ago for 3 days ahead.
- Steps: Cancel online.
- Expected: Succeeds, free of charge.

**TC-04: Cancelled appointment is reflected everywhere**
- Preconditions: TC-01 completed.
- Steps: 1. Reload the appointment list. 2. View appointment history.
- Expected: Appointment shown as Cancelled consistently; cannot be cancelled again.

### Negative

**TC-05: Online cancel attempt inside 24h (23h 59min before)**
- Preconditions: Appointment starts in 23 h 59 min.
- Steps: Try to cancel online.
- Expected: Online cancellation is rejected or the option is unavailable; the user is told to cancel by phone; appointment remains Scheduled; no charge applied.

**TC-06: Cancel online 1 hour before start**
- Preconditions: Appointment starts in 1 h.
- Steps: Try to cancel online.
- Expected: Same as TC-05.

**TC-07: Cancel online after the start time has passed**
- Preconditions: Appointment start time was 10 min ago.
- Steps: Try to cancel online.
- Expected: Online cancellation not possible; status unchanged. (Exact message/phone guidance per OQ-6.)

**TC-08: Bypass attempt via direct request/URL after cutoff**
- Preconditions: Appointment starts in 2 h; patient knows the cancel endpoint/URL.
- Steps: Send the cancel request directly (API call or saved link), bypassing the UI.
- Expected: Server-side validation rejects it; appointment unchanged.

**TC-09: Cancel button left open across the cutoff (stale page)**
- Preconditions: Appointment starts in 24 h 02 min; cancel page loaded.
- Steps: 1. Wait 3 minutes (now under 24h). 2. Click Confirm.
- Expected: Server evaluates the time at submission; request is rejected with phone guidance; appointment unchanged.

**TC-10: Cancel another patient's appointment**
- Preconditions: Patient A logged in; appointment belongs to Patient B, 7 days away.
- Steps: Attempt to cancel B's appointment by ID.
- Expected: Request denied (authorization failure); appointment unchanged.

**TC-11: Unauthenticated cancellation**
- Preconditions: Not logged in.
- Steps: Open the cancel link/endpoint.
- Expected: Redirected to login or denied; no change.

**TC-12: Cancel an already cancelled appointment**
- Preconditions: Appointment already Cancelled.
- Steps: Attempt to cancel again.
- Expected: Option unavailable or a clear "already cancelled" message; no duplicate processing or charge.

**TC-13: Double-submit / concurrent cancel**
- Preconditions: Appointment 5 days away.
- Steps: Click Confirm twice quickly (or from two sessions).
- Expected: Cancelled once; no duplicate side effects; no error corruption.

### Boundary

**TC-14: Exactly 24h 00min 00s before start**
- Preconditions: Cancellation submitted at exactly start minus 24:00:00.
- Steps: Cancel online.
- Expected: Per clarification of OQ-1 ("up to 24 hours" suggests allowed). Result must be deterministic and consistent between UI and server.

**TC-15: 1 second before the cutoff (24h 00min 01s)**
- Expected: Allowed, free.

**TC-16: 1 second after the cutoff (23h 59min 59s)**
- Expected: Online cancellation rejected; phone guidance shown.

**TC-17: Time zone difference**
- Preconditions: Patient's browser time zone differs from the clinic's; appointment 24h 30min away by clinic time.
- Steps: Cancel online.
- Expected: Cutoff calculated in the time zone defined by OQ-3; result matches that rule regardless of device time zone.

**TC-18: Daylight saving transition inside the 24h span**
- Preconditions: Appointment the day after a DST change; submission about 24h 30min before start in real elapsed time.
- Expected: Cutoff uses real elapsed 24 hours (or the rule clarified in OQ-4); no off-by-one-hour error.

**TC-19: Appointment at midnight / early morning**
- Preconditions: Appointment at 00:15 two days ahead.
- Steps: Cancel at 00:10 the day before... (just over 24h) and again at 00:20.
- Expected: Cutoff computed from the exact start time, not by calendar date.

**TC-20: Appointment rescheduled to a new time**
- Preconditions: Appointment moved from 5 days ahead to 12 hours ahead.
- Steps: Try online cancel.
- Expected: Rule uses the current start time; online cancel rejected.

**TC-21: Appointment booked less than 24h in advance**
- Preconditions: Appointment booked now for 6 hours ahead.
- Steps: Try online cancel immediately after booking.
- Expected: Per requirement, online cancel is not allowed (inside 24h); confirm with OQ-7.

### Non-functional / UI

**TC-22: Messaging and discoverability of the rule**
- Steps: View the cancel screen for an appointment (a) outside and (b) inside 24h.
- Expected: (a) Cancel is offered and states it is free; (b) the user is clearly told to call, with the phone number visible.

**TC-23: Accessibility of cancel flow**
- Steps: Complete cancel using keyboard only and a screen reader.
- Expected: All controls reachable; confirmation and error messages announced.

## Open Questions (not assumed in the tests above)
1. Is "up to 24 hours before" inclusive at exactly 24:00:00? (Affects TC-14.)
2. Is the 24 hours measured from the appointment start time to the moment of submission (server time), and which clock is authoritative?
3. Which time zone applies: clinic, patient, or UTC?
4. How is the cutoff handled across DST changes: elapsed hours or wall-clock?
5. What does "free of charge" imply: do any fees exist for phone cancellations or late cancellations? Is a prepaid deposit refunded on online cancellation?
6. What exactly should the system show after the cutoff: hide the button, disable it, or show an error? Is a phone number displayed? What are phone hours, and what if the clinic is closed at that time?
7. Can online cancellation be used for appointments booked less than 24 hours ahead (never in the free window)?
8. Which appointment types are covered (all, or exclude e.g. surgeries, telehealth, group sessions)? Are there exceptions (emergencies)?
9. Who can cancel: only the patient, or also guardians/proxies/caregivers for minors or dependents?
10. Are there confirmations, e-mail/SMS notifications, or calendar updates after cancel, and are these required?
11. Is a cancellation reason mandatory or optional?
12. Is cancelled slot released immediately for other patients/waiting list?
13. For recurring or multi-appointment series, does cancelling apply to one or all?
14. Are there audit/logging requirements for cancellations, including those rejected after the cutoff?
15. Can an online cancellation be undone, and by whom?
16. After the start time has passed (no-show), is the appointment handled as a phone-only or a different flow?
