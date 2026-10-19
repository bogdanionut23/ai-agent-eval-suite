# TC15 - Test cases: Quiet hours for push notifications

**Requirement:** Users can set quiet hours (for example 22:00 to 07:00). During quiet hours no push notifications are sent; they are delivered when quiet hours end.

**Conventions**
- Examples use the window 22:00-07:00 (overnight, crosses midnight) unless stated otherwise.
- Where a test depends on an open question (section 2), the assumed answer is stated in the test. If the question is resolved differently, the test must be updated.
- Notification "generated" means the backend produces a push notification for a user. "Sent" means the push is handed to the push provider (APNs/FCM/Web Push).

---

## 1. Test cases

### 1.1 Positive scenarios

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| QH-P01 | Set overnight quiet hours | User is logged in, quiet hours not configured | 1. Open notification settings. 2. Enable quiet hours. 3. Set start 22:00 and end 07:00. 4. Save. | Setting is saved and shown as 22:00-07:00 after reload. |
| QH-P02 | Push suppressed during overnight window | QH-P01 done, user has a push-enabled device | 1. Set the device clock (or test clock) to 02:00. 2. Trigger a push-generating event for the user. | No push is sent to the provider. |
| QH-P03 | Push suppressed at start of window | As QH-P01 | Trigger a push event at 22:00:00. | No push is sent. |
| QH-P04 | Push delivered just before window starts | As QH-P01 | Trigger a push event at 21:59:59. | Push is sent immediately. |
| QH-P05 | Queued notifications delivered when quiet hours end | As QH-P01; 3 push events generated at 23:00, 01:30 and 05:45 | 1. Let the clock pass 07:00. | All 3 pushes are sent once, shortly after 07:00. None is sent before 07:00. |
| QH-P06 | Push delivered at end of window | As QH-P01 | Trigger a push event at 07:00:00. | Push is sent immediately (end is exclusive, see OQ-3). |
| QH-P07 | Same-day quiet window | Quiet hours set to 13:00-14:00 | Trigger push events at 12:59, 13:30 and 14:00. | 12:59 and 14:00 are sent. 13:30 is suppressed and delivered at 14:00. |
| QH-P08 | Disabling quiet hours restores delivery | Quiet hours 22:00-07:00 active at 23:00 | 1. Disable quiet hours. 2. Trigger a push event. | Push is sent immediately. |
| QH-P09 | Editing the window changes behaviour | Quiet hours 22:00-07:00 | 1. Change to 23:00-06:00. 2. Trigger a push at 22:30. | Push at 22:30 is sent (outside the new window). |
| QH-P10 | No loss of queued notifications | As QH-P05 with 50 events in the window | Let the window end and count received pushes. | 50 generated, 50 delivered. No duplicates. |

### 1.2 Negative scenarios

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| QH-N01 | Reject invalid time format | Settings screen open | Enter start "25:00" and save. | Save is blocked with a validation message. Previous value is kept. |
| QH-N02 | Reject non-time input | Settings screen open | Enter start "ab:cd" and save. | Save is blocked with a validation message. |
| QH-N03 | Reject missing end time | Settings screen open | Set start 22:00, leave end empty, save. | Save is blocked with a validation message. |
| QH-N04 | Quiet hours off means no suppression | Quiet hours never enabled | Trigger push events at 23:00 and 03:00. | Both pushes are sent immediately. |
| QH-N05 | Quiet hours are per user | User A has 22:00-07:00, user B has none | At 02:00, trigger a push for user B. | User B's push is sent. User A's pushes are suppressed. |
| QH-N06 | Other users cannot change quiet hours | User A and user B exist | Send an API request to update user A's quiet hours while logged in as user B. | Request is rejected (403 or equivalent). User A's setting is unchanged. |
| QH-N07 | Push provider failure at window end | Queued pushes exist; provider returns an error at 07:00 | Let the window end while the provider is failing, then restore it. | Failed pushes are retried according to the retry policy (OQ-9). No silent loss. |
| QH-N08 | Device offline at window end | Device has no connectivity from 06:50 to 07:30 | Let the window end, then reconnect the device at 07:30. | Queued pushes are delivered after reconnect, per the delivery policy (OQ-9). |
| QH-N09 | Notification not suppressed for non-push channel | Quiet hours active; in-app inbox enabled | Trigger a notification at 02:00. | Push is not sent. Inbox item is stored (assumes OQ-1 answer: quiet hours apply to push only). |
| QH-N10 | Queued push not sent after its content is no longer valid | A push is queued at 23:00 for a resource the user deletes at 01:00 | Let the window end. | Behaviour follows the expiry rule (OQ-8). Test passes once the rule is defined and applied. |

