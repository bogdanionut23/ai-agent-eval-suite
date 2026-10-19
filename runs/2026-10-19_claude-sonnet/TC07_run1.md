# TC07 - Automatic logout after 30 minutes of inactivity

Requirement: A logged-in user is logged out automatically after 30 minutes without activity. Any page load or form submit counts as activity.

## Assumptions used for the test cases (no behaviour invented)

- "Activity" is limited to what the requirement states: page load and form submit. Anything else (mouse movement, typing, scrolling, AJAX calls) is treated as an open question and is NOT used as an expected result.
- The 30-minute timer is measured from the last activity (idle timeout), not from login time.
- Where the exact post-logout behaviour is unspecified, the expected result says only "user is logged out (session ended, protected content no longer accessible)". Specifics such as redirect target or message are in the open questions.
- Test environments should allow the clock or timeout to be controlled (for example, a configurable timeout or a time-travel tool) so that 30-minute waits are practical. Real-time runs are noted where relevant.

## Test cases

### Positive

**TC07-01 - User is logged out after 30 minutes of no activity**
- Preconditions: Valid user account exists. User is logged in and on any authenticated page.
- Steps:
  1. Log in and note the time of the last page load (T0).
  2. Perform no page loads and no form submits.
  3. At T0 + 30 min (plus a small tolerance), attempt to load an authenticated page.
- Expected result: The user is logged out. The authenticated page is not shown, and the user is treated as not logged in.

**TC07-02 - Page load resets the inactivity timer**
- Preconditions: User is logged in.
- Steps:
  1. Log in at T0.
  2. At T0 + 20 min, load another authenticated page (activity).
  3. Do nothing further.
  4. At T0 + 40 min (20 min after the page load), load an authenticated page.
- Expected result: The user is still logged in at T0 + 40 min, because the timer restarted at T0 + 20 min. The user is logged out only after T0 + 50 min of continued inactivity.

**TC07-03 - Form submit resets the inactivity timer**
- Preconditions: User is logged in; an authenticated page with a form (for example profile settings) is available.
- Steps:
  1. Log in at T0 and open the form page.
  2. At T0 + 25 min, submit the form with valid data.
  3. At T0 + 50 min (25 min after the submit), load an authenticated page.
- Expected result: The user is still logged in at T0 + 50 min. The timer restarted at the form submit.

**TC07-04 - Repeated activity keeps the session alive indefinitely**
- Preconditions: User is logged in.
- Steps:
  1. Every 10 minutes for 2 hours, alternate between a page load and a form submit.
  2. After the final activity, load an authenticated page.
- Expected result: The user stays logged in throughout and is never logged out while activity gaps are under 30 minutes.

**TC07-05 - After auto logout, logging in again works and starts a fresh timer**
- Preconditions: User was automatically logged out after 30 minutes idle.
- Steps:
  1. Log in again with valid credentials.
  2. Wait 29 minutes with no activity, then load a page.
- Expected result: Login succeeds, and the user is logged in at the 29-minute mark. The timer is based on the new session, not the old one.

### Negative

**TC07-06 - Access to an authenticated page after timeout is denied**
- Preconditions: User was logged in and has been idle for more than 30 minutes (for example 31 minutes).
- Steps:
  1. Without logging in again, request an authenticated page directly (by URL or browser Back).
- Expected result: Protected content is not displayed. The user is not treated as logged in.

**TC07-07 - Form submit after timeout is rejected and data is not processed as an authenticated action**
- Preconditions: User is logged in, has a form open, and then stays idle for 31 minutes.
- Steps:
  1. After 31 minutes, submit the form.
- Expected result: The submission is not accepted as an authenticated action. The user is logged out. (How the entered data is handled is an open question.)

**TC07-08 - Timeout applies when a second tab or browser is used after the idle period**
- Preconditions: User is logged in in tab A. Tab B shares the same session.
- Steps:
  1. Leave both tabs idle for 31 minutes.
  2. Load a page in tab B.
- Expected result: The user is logged out in tab B. Tab A, on its next page load, also shows the user as logged out.

**TC07-09 - Idle time does not log out a user who was never logged in (no side effects)**
- Preconditions: Anonymous visitor, not logged in.
- Steps:
  1. Stay on a public page for 31 minutes, then load another public page.
- Expected result: Public pages remain accessible. No error occurs and no logout message is shown.

**TC07-10 - Logged-out user cannot reuse the old session identifier**
- Preconditions: User was automatically logged out after idle timeout. The old session cookie or token value was captured beforehand.
- Steps:
  1. Send a request to an authenticated page using the captured old session cookie or token.
- Expected result: The request is rejected. The old session is no longer valid.

### Boundary

**TC07-11 - Just under the limit (29 min 59 s)**
- Preconditions: User is logged in; last activity at T0.
- Steps:
  1. Do nothing until T0 + 29:59.
  2. Load an authenticated page.
- Expected result: The user is still logged in, and the page loads normally. This load also resets the timer.

