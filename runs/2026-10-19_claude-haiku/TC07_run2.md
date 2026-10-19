# TC07 - Test cases: automatic logout after 30 minutes of inactivity

Requirement: A logged-in user is logged out automatically after 30 minutes without activity. Any page load or form submit counts as activity.

Conventions used below:
- "Idle" means no counted activity event.
- Unless a step says otherwise, the user starts logged in on a protected page with the timer freshly started.
- Where a test depends on an open question (see the last section), the dependency is marked "[depends on OQ-n]".

---

## Positive scenarios

### TC07-01 - Idle session is logged out after 30 minutes
- Preconditions: User is logged in. No activity since login.
- Steps:
  1. Log in and land on a protected page.
  2. Do nothing for 30 minutes.
  3. Trigger a page load of a protected page (or wait for the client to react, if it does so on its own).
- Expected result: The session has ended. The protected page does not render. The user is redirected to the login page. A subsequent request for protected data is rejected.

### TC07-02 - Page load resets the inactivity timer
- Preconditions: User is logged in.
- Steps:
  1. Log in. Wait 25 minutes.
  2. Load a protected page.
  3. Wait a further 25 minutes (50 minutes after login, 25 minutes after the last activity).
  4. Load a protected page.
- Expected result: The user is still logged in at step 4. The protected page renders.

### TC07-03 - Form submit resets the inactivity timer
- Preconditions: User is logged in. A form exists that saves successfully.
- Steps:
  1. Log in. Wait 25 minutes.
  2. Submit a valid form.
  3. Confirm the submit succeeded.
  4. Wait 25 minutes, then load a protected page.
- Expected result: The form submit is saved. The user is still logged in and the protected page renders.

### TC07-04 - Session stays alive while activity keeps occurring within each 30-minute window
- Preconditions: User is logged in.
- Steps:
  1. Log in. Every 20 minutes, alternate between a page load and a form submit, for 2 hours.
  2. Load a protected page at the end.
- Expected result: The user is not logged out at any point during the 2 hours.

### TC07-05 - Login after a timeout starts a new 30-minute window
- Preconditions: The user was logged out by inactivity (TC07-01).
- Steps:
  1. Log in again.
  2. Wait 20 minutes, then load a protected page.
  3. Wait 20 minutes, then load a protected page again.
- Expected result: The user stays logged in through both checks. The timer starts fresh at the new login.

---

## Negative scenarios

### TC07-06 - Form submit after expiry is rejected
- Preconditions: User is logged in. A form is open in the browser.
- Steps:
  1. Log in and open a form. Leave the page idle for 30 minutes.
  2. Fill in valid data and submit.
- Expected result: The submit is refused. No data is written to the database. The user is sent to the login page or shown a session-expired message. The submit does not count as activity and does not revive the session.

### TC07-07 - Page load after expiry does not show protected content
- Preconditions: User is logged in.
- Steps:
  1. Log in. Leave the session idle for 30 minutes.
  2. Navigate to a protected URL (typed or from a bookmark).
- Expected result: The protected page is not shown. The user is redirected to login. The requested URL is kept for redirect after login, if that feature exists [depends on OQ-6].

### TC07-08 - Back button after logout does not show cached protected content
- Preconditions: User was on a protected page before the timeout.
- Steps:
  1. Log in and load a protected page. Leave it idle for 30 minutes until logged out.
  2. Press the browser Back button.
- Expected result: The cached protected page is not displayed with live data. The user is redirected to login, or the page shows a login prompt. No protected data is shown.

### TC07-09 - API or AJAX call after expiry returns 401 and does not revive the session
- Preconditions: User is logged in. A page makes background requests.
- Steps:
  1. Log in. Leave the page idle for 30 minutes.
  2. Trigger a background request (for example a data fetch or a dropdown load) that the page would normally make.
- Expected result: The server returns 401 (or equivalent). The client handles it by sending the user to the login page. The request does not extend the session.

### TC07-10 - Activity after expiry does not restore the session
- Preconditions: Session has just expired (for example at 30:01).
- Steps:
  1. Let the session expire.
  2. Click, type or scroll on the expired page. Then load a protected page.
- Expected result: The session stays ended. Client-side interaction alone does not restore it. The user must log in again.

### TC07-11 - Expired session cannot be used from a second browser or tab
- Preconditions: Same user logged in in two tabs or browsers.
- Steps:
  1. Log in in Tab A and Tab B. Leave both idle for 30 minutes.
  2. In Tab B, load a protected page.
- Expected result: The session in Tab B has also expired. The user is redirected to login.

### TC07-12 - Client-side timer cannot be the only control
- Preconditions: User is logged in. Browser devtools are available.
- Steps:
  1. Log in. Disable or stop the client-side idle timer (for example by blocking its script or changing the JS timer in devtools).
  2. Leave the session idle for 30 minutes.
  3. Send a request for protected data directly (for example via curl with the session cookie).
- Expected result: The server rejects the request because the server-side session has expired. Expiry is enforced on the server, not only in the browser.

### TC07-13 - Activity from a non-counted source does not reset the timer [depends on OQ-1]
- Preconditions: User is logged in.
- Steps:
  1. Log in. Only do things outside the defined activity types (for example move the mouse or scroll on a static page with no page load and no submit) for 30 minutes.
  2. Load a protected page after 30 minutes.
- Expected result: Per the requirement wording, the session has expired, because only page loads and form submits are listed as activity. Confirm the actual behaviour with the product owner [depends on OQ-1].

---