### 1.3 Boundary scenarios

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| QH-B01 | Minimum window length | Quiet hours 07:00-07:01 | Trigger push at 07:00:30. Then at 07:01:00. | 07:00:30 is suppressed and delivered at 07:01. 07:01:00 is sent immediately. |
| QH-B02 | Window ending at midnight | Quiet hours 23:00-00:00 | Trigger push at 23:59:59 and at 00:00:00. | 23:59:59 is suppressed. 00:00:00 is sent (OQ-3). |
| QH-B03 | Window starting at midnight | Quiet hours 00:00-06:00 | Trigger push at 00:00:00 and 05:59:59. | Both suppressed. 06:00:00 is sent. |
| QH-B04 | Maximal window | Quiet hours 00:01-00:00 (or the longest valid window per OQ-4) | Trigger pushes at 00:00:30 and 12:00. | Both suppressed. Behaviour at 00:00 follows OQ-3. |
| QH-B05 | Start equals end | Quiet hours 22:00-22:00 | Trigger push at 22:00 and at 12:00. | Behaviour must match the rule defined in OQ-5. Test blocked until OQ-5 is answered. |
| QH-B06 | Time zone change during window | User in UTC+1, window 22:00-07:00 local. Travels to UTC-5 at 23:00 local | Change the device time zone while in the window. | Window is applied in the new local time (OQ-2). Pushes stay suppressed until the new 07:00 local. |
| QH-B07 | DST start night | Window 22:00-07:00 spanning a spring-forward transition (02:00 becomes 03:00) | Trigger pushes around 01:59, 03:00 and 07:00 local. | Window still ends at 07:00 local. Queued pushes are released at the correct local time (OQ-2). |
| QH-B08 | DST end night | Window 22:00-07:00 spanning a fall-back transition (02:00 repeats) | Trigger pushes around both 02:30 occurrences. | Both occurrences are suppressed. No push is sent twice at window end. |
| QH-B09 | Large queue at window end | 5,000 events queued during the window | Let the window end and measure delivery time. | All 5,000 are delivered. Delivery is spread so the provider rate limit is not exceeded (rate limit value per OQ-10). |
| QH-B10 | Event exactly at window end with a queued burst | Queue of 20 pushes; a new push at 07:00:00.000 | Observe ordering and timing. | No push is delivered before 07:00:00. All 21 are delivered once. Order follows OQ-7. |
| QH-B11 | Rapid edits of the window | Window 22:00-07:00 | Change the end time to 06:00, then 08:00, then 07:00 within 1 minute. | Only the final value is applied. Pushes are suppressed as per 22:00-07:00 at 06:30. |
| QH-B12 | Maximum time value | Quiet hours end "23:59" | Save, then trigger a push at 23:58 and at 00:00. | Save succeeds. 23:58 is suppressed. 00:00 is sent if the window does not cover it. |

---

## 2. Open questions

These questions are not answered by the requirement. No behaviour has been assumed beyond what is stated in the test cases.

1. **Scope of suppression.** Does quiet hours apply only to push notifications, or also to email, SMS and in-app notifications? (The requirement only says push.) Test QH-N09 assumes push only.
2. **Time zone.** Are quiet hours evaluated in the device's local time, a time zone stored on the user profile, or UTC? What happens when the user travels or changes time zone? How are DST transitions handled?
3. **Boundary inclusion.** Is the start time inclusive and the end time exclusive (22:00 suppressed, 07:00 delivered)? Test cases assume this. The requirement does not say.
4. **Minimum and maximum window.** Is the minimum window length 1 minute? Is there a maximum? Are minute values only, or are seconds allowed?
5. **Start equals end.** What does 22:00-22:00 mean: rejected, disabled, or a full 24-hour window?
6. **Critical or exempt notifications.** Are any notifications exempt from quiet hours (security alerts, payment failures, calls, user-initiated actions)? Is there an override?
7. **Queued delivery order and grouping.** When the window ends, are queued pushes sent in original order, individually or as a summary/grouped push? Is there a limit on the number of queued pushes?
8. **Expiry of queued notifications.** Should a notification queued during quiet hours expire after some time, or be dropped if its underlying event no longer exists? Is a "stale" notification still sent at 07:00?
9. **Failure and retry.** What happens if the device is offline or the push provider fails at window end? How many retries, over what period, and is the notification dropped after that?
10. **Rate limit at window end.** Is there a throttle when releasing a large queue? What is the rate limit?
11. **Multiple devices.** Is quiet hours a per-user or per-device setting? If the user has several devices, are pushes suppressed on all of them?
12. **Editing and disabling mid-window.** If quiet hours are disabled or the window is shortened while notifications are queued, are the queued notifications sent immediately, kept, or dropped?
13. **Recurrence.** Does the setting apply every day, or can the user choose days of the week? Is there a one-off override ("until tomorrow")?
14. **Default state.** Is quiet hours off by default? Is a default window (for example 22:00-07:00) suggested on first enable?
15. **Validation rules.** Which input formats are accepted (24-hour only, 12-hour with AM/PM, minutes required)? What is the exact validation error text?
16. **Badge and in-app state.** Does the app icon badge count update during quiet hours? Is the in-app notification list updated in real time?
17. **Where the setting lives.** Is quiet hours stored on the server (so it applies to all clients) or on the device? Is it synced across apps (iOS, Android, web)?
18. **Permissions.** Who can change quiet hours? Can an administrator or account owner set quiet hours for a user?
19. **Accessibility and localisation.** Are time pickers required to support 12-hour formats and locale-specific display? Is there a text-based input option?
20. **Analytics and audit.** Should suppressed or deferred pushes be logged or counted for analytics? Is the change of quiet hours audited?

---

## 3. Coverage summary

| Area | Test cases |
|---|---|
| Core behaviour (suppress, then deliver at end) | QH-P02, QH-P03, QH-P05, QH-P06, QH-P10 |
| Window definition and validation | QH-P01, QH-P07, QH-P09, QH-N01 to QH-N03, QH-B05, QH-B12 |
| Enable, disable, scope | QH-P08, QH-N04, QH-N05, QH-N06, QH-N09 |
| Failure and offline handling | QH-N07, QH-N08 |
| Boundaries (time, length, DST, time zone, volume) | QH-B01 to QH-B11 |
| Ordering and expiry | QH-B10, QH-N10 |
