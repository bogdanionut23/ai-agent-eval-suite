# Test Cases: Quiet Hours for Push Notifications

Requirement: Users can set quiet hours (for example 22:00 to 07:00). During quiet hours no push notifications are sent; they are delivered when quiet hours end.

Assumptions used only where the requirement is explicit: a quiet-hours window has a start and an end time, and "delivered when quiet hours end" means queued notifications are sent after the window closes. Everything else is listed under Open Questions. Expected results that depend on an open question are marked [OQ-n] and must be confirmed before execution.

## Positive scenarios

**TC-QH-001: Set quiet hours that cross midnight**
- Preconditions: User logged in, push enabled, no quiet hours set.
- Steps: 1. Open notification settings. 2. Set quiet hours 22:00 to 07:00. 3. Save.
- Expected: Setting saved and shown as 22:00-07:00 after reopening the screen and after re-login.

**TC-QH-002: Set quiet hours within a single day**
- Preconditions: As TC-QH-001.
- Steps: 1. Set quiet hours 13:00 to 15:00. 2. Save.
- Expected: Saved and displayed correctly.

**TC-QH-003: No push during quiet hours**
- Preconditions: Quiet hours 22:00-07:00; device time 23:00; app in background.
- Steps: 1. Trigger a push-eligible event (e.g. new message) for the user.
- Expected: No push is shown on the device and none is sent to the push provider at 23:00.

**TC-QH-004: Push outside quiet hours is unaffected**
- Preconditions: Quiet hours 22:00-07:00; time 12:00.
- Steps: 1. Trigger a push-eligible event.
- Expected: Push is delivered normally and promptly.

**TC-QH-005: Notification held during quiet hours is delivered when they end**
- Preconditions: Quiet hours 22:00-07:00; one event triggered at 23:00 (held).
- Steps: 1. Wait until 07:00 (or advance the clock/scheduler in a test environment). 2. Observe the device.
- Expected: The held notification is delivered at or shortly after 07:00 (tolerance [OQ-6]), not before.

**TC-QH-006: Several notifications held, then delivered at end**
- Preconditions: Quiet hours 22:00-07:00.
- Steps: 1. Trigger 3 different events at 22:30, 01:00, 03:00. 2. Reach 07:00.
- Expected: All 3 are delivered after 07:00; none earlier. Order and grouping per [OQ-4].

**TC-QH-007: Edit existing quiet hours**
- Preconditions: Quiet hours 22:00-07:00 saved.
- Steps: 1. Change to 23:00-06:00. 2. Save. 3. Trigger an event at 22:30.
- Expected: Push is delivered at 22:30 because it is now outside the window.

**TC-QH-008: Quiet hours apply across the user's devices**
- Preconditions: User has two devices with push enabled; quiet hours active.
- Steps: 1. Trigger an event. 
- Expected: No push on either device during quiet hours; both receive it afterwards ([OQ-9]).

## Boundary scenarios

**TC-QH-009: Event exactly at start time (22:00:00)**
- Preconditions: Quiet hours 22:00-07:00.
- Steps: 1. Trigger an event at 22:00:00.
- Expected: Treated per [OQ-2] (start inclusive assumed to be suppressed; confirm).

**TC-QH-010: Event one second before start (21:59:59)**
- Steps: Trigger an event at 21:59:59.
- Expected: Delivered immediately.

**TC-QH-011: Event exactly at end time (07:00:00)**
- Steps: Trigger an event at 07:00:00.
- Expected: Delivered immediately (not held) per [OQ-2]; confirm end boundary.

**TC-QH-012: Event one second before end (06:59:59)**
- Steps: Trigger an event at 06:59:59.
- Expected: Held and delivered at 07:00.

**TC-QH-013: Window spanning midnight handled on both sides**
- Preconditions: Quiet hours 22:00-07:00.
- Steps: 1. Trigger events at 23:59 and 00:01.
- Expected: Both held.

**TC-QH-014: Shortest window**
- Preconditions: None.
- Steps: 1. Set 10:00 to 10:01 (or minimum allowed [OQ-3]). 2. Trigger an event at 10:00:30.
- Expected: Held until 10:01.

**TC-QH-015: Longest window (e.g. 00:00 to 23:59)**
- Steps: Set 00:00-23:59, trigger an event at 12:00.
- Expected: Held until 23:59. If the system rejects it, a clear message is shown ([OQ-3]).

**TC-QH-016: Daylight saving transitions**
- Preconditions: Quiet hours 22:00-07:00; user time zone observes DST.
- Steps: 1. Trigger events on the spring-forward night and the fall-back night. 
- Expected: Window start and end follow local wall-clock time; delivery occurs at local 07:00 ([OQ-5]).

