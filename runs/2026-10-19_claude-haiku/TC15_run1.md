# TC15 - Quiet hours for push notifications: test cases

## Requirement

Users can set quiet hours (for example 22:00 to 07:00). During quiet hours no push notifications are sent; they are delivered when quiet hours end.

## Notes on the test cases

- Test cases that depend on an open question are marked "Depends on OQ-n". Their expected result must be filled in once that question is answered. No behaviour has been assumed for them.
- Unless a test says otherwise, the precondition is: user is signed in, push notifications are enabled at OS and app level, and the test device is on the time zone stated in OQ-1.
- Test time is set with a controlled clock or a test-time override where possible, so boundary times can be hit exactly.

---

## Positive scenarios

### TC15-01 - Set quiet hours to 22:00-07:00

- Preconditions: User is signed in. Quiet hours are not yet configured.
- Steps:
  1. Open Settings > Notifications > Quiet hours.
  2. Set the start time to 22:00.
  3. Set the end time to 07:00.
  4. Tap Save.
  5. Leave the screen and reopen it.
- Expected result: The save succeeds with a confirmation. After reopening, the window shows 22:00 start and 07:00 end.

### TC15-02 - Push notification during quiet hours is not sent

- Preconditions: Quiet hours set to 22:00-07:00 (TC15-01). Clock is at 23:30 local time. Device is online.
- Steps:
  1. Trigger an event that normally produces a push notification (for example, a new message).
  2. Observe the device for 2 minutes.
- Expected result: No push notification is shown or sounded. The push provider is not called for this recipient (verify in server logs or the push gateway mock).

### TC15-03 - Suppressed notifications are delivered when quiet hours end

- Preconditions: Quiet hours 22:00-07:00. Three push notifications were triggered during quiet hours (for example at 23:00, 01:30, 05:45).
- Steps:
  1. Advance the clock to 07:00 local time.
  2. Observe the device.
- Expected result: All three held notifications are delivered at or shortly after 07:00. None is dropped. Each one appears on the device (content and count match what was triggered).
- Depends on: OQ-3 (batch vs individual delivery, timing tolerance).

### TC15-04 - Notification outside quiet hours is delivered immediately

- Preconditions: Quiet hours 22:00-07:00. Clock at 12:00 local time.
- Steps:
  1. Trigger a push notification.
- Expected result: The notification is delivered immediately and is not held.

### TC15-05 - Quiet hours disabled: notification at a quiet-hour time is delivered

- Preconditions: Quiet hours were configured, then turned off (toggle set to Off and saved). Clock at 23:30.
- Steps:
  1. Trigger a push notification.
- Expected result: The notification is delivered immediately.

### TC15-06 - Edit an existing quiet-hours window

- Preconditions: Quiet hours set to 22:00-07:00.
- Steps:
  1. Change the window to 23:00-06:00 and save.
  2. Clock at 22:30, trigger a push notification.
  3. Clock at 23:30, trigger a push notification.
- Expected result: The 22:30 notification is delivered (outside the new window). The 23:30 notification is held. The change applies to the next notification without an app restart.

### TC15-07 - Quiet hours survive app restart and sign-out/sign-in

- Preconditions: Quiet hours set to 22:00-07:00.
- Steps:
  1. Force-close the app and reopen it. Check the setting.
  2. Sign out, sign back in. Check the setting.
- Expected result: The 22:00-07:00 setting is still in place after each step. Notifications during the window are still held.
- Depends on: OQ-4 (if the setting is per device, sign-out behaviour may differ).

### TC15-08 - Quiet hours set on one device apply to a second device

- Preconditions: User is signed in on device A and device B, both with push enabled.
- Steps:
  1. Set quiet hours to 22:00-07:00 on device A.
  2. At 23:00, trigger a push notification.
- Expected result: Depends on OQ-4. Record whether device B receives the notification.

---

## Negative scenarios

### TC15-09 - Invalid time value is rejected

- Preconditions: Quiet hours screen open.
- Steps:
  1. Enter 25:00 as the start time. Tap Save.
  2. Enter 7:60 as the end time. Tap Save.
  3. Enter "abc" in the time field (if the field accepts text).
