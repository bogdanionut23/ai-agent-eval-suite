# TC15 Run 2: Quiet Hours for Push Notifications

**Requirement:** Users can set quiet hours (for example 22:00 to 07:00). During quiet hours no push notifications are sent; they are delivered when quiet hours end.

Assumptions used for the test steps (not specified in the requirement, see Open Questions): quiet hours are stored per user, the window is interpreted in the user's local time, and the start time is inclusive and the end time is exclusive.

---

## Positive scenarios

### TC15-P01: Set an overnight quiet-hours window
- **Preconditions:** Logged-in user with push notifications enabled. Quiet hours currently not set.
- **Steps:**
  1. Open the notification settings.
  2. Enable quiet hours.
  3. Set start time to 22:00 and end time to 07:00.
  4. Save.
  5. Reopen the notification settings.
- **Expected result:** The window is saved and shown as 22:00 to 07:00. The value persists after reload and is applied to the user's account.

### TC15-P02: Push notification during quiet hours is not sent
- **Preconditions:** Quiet hours set to 22:00 to 07:00. Device time is 23:00 local. Push notifications enabled.
- **Steps:**
  1. Trigger an event that produces a push notification for the user (for example, a new message).
  2. Observe the push gateway / device.
- **Expected result:** No push notification is sent to the device at 23:00. The notification is recorded as held for later delivery.

### TC15-P03: Held notifications are delivered when quiet hours end
- **Preconditions:** Quiet hours 22:00 to 07:00. Two notifications were triggered at 23:00 and 02:00 and held.
- **Steps:**
  1. Advance the clock (or wait) to 07:00 local.
  2. Observe the device.
- **Expected result:** Both held notifications are delivered at or shortly after 07:00. Nothing is delivered before 07:00. Delivery order matches creation order (or the documented order, see Open Questions).

### TC15-P04: Notification outside quiet hours is sent immediately
- **Preconditions:** Quiet hours 22:00 to 07:00. Device time is 12:00 local.
- **Steps:**
  1. Trigger a push notification.
- **Expected result:** The push is sent immediately. Nothing is held.

### TC15-P05: Daytime window (start earlier than end on the same day)
- **Preconditions:** Quiet hours set to 13:00 to 14:00.
- **Steps:**
  1. Trigger a push at 13:30.
  2. Trigger a push at 12:59 and at 14:01.
- **Expected result:** The 13:30 push is held. The 12:59 and 14:01 pushes are sent immediately.

### TC15-P06: Disable quiet hours
- **Preconditions:** Quiet hours 22:00 to 07:00 enabled. Device time 23:00.
- **Steps:**
  1. Disable quiet hours in settings.
  2. Trigger a push notification.
- **Expected result:** The push is sent immediately and is not held.

### TC15-P07: Change the window and it takes effect
- **Preconditions:** Quiet hours 22:00 to 07:00. Device time 21:00.
- **Steps:**
  1. Change the window to 21:30 to 06:00.
  2. Trigger a push at 21:45.
- **Expected result:** The push at 21:45 is held. The new window is used without requiring a logout or app restart.

### TC15-P08: Quiet hours are per user
- **Preconditions:** User A has quiet hours 22:00 to 07:00. User B has no quiet hours. Both are at 23:00 local.
- **Steps:**
  1. Trigger a push for User A.
  2. Trigger a push for User B.
- **Expected result:** User A's push is held. User B's push is sent.

---

## Negative scenarios

### TC15-N01: Invalid time format is rejected
- **Preconditions:** Quiet hours settings open.
- **Steps:**
  1. Enter start time `25:00` and end time `07:00`.
  2. Save.
- **Expected result:** Save is rejected with a validation message. The previous setting is kept.

### TC15-N02: Non-time input is rejected
- **Preconditions:** Quiet hours settings open.
- **Steps:**
  1. Enter start time `abc`.
  2. Save.
- **Expected result:** Save is rejected with a validation message. No window is stored.

