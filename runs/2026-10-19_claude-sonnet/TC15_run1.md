# Test cases: Quiet hours for push notifications

Requirement: Users can set quiet hours (for example 22:00 to 07:00). During quiet hours no push notifications are sent; they are delivered when quiet hours end.

Assumptions are kept to a minimum. Where the requirement does not define behaviour, the test case states what is verified and refers to an open question (OQ) instead of inventing an expected result. Cases marked "pending OQ" cannot be finalised until the question is answered.

## Open questions

| ID | Question | Why it matters |
|----|----------|----------------|
| OQ1 | Which time zone applies: the device's current time zone, a profile time zone, or the server's? What happens when the user travels or changes time zone? | Determines when quiet hours start and end. |
| OQ2 | How are daylight saving time (DST) transitions handled (the 22:00-07:00 window is 8 or 10 hours long on those nights)? | Boundary behaviour on two nights a year. |
| OQ3 | Are the start and end times inclusive or exclusive? Is a notification at exactly 22:00:00 or exactly 07:00:00 suppressed? | Defines the boundary expectations. |
| OQ4 | What is the time precision (minutes, or seconds)? Is there a minimum or maximum window length? | Boundary of valid input. |
| OQ5 | What are start == end (e.g. 22:00 to 22:00) and 00:00 to 24:00 supposed to mean (no quiet hours, or 24 hours of quiet)? | Ambiguous input. |
| OQ6 | Does "delivered when quiet hours end" mean all held notifications at once at end time, or are they spread out or batched or summarised? Is there a rate limit or ordering guarantee (original order)? | Defines delivery expectations after quiet hours. |
| OQ7 | Is there a limit on how many notifications are held, or a maximum holding time? What happens to notifications that expire (time-sensitive, e.g. one-time codes, calls) or become obsolete before 07:00? | Possible loss or stale messages. |
| OQ8 | Are there exceptions (critical or security alerts, incoming calls, alarms, messages from favourite contacts)? | "No push notifications" is stated absolutely. |
| OQ9 | Does the rule apply to all notification types and channels (push only, or also in-app, email, SMS)? Are in-app badges and inbox entries still updated during quiet hours? | Scope. |
| OQ10 | Can users set several windows, different windows per weekday, or only one global window? Can quiet hours be turned off without deleting the setting? | Data model and UI. |
| OQ11 | Does the setting apply per user or per device? With several devices, is each device handled separately? | Multi-device behaviour. |
| OQ12 | When the user changes the setting while it is active (e.g. shortens the window or disables it at 23:00), are held notifications released immediately? Does a change apply immediately or from the next day? | Edit during an active window. |
| OQ13 | Is the notification suppressed on the server (not sent) or on the client (sent but silenced)? What is the expected behaviour if the device is offline at 07:00? | Defines what to observe in testing. |
| OQ14 | What is the default for new users (quiet hours off)? Is the example 22:00 to 07:00 a default or only an example? | Initial state. |
| OQ15 | Is the notification the user sees after quiet hours the original (with original timestamp) or a new one? Are duplicates collapsed? | Content expectations. |

## Positive cases

**TC-QH-001: Set quiet hours that cross midnight**
- Preconditions: User logged in; quiet hours not set; push enabled at OS level.
- Steps: 1. Open notification settings. 2. Set quiet hours start 22:00 and end 07:00. 3. Save. 4. Re-open settings.
- Expected: Setting is saved without error and shows 22:00 to 07:00 after re-opening and after app restart.

**TC-QH-002: Set quiet hours within a single day**
- Preconditions: As TC-QH-001.
- Steps: 1. Set start 13:00 and end 15:00. 2. Save.
- Expected: Saved and displayed as 13:00 to 15:00. Notifications at 12:00 and 16:00 are delivered normally; one at 14:00 is held.

**TC-QH-003: Notification during quiet hours is not pushed**
- Preconditions: Quiet hours 22:00-07:00 active; current time 23:30 (user time zone); a triggering event is available.
- Steps: 1. Trigger an event that normally causes a push notification. 2. Observe the device for 1 minute.
- Expected: No push notification is displayed, no sound or vibration.

