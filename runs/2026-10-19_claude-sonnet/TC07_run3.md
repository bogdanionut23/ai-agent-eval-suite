# Test cases: automatic logout after 30 minutes of inactivity

Requirement: A logged-in user is logged out automatically after 30 minutes without activity. Any page load or form submit counts as activity.

## Assumptions used for the tests (no behaviour invented beyond these)
- "30 minutes" means 30:00 of elapsed time since the last activity.
- "Logged out" means the session is ended and protected pages are no longer accessible without logging in again.
- Time can be controlled (clock manipulation, a test configuration, or real waiting).
- Where the requirement is silent, the expected result is marked "see OQ-n" and is not asserted.

## Positive

### TC-001 Logout after 30 minutes of no activity
- Preconditions: User has a valid account and is logged in. The user then does nothing.
- Steps:
  1. Log in and land on any page.
  2. Perform no page load or form submit for 30 minutes.
  3. After 30 minutes, try to open a protected page.
- Expected: The user is logged out. The protected page is not shown and the user must log in again.

### TC-002 Page load resets the inactivity timer
- Preconditions: User is logged in.
- Steps:
  1. Wait 29 minutes without activity.
  2. Load or navigate to another page.
  3. Wait another 29 minutes without activity.
  4. Load a protected page.
- Expected: The user is still logged in at step 4. The timer restarted at step 2 (58 minutes since login, never 30 minutes between activities).

### TC-003 Form submit resets the inactivity timer
- Preconditions: User is logged in and has a page with a form (for example profile settings).
- Steps:
  1. Wait 29 minutes without activity.
  2. Submit a valid form.
  3. Wait 29 minutes without activity.
  4. Load a protected page.
- Expected: The user is still logged in at step 4. The form submit counted as activity.

### TC-004 Repeated activity keeps the session alive beyond 30 minutes total
- Preconditions: User is logged in.
- Steps:
  1. Every 10 minutes, alternate between a page load and a form submit.
  2. Continue for 90 minutes.
- Expected: The user is never logged out during the 90 minutes.

### TC-005 Logged out user must log in again after timeout
- Preconditions: Session expired through inactivity (TC-001).
- Steps:
  1. Log in again with valid credentials.
  2. Load a protected page.
- Expected: Login succeeds and a new inactivity period of 30 minutes starts.

## Boundary

### TC-006 Activity at 29 minutes 59 seconds
- Preconditions: User is logged in, last activity at T0.
- Steps:
  1. At T0 + 29:59 load a page.
- Expected: The page loads normally, the user stays logged in and the timer restarts.

### TC-007 Activity at exactly 30 minutes 00 seconds
- Preconditions: User is logged in, last activity at T0.
- Steps:
  1. At T0 + 30:00 load a page.
- Expected: Not defined by the requirement (see OQ-1). Record the observed behaviour and confirm with the product owner.

### TC-008 Activity at 30 minutes 01 seconds
- Preconditions: User is logged in, last activity at T0.
- Steps:
  1. At T0 + 30:01 load a page.
- Expected: The user is logged out and is not shown the protected page (see OQ-3 for the exact destination).

### TC-009 Timer restarts from the latest activity, not the first
- Preconditions: User is logged in at T0.
- Steps:
  1. At T0 + 20:00 load a page.
  2. At T0 + 45:00 (25 minutes after the last activity) load a page.
- Expected: The user is still logged in at step 2. Logout would have happened at T0 + 50:00 if there were no further activity.

### TC-010 Logout takes effect close to 30 minutes with no further activity
- Preconditions: User is logged in and idle, with a page left open.
- Steps:
  1. Do nothing for 30 minutes.
  2. Check the session state at 30:00 plus the tolerance agreed in OQ-4.
- Expected: The session is ended within the agreed tolerance.

## Negative

### TC-011 Protected page access after timeout is denied
- Preconditions: Session expired through inactivity.
- Steps:
  1. Open a bookmarked or previously visited protected URL directly.
- Expected: Access is denied and the user is asked to log in. No protected data is shown.

### TC-012 Form submit after timeout is rejected
- Preconditions: User left a page with a filled-in form open for more than 30 minutes.
- Steps:
  1. Submit the form after the session has expired.
- Expected: The submit is not processed as an authenticated action. The user is logged out. Handling of the entered data is not defined (see OQ-5).

### TC-013 Expired session cannot be reused through the Back button
- Preconditions: Session expired through inactivity.
- Steps:
  1. Use the browser Back button to return to a protected page.
