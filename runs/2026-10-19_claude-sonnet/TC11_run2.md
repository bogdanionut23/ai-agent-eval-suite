# Test Cases: Online Appointment Cancellation (24-hour rule)

Requirement: A patient can cancel an appointment online free of charge up to 24 hours before its start time. Later cancellations must be made by phone.

Notation: T = appointment start time. The cutoff is T - 24h. All times are in the clinic's time zone unless stated otherwise (see open questions Q1, Q2). Cases whose result depends on an open question are marked [Q#] and state the assumption used.

## Positive

**TC-001 - Cancel well before the cutoff**
- Preconditions: Patient is logged in. A confirmed appointment exists at T = now + 7 days.
- Steps: 1. Open "My appointments". 2. Select the appointment. 3. Choose "Cancel". 4. Confirm.
- Expected: Cancellation succeeds. Status becomes "Cancelled". No fee is charged. The slot is released. A confirmation is shown (and sent, if notifications exist, see Q9).

**TC-002 - Cancel shortly before the cutoff (more than 24h)**
- Preconditions: Confirmed appointment at T = now + 25h.
- Steps: As TC-001.
- Expected: Cancellation succeeds free of charge.

**TC-003 - Cancel at a distant future date**
- Preconditions: Confirmed appointment at T = now + 6 months (if booking that far ahead is allowed).
- Steps: As TC-001.
- Expected: Cancellation succeeds free of charge.

**TC-004 - Cancelled slot becomes bookable again**
- Preconditions: TC-001 completed.
- Steps: 1. As another patient (or the same), search availability for that slot.
- Expected: The slot appears as available. [Q8]

**TC-005 - Cancelled appointment no longer shown as upcoming**
- Preconditions: TC-001 completed.
- Steps: 1. Reload "My appointments" in upcoming view.
- Expected: Appointment is not listed as upcoming; it appears in history/cancelled with status "Cancelled".

## Boundary

**TC-010 - Exactly 24h 00m 00s before start** [Q3]
- Preconditions: Appointment at T. Current time = T - 24:00:00.
- Steps: Attempt online cancellation.
- Expected: Per the wording "up to 24 hours before", cancellation is allowed (assumed inclusive). Verify the actual behaviour matches the confirmed answer to Q3.

**TC-011 - 1 minute earlier than the cutoff**
- Preconditions: Current time = T - 24h 01m.
- Steps: Attempt online cancellation.
- Expected: Allowed, free of charge.

**TC-012 - 1 second earlier than the cutoff**
- Preconditions: Current time = T - 24h 00m 01s (time can be controlled/mocked).
- Steps: Attempt online cancellation.
- Expected: Allowed, free of charge.

**TC-013 - 1 second after the cutoff** [Q3]
- Preconditions: Current time = T - 23h 59m 59s.
- Steps: Attempt online cancellation.
- Expected: Online cancellation is refused. The user is told to cancel by phone and is shown the phone number. The appointment stays "Confirmed".

**TC-014 - 1 minute after the cutoff**
- Preconditions: Current time = T - 23h 59m.
- Steps: Attempt online cancellation.
- Expected: Refused as in TC-013.

**TC-015 - Page opened before the cutoff, submitted after it**
- Preconditions: Appointment at T. Patient opens the cancel dialog at T - 24h 02m and waits.
- Steps: 1. Click "Confirm" at T - 23h 58m.
- Expected: The server evaluates the time at submission, rejects the request, and the appointment stays "Confirmed". The UI explains to cancel by phone. (The check must not rely on page-load time.)

**TC-016 - Cancel button state across the cutoff**
- Preconditions: Appointment at T; "My appointments" page open at T - 24h 01m.
- Steps: 1. Refresh the page at T - 23h 59m.
- Expected: The online "Cancel" action is no longer available (or is disabled with an explanation and phone number). Q4 covers what exactly is displayed.