**TC-QH-017: Time zone change**
- Preconditions: Quiet hours 22:00-07:00; user travels to a time zone 5 hours different.
- Steps: 1. Change device/profile time zone. 2. Trigger events.
- Expected: Window is evaluated per [OQ-5].

## Negative scenarios

**TC-QH-018: Start equals end (22:00 to 22:00)**
- Steps: Set and save.
- Expected: Rejected with a validation message, or defined behaviour per [OQ-3] (no silent 24h or 0h window).

**TC-QH-019: Invalid time values**
- Steps: Enter 25:00, 12:60, empty, non-numeric text via UI and via API.
- Expected: Rejected with validation error; previously saved setting unchanged.

**TC-QH-020: Save fails (network error)**
- Steps: Go offline, set quiet hours, save.
- Expected: Error shown; setting is not shown as saved; no partial state.

**TC-QH-021: Unauthenticated or other user's settings via API**
- Steps: Call the quiet-hours API without a token and with another user's ID.
- Expected: 401 / 403; no data changed.

**TC-QH-022: Quiet hours disabled or cleared**
- Preconditions: Quiet hours active; a notification is currently held.
- Steps: 1. Remove/disable quiet hours. 
- Expected: Normal delivery resumes for new events; handling of the held notification per [OQ-7].

**TC-QH-023: Push disabled at OS or app level**
- Preconditions: Quiet hours set; push permission revoked.
- Steps: Trigger an event, reach 07:00.
- Expected: Nothing is delivered before or after quiet hours; no error.

**TC-QH-024: Device offline when quiet hours end**
- Preconditions: A notification is held; device offline at 07:00.
- Steps: Reach 07:00, then reconnect at 07:30.
- Expected: Delivered once on reconnect, no duplicates, subject to provider TTL [OQ-8].

**TC-QH-025: Held notification becomes stale**
- Preconditions: Held notification about an event that is no longer relevant by 07:00 (e.g. message deleted, order cancelled).
- Steps: Reach 07:00.
- Expected: Per [OQ-8]; not silently inconsistent.

**TC-QH-026: Service restart during quiet hours**
- Preconditions: Held notifications exist.
- Steps: Restart the notification service/scheduler at 02:00.
- Expected: Held notifications are not lost and are delivered once at 07:00.

**TC-QH-027: No duplicate delivery**
- Steps: Hold a notification, reach 07:00, check the device and the delivery log.
- Expected: Exactly one delivery per held notification.

**TC-QH-028: Concurrent edit of settings**
- Steps: Change quiet hours on two devices at the same time.
- Expected: One consistent final value; no corrupted data.

## Open questions (not assumed above)

1. **Scope of notifications:** Does "no push notifications" cover all types, including critical ones (security alerts, OTP codes, emergency, calls, alarms)? Is there an exception list or an override?
2. **Boundary inclusivity:** Are start and end times inclusive or exclusive (is 22:00:00 quiet, is 07:00:00 quiet)? What time resolution (minutes or seconds)?
3. **Validation rules:** Are windows with start = end allowed? Minimum and maximum length? Is the time picker 15-min, 5-min or 1-min granularity?
4. **Delivery behaviour at end:** Are all held notifications sent at once (burst) or throttled? Are they collapsed or summarised? Original order kept? Is there a cap on how many are queued, and what happens when it is exceeded?
5. **Time zone and DST:** Which time zone is used (device, profile, server)? What happens on DST changes and when the user travels?
6. **Delivery latency:** What is the acceptable delay after quiet hours end (seconds, minutes)?
7. **Changing settings with queued items:** If quiet hours are edited, disabled or shortened while notifications are held, are they released immediately?
8. **Expiry of held items:** Do held notifications expire (TTL) or get dropped if outdated, or if the device is offline for a long time? Should the delivered notification show the original or the delivery time?
9. **Multiple devices and platforms:** Is the setting per user or per device? Does it differ for iOS, Android and web push?
10. **Multiple windows and days:** Can a user set more than one window or different windows per weekday? Is the single default window on or off for new users?
11. **Other channels:** Do in-app notifications, email, SMS and badge counts continue during quiet hours? Is the in-app inbox still updated?
12. **Silent or data pushes:** Are background data pushes that show no UI also suppressed?
13. **Where it is enforced:** Server-side suppression (no send) or client-side silencing? The wording "no push notifications are sent" suggests server side; please confirm.
14. **Do Not Disturb interaction:** How does this interact with OS Do Not Disturb / Focus modes and with per-category user preferences?
15. **Non-functional:** Expected scale of held notifications, privacy of queued content, and audit/logging needs.
