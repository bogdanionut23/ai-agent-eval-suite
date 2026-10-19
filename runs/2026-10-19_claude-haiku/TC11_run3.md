# TC11: Online appointment cancellation (24-hour rule)

**Requirement:** A patient can cancel an appointment online free of charge up to 24 hours before its start time. Later cancellations must be made by phone.

## Test cases

### Positive scenarios

**TC11-01: Cancel online more than 24 hours before start (free of charge)**
- **Preconditions:** Patient is logged in to the online portal. Patient has an upcoming, active appointment whose start time is 48 hours from now. The appointment carries no cancellation fee.
- **Steps:**
  1. Open the list of upcoming appointments.
  2. Select the appointment and choose "Cancel".
  3. Confirm the cancellation.
- **Expected result:** The appointment is cancelled. No cancellation fee is charged or shown. The patient sees a confirmation of the cancellation. The slot is released.

**TC11-02: Cancel online shortly over 24 hours before start**
- **Preconditions:** As TC11-01. The appointment starts in 24 hours and 1 minute.
- **Steps:** As TC11-01.
- **Expected result:** The cancellation succeeds free of charge, as in TC11-01.

**TC11-03: Cancel online with a large notice period**
- **Preconditions:** As TC11-01. The appointment starts in 30 days.
- **Steps:** As TC11-01.
- **Expected result:** The cancellation succeeds free of charge.

### Negative scenarios

**TC11-04: Online cancellation refused within 24 hours of start**
- **Preconditions:** Patient is logged in. Patient has an active appointment that starts in 23 hours and 59 minutes.
- **Steps:**
  1. Open the list of upcoming appointments.
  2. Select the appointment and choose "Cancel".
- **Expected result:** The online cancellation is not allowed. The patient is told that cancellations within 24 hours must be made by phone, and the phone number is shown. The appointment remains active.

**TC11-05: Online cancellation refused after the appointment start time**
- **Preconditions:** Patient is logged in. The appointment start time has already passed, and the appointment is still marked active.
- **Steps:** As TC11-04.
- **Expected result:** The online cancellation is refused with the same phone instruction as TC11-04. The appointment state is unchanged.

**TC11-06: Phone-only message is not bypassed by the API**
- **Preconditions:** Patient has an active appointment starting in 2 hours. A valid session token exists.
- **Steps:** Send a cancel request for the appointment directly to the cancellation endpoint, bypassing the UI.
- **Expected result:** The request is rejected with an error that states the phone requirement. The appointment remains active.

**TC11-07: Unauthenticated user cannot cancel**
- **Preconditions:** No user is logged in. An appointment ID is known.
- **Steps:** Attempt to cancel the appointment through the online cancel function.
- **Expected result:** The request is rejected. The user is asked to log in. The appointment is unchanged.

**TC11-08: Patient cannot cancel another patient's appointment**
- **Preconditions:** Patient A is logged in. Patient B has an appointment more than 24 hours away.
- **Steps:** Attempt to cancel Patient B's appointment by its ID.
- **Expected result:** The request is rejected as unauthorised. Patient B's appointment is unchanged.

**TC11-09: Already cancelled appointment cannot be cancelled again**
- **Preconditions:** Patient is logged in. Appointment was cancelled earlier and is no longer active. It starts more than 24 hours away.
- **Steps:** Attempt to cancel the appointment again.
- **Expected result:** The action is not offered or is rejected with a clear message. No duplicate cancellation is recorded and no fee is applied.

**TC11-10: Cancellation fails on a system error, no partial state**
- **Preconditions:** Patient is logged in. Appointment starts in 3 days. The scheduling back end is made to fail during the cancel call.
- **Steps:** Confirm the cancellation.
- **Expected result:** The patient sees an error message. The appointment remains active, not half-cancelled. The patient is not told it was cancelled.

### Boundary scenarios

**TC11-11: Exactly 24 hours before start**
- **Preconditions:** Patient is logged in. Appointment starts in exactly 24:00:00 from the current time.
- **Steps:** Attempt an online cancellation.
- **Expected result:** Depends on the open question on the inclusive/exclusive boundary (Q1). Do not finalise the expected result until Q1 is answered.

**TC11-12: One second before the 24-hour boundary**
- **Preconditions:** As TC11-11, but the start time is 23:59:59 away.
- **Steps:** Attempt an online cancellation.
- **Expected result:** Refused, with the phone instruction (per "later cancellations must be made by phone").

**TC11-13: One second after the 24-hour boundary**
- **Preconditions:** As TC11-11, but the start time is 24:00:01 away.
- **Steps:** Attempt an online cancellation.
- **Expected result:** Allowed free of charge.

**TC11-14: Time zone of the clinic versus the patient**
- **Preconditions:** Patient is in a different time zone from the clinic. The appointment start time is 25 hours away in clinic time and 23 hours away in patient time. (Exact zone rule is open, see Q3.)
- **Steps:** Attempt an online cancellation.
- **Expected result:** Depends on Q3. The result must be the same whichever zone the check uses, or the rule must be stated.

**TC11-15: Daylight saving time change inside the 24-hour window**
- **Preconditions:** A DST change falls between now and the appointment start, so the real elapsed time is 24 hours but the wall-clock difference is not.
- **Steps:** Attempt an online cancellation near the boundary.
- **Expected result:** The check uses elapsed time (to be confirmed, see Q3), and the decision is consistent with the rule.

**TC11-16: Appointment start is 0 seconds away (at the start time)**
- **Preconditions:** Patient is logged in. Appointment start time is now.
- **Steps:** Attempt an online cancellation.
- **Expected result:** Refused with the phone instruction.

## Open questions

1. **Boundary:** Is "up to 24 hours before" inclusive? Is a cancellation at exactly 24:00:00 allowed online?
2. **Fee:** The requirement says "free of charge" but gives no fee amount or rule for phone cancellations. What fee applies to a later cancellation, if any? Does an online cancellation after the cut-off get any fee message?
3. **Time zone:** Is the 24-hour check measured against the clinic's time zone, the patient's time zone, or UTC? Is elapsed time used across DST changes?
4. **Refused online cancellation:** Must the system show the phone number and the reason, or only a generic message? Should it be shown in the UI and also be enforced by the API?
5. **Who can cancel:** Only the patient, or also a carer, guardian, or proxy account? How is authentication done for "patient"?
6. **Appointment types:** Do rules differ for appointment types (e.g. procedures, tests, telehealth), recurring series, or group sessions?
7. **Already-started or past appointments:** What is the expected behaviour when the start time has passed and the appointment was never marked completed? Is the state "no-show" or still "active"?
8. **Rescheduling:** Does rescheduling online count as cancellation, and is it subject to the same 24-hour rule?
9. **Confirmation:** Does the patient get an email or SMS confirmation of a successful cancellation? Is that also required for a refused request?
10. **Slot release:** Is the freed slot immediately available to other patients, and is there a waiting-list step?
11. **Phone channel:** Is the phone cancellation process itself in scope for these tests? If so, what is the expected behaviour for a phone cancellation inside 24 hours?
12. **Audit:** Must every online attempt, including refused ones, be logged?
