# Test Cases: Automatic Logout After 30 Minutes of Inactivity

Requirement: A logged-in user is logged out automatically after 30 minutes without activity. Any page load or form submit counts as activity.

Notation: T = time since the last activity (page load or form submit). "Logged out" means the session is invalidated server-side and the user is sent to the login page (assumed; see open questions).

## Positive scenarios

### TC-01: User stays logged in under 30 minutes of inactivity
- Preconditions: User is logged in. Last activity at T=0.
- Steps:
  1. Do nothing for 29 minutes.
  2. Load any protected page.
- Expected: The page loads normally. The user is still logged in.

### TC-02: User is logged out after 30 minutes without activity
- Preconditions: User is logged in. Last activity at T=0.
- Steps:
  1. Do nothing for 31 minutes.
  2. Try to load a protected page.
- Expected: The session is expired. The user is redirected to the login page. The protected content is not shown.

### TC-03: Page load resets the inactivity timer
- Preconditions: User is logged in at T=0.
- Steps:
  1. Wait 20 minutes.
  2. Load any page.
  3. Wait another 20 minutes (40 minutes since login, 20 since the last page load).
  4. Load a protected page.
- Expected: The user is still logged in. The timer restarted at step 2.

### TC-04: Form submit resets the inactivity timer
- Preconditions: User is logged in at T=0. A page with a form is open.
- Steps:
  1. Wait 25 minutes.
  2. Submit a valid form.
  3. Wait 25 minutes.
  4. Load a protected page.
- Expected: The form submit succeeds. After step 3 the user is still logged in, because the timer restarted at step 2.

### TC-05: Re-login after timeout works and starts a new session
- Preconditions: The user was logged out by the timeout (TC-02).
- Steps:
  1. Log in with valid credentials.
  2. Load a page.
- Expected: Login succeeds. A new 30-minute timer starts. No stale session data is reused.

### TC-06: Repeated activity keeps the session alive indefinitely
- Preconditions: User is logged in.
- Steps:
  1. Load a page every 10 minutes for 2 hours.
- Expected: The user is never logged out.

## Boundary scenarios

### TC-07: Just under the limit (29 min 59 s)
- Preconditions: User is logged in. Last activity at T=0.
- Steps:
  1. Wait 29 minutes 59 seconds.
  2. Load a protected page.
- Expected: The user is still logged in. The page loads.

### TC-08: Exactly 30 minutes
- Preconditions: User is logged in. Last activity at T=0.
- Steps:
  1. Wait exactly 30 minutes 0 seconds.
  2. Load a protected page.
- Expected: Behaviour depends on whether the limit is inclusive or exclusive. See open question Q1. Record the actual result and confirm it against the answer.

### TC-09: Just over the limit (30 min 1 s)
- Preconditions: User is logged in. Last activity at T=0.
- Steps:
  1. Wait 30 minutes 1 second.
  2. Load a protected page.
- Expected: The user is logged out and redirected to the login page.

### TC-10: Activity just before expiry
- Preconditions: User is logged in. Last activity at T=0.
- Steps:
  1. Wait 29 minutes 59 seconds.
  2. Submit a form.
  3. Wait 29 minutes 59 seconds.
  4. Load a protected page.
- Expected: The user is still logged in. The timer reset at step 2.

### TC-11: Request that straddles the expiry
- Preconditions: User is logged in. A form submit takes several seconds to process.
- Steps:
  1. Start a form submit at T=29:58, so that the server finishes after T=30:00.
- Expected: The behaviour is consistent and defined (see Q5). It must not partially save data and then log the user out without telling them.

### TC-12: Session cookie or token expiry after the timeout
- Preconditions: User is logged in.
- Steps:
  1. Wait 31 minutes.
  2. Replay the old session cookie or token in a request.
- Expected: The server rejects the old session. No access is granted.

## Negative scenarios

### TC-13: Form submit after expiry is rejected
- Preconditions: User is logged in with a form open. Last activity at T=0.
- Steps:
  1. Wait 31 minutes.
  2. Submit the form.
- Expected: The submit is not processed. The user is redirected to login. No data is saved. (Whether the entered data is preserved is Q6.)

### TC-14: Direct URL access after expiry
- Preconditions: User is logged in. Last activity at T=0.
- Steps:
  1. Wait 31 minutes.
  2. Type the URL of a protected page into the address bar.
- Expected: The user is redirected to login. The protected content is not shown.

### TC-15: Back button after timeout logout
- Preconditions: The user was logged out by the timeout.
- Steps:
  1. Press the browser Back button to a previously viewed protected page.
- Expected: Protected data is not accessible. The page is not served from cache with live data, or any further action requires login.

