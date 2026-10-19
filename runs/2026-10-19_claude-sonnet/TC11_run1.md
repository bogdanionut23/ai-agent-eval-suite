# Test Cases: Online Appointment Cancellation (24-hour rule)

Requirement: A patient can cancel an appointment online free of charge up to 24 hours before its start time. Later cancellations must be made by phone.

## Assumptions used for the test cases (to be confirmed, see open questions)
- "Up to 24 hours before" is read as: online cancellation is allowed while (start time - now) >= 24h 00m 00s. The exact boundary (exactly 24:00:00) is an open question (OQ-1); TC-06 is written against the most literal reading and must be confirmed.
- Time is measured in the clinic's local time zone.
- No behaviour is invented for the phone channel beyond what the requirement states (it is the only route after the cut-off).

## Test cases

### Positive

**TC-01 Cancel well before the cut-off**
- Preconditions: Patient logged in; has a confirmed appointment starting in 5 days.
- Steps: 1. Open "My appointments". 2. Select the appointment. 3. Choose "Cancel appointment". 4. Confirm.
- Expected: Cancellation succeeds online; appointment status becomes Cancelled; no fee is charged; patient sees a confirmation; the slot is released.

**TC-02 Cancel just inside the limit (24h 01min before)**
- Preconditions: Patient logged in; appointment starts in 24h 01min.
- Steps: As TC-01.
- Expected: Cancellation succeeds online, free of charge.

**TC-03 Cancel the same appointment type for different booking sources**
- Preconditions: Patient has one appointment booked online and one booked by phone/reception, both > 48h away.
- Steps: Cancel each online.
- Expected: Both can be cancelled online free of charge (see OQ-6 if the business rule differs).

**TC-04 Cancellation is reflected everywhere**
- Preconditions: Appointment starts in 3 days; patient has notifications enabled.
- Steps: Cancel online; then check the appointment list, the calendar view, and email/SMS.
- Expected: Appointment shows as Cancelled in all views; a confirmation notification is sent (only if notifications are in scope, see OQ-8).

### Boundary

**TC-05 Cancel at 24h 00min 01s before start**
- Preconditions: Appointment at 14:00:00 on day D; test clock set to 14:00:00 - 24h - 1s... i.e. 13:59:59 on D-1 is NOT this case; use 13:59:59 on D-1 minus 0 = 24h 00m 01s before start is 13:59:59 on D-1.
- Steps: Attempt online cancellation.
- Expected: Allowed, free of charge.

**TC-06 Cancel at exactly 24h 00min 00s before start**
- Preconditions: Appointment at 14:00:00 on day D; test clock at 14:00:00 on D-1.
- Steps: Attempt online cancellation.
- Expected (per literal reading of "up to 24 hours"): Allowed, free of charge. Mark as BLOCKED until OQ-1 is answered.

**TC-07 Cancel at 23h 59min 59s before start**
- Preconditions: Appointment at 14:00:00 on day D; test clock at 14:00:01 on D-1.
- Steps: Attempt online cancellation.
- Expected: Online cancellation is not offered or is rejected; patient is told to cancel by phone; appointment remains Active; no fee behaviour is invented (see OQ-3).

**TC-08 Cancel at 23h 00min before start**
- Preconditions: Appointment starts in 23h.
- Steps: Attempt online cancellation.
- Expected: Same as TC-07.

**TC-09 Cut-off crossing during the session**
- Preconditions: Appointment at 14:00 on day D; patient opens the cancel dialog at 13:58 on D-1 (just before the cut-off) and waits.
- Steps: 1. Open cancel dialog at 13:58. 2. Wait until 14:01. 3. Click Confirm.
- Expected: The system evaluates the rule at the time of submission (server time), so the request is rejected with a message to call; appointment stays Active. (Confirm intended behaviour in OQ-2.)

**TC-10 Midnight and date boundaries**
- Preconditions: Appointment at 00:30 on day D.
- Steps: Cancel at 00:30 on D-1 (exactly 24h) and at 00:31 on D-1.
- Expected: Behaviour is consistent with TC-06 and TC-07; the date change does not cause errors.

**TC-11 Daylight-saving change between cancellation time and appointment**
- Preconditions: Appointment the morning after a DST switch (clock moves forward or back); cancellation attempted so that the wall-clock difference is 24h but the real elapsed time is 23h or 25h.
- Steps: Attempt online cancellation at both sides of the real 24h point.
- Expected: The rule is applied on real elapsed time (or as defined by OQ-4); no off-by-one-hour error.

**TC-12 Time zone difference (patient abroad)**
- Preconditions: Patient's device/browser is in a time zone different from the clinic.
- Steps: Attempt cancellation near the cut-off.
- Expected: The cut-off is calculated against clinic time, independent of device time zone (see OQ-4).

### Negative

**TC-13 Cancel within 24h, online option unavailable**
- Preconditions: Appointment starts in 6 hours.
- Steps: Open the appointment details.
- Expected: Online cancel button is disabled or hidden; a message explains that cancellation within 24h must be done by phone and shows the phone number.

**TC-14 Bypass via direct URL / API call**
- Preconditions: Appointment starts in 6 hours; patient authenticated.
- Steps: Send the cancel request directly (API call or saved URL) without using the UI.
- Expected: Server rejects the request (4xx with clear error); appointment remains Active. The rule is enforced server-side, not only in the UI.

**TC-15 Manipulated client clock**
- Preconditions: Appointment starts in 6 hours; patient's device clock set 3 days earlier.
- Steps: Attempt online cancellation.
- Expected: Rejected; server time is used.