**TC-017 - Appointment starting in less than a minute / already started** [Q5]
- Preconditions: Current time = T - 30s, and then T + 5 min.
- Steps: Attempt online cancellation.
- Expected: Refused in both cases. Online cancellation is not possible; message indicates phone. (Behaviour for started appointments is unspecified, see Q5.)

**TC-018 - Day boundary: appointment just after midnight**
- Preconditions: Appointment at 00:15 on day D+1. Current time is 00:14 on day D (24h 01m before).
- Steps: Attempt online cancellation.
- Expected: Allowed. The check uses elapsed time, not calendar days.

**TC-019 - Daylight saving time change between now and the appointment** [Q2]
- Preconditions: Appointment at 09:00 on the day after a clock change (spring forward / fall back). Current time is 09:00 on the previous day, local time.
- Steps: Attempt online cancellation.
- Expected: Cutoff is computed on real elapsed time (24 real hours; 23 or 25 wall-clock hours across a DST shift). Verify against the confirmed rule in Q2.

**TC-020 - Patient in a different time zone than the clinic** [Q1]
- Preconditions: Patient's browser/device is set to a time zone several hours different from the clinic's.
- Steps: Attempt cancellation at 24h 30m and at 23h 30m before the appointment (clinic time).
- Expected: The decision is based on a single reference time (assumed clinic time/UTC instant), regardless of the patient's device clock or time zone. Changing the device clock does not change the outcome.

## Negative

**TC-030 - Late cancellation attempt through the UI**
- Preconditions: Appointment at T = now + 3h.
- Steps: 1. Open the appointment. 2. Look for the cancel option.
- Expected: Online cancel is unavailable or rejected. A clear message says late cancellations must be made by phone, with the phone number and opening hours. The appointment remains "Confirmed".

**TC-031 - Late cancellation attempt by calling the API directly**
- Preconditions: Appointment at T = now + 3h. Valid patient session/token.
- Steps: 1. Send the cancel request directly to the cancellation endpoint, bypassing the UI.
- Expected: Request is rejected with a business-rule error (not a 500). The appointment status is unchanged. No fee or side effect.

**TC-032 - Tampered client clock**
- Preconditions: Appointment at T = now + 3h. Set the device clock back by 2 days.
- Steps: Attempt online cancellation.
- Expected: Rejected, because the server time is used.

**TC-033 - Cancel a different patient's appointment**
- Preconditions: Patient A logged in; appointment belongs to patient B, more than 24h away.
- Steps: 1. Send a cancel request for B's appointment ID as A.
- Expected: Rejected (403/404). Appointment unchanged.

**TC-034 - Not logged in / expired session**
- Preconditions: No valid session.
- Steps: Attempt to cancel (via UI link or API).
- Expected: Redirected to login or 401. No cancellation occurs. After login, the user can cancel, subject to the 24h rule.

**TC-035 - Cancel an already cancelled appointment**
- Preconditions: Appointment already "Cancelled".
- Steps: 1. Repeat the cancel request (e.g. via a stale tab or a replayed API call).
- Expected: No second cancellation, no error 500, no duplicate notification or fee. A friendly "already cancelled" result. [Q10]

**TC-036 - Double submit**
- Preconditions: Appointment at T = now + 3 days.
- Steps: 1. Double-click "Confirm" or submit twice quickly.
- Expected: Exactly one cancellation is processed. No duplicate emails/side effects.

**TC-037 - Cancel a non-existent or invalid appointment ID**
- Preconditions: Logged in.
- Steps: 1. Request cancellation of an ID that does not exist, is malformed, or is empty.
- Expected: Clean error (404/400). No data leak or server error.

**TC-038 - Cancel an appointment that is not in a cancellable state** [Q6]
- Preconditions: Appointments in states such as "Completed", "No-show", "Pending approval", "Rescheduled".
- Steps: Attempt online cancellation of each.
- Expected: Behaviour per the confirmed rule. At minimum, completed/past appointments cannot be cancelled and no fee is incurred.