### TC15-N03: Missing start or end time is rejected
- **Preconditions:** Quiet hours settings open.
- **Steps:**
  1. Enable quiet hours. Leave the end time empty.
  2. Save.
- **Expected result:** Save is rejected and the user is told which field is missing.

### TC15-N04: Push notification is not lost when the window starts
- **Preconditions:** Quiet hours 22:00 to 07:00. A push is triggered at 21:59:59 and is in transit when the window starts.
- **Steps:**
  1. Trigger the push at 21:59:59. Observe delivery.
- **Expected result:** The push is sent (it was sent before the window began). It is not dropped.

### TC15-N05: Held notifications are not sent early when quiet hours are shortened
- **Preconditions:** Quiet hours 22:00 to 07:00. Two notifications held at 23:00.
- **Steps:**
  1. At 23:30, change the window to 22:00 to 23:15.
- **Expected result:** The held notifications are not sent at 23:30 while quiet hours still apply. (Expected behaviour on release is an open question. See Q6.)

### TC15-N06: Disabling quiet hours during the window
- **Preconditions:** Quiet hours 22:00 to 07:00. Two notifications held at 23:00. Current time 23:30.
- **Steps:**
  1. Disable quiet hours at 23:30.
  2. Observe the device.
- **Expected result:** Expected behaviour is not specified. See Q6. Test should not assume either outcome until the requirement is clarified.

### TC15-N07: Quiet hours do not suppress notifications when push is disabled by the user
- **Preconditions:** Push notifications disabled at OS or app level. Quiet hours 22:00 to 07:00.
- **Steps:**
  1. Trigger a push at 23:00.
- **Expected result:** No push is sent (already suppressed by the permission). Held queue is not used to deliver it later against the user's choice. Expected behaviour to be confirmed. See Q9.

---

## Boundary scenarios

### TC15-B01: Start boundary (inclusive)
- **Preconditions:** Quiet hours 22:00 to 07:00.
- **Steps:**
  1. Trigger a push at 21:59:59.
  2. Trigger a push at 22:00:00.
- **Expected result:** The 21:59:59 push is sent. The 22:00:00 push is held.

### TC15-B02: End boundary (exclusive)
- **Preconditions:** Quiet hours 22:00 to 07:00. Held notifications exist.
- **Steps:**
  1. Trigger a push at 06:59:59.
  2. Observe delivery at 07:00:00.
- **Expected result:** Held notifications are not delivered at 06:59:59. Delivery starts at 07:00:00. A push triggered at 07:00:00 is sent immediately.

### TC15-B03: Midnight crossing
- **Preconditions:** Quiet hours 22:00 to 07:00.
- **Steps:**
  1. Trigger pushes at 23:59:59 and 00:00:00.
- **Expected result:** Both are held. The window does not break at midnight.

### TC15-B04: Minimal window length (one minute)
- **Preconditions:** Quiet hours 06:59 to 07:00.
- **Steps:**
  1. Trigger a push at 06:58:59, 06:59:00 and 07:00:00.
- **Expected result:** The pushes at 06:59:00 is held and the others are sent. Confirms the one-minute window is handled.

### TC15-B05: Start equals end
- **Preconditions:** Quiet hours settings open.
- **Steps:**
  1. Set start and end to 08:00.
  2. Save.
- **Expected result:** Behaviour is not specified. Test records the actual behaviour and checks it against the answer to Q4. Do not assume it is valid.

### TC15-B06: Minimum and maximum time values
- **Preconditions:** Quiet hours settings open.
- **Steps:**
  1. Set start to 00:00 and end to 23:59. Save.
  2. Trigger a push at 12:00 and at 00:00.
  3. Set start to 23:59 and end to 00:00. Save.
- **Expected result:** Valid values are accepted. Behaviour for the 00:00 to 23:59 window is applied as described in the requirement. Expected result for the second window is covered by Q4.

### TC15-B07: Daylight saving time start (spring forward)
- **Preconditions:** Quiet hours 02:00 to 03:00 on a DST change day. User in a timezone with DST.
- **Steps:**
  1. Trigger a push during the non-existent local hour.
  2. Observe whether the window is applied and when held items are released.
