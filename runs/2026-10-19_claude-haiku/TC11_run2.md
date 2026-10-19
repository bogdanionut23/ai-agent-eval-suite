# TC11 - Test cases: online appointment cancellation (run 2)

## Requirement

A patient can cancel an appointment online free of charge up to 24 hours before its start time. Later cancellations must be made by phone.

## Assumptions used for the test cases

- "Online" means the patient cancels through the patient-facing web or app channel, with the patient logged in.
- "Free of charge" means no cancellation fee is charged for an online cancellation within the window.
- Times are compared as absolute instants, and the start time is the appointment's start in the clinic's time zone.
- Expected results that depend on an open question are marked "(pending OQ-n)".

## Test cases

### Positive scenarios

**TC11-01 - Cancel online well before the 24-hour window**
- Preconditions: Patient is logged in. Patient has one upcoming, confirmed appointment starting 72 hours from now. No cancellation fee applies to the appointment.
- Steps:
  1. Open "My appointments".
  2. Select the appointment.
  3. Choose "Cancel appointment".
  4. Confirm the cancellation.
- Expected result: The appointment status changes to "Cancelled". A confirmation message is shown with no fee or charge mentioned. The slot is released for other bookings. A cancellation confirmation is sent to the patient (channel per OQ-7).

**TC11-02 - Cancel online at 25 hours before start**
- Preconditions: Patient is logged in. Appointment starts exactly 25 hours from now.
- Steps: Same as TC11-01.
- Expected result: Cancellation succeeds, the status is "Cancelled", and no fee is charged.

**TC11-03 - Cancel online with a prepaid appointment inside the free window (if prepayment exists)**
- Preconditions: Patient is logged in. Appointment starts in 48 hours. Appointment was prepaid.
- Steps: Cancel the appointment online.
- Expected result: Cancellation succeeds. The prepayment is refunded in full, or handled as stated in the refund rule (pending OQ-5).

**TC11-04 - Cancelled appointment no longer appears as active**
- Preconditions: TC11-01 completed.
- Steps: Open "My appointments" and the upcoming list.
- Expected result: The cancelled appointment is shown under cancelled items or is removed from upcoming items. It is not counted as an active booking.

### Negative scenarios

**TC11-05 - Online cancel blocked inside the 24-hour window**
- Preconditions: Patient is logged in. Appointment starts in 23 hours.
- Steps:
  1. Open "My appointments".
  2. Select the appointment.
  3. Attempt "Cancel appointment".
- Expected result: Online cancellation is not completed. The appointment status is unchanged. The patient sees a message that online cancellation is no longer possible and that the cancellation must be made by phone. The message includes the clinic phone number (pending OQ-3).

**TC11-06 - Online cancel blocked for a past appointment**
- Preconditions: Patient is logged in. Appointment start time has already passed.
- Steps: Attempt to cancel the appointment online.
- Expected result: Cancellation is refused. The status is unchanged. The patient is directed to phone cancellation or the message says the appointment has already started or passed (pending OQ-4).

**TC11-07 - Patient cannot cancel another patient's appointment**
- Preconditions: Patient A is logged in. Appointment belongs to patient B and starts in 72 hours.
- Steps: Attempt to cancel patient B's appointment by changing the appointment ID in the request or URL.
- Expected result: The request is refused with an authorization error. Patient B's appointment is unchanged. The attempt is logged.

**TC11-08 - Unauthenticated user cannot cancel**
- Preconditions: User is not logged in. A valid appointment ID is known.
- Steps: Call the online cancellation endpoint or open the cancellation page directly.
- Expected result: The user is asked to log in or receives an authorization error. Nothing is cancelled.

**TC11-09 - Already cancelled appointment cannot be cancelled again**
- Preconditions: Appointment was cancelled earlier, with more than 24 hours remaining.
- Steps: Attempt to cancel the same appointment again.
- Expected result: The cancel action is not offered, or a message says the appointment is already cancelled. No second cancellation or fee is recorded.

**TC11-10 - Late cancellation online does not cancel the appointment even when the request is sent directly to the API**
- Preconditions: Appointment starts in 2 hours. Patient is logged in.
- Steps: Send the cancellation request directly to the API, bypassing the UI.
- Expected result: The server rejects the request with a clear error (for example, 409 or 422, with a "phone cancellation required" reason). The appointment remains active. The rule is enforced on the server, not only in the UI.

**TC11-11 - Double submit does not cancel twice or charge twice**
- Preconditions: Appointment starts in 72 hours. Patient is logged in.
- Steps: Click "Confirm cancellation" twice quickly, or send two identical requests at the same time.
- Expected result: The appointment is cancelled once. Only one confirmation is sent. No duplicate fee or refund is created.