## Boundary scenarios

### TC07-14 - Activity at 29:59 resets the timer to a full 30 minutes
- Preconditions: User is logged in.
- Steps:
  1. Log in. Wait 29 minutes 59 seconds.
  2. Load a protected page.
  3. Wait 29 minutes 59 seconds again, then load a protected page.
- Expected result: The user is still logged in at both checks. The full 30-minute window restarts at the last activity.

### TC07-15 - Idle for 29:59 keeps the session active
- Preconditions: User is logged in.
- Steps:
  1. Log in. Do nothing for 29 minutes 59 seconds.
  2. Load a protected page.
- Expected result: The session is still active. The protected page renders.

### TC07-16 - Idle for 30:01 logs the user out
- Preconditions: User is logged in.
- Steps:
  1. Log in. Do nothing for 30 minutes 1 second.
  2. Load a protected page.
- Expected result: The session has expired. The user is redirected to login.

### TC07-17 - Exact 30:00 boundary [depends on OQ-2]
- Preconditions: User is logged in.
- Steps:
  1. Log in. Do nothing for exactly 30:00.
  2. Load a protected page.
- Expected result: Behaviour is not specified. Record the actual outcome and confirm the expected result with the product owner. Do not treat either result as a defect until the boundary is defined [depends on OQ-2].

### TC07-18 - Multiple activity events in quick succession reset the timer once and do not break anything
- Preconditions: User is logged in.
- Steps:
  1. Log in. At 29:00, submit a form three times in quick succession and load a protected page twice.
  2. Wait 29 minutes, then load a protected page.
- Expected result: No errors occur. The session stays active. The timer restarts from the most recent activity, not from the first or last of the burst in a way that causes early expiry.

### TC07-19 - Timer resets correctly across midnight and a daylight-saving change
- Preconditions: User is logged in. Client and server clocks run in a time zone with a DST change.
- Steps:
  1. Log in at 01:50 on the day of a DST change. Wait 30 minutes across the clock change. Load a protected page.
- Expected result: The timeout is based on elapsed time, not on wall-clock time. The session expires exactly 30 minutes after the last activity, unaffected by the clock change.

---

## Other scenarios

### TC07-20 - Form data entered before the timeout is preserved or discarded consistently [depends on OQ-4]
- Preconditions: User is logged in. A long form is partly filled.
- Steps:
  1. Log in. Open a form and type data. Leave the page idle for 30 minutes.
  2. Log in again and return to the form.
- Expected result: Behaviour is not specified. Confirm with the product owner whether the typed data is kept, discarded, or saved as a draft [depends on OQ-4]. Whatever the rule is, it must be applied the same way every time.

### TC07-21 - Laptop sleep or a background tab across 30 minutes of wall-clock time
- Preconditions: User is logged in. Device can be put to sleep.
- Steps:
  1. Log in. Put the device to sleep for 35 minutes. Wake the device and load a protected page.
- Expected result: The session has expired, because the server-side inactivity period is over 30 minutes [depends on OQ-3 for client timer behaviour].

### TC07-22 - Long request still running at the 30-minute mark
- Preconditions: User submits a form that takes several minutes to process.
- Steps:
  1. Log in. At 29:30, submit a form whose processing takes 2 minutes.
  2. Check the result and then load a protected page.
- Expected result: Behaviour is not specified. Confirm with the product owner whether the in-flight request counts as activity, and whether it completes if the session expires mid-request [depends on OQ-7].

---

## Open questions

These are not tested as fact. Each one needs an answer before the related test case can pass or fail reliably.

- OQ-1: What exactly counts as activity? The requirement names only page loads and form submits. Do clicks, keystrokes, scrolling, mouse movement, background AJAX calls, polling, or token refresh count? (Affects TC07-13 and TC07-09.)
- OQ-2: Is the 30-minute limit inclusive or exclusive? What is the precision (second, minute)? (Affects TC07-14 to TC07-17.)
- OQ-3: Is the timeout enforced on the server, in the client, or both? Which one is authoritative when they disagree? (Affects TC07-12 and TC07-21.)
- OQ-4: What happens to unsaved form data when the session expires? Is it discarded, kept in the browser, or saved as a draft? (Affects TC07-20.)
- OQ-5: Is there a warning before logout, such as a countdown or a "stay logged in" prompt? Can the user extend the session from that prompt? If so, does the prompt's click count as activity?
- OQ-6: What does the user see after the timeout: a message, a redirect target, and a return-to URL after re-login?
- OQ-7: Does an in-flight request complete if the session expires during it? Does a long-running request count as activity?
- OQ-8: Do multiple tabs or windows share one activity timer, or does each tab track its own? (Affects TC07-11.)
- OQ-9: Does a validation-failed form submit count as activity? Does a server error on submit count?
- OQ-10: Does a page served from the browser cache (Back or Forward navigation) count as a page load?
- OQ-11: Is "remember me" or any persistent-login feature exempt from the 30-minute rule? If not exempt, does that conflict with the requirement?
- OQ-12: Does the rule apply to API-only clients (mobile apps, tokens, service accounts)? Or only to browser sessions?
- OQ-13: Is there an absolute maximum session length, independent of activity? If so, what happens to the 30-minute rule when that limit is reached first?
- OQ-14: Is the 30-minute value configurable per environment or per role? If so, what are the allowed bounds?
- OQ-15: Is the automatic logout recorded in an audit log? Which fields are logged?
- OQ-16: Do pages that do not require login (public pages) reset or ignore the timer?