**TC-QH-004: Notification outside quiet hours is pushed normally**
- Preconditions: Quiet hours 22:00-07:00; current time 12:00.
- Steps: 1. Trigger an event.
- Expected: Push notification arrives as normal.

**TC-QH-005: Held notification is delivered when quiet hours end**
- Preconditions: Quiet hours 22:00-07:00; one notification triggered at 23:30 and held.
- Steps: 1. Wait until 07:00 (or advance the test clock). 2. Observe the device.
- Expected: The held notification is delivered at or shortly after 07:00 (exact tolerance per OQ6). It is not delivered before 07:00.

**TC-QH-006: Several held notifications are all delivered**
- Preconditions: Quiet hours 22:00-07:00; 5 notifications triggered between 22:30 and 06:30.
- Steps: 1. Advance to 07:00. 2. Observe.
- Expected: All 5 notifications are delivered, none lost, none duplicated. Order and grouping: pending OQ6.

**TC-QH-007: Setting persists across sessions and devices**
- Preconditions: Quiet hours saved on device A.
- Steps: 1. Log out and log in again. 2. Check settings on device A and on device B (if applicable).
- Expected: Setting is unchanged on the same account. Per-device vs per-user behaviour: pending OQ11.

**TC-QH-008: Quiet hours repeat every day**
- Preconditions: Quiet hours 22:00-07:00 set.
- Steps: 1. Trigger notifications at 23:00 on three consecutive days.
- Expected: Suppressed each night and delivered each morning (assuming no weekday restriction, see OQ10).

**TC-QH-009: Change the quiet hours**
- Preconditions: Quiet hours 22:00-07:00.
- Steps: 1. Change to 23:00-06:00. 2. Save. 3. Trigger a notification at 22:30.
- Expected: Notification at 22:30 is delivered immediately; one at 23:30 is held until 06:00.

## Negative cases

**TC-QH-010: Invalid time format is rejected**
- Preconditions: Settings screen open (if the UI uses free text or API).
- Steps: 1. Enter start "25:00" or "ab:cd". 2. Save.
- Expected: Validation error, nothing saved, previous value retained.

**TC-QH-011: Empty start or end**
- Steps: 1. Leave end time empty, enter start 22:00. 2. Save.
- Expected: Validation error or defined behaviour; no partially saved window.

**TC-QH-012: Start equal to end**
- Steps: 1. Set start 22:00 and end 22:00. 2. Save.
- Expected: Pending OQ5. Test verifies that the system either rejects with a clear message or applies the documented meaning, and does not crash or behave inconsistently.

**TC-QH-013: API request with invalid payload**
- Preconditions: Authenticated API client.
- Steps: 1. Send a request to set quiet hours with null, a string, or out-of-range values (-1:00, 24:01).
- Expected: 4xx error with a descriptive message; stored setting unchanged.

**TC-QH-014: Unauthorised change of another user's quiet hours**
- Preconditions: Users A and B exist.
- Steps: 1. As user A, send a request to set quiet hours for user B.
- Expected: Rejected (403 or 404); user B's setting unchanged.

**TC-QH-015: Quiet hours disabled or not set**
- Preconditions: Quiet hours off (or never set).
- Steps: 1. Trigger a notification at 23:30.
- Expected: Notification is delivered immediately.

**TC-QH-016: Push disabled at OS level**
- Preconditions: Quiet hours set; push permission denied in the OS.
- Steps: 1. Trigger a notification during and after quiet hours.
- Expected: No notification is displayed at any time; the system does not crash, and no notification appears at 07:00 only because of the quiet-hours release.

**TC-QH-017: Notifications with exceptions (critical, security)**
- Preconditions: Quiet hours active.
- Steps: 1. Trigger a security alert or incoming call.
- Expected: Pending OQ8. Verify against the documented rule; the requirement as written says no push notifications.