**TC11-12 - Network failure during cancellation**
- Preconditions: Appointment starts in 72 hours. Connection drops after the request is sent.
- Steps: Confirm cancellation, then reload and check the appointment.
- Expected result: Either the appointment is fully cancelled with a confirmation, or it stays active with a retry option. It is never left in a half-cancelled state (for example, cancelled with no slot release, or with a fee charged and no status change).

### Boundary scenarios

**TC11-13 - Exactly 24 hours before start (pending OQ-1)**
- Preconditions: Patient is logged in. Appointment starts at exactly 24:00:00 from the current server time.
- Steps: Attempt online cancellation.
- Expected result: If the boundary is inclusive, cancellation succeeds with no fee (pending OQ-1). If exclusive, the request is blocked with the phone instruction.

**TC11-14 - One minute inside the window (23 hours 59 minutes before start)**
- Preconditions: Appointment starts in 23 hours 59 minutes.
- Steps: Attempt online cancellation.
- Expected result: Cancellation is blocked. The patient is directed to phone cancellation.

**TC11-15 - One minute outside the window (24 hours 1 minute before start)**
- Preconditions: Appointment starts in 24 hours 1 minute.
- Steps: Attempt online cancellation.
- Expected result: Cancellation succeeds with no fee.

**TC11-16 - Window measured in the clinic's time zone across a daylight saving change**
- Preconditions: Clinic time zone switches to or from daylight saving time within the 24 hours before the appointment. Patient is in a different time zone.
- Steps: Attempt cancellation at the boundary, once in the patient's local time zone and once in the clinic's time zone.
- Expected result: The 24-hour rule is applied to the actual elapsed time, and the result is the same regardless of the viewer's time zone (pending OQ-2).

**TC11-17 - Client clock is wrong**
- Preconditions: Patient's device clock is set 2 hours ahead, so the appointment looks like it is more than 24 hours away when it is inside the window.
- Steps: Attempt online cancellation.
- Expected result: The server decides based on server time. The cancellation is blocked.

**TC11-18 - Multiple appointments, only one in the window**
- Preconditions: Patient has appointment X starting in 30 hours and appointment Y starting in 10 hours.
- Steps: Cancel X online, then attempt to cancel Y online.
- Expected result: X is cancelled. Y is blocked with the phone instruction. The decision is made per appointment.

**TC11-19 - Cancellation at the start time itself**
- Preconditions: Appointment start time is now.
- Steps: Attempt online cancellation.
- Expected result: Cancellation is blocked with the phone instruction (or the appointment is shown as started, per OQ-4).

## Open questions

1. **Boundary:** Is "up to 24 hours before" inclusive? Is cancelling at exactly 24:00:00 free online?
2. **Time zone:** Is the 24-hour window calculated from the clinic's time zone, the patient's time zone, or UTC? Which "start time" applies if the clinic is in a different time zone?
3. **Phone instruction:** What exact message is shown when online cancellation is blocked? Is the clinic phone number shown on screen? Is it per clinic or global?
4. **Past or started appointments:** What is the expected behavior for an appointment that has already started or passed? Is it blocked online, and what message is shown?
5. **Fees and refunds:** What is the fee amount for a late cancellation by phone? How is a prepaid appointment refunded for an online cancellation inside the free window, and for a phone cancellation?
6. **Who may cancel:** Can a guardian, proxy, or caregiver cancel on behalf of a patient online? Do the same rules apply?
7. **Confirmation:** Which channel is used for the cancellation confirmation (email, SMS, in-app), and is it required?
8. **Reversal:** Can a patient undo an online cancellation? If so, does the 24-hour rule still apply?
9. **Other statuses:** Can an appointment in a non-standard status (pending, rescheduled, waitlisted, no-show, partially paid) be cancelled online? If so, which rule applies?
10. **Multi-visit or multi-slot bookings:** Which start time counts when one booking has several visits or services?
11. **Phone line hours:** What happens if a patient needs to cancel by phone but the phone line is closed? Is there any alternative (email, form) for late cancellations, or is phone the only route?
12. **Audit and notifications:** Must the clinic be notified of an online cancellation immediately? Is a log of blocked attempts required?
13. **Scope of "online":** Does "online" include the mobile app, the web portal, and any third-party booking widget? Are they all subject to the same rule?
14. **Rescheduling:** Is rescheduling within the 24-hour window allowed online, or is it treated as a cancellation?
