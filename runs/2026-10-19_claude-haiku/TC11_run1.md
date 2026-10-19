# TC11 - Test cases: online appointment cancellation (24-hour rule)

**Requirement:** A patient can cancel an appointment online free of charge up to 24 hours before its start time. Later cancellations must be made by phone.

## Assumptions used for the test cases

These assumptions are needed to write concrete steps. Each is also listed under Open questions.

- "Online" means the patient-facing web or app channel, authenticated as the patient who owns the appointment.
- "Free of charge" means no cancellation fee is charged. The fee rules for phone cancellations are not defined in the requirement.
- The 24-hour boundary is compared against the appointment start time in the clinic's local time zone.
- Test data uses fixed timestamps so the boundary can be set exactly.

## Test cases

### Positive scenarios

**TC11-P01 - Cancel online well ahead of start time, no fee**
- Preconditions: Patient "P1" is logged in. P1 has a confirmed future appointment A1 with start time T. The current time is T minus 72 hours. A1 has no cancellation fee on record.
- Steps:
  1. Open "My appointments".
  2. Select A1 and choose "Cancel appointment".
  3. Confirm the cancellation.
- Expected result: A1 status changes to cancelled. The confirmation screen shows that no fee applies. No fee is charged or recorded. The slot is released for other bookings. A cancellation confirmation is sent to P1.

**TC11-P02 - Cancel online at exactly the 24-hour mark (boundary, inclusive)**
- Preconditions: P1 is logged in. Appointment A2 starts at T. The current time is exactly T minus 24:00:00.
- Steps:
  1. Open A2 and choose "Cancel appointment".
  2. Confirm the cancellation.
- Expected result: The cancellation succeeds online. No fee is charged. Status is cancelled. (Whether this is the correct result depends on Open question Q1.)

**TC11-P03 - Cancel online one minute before the 24-hour mark**
- Preconditions: P1 is logged in. Appointment A3 starts at T. The current time is T minus 24:01:00.
- Steps:
  1. Open A3 and choose "Cancel appointment".
  2. Confirm the cancellation.
- Expected result: The cancellation succeeds online with no fee.

**TC11-P04 - Online cancel option is visible for an eligible appointment**
- Preconditions: P1 is logged in. Appointment A4 starts at T. The current time is T minus 48 hours.
- Steps:
  1. Open A4 in "My appointments".
- Expected result: The "Cancel appointment" action is shown and enabled. No phone-only message is shown.

**TC11-P05 - Cancel one of several future appointments**
- Preconditions: P1 has appointments A5 (start T1, more than 24 hours away) and A6 (start T2, less than 24 hours away).
- Steps:
  1. Cancel A5 online.
- Expected result: A5 is cancelled with no fee. A6 is unchanged and still confirmed.

### Negative scenarios

**TC11-N01 - Online cancel is refused inside the 24-hour window**
- Preconditions: P1 is logged in. Appointment A7 starts at T. The current time is T minus 23:59:00.
- Steps:
  1. Open A7 and choose "Cancel appointment".
  2. Confirm the cancellation.
- Expected result: The cancellation is not made. A message tells P1 that the appointment cannot be cancelled online within 24 hours of the start time, and that they must cancel by phone. The phone number is shown. A7 stays confirmed. No fee is charged.

**TC11-N02 - Cancel option is hidden or disabled inside the window**
- Preconditions: P1 is logged in. Appointment A8 starts at T. The current time is T minus 2 hours.
- Steps:
  1. Open A8 in "My appointments".
- Expected result: No enabled "Cancel appointment" action is offered. The page shows the phone cancellation instruction instead.

**TC11-N03 - Cancel a past appointment online**
- Preconditions: P1 is logged in. Appointment A9 started at T minus 1 day and is marked as attended or not attended.
- Steps:
  1. Open A9 and try to cancel it online.
- Expected result: Cancellation is refused. The page does not offer an online cancel action for a past appointment. No status change.