- Expected: Protected content is not retrievable from a stale authenticated state. Any action needing the server requires login.

### TC-014 Reusing the old session identifier or token fails
- Preconditions: Session token captured before the timeout. Session expired through inactivity.
- Steps:
  1. Send a request with the old session cookie or token.
- Expected: The request is rejected as unauthenticated.

### TC-015 Logged-out user is not affected by the timer
- Preconditions: User is not logged in.
- Steps:
  1. Browse public pages for 40 minutes.
- Expected: No logout event or error occurs. The timeout only applies to a logged-in user.

### TC-016 Failed or invalid requests and activity classification
- Preconditions: User is logged in.
- Steps:
  1. Wait 29 minutes.
  2. Submit a form with invalid data that fails validation.
  3. Wait 29 minutes and load a protected page.
- Expected: Whether a failed submit counts as activity is not defined (see OQ-6). Record the observed behaviour.

### TC-017 Non-page traffic does not count as activity
- Preconditions: User is logged in.
- Steps:
  1. Wait 29 minutes.
  2. Let background requests run (for example AJAX polling, mouse movement, scrolling, typing without submitting).
  3. Wait 2 more minutes and load a protected page.
- Expected: Per the requirement only page load and form submit count, so the user is logged out at the 30-minute mark. Confirm in OQ-7 whether other events count.

## Multi-session and environment

### TC-018 Multiple tabs
- Preconditions: User is logged in with two tabs open.
- Steps:
  1. In tab A, do nothing. In tab B, load pages every 10 minutes for 40 minutes.
  2. Load a page in tab A.
- Expected: Not defined (see OQ-8). The session is expected to be shared, so activity in tab B would normally keep tab A logged in. Confirm.

### TC-019 Multiple devices or browsers
- Preconditions: User is logged in on device 1 and device 2.
- Steps:
  1. Stay idle on device 1 for 30 minutes. Remain active on device 2.
- Expected: Device 1 is logged out. Device 2 stays logged in, assuming independent sessions (see OQ-9).

### TC-020 Client clock changes
- Preconditions: User is logged in.
- Steps:
  1. Change the device clock forward 31 minutes, then load a page.
  2. Reset the clock and change it backward 31 minutes, then wait 30 real minutes.
- Expected: The timeout is based on server time, so a client clock change neither logs the user out early nor extends the session (see OQ-10).

### TC-021 Session survives server-side events consistently
- Preconditions: User is logged in.
- Steps:
  1. Restart the application server or fail over to another node. Load a page within 30 minutes.
- Expected: Not defined (see OQ-11). Record the behaviour.

### TC-022 Daylight saving or timezone change
- Preconditions: User is logged in shortly before a DST change.
- Steps:
  1. Stay idle across the DST boundary for 30 real minutes.
- Expected: Logout happens after 30 elapsed minutes, not 30 wall-clock minutes.

## Open questions
- OQ-1: Is the limit inclusive or exclusive? At exactly 30:00 is the user still logged in or already logged out?
- OQ-2: Is the "30 minutes" configurable (per role, per tenant, per environment)? Are there users such as administrators with a different value?
- OQ-3: What does the user see after being logged out: a redirect to the login page, a message about the timeout, a return to the original URL after re-login?
- OQ-4: How precise must the logout be? Does it need to happen exactly at 30:00 (a client side timer or push), or only be enforced on the next request?
- OQ-5: What happens to unsaved data in a form submitted after expiry: lost, preserved, or resubmitted after login?
- OQ-6: Does a failed request (validation error, 4xx or 5xx, a submit that is rejected) count as activity?
- OQ-7: Do other interactions count as activity: AJAX or API calls, file downloads, WebSocket traffic, scrolling, mouse movement, keystrokes? Does a page load triggered automatically (auto-refresh) count?
- OQ-8: How are multiple tabs of the same session treated?
- OQ-9: Are multiple concurrent sessions per user allowed, and is the timer per session?
- OQ-10: Which clock is authoritative (server or client)?
- OQ-11: Should sessions and timers survive server restarts or deployments?
- OQ-12: Is a warning shown before logout (for example at 25 minutes) with a way to extend the session? None is specified, so none is tested.
- OQ-13: Is there a "remember me" or persistent login option, and does it change the timeout?
- OQ-14: Does the timeout apply to the API and mobile clients as well as the web UI?
- OQ-15: Should the logout be audited or logged (event record, security log)?