- Expected result: Each save is blocked with a validation message. The previously saved setting is unchanged.

### TC15-10 - Missing start or end time blocks save

- Preconditions: Quiet hours screen open, no window configured.
- Steps:
  1. Set only a start time of 22:00. Leave end empty. Tap Save.
  2. Clear the start time, set only an end time of 07:00. Tap Save.
- Expected result: Save is blocked in both cases with a message that names the missing field.

### TC15-11 - Notification is not held when quiet hours are not active

- Preconditions: Quiet hours 22:00-07:00. Clock at 21:59 (one minute before start).
- Steps:
  1. Trigger a push notification.
- Expected result: The notification is delivered immediately. It must not be held as a quiet-hours notification.

### TC15-12 - Push provider failure during quiet hours does not send early

- Preconditions: Quiet hours active. Push provider is mocked to fail on any send.
- Steps:
  1. Trigger a push notification during quiet hours. Confirm it is held and not sent.
  2. Advance the clock to 07:00.
- Expected result: No send attempt is made during quiet hours (no failed attempts are logged for the held notification). At 07:00 the held notification is sent once the provider is healthy, and the send is not duplicated.
- Depends on: OQ-3 (retry policy when the provider is unavailable at quiet end).

### TC15-13 - Held notification is not delivered after the user has cleared it

- Preconditions: Quiet hours active. A notification was held at 23:00.
- Steps:
  1. Delete or mark the underlying item as read (for example, the message is opened in the app) before 07:00.
  2. Advance the clock to 07:00.
- Expected result: Depends on OQ-3 and OQ-6. Record whether the stale notification is dropped or sent.

### TC15-14 - Unauthorised user cannot change another user's quiet hours

- Preconditions: User A and User B are separate accounts.
- Steps:
  1. As User A, send a request to set quiet hours for User B's account (via API or tampered client).
- Expected result: The request is rejected with an authorisation error. User B's setting is unchanged.

---

## Boundary scenarios

### TC15-15 - Start boundary: 21:59 vs 22:00

- Preconditions: Quiet hours 22:00-07:00.
- Steps:
  1. Clock at 21:59. Trigger a push notification.
  2. Clock at 22:00. Trigger a push notification.
- Expected result: The 21:59 notification is delivered. The 22:00 notification is held.
- Depends on: OQ-2 (confirm that 22:00 is inclusive as the start of the window).

### TC15-16 - End boundary: 06:59 vs 07:00

- Preconditions: Quiet hours 22:00-07:00. Notifications held from earlier in the night.
- Steps:
  1. Clock at 06:59. Trigger a push notification.
  2. Clock at 07:00. Trigger a push notification and observe the held queue.
- Expected result: The 06:59 notification is held. At 07:00, the held queue is released and the 07:00 notification is delivered.
- Depends on: OQ-2 (whether 07:00 is the first minute outside the window).

### TC15-17 - Midnight crossing

- Preconditions: Quiet hours 22:00-07:00.
- Steps:
  1. Clock at 23:59. Trigger a notification. Confirm it is held.
  2. Clock at 00:00 (next day). Trigger a notification. Confirm it is still held.
- Expected result: Quiet hours do not reset at midnight. Both notifications are held.

### TC15-18 - Minute resolution

- Preconditions: Quiet hours set to 22:15-07:45.
- Steps:
  1. Clock at 22:14, trigger a notification.
  2. Clock at 22:15, trigger a notification.
- Expected result: 22:14 is delivered. 22:15 is held (subject to OQ-2).
- Depends on: OQ-7 (minute granularity).

### TC15-19 - Window with full-day extent (00:00 to 23:59)

- Preconditions: Quiet hours set to 00:00-23:59.
- Steps:
  1. Clock at 12:00. Trigger a notification.
- Expected result: Depends on OQ-8. If accepted, the notification is held. If rejected at save, save is blocked with a message.

### TC15-20 - Equal start and end time (22:00-22:00)

- Preconditions: Quiet hours screen open.
- Steps:
  1. Set start 22:00 and end 22:00. Tap Save.
- Expected result: Depends on OQ-8. Either the save is rejected with a message, or the window is interpreted as defined there. No undefined state is saved.