**TC11-N04 - Cancel an appointment that has already been cancelled**
- Preconditions: P1 is logged in. Appointment A10 is already cancelled. The current time is T minus 72 hours.
- Steps:
  1. Open A10 and try to cancel it again.
- Expected result: No second cancellation is recorded. The system shows that the appointment is already cancelled. No fee is applied.

**TC11-N05 - Another patient's appointment cannot be cancelled**
- Preconditions: P1 is logged in. Appointment A11 belongs to patient P2. The current time is T minus 72 hours.
- Steps:
  1. Try to cancel A11 by its ID, for example by editing the URL or request.
- Expected result: The request is refused with an access error. A11 is unchanged. The error does not reveal details of P2's appointment.

**TC11-N06 - Unauthenticated cancel request**
- Preconditions: No user is logged in. Appointment A12 belongs to a patient and is more than 24 hours away.
- Steps:
  1. Send a cancel request for A12 without a valid session.
- Expected result: The request is rejected and the user is asked to log in. A12 is unchanged.

**TC11-N07 - Double submit of the cancel action**
- Preconditions: P1 is logged in. Appointment A13 is more than 24 hours away.
- Steps:
  1. Choose "Cancel appointment" and confirm.
  2. Immediately confirm again, or resubmit the same request.
- Expected result: A13 is cancelled once. Only one cancellation record and one confirmation message exist. No duplicate fee or error state.

**TC11-N08 - Cancel is interrupted by a network failure**
- Preconditions: P1 is logged in. Appointment A14 is more than 24 hours away. The network drops during the confirm request.
- Steps:
  1. Confirm the cancellation.
  2. Lose the connection before a response is received.
  3. Reload "My appointments" after reconnecting.
- Expected result: The system shows either a fully cancelled appointment or a fully confirmed one. It never shows a partial state. If it is shown as cancelled, no fee is recorded. If it is shown as confirmed, the patient can retry.

**TC11-N09 - Time check uses server time, not the device clock**
- Preconditions: P1 is logged in on a device whose clock is set 2 hours ahead. Appointment A15 starts in 10 hours, server time.
- Steps:
  1. Open A15 and cancel it online.
- Expected result: The cancellation is evaluated against server time. It succeeds because the appointment is more than 24 hours away. (Device clock must not change the result.)

### Boundary scenarios

**TC11-B01 - Exactly 24 hours and 1 second before start (just outside the window)**
- Preconditions: P1 is logged in. Appointment A16 starts at T. The current time is T minus 24:00:01.
- Steps:
  1. Cancel A16 online.
- Expected result: The cancellation succeeds online with no fee.

**TC11-B02 - Exactly 24 hours before start (inclusive boundary)**
- Preconditions: Same as TC11-P02.
- Steps:
  1. Cancel online at T minus 24:00:00.
- Expected result: Depends on Q1. If "up to 24 hours" includes the 24-hour mark, the cancellation succeeds with no fee.

**TC11-B03 - One second inside the window**
- Preconditions: P1 is logged in. Appointment A17 starts at T. The current time is T minus 23:59:59.
- Steps:
  1. Cancel A17 online.
- Expected result: The cancellation is refused. The phone instruction is shown. A17 stays confirmed.

**TC11-B04 - Appointment booked less than 24 hours before its start**
- Preconditions: P1 books A18 at time B, where B is 3 hours before the start time T.
- Steps:
  1. Immediately try to cancel A18 online.
- Expected result: The cancellation is refused, because the appointment is inside the 24-hour window. The phone instruction is shown.

**TC11-B05 - Start time is exactly now**
- Preconditions: P1 is logged in. Appointment A19 starts at the current time T.
- Steps:
  1. Try to cancel A19 online.
- Expected result: The cancellation is refused. The phone instruction is shown.

**TC11-B06 - Daylight saving time change between cancel time and start time**
- Preconditions: P1 is logged in. The clinic's time zone changes by one hour during the 24-hour window before appointment A20. The start time is 25 hours of elapsed time away, but the wall-clock difference is 24 hours (or the reverse).
- Steps:
  1. Cancel A20 online at the boundary.