**TC-QH-018: Device offline at the end of quiet hours**
- Preconditions: Quiet hours 22:00-07:00; notification held; device offline at 07:00.
- Steps: 1. Reconnect the device at 08:00.
- Expected: Held notification is delivered after reconnection (or expires, per OQ7/OQ13); no duplicates.

## Boundary cases

**TC-QH-019: Notification one minute before start**
- Preconditions: Quiet hours 22:00-07:00.
- Steps: 1. Trigger at 21:59.
- Expected: Delivered immediately.

**TC-QH-020: Notification exactly at start time**
- Steps: 1. Trigger at 22:00:00.
- Expected: Pending OQ3 (inclusive or exclusive). Result must be consistent with the documented rule and the same on repeated runs.

**TC-QH-021: Notification one minute after start**
- Steps: 1. Trigger at 22:01.
- Expected: Held.

**TC-QH-022: Notification one minute before end**
- Steps: 1. Trigger at 06:59.
- Expected: Held and delivered at 07:00.

**TC-QH-023: Notification exactly at end time**
- Steps: 1. Trigger at 07:00:00.
- Expected: Pending OQ3. Either delivered immediately or treated as part of the release batch, but delivered once only.

**TC-QH-024: Notification one minute after end**
- Steps: 1. Trigger at 07:01.
- Expected: Delivered immediately.

**TC-QH-025: Midnight crossing**
- Preconditions: Quiet hours 22:00-07:00.
- Steps: 1. Trigger notifications at 23:59:59, 00:00:00 and 00:00:01.
- Expected: All three are held; no gap in suppression at midnight and none is delivered early.

**TC-QH-026: Shortest and longest windows**
- Steps: 1. Set 22:00-22:01. 2. Set 22:01-22:00 (almost 24 hours). 3. Trigger notifications inside and outside the windows.
- Expected: Accepted or rejected per OQ4/OQ5; if accepted, boundaries behave as in TC-QH-019 to TC-QH-024.

**TC-QH-027: Whole-day values 00:00 to 00:00 and 00:00 to 23:59**
- Steps: 1. Set each window and save.
- Expected: Pending OQ5; behaviour must be defined and consistent.

**TC-QH-028: DST change inside quiet hours**
- Preconditions: Time zone with DST; quiet hours 22:00-07:00 on the night of the change.
- Steps: 1. Trigger notifications before and after the clock change. 2. Observe release time.
- Expected: Pending OQ2. Release occurs at 07:00 local wall-clock time, no early or double delivery.

**TC-QH-029: Time zone change while quiet hours are active**
- Preconditions: Quiet hours active; user travels (device time zone changes).
- Steps: 1. Change the device time zone. 2. Trigger a notification.
- Expected: Pending OQ1; behaviour follows the documented time zone source.

**TC-QH-030: Large number of held notifications**
- Preconditions: Quiet hours active.
- Steps: 1. Trigger 500 notifications. 2. Advance to 07:00.
- Expected: Pending OQ6/OQ7 (limits). At minimum: no crash, behaviour matches the documented limit, and the user is not flooded in a way that contradicts the specification.

**TC-QH-031: Setting changed while quiet hours are active**
- Preconditions: Quiet hours 22:00-07:00; notification held at 23:00.
- Steps: 1. At 23:30 turn off quiet hours (or change end to 23:45).
- Expected: Pending OQ12; the held notification is released or retained per the documented rule.

**TC-QH-032: Device clock manipulated**
- Steps: 1. Change device time manually to 12:00 at 23:30 real time. 2. Trigger a notification.
- Expected: Pending OQ13; if suppression is server-side the result follows server time, if client-side it follows the device. Verify no way to bypass or break the rule unintentionally.

## Coverage summary

- Positive: TC-QH-001 to 009
- Negative: TC-QH-010 to 018
- Boundary: TC-QH-019 to 032
- Cases that depend on an open question: 012, 017, 018, 020, 023, 026 to 032 and partly 006, 007.