**TC-039 - Declining the confirmation dialog**
- Preconditions: Appointment at T = now + 3 days.
- Steps: 1. Click "Cancel". 2. Choose "No / Keep appointment" in the confirmation.
- Expected: The appointment stays "Confirmed".

**TC-040 - Concurrent cancellation and rescheduling or clinic-side change** [Q7]
- Preconditions: Same appointment open in two sessions (or patient and staff).
- Steps: 1. Patient cancels in session 1. 2. Session 2 tries to reschedule or cancel it.
- Expected: The second action fails with a clear message; the data stays consistent (no appointment that is both cancelled and rescheduled).

## Non-functional / side effects

**TC-050 - No charge on in-time cancellation**
- Preconditions: Appointment with a deposit or prepayment, if the system supports them (Q11).
- Steps: Cancel at T - 48h.
- Expected: No fee is charged; any prepaid amount is handled per the confirmed refund rule.

**TC-051 - Confirmation message or email**
- Preconditions: Notifications enabled.
- Steps: Cancel at T - 48h.
- Expected: Patient receives an on-screen confirmation (and an email/SMS if the system defines it) with the correct appointment details and time zone. [Q9]

**TC-052 - Audit trail**
- Preconditions: TC-001 completed.
- Steps: Check the audit log or admin view.
- Expected: Records who cancelled, when (server timestamp), and via which channel (online). [Q9]

**TC-053 - Accessibility and wording of the late-cancel message**
- Preconditions: Appointment within 24h.
- Steps: Open the appointment using a screen reader and keyboard only.
- Expected: Disabled/missing cancel control and the phone instruction are announced and reachable; the phone number is a tappable link on mobile.

**TC-054 - Phone cancellation path exists for late cancellations** [Q12]
- Preconditions: Appointment at T = now + 3h.
- Steps: Cancel by phone through the reception.
- Expected: Staff can cancel it; the status becomes "Cancelled". Whether a fee applies is per Q12. (This is outside the online flow but verifies the other half of the requirement.)

## Open questions (not assumed in the test cases above)

1. Which time zone defines the 24-hour window: the clinic's, the patient's, or UTC? What if the patient is travelling?
2. How is DST handled: 24 elapsed hours or 24 wall-clock hours?
3. Is the boundary inclusive? "Up to 24 hours before" suggests exactly 24:00:00 is still allowed, but this should be confirmed. What is the time precision (seconds, minutes)?
4. When the window has passed, should the Cancel button be hidden, disabled, or shown with an error? What exact message, phone number, and opening hours are displayed?
5. What happens for an appointment that has already started or passed? Is phone cancellation still possible, and is this a no-show?
6. Which appointment states can be cancelled (pending, confirmed, rescheduled, checked in, telehealth, recurring)? For a recurring series, does cancelling affect one occurrence or all?
7. How should conflicts with staff-side changes (clinic cancels or reschedules) be handled?
8. Does a cancelled slot go back to the public pool immediately, or is there a waitlist process?
9. What notifications (email/SMS) are sent on cancellation, to whom, and what is audited?
10. Should repeating a cancel on an already-cancelled appointment return success (idempotent) or an error?
11. "Free of charge" implies charges exist. What is the fee for late cancellation by phone, if any? Does the online flow involve prepayments or deposits, and how are refunds handled?
12. Are late cancellations by phone free, charged, or subject to a no-show policy? Who is allowed to waive the fee?
13. Are there exceptions (emergencies, clinic-initiated changes, different appointment types or specialties with different windows, or minors/guardians cancelling on someone's behalf)?
14. Can a patient reschedule online instead of cancelling, and does the same 24-hour rule apply?
15. Are there limits on the number of online cancellations per patient, or on cancellations of appointments booked very recently (e.g. booked 2 hours before the start)? Does a booking made less than 24h before the start allow any online cancellation at all?
16. Does the rule apply to all channels (web, mobile app, email link, kiosk)? Is a one-click cancel link in a reminder email in scope?