- Expected result: The window is measured in elapsed real time (or in the rule the business defines, see Q4). The result is consistent with that rule and does not differ by an hour because of the clock change.

**TC11-B07 - Time zone of the patient differs from the clinic**
- Preconditions: P1 is in a different time zone from the clinic. A21 starts at T clinic-local. The current time is T minus 24:00:30 in the clinic's zone but T minus 23:59:30 in the patient's zone.
- Steps:
  1. Cancel A21 online.
- Expected result: The rule is applied in the zone defined by the business (see Q4). The result is the same no matter which zone the patient's device reports.

**TC11-B08 - Multiple appointments in one booking at different times**
- Preconditions: P1 has one booking with two appointments: A22 at T1 (more than 24 hours away) and A23 at T2 (less than 24 hours away).
- Steps:
  1. Try to cancel the whole booking online.
- Expected result: The system must not cancel A23 online. Behaviour for A22 depends on Q6. Either A22 is cancelled alone, or the whole booking is blocked with a clear message. The result is consistent and the patient sees what happened.

**TC11-B09 - Appointment start time is moved by the clinic**
- Preconditions: P1 has A24 at T, more than 24 hours away. The clinic moves the start to T plus 3 hours, so A24 is still more than 24 hours away. A second change moves the start to 10 hours away.
- Steps:
  1. After the first move, cancel A24 online.
  2. After the second move, try to cancel A24 online.
- Expected result: Step 1 succeeds. Step 2 is refused, because the window is measured from the current start time, not the original booking time (see Q5).

## Open questions

These are not answered by the requirement. No behaviour has been invented for them.

- **Q1 - Boundary inclusion.** Does "up to 24 hours before" include exactly 24 hours? (TC11-P02, TC11-B02)
- **Q2 - What happens to a late cancel attempt online?** Is the online action blocked and only a phone instruction shown, or is the cancellation accepted with a fee? If a fee applies, what is the amount and who is charged?
- **Q3 - What is the phone cancellation rule?** Does a phone cancellation inside the window carry a fee? Are there any exceptions, for example illness or emergency?
- **Q4 - Time zone and clock.** Which clock is authoritative: server time, clinic time, or patient time? Is the window measured in elapsed time or in wall-clock time across daylight saving changes?
- **Q5 - Rescheduled appointments.** When the clinic moves an appointment, is the window measured from the new start time or from the original booking?
- **Q6 - Multi-appointment bookings.** Can one appointment in a booking be cancelled online while another is inside the window? Or is the whole booking blocked?
- **Q7 - Booking inside the window.** Can a patient book an appointment less than 24 hours before it starts? If so, can it ever be cancelled online? (TC11-B04)
- **Q8 - Appointment types.** Do all appointment types follow this rule, including telehealth, procedures, or follow-ups? Are any types excluded?
- **Q9 - Past or in-progress appointments.** What is the online behaviour for an appointment that has already started or finished? (TC11-N03)
- **Q10 - Confirmation and notification.** What must the patient see and receive after an online cancellation? Is an email or SMS sent, and does the phone instruction appear in it?
- **Q11 - Recording the cancellation.** What is stored: the cancellation time, the channel (online or phone), the actor, and the reason? Is a reason required from the patient?
- **Q12 - Reinstating a cancelled appointment.** Can a patient undo an online cancellation? Is the slot guaranteed to be free again?
- **Q13 - Clinic-initiated cancellations.** Does this rule apply only to patient cancellations? Are clinic cancellations out of scope?
- **Q14 - Phone channel.** Is the phone requirement enforced by the system (for example, the online action is disabled), or is it only information for the patient? How does the system know that the patient has cancelled by phone?
- **Q15 - Error message.** What exact text and phone number are shown when an online cancel is refused? Are there localisation requirements?