### TC15-21 - Same-day window (for example 13:00-14:00)

- Preconditions: Quiet hours screen open.
- Steps:
  1. Set start 13:00 and end 14:00. Save.
  2. Clock at 13:30, trigger a notification.
  3. Clock at 14:30, trigger a notification.
- Expected result: Depends on OQ-8. If supported: 13:30 is held, 14:30 is delivered at once. If not supported: save is rejected.

### TC15-22 - Queue released at the end of quiet hours with many notifications

- Preconditions: Quiet hours 22:00-07:00. 500 push notifications triggered during the window.
- Steps:
  1. Advance the clock to 07:00.
  2. Measure delivery of all 500 notifications.
- Expected result: All 500 are delivered. None are duplicated or lost. Delivery completes within the time agreed in OQ-3.
- Depends on: OQ-3.

### TC15-23 - Device offline at end of quiet hours

- Preconditions: Quiet hours 22:00-07:00. One notification held. Device is in airplane mode at 06:50.
- Steps:
  1. Restore connectivity at 07:10.
- Expected result: Depends on OQ-3. Record whether the held notification is delivered on reconnect or dropped.

### TC15-24 - Time zone change during quiet hours

- Preconditions: Quiet hours 22:00-07:00 in local time. Device set to UTC+1. At 23:00 local, one notification is held.
- Steps:
  1. Change device time zone to UTC-5 (local time now 17:00).
  2. Trigger a notification.
- Expected result: Depends on OQ-1. The result must be recorded for the chosen time zone model.

### TC15-25 - Daylight saving transition inside the window

- Preconditions: Quiet hours 22:00-07:00. Test on the night of a DST change (clock moves forward from 02:00 to 03:00, or back from 03:00 to 02:00).
- Steps:
  1. Set the clock to the night of the change. Trigger notifications at 01:30, 02:30 and 03:30 local time.
  2. Observe holding and release at 07:00.
- Expected result: Depends on OQ-1. The window must be applied by the rule chosen there, and no notification is lost or released early because of the time jump.

---

## Open questions

These points are not defined in the requirement. Test results for the linked cases cannot be judged until they are answered.

- OQ-1 - Time zone. Is quiet hours interpreted in the device's local time, the account's profile time zone, or UTC? What happens when a user travels or changes time zone, and how are daylight saving changes handled? (TC15-24, TC15-25, and all timing tests)
- OQ-2 - Boundary inclusivity. Is the start time inclusive? Is the end time the first minute outside the window? (TC15-15, TC15-16, TC15-18)
- OQ-3 - Held notification delivery. Are held notifications sent one by one, as a batch, or staggered? Is there a time tolerance after 07:00? Is there a cap or expiry on held notifications? What happens if the push provider or the device is unavailable at quiet end? (TC15-03, TC15-12, TC15-22, TC15-23)
- OQ-4 - Scope of the setting. Is quiet hours per account, per device, or per notification category? Does it survive sign-out? (TC15-07, TC15-08)
- OQ-5 - Which notifications are covered. Does "push notification" include silent or data-only pushes, badge updates, and in-app notifications? Are any categories exempt (for example security, emergency, or call alerts)? Is the default for a new user on or off?
- OQ-6 - Stale notifications. Should a held notification be dropped if the underlying event has been read or deleted before quiet hours end? (TC15-13)
- OQ-7 - Input granularity and format. Is the time picker limited to whole minutes or fixed steps (for example 15 minutes)? Is 12-hour input supported?
- OQ-8 - Validation rules. Is a same-day window (for example 13:00-14:00) allowed? Is start equal to end allowed? Is a 00:00-23:59 window allowed? Is there a minimum or maximum window length? (TC15-19, TC15-20, TC15-21)
- OQ-9 - Multiple devices. When the user is signed in on several devices, does each device hold or deliver independently? (TC15-08)
- OQ-10 - Scope limit. Does the requirement cover only push notifications, or also email, SMS and in-app alerts? This document assumes push only.
- OQ-11 - Override. Can a user temporarily pause or override quiet hours (for example "until tomorrow")? This is not in the requirement and no tests are written for it.