**TC07-12 - Exactly at the limit (30 min 00 s)**
- Preconditions: User is logged in; last activity at T0.
- Steps:
  1. Do nothing until exactly T0 + 30:00.
  2. Load an authenticated page.
- Expected result: Behaviour at exactly 30:00 is not defined by the requirement ("after 30 minutes"). Record the actual outcome and confirm it with the product owner (see open question Q3). Do not mark pass or fail until clarified.

**TC07-13 - Just over the limit (30 min 01 s)**
- Preconditions: User is logged in; last activity at T0.
- Steps:
  1. Do nothing until T0 + 30:01.
  2. Load an authenticated page.
- Expected result: The user is logged out.

**TC07-14 - Activity just before expiry extends the session by a full period**
- Preconditions: User is logged in; last activity at T0.
- Steps:
  1. At T0 + 29:59, load an authenticated page.
  2. At T0 + 29:59 + 29:59 (that is, 29:59 after the load), load another authenticated page.
- Expected result: The user is still logged in on both loads.

**TC07-15 - Activity at the same moment as expiry (race condition)**
- Preconditions: User is logged in; last activity at T0.
- Steps:
  1. Submit a form at approximately T0 + 30:00 (use automation to hit the boundary as precisely as possible).
- Expected result: The outcome is consistent and deterministic: either the request is processed and the timer reset, or the user is logged out. There is no partial state (for example, data saved while the user is shown as logged out). Record the result and confirm against the requirement (see Q3).

**TC07-16 - Very long form entry that straddles the timeout**
- Preconditions: User is logged in and opens a long form at T0.
- Steps:
  1. Type in the form continuously for 35 minutes without any page load or submit.
  2. Submit the form at T0 + 35 min.
- Expected result: Per the stated requirement, typing is not listed as activity, so the user is logged out before the submit. Record the actual behaviour and confirm against Q1 and Q5. Do not mark pass or fail until clarified.

**TC07-17 - Timeout still correct across system clock or time zone changes**
- Preconditions: User is logged in; test environment allows changing client time zone (and, if possible, server-side clock handling such as DST).
- Steps:
  1. Log in and note the activity time.
  2. Change the client device time zone or clock by one hour (forward and backward in separate runs).
  3. Load a page at 29 minutes of real elapsed time, then again at 31 minutes of real elapsed time in a fresh run.
- Expected result: The timeout is based on elapsed server-measured time, not on the client clock. The user is logged in at 29 minutes and logged out at 31 minutes regardless of the client clock change.

## Open questions (not assumed in the test cases above)

1. **Definition of activity.** The requirement says "any page load or form submit". Do these also count, or not: AJAX/fetch/API calls made in the background, keep-alive or polling requests, mouse movement, typing, scrolling, file uploads/downloads, WebSocket traffic? Does a static asset request (images, CSS) count as a page load?
2. **What does "page load" include?** Does a failed load (404, 500, 403) reset the timer? Does a browser refresh or Back/Forward navigation served from cache count?
3. **Exact boundary.** Is a user logged out when idle time is exactly 30:00 or only strictly greater than 30:00? What time precision and tolerance are acceptable (seconds, minutes)?
4. **Does a form submit with invalid data (validation failure) count as activity?** Does a submit that is rejected by the server?
5. **Handling of in-progress work.** What happens to unsaved form data when the user is logged out? Is there a warning before logout (for example at 25 or 29 minutes), a countdown, or an option to extend the session?
6. **Post-logout behaviour.** What does the user see after auto logout: redirect to the login page, a message such as "session expired", a specific status code (401/403) for API calls? After re-login, is the user returned to the page they were on?
7. **Where is the timer enforced and measured?** Server side, client side, or both? Is it based on server time? What happens if the browser tab is closed or the device sleeps and then resumes?
8. **Multiple tabs, windows and devices.** Does activity in one tab keep all tabs alive? Is the timeout per session or per user (what if the user is logged in on two devices)?
9. **"Remember me" or persistent login.** Does the timeout apply to users who chose to stay logged in? Does it apply to all roles (admin, regular, API/service accounts)?
10. **Configurability.** Is 30 minutes fixed, or configurable per environment/tenant/role? Is there also an absolute maximum session length regardless of activity?
11. **Security and audit.** Must the server-side session be invalidated and the token revoked on timeout? Must the auto logout be logged or audited? Should other things (for example, single-sign-on session, refresh tokens) be terminated too?
12. **Pages exempt from the rule.** Are there pages (for example, long-running video, dashboards, or payment steps) that should be handled differently?
13. **Is "without activity" measured from login for a user who logs in but takes no further action?** (Assumed yes in TC07-01; please confirm that login itself counts as the first activity.)

## Coverage summary

- Positive: TC07-01 to TC07-05
- Negative: TC07-06 to TC07-10
- Boundary: TC07-11 to TC07-17 (TC07-12, TC07-15 and TC07-16 are blocked pending answers to the open questions)
