# Test Cases: Quiet Hours for Push Notifications

Requirement: Users can set quiet hours (for example 22:00 to 07:00). During quiet hours no push notifications are sent; they are delivered when quiet hours end.

Assumptions used only where the requirement is explicit: a quiet-hours window has a start and an end time; it can cross midnight (the 22:00 to 07:00 example); notifications raised during the window are held and delivered after it ends. Everything else is listed under Open Questions, and the tests that depend on it are marked **[Blocked: Qn]**. Expected results for those tests are provisional until the question is answered.

## Positive

**TC-01: Set quiet hours that cross midnight**
- Preconditions: User logged in, push enabled, no quiet hours set.
- Steps: 1. Open notification settings. 2. Set quiet hours 22:00 to 07:00. 3. Save. 4. Reopen settings.
- Expected: Settings save without error; 22:00 and 07:00 show on reopening.

**TC-02: Set quiet hours within a single day**
- Preconditions: As TC-01.
- Steps: 1. Set 13:00 to 15:00. 2. Save. 3. Reopen settings.
- Expected: Saved and displayed correctly.

**TC-03: Notification during quiet hours is not sent**
- Preconditions: Quiet hours 22:00 to 07:00; device time 23:00; device online with push permission.
- Steps: 1. Trigger a push-eligible event for the user (e.g. new message).
- Expected: No push arrives on the device at 23:00. The event is not lost (see TC-05).

**TC-04: Notification outside quiet hours is sent normally**
- Preconditions: Same settings; time 12:00.
- Steps: 1. Trigger a push-eligible event.
- Expected: Push is delivered immediately.

**TC-05: Held notification is delivered when quiet hours end**
- Preconditions: Quiet hours 22:00 to 07:00; one notification raised at 23:00 and held.
- Steps: 1. Advance time to 07:00 (or wait). 2. Observe the device.
- Expected: The held notification is delivered at or just after 07:00, with its original content.

**TC-06: Multiple held notifications are all delivered**
- Preconditions: Quiet hours active.
- Steps: 1. Trigger 5 distinct events during quiet hours. 2. Let quiet hours end.
- Expected: All 5 are delivered after 07:00 with no loss and no duplicates. Order and grouping: **[Blocked: Q4]**.

**TC-07: Edit quiet hours**
- Preconditions: Quiet hours 22:00 to 07:00 saved.
- Steps: 1. Change to 23:00 to 06:00. 2. Save. 3. Trigger an event at 22:30.
- Expected: The new window applies; the 22:30 notification is sent normally.

**TC-08: Remove/disable quiet hours**
- Preconditions: Quiet hours set.
- Steps: 1. Disable or clear quiet hours. 2. Trigger an event at 23:00.
- Expected: Push is delivered immediately. **[Blocked: Q9]** on whether a disable toggle exists.

**TC-09: Settings persist across sessions and devices**
- Preconditions: Quiet hours saved.
- Steps: 1. Log out and log in. 2. Log in on a second device.
- Expected: Same quiet hours shown on both.

## Boundary

**TC-10: Notification exactly at start time**
- Preconditions: Quiet hours 22:00 to 07:00.
- Steps: 1. Trigger an event at 22:00:00.
- Expected: **[Blocked: Q2]** Whether the start is inclusive. Provisional: suppressed.

**TC-11: Notification one second before start**
- Steps: Trigger at 21:59:59.
- Expected: Delivered immediately.

**TC-12: Notification exactly at end time**
- Steps: Trigger at 07:00:00.
- Expected: **[Blocked: Q2]** Provisional: delivered immediately (quiet hours have ended).

**TC-13: Notification one second before end**
- Steps: Trigger at 06:59:59.
- Expected: Held and delivered at the end of quiet hours.

**TC-14: Start equals end (e.g. 22:00 to 22:00)**
- Steps: Try to save.
- Expected: **[Blocked: Q3]** Either rejected with a clear message, or treated as 0 hours or 24 hours. Must not be silently accepted with undefined behaviour.

**TC-15: Longest window (e.g. 00:00 to 23:59)**
- Steps: Save and trigger an event at 12:00 and at 23:59:30.
- Expected: Saved; notifications suppressed throughout, delivered at the end. **[Blocked: Q3]** on the maximum allowed length.

**TC-16: Shortest window (e.g. 22:00 to 22:01)**
- Steps: Save; trigger at 22:00:30.
- Expected: **[Blocked: Q3]** Accepted or rejected per the minimum length; if accepted, the notification is held for about 30 seconds.

**TC-17: Midnight crossing at 00:00**
- Preconditions: 22:00 to 07:00.
- Steps: Trigger events at 23:59:59 and at 00:00:01.
- Expected: Both are suppressed; there is no gap at midnight.