**TC-16 Cancel another patient's appointment**
- Preconditions: Patient A logged in; Patient B has an appointment in 5 days.
- Steps: Patient A sends a cancel request for B's appointment ID.
- Expected: Rejected (403/404); B's appointment unaffected.

**TC-17 Unauthenticated cancellation**
- Preconditions: Not logged in.
- Steps: Open a cancel link or call the cancel endpoint.
- Expected: Redirected to login or rejected; nothing is cancelled.

**TC-18 Already cancelled appointment**
- Preconditions: Appointment already cancelled.
- Steps: Attempt to cancel again (also via repeated request / double click).
- Expected: No second cancellation; no duplicate notification or error crash; clear message.

**TC-19 Past appointment**
- Preconditions: Appointment start time is in the past (completed or no-show).
- Steps: Attempt online cancellation.
- Expected: Not possible; no state change.

**TC-20 Appointment already started (in progress)**
- Preconditions: Appointment started 10 minutes ago.
- Steps: Attempt online cancellation.
- Expected: Rejected; patient is told to contact the clinic by phone or no action offered (see OQ-5).

**TC-21 Concurrent cancellation**
- Preconditions: Same appointment open in two sessions/devices (or patient and staff cancel simultaneously).
- Steps: Confirm cancellation in both at nearly the same time.
- Expected: Exactly one cancellation is recorded; the other gets a clear "already cancelled" message; slot released once; no double refund/charge.

**TC-22 Backend failure during cancellation**
- Preconditions: Appointment in 5 days; simulate DB or service error.
- Steps: Confirm cancellation.
- Expected: Clear error shown; appointment state remains consistent (either fully cancelled or still Active); the slot is not released unless the cancellation was saved.

**TC-23 User aborts the confirmation dialog**
- Preconditions: Appointment in 5 days.
- Steps: Start cancellation, then choose "No / Back".
- Expected: Appointment remains Active; no notification sent.

### Non-functional / other

**TC-24 Free of charge verification**
- Preconditions: Appointment (prepaid or with a deposit, if applicable) in 3 days.
- Steps: Cancel online; review billing/payment records.
- Expected: No cancellation fee is created. Handling of any prepaid amount is not tested until OQ-7 is answered.

**TC-25 Accessibility and responsive behaviour**
- Preconditions: Appointment in 3 days.
- Steps: Perform cancellation using keyboard only, a screen reader, and a mobile viewport.
- Expected: The flow is usable and the confirmation and error messages are announced/readable.

**TC-26 Multiple appointments**
- Preconditions: Patient has appointments at 3 days, 20h and 2 days from now.
- Steps: Try cancelling each one.
- Expected: The 3-day and 2-day appointments can be cancelled online; the 20h one cannot. Each is evaluated independently.

**TC-27 Audit trail**
- Preconditions: Appointment in 3 days.
- Steps: Cancel online; inspect the audit log.
- Expected: Log records who, when (timestamp), channel (online) and the appointment ID. (Only if auditing is required, see OQ-8.)

## Open questions (not assumed in the test cases above)

1. **Exact boundary:** Is cancellation at exactly 24h 00m 00s before start allowed ("up to 24 hours" inclusive) or not? What granularity is used (seconds, minutes)?
2. **When is the rule evaluated:** At the time the patient opens the dialog or at the time the cancellation is submitted? Which clock (server time) is authoritative?
3. **Late cancellation consequences:** The requirement says cancellations within 24h must be by phone. Is a phone cancellation also free, or is a fee charged? Is a no-show treated differently? "Free of charge" is stated only for online cancellation, which implies something else may apply later but it is not defined.
4. **Time zone and DST:** Which time zone defines the start time and cut-off (clinic, patient)? Is 24 hours real elapsed time or 24 wall-clock hours across DST changes?
5. **Started or past appointments:** What should happen for appointments already in progress or in the past? Is phone cancellation still possible?
6. **Scope of appointments:** Does the rule apply to all appointment types (e.g. first visits, telemedicine, procedures, specialist visits, group sessions) and all booking sources (online, phone, reception)? Are there exceptions with a longer notice period?
7. **Payments:** If a deposit or prepayment exists, is it refunded on online cancellation? How fast?
8. **Notifications and audit:** Must the patient receive an email/SMS confirmation? Must the clinic be notified? Is an audit record required?
9. **Phone cancellation details:** What are the phone hours? What if the clinic is closed in the 24h window (e.g. an appointment early Monday morning, cut-off on Sunday)? Is there an out-of-hours option?
10. **Rescheduling:** Is rescheduling (changing time) covered by the same rule or does it follow a different one?
11. **Who can cancel:** Can a guardian, caregiver or proxy cancel for a minor or dependent? Are guest/unregistered patients supported (cancel link from email)?
12. **UI behaviour after cut-off:** Should the online option be hidden, disabled, or shown with an explanation and the phone number? Exact wording and languages are not defined.
13. **Recurring or multi-visit series:** Is cancellation per occurrence, per series, or both?
14. **Waitlist / slot release:** Should the cancelled slot be offered to the waiting list automatically?

## Notes
- TC-05 preconditions wording is deliberately minimal: use a controllable test clock; 24h 00m 01s before 14:00:00 on day D is 13:59:59 on D-1.
- TC-06, TC-09, TC-11, TC-12, TC-20 and TC-24 depend on open questions and should be treated as blocked until answered.