### TC-16: API / AJAX request after expiry
- Preconditions: User is logged in. Last activity at T=0.
- Steps:
  1. Wait 31 minutes.
  2. Send an API or background request with the old session.
- Expected: The server returns an unauthenticated response (for example 401). The UI handles it by sending the user to login.

### TC-17: Failed login or unauthenticated page load does not extend a session
- Preconditions: A session has expired. Another user or an anonymous visitor is using the same browser.
- Steps:
  1. Load the login page or submit a wrong password.
  2. Attempt to open a protected page.
- Expected: No session is revived. Protected pages still require login.

### TC-18: Non-user-initiated requests do not reset the timer
- Preconditions: User is logged in. The page runs background polling or auto-refresh (if any exists). See Q3.
- Steps:
  1. Leave the page idle for 31 minutes with background polling running.
  2. Load a protected page.
- Expected: Per the expected answer to Q3. Without a clarification, record the observed behaviour.

## Multi-session and environment scenarios

### TC-19: Activity in one tab keeps other tabs alive
- Preconditions: User is logged in with two tabs open in the same browser.
- Steps:
  1. Leave tab A idle. Load a page in tab B at T=20.
  2. At T=35, interact with tab A.
- Expected: Per Q4, the session is shared, so tab A should still be logged in (15 minutes since the last activity).

### TC-20: Logout in one tab affects all tabs
- Preconditions: Two tabs open. The session times out.
- Steps:
  1. After 31 minutes, load a page in tab A.
  2. Switch to tab B and load a page.
- Expected: Both tabs require login.

### TC-21: Separate devices or browsers have independent timers
- Preconditions: The same user is logged in on two devices.
- Steps:
  1. Keep device A active every 10 minutes.
  2. Leave device B idle for 31 minutes.
  3. Load a page on both.
- Expected: Device A stays logged in. Device B is logged out (assuming independent sessions, see Q4).

### TC-22: Client clock changed
- Preconditions: User is logged in.
- Steps:
  1. Move the client machine clock forward 1 hour, then load a page.
  2. Separately, move the clock back 1 hour and wait 31 real minutes.
- Expected: Timing is enforced by the server clock. A client clock change neither logs the user out early nor extends the session.

### TC-23: Time zone or daylight-saving change during the session
- Preconditions: User is logged in near a DST transition.
- Steps:
  1. Stay idle across the DST change for 29 minutes of real elapsed time.
  2. Load a page.
- Expected: The user is still logged in. Elapsed time is measured independently of local time.

### TC-24: Inactivity while the browser or machine sleeps
- Preconditions: User is logged in.
- Steps:
  1. Put the laptop to sleep for 31 minutes.
  2. Wake it and click a link.
- Expected: The user is logged out and sent to login (the elapsed wall-clock time exceeded 30 minutes).

### TC-25: Tab is open but unattended; user is notified
- Preconditions: User is logged in with a page open.
- Steps:
  1. Do nothing for 31 minutes without reloading.
  2. Look at the page.
- Expected: Per Q2 (whether the page itself reacts without a request). Record the actual behaviour.

## Open questions (not assumed in the test cases above)

1. Is the limit inclusive or exclusive at exactly 30:00? Does "after 30 minutes" mean at or after 30:00 or strictly more than 30:00?
2. Is the logout enforced only on the next request (server-side), or does the UI actively log the user out and show a message at 30 minutes (client timer)? What should the user see (message, redirect target)?
3. What counts as activity beyond "page load or form submit"? Do AJAX/API calls, background polling, WebSocket traffic, scrolling, mouse movement, typing in a form without submitting, or video playback reset the timer? Does a failed or rejected request count?
4. Is the timer per session, per tab, or per device? Is the session shared across tabs? Are multiple concurrent sessions allowed?
5. What happens to a request that is in progress at the moment of expiry? Is the timer measured from the request start or end?
6. Is unsaved form data preserved after a timeout? After re-login, is the user returned to the page they were on?
7. Does the requirement apply to all roles and all pages, including public pages and the login page? Are there exemptions (for example "remember me", admin, or long-running operations like uploads)?
8. Is the 30-minute value configurable, and who can change it?
9. Is the user warned before logout (for example a countdown at 28 minutes) and can they extend the session?
10. What is cleaned up on logout (server session, cookies, tokens, local/session storage, refresh tokens)? Is the logout recorded in an audit log?
11. Which time source is authoritative (server clock) and how is clock skew between servers in a cluster handled?
12. How does "remember me" or persistent login interact with the timeout?
13. Are there security or compliance standards behind the 30 minutes (for example OWASP or PCI) that should shape the tests?