**TC-18: Window ending exactly at midnight (e.g. 20:00 to 00:00)**
- Steps: Save; trigger at 23:59:59 and 00:00:00.
- Expected: First held, second delivered (subject to Q2).

**TC-19: Time zone and DST**
- Preconditions: User in a time zone with DST; quiet hours 22:00 to 07:00.
- Steps: 1. On the spring-forward night, trigger events. 2. On the fall-back night, trigger events. 3. Change the device time zone, then trigger an event.
- Expected: **[Blocked: Q5]** The window follows the defined time zone; delivery occurs at the intended local 07:00; no notification is lost or duplicated at DST changes.

## Negative

**TC-20: Invalid time value**
- Steps: Attempt to save a time like 25:00 or 12:60 (via UI or API).
- Expected: Rejected with a validation error; previous settings unchanged.

**TC-21: Missing start or end**
- Steps: Set only the start time and save.
- Expected: Rejected with a validation message; nothing partially saved.

**TC-22: Non-time input**
- Steps: Send a non-time string (e.g. "abc", empty, null) via the API.
- Expected: 400-level error, no change to settings.

**TC-23: Unauthenticated or other-user access**
- Steps: Call the quiet-hours API without a token, then with another user's token for the first user's settings.
- Expected: 401 and 403 respectively; settings unchanged.

**TC-24: Push permission disabled at OS level**
- Preconditions: Quiet hours set; OS notification permission off.
- Steps: Let quiet hours end with held notifications.
- Expected: Nothing is displayed; no crash or error. Whether they show when permission returns: **[Blocked: Q10]**.

**TC-25: Device offline when quiet hours end**
- Preconditions: Held notifications; device offline at 07:00.
- Steps: Bring the device online at 07:30.
- Expected: **[Blocked: Q7]** Delivery on reconnect, or expired if past TTL.

**TC-26: App uninstalled or user logged out while notifications are held**
- Expected: No delivery to the removed token; no errors or retry storms.

**TC-27: Concurrent edit during quiet hours**
- Steps: While notifications are held, change the window so that it ends now.
- Expected: **[Blocked: Q8]** Held notifications are released per the defined rule.

**TC-28: Non-push channels unaffected**
- Preconditions: Quiet hours active.
- Steps: Trigger events that also generate email, SMS or in-app notifications.
- Expected: **[Blocked: Q1]** Only push is suppressed, per the requirement; the in-app inbox still records the event.

**TC-29: Large backlog**
- Steps: Hold 1,000+ notifications, then let the window end.
- Expected: All are delivered without crashing; delivery is rate-limited so the device and push provider are not flooded. **[Blocked: Q4]** on the batching rule.

## Open Questions (not assumed above)

1. **Scope of "push notifications":** Do all types (e.g. security alerts, calls, OTP, emergency or critical alerts) get suppressed, or can some bypass quiet hours? Are other channels (email, SMS, in-app badges) affected?
2. **Boundary inclusivity:** Is the start inclusive and the end exclusive? Exactly what time is "when quiet hours end" (07:00:00 sharp)?
3. **Valid ranges:** Are start equal to end, very short windows or 24-hour windows allowed? What is the minimum and maximum length? What is the time granularity (minutes, 5 or 15 minute steps)?
4. **Delivery format:** Are held notifications sent one by one (a burst), or collapsed into a digest or grouped? Original order preserved? Is there a cap on how many are held?
5. **Time zone:** Is the window based on the device time zone, the account profile time zone or the server time? What happens when the user travels or at DST changes?
6. **Multiple windows and days:** Is only one window allowed? Can there be different windows per weekday, or an exception schedule?
7. **Expiry:** Do held notifications expire (time-sensitive content, e.g. a "meeting in 5 minutes" alert)? What if the device is offline at delivery time? Which TTL applies?
8. **Changing settings mid-window:** If the user edits or disables quiet hours while notifications are held, are they released immediately? Does a change take effect right away or from the next day?
9. **Defaults and UX:** Is quiet hours off by default? Is there an on/off toggle separate from the saved times? Is there a confirmation or indicator when active?
10. **Multi-device and OS permissions:** Is the setting per account or per device? What happens to held notifications when OS permission is off at delivery time?
11. **Interaction with OS-level Do Not Disturb/Focus modes:** Is the app expected to respect these in addition to its own setting?
12. **Non-functional:** What is the acceptable delivery latency after quiet hours end? What is the expected scale of held notifications?
13. **Cancelled events:** If the underlying event is cancelled, read elsewhere or becomes irrelevant while held (e.g. the message was read on desktop), should the notification still be delivered?