- **Expected result:** No crash or duplicate delivery. Release occurs once. Exact behaviour is an open question (Q3).

### TC15-B08: Daylight saving time end (fall back)
- **Preconditions:** Quiet hours 22:00 to 07:00 on a DST end day.
- **Steps:**
  1. Trigger a push at 01:30 (repeated hour). Observe.
  2. Confirm release at 07:00 local.
- **Expected result:** Window applied consistently. Release at 07:00 local, not 06:00 or 08:00. Exact behaviour is an open question (Q3).

### TC15-B09: Time zone change while quiet hours are active
- **Preconditions:** User in timezone A with quiet hours 22:00 to 07:00. Device moves to timezone B.
- **Steps:**
  1. Change device timezone so local time is inside the window in B.
  2. Trigger a push.
- **Expected result:** Behaviour not specified. See Q2.

### TC15-B10: Large held queue
- **Preconditions:** Quiet hours 22:00 to 07:00. User receives 500 pushes during the window.
- **Steps:**
  1. Trigger 500 pushes between 22:00 and 06:59.
  2. Observe delivery at 07:00.
- **Expected result:** All held pushes are delivered, or the documented cap and overflow rule applies (Q7). No crash, no timeout of the release job.

### TC15-B11: Held notification older than the retention limit
- **Preconditions:** Quiet hours 22:00 to 07:00. Retention limit for held notifications is not defined (Q7).
- **Steps:**
  1. Hold a push at 23:00 and keep the window active for a long period (for example, the user is offline for 48 hours).
- **Expected result:** Behaviour not specified. See Q7.

### TC15-B12: Concurrent trigger at the end boundary
- **Preconditions:** Quiet hours 22:00 to 07:00. Held queue has items. Many pushes are triggered at 06:59:59.9.
- **Steps:**
  1. Trigger pushes around 07:00:00 from several services at once.
- **Expected result:** Each notification is sent exactly once. None are lost or duplicated during the transition.

---

## Open questions

1. **Timezone basis:** Is quiet hours evaluated in the user's profile timezone, or the device's current local time? What happens when the user travels?
2. **Timezone change while active:** Which timezone applies when the device or profile timezone changes during a window?
3. **DST:** How are the non-existent hour (spring forward) and the repeated hour (fall back) handled? Does release happen at the first 07:00 wall-clock time?
4. **Start equals end:** Is `08:00 to 08:00` rejected, treated as disabled, or treated as a 24-hour window?
5. **Boundary inclusivity:** Is the start time inclusive (22:00 held) and the end time exclusive (07:00 sent)? The requirement does not say.
6. **Release behaviour:** When quiet hours are shortened or disabled while notifications are held, are they sent immediately, or kept until the original end time? The requirement only says "when quiet hours end".
7. **Held queue limits:** Is there a maximum queue size, a retention limit, or an expiry for held notifications? What happens to overflow?
8. **Delivery format at release:** Are held notifications delivered as individual pushes, as a single digest, or ordered in some specific way? Are they grouped or collapsed?
9. **Permissions and suppression:** If the user disables push at OS or app level, are held notifications discarded? Should they be sent when push is re-enabled?
10. **Exceptions:** Do critical or security notifications (for example, login alerts, calls, emergencies) bypass quiet hours? Are non-push channels (in-app, email, SMS) affected?
11. **Scope:** Is quiet hours per user account or per device? If the user has several devices, is the window applied to all of them?
12. **Multiple windows:** Can the user set more than one window, or different windows per weekday?
13. **Default state:** Is quiet hours on or off by default? Is there a default window?
14. **Input granularity and format:** Are times set to the minute only? Is a 12-hour clock format supported?
15. **Offline device at release:** If the device is offline at 07:00, is delivery attempted when it reconnects, or is the notification dropped?
16. **Feedback to the user:** Does the UI show that notifications are being held (for example a count or badge)?
17. **Relation to OS focus modes:** Does this feature interact with iOS Focus or Android Do Not Disturb, and which setting wins?
