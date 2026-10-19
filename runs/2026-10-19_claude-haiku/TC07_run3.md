# Test cases: automatic logout after 30 minutes of inactivity

**Requirement:** A logged-in user is logged out automatically after 30 minutes without activity. Any page load or form submit counts as activity.

**Working assumptions (used only where the requirement is silent; each is also listed under Open questions):**
- The idle timer starts at the user's last counted activity (initially, the login).
- A counted activity restarts the full 30-minute window.
- Expiry is enforced by the server. A logged-out user has no valid session and any protected request is refused.
- Times below are measured from the last counted activity, written as mm:ss.

---

## Positive scenarios

### TC-01: User is logged out after 30 minutes with no activity

**Preconditions**
- User account exists and is active.
- User is logged in and has just reached a protected page (last counted activity at 00:00).
- No other activity occurs in the session.

**Steps**
1. Wait until 30 minutes have passed since the last counted activity (00:00 to 30:00). Do not load pages or submit forms.
2. Load a protected page, or trigger a request that needs an authenticated session.

**Expected result**
- The session is no longer valid.
- The user is redirected to the login page (or receives an unauthenticated response).
- Protected content is not shown.

---

### TC-02: Page load resets the 30-minute window

**Preconditions**
- User is logged in. Last counted activity at login (00:00).

**Steps**
1. At 25:00, load a protected page.
2. At 50:00 (25 minutes after the page load, 55 minutes after login), load a protected page again.
3. At 80:00 (30 minutes after the page load at 50:00), load a protected page.

**Expected result**
- At 25:00, 50:00 and 80:00 the user remains logged in and the pages load normally.
- The logout clock is restarted by each page load, not counted from login.

---

### TC-03: Form submit resets the 30-minute window

**Preconditions**
- User is logged in. Last counted activity at login (00:00).
- A form exists on a protected page and is valid.

**Steps**
1. At 20:00, submit a valid form.
2. At 45:00 (25 minutes after the submit), submit a valid form again.
3. At 74:00 (29 minutes after the second submit), load a protected page.

**Expected result**
- Each valid submit is processed and the user stays logged in.
- The page at 74:00 loads normally.

---

### TC-04: Idle timeout occurs after a page load, not only after login

**Preconditions**
- User logs in at 00:00 (login is the first counted activity).

**Steps**
1. At 10:00, load a protected page (counted activity).
2. At 39:59 (29 minutes 59 seconds after the page load, 39 minutes 59 seconds after login), load a protected page.
3. Wait until 70:01 with no counted activity, then load a protected page.

**Expected result**
- At 39:59 the user is still logged in. The window is measured from the 10:00 page load, not from login, which passed 30 minutes earlier.
- At 70:01 (30 minutes 2 seconds after the 39:59 load) the user is logged out and redirected to login.

---

### TC-05: No logout before 30 minutes of inactivity

**Preconditions**
- User is logged in. Last counted activity at login (00:00).

**Steps**
1. Wait until 15:00 with no counted activity.
2. Load a protected page.

**Expected result**
- The user is still logged in and the page loads normally. Early logout is a defect.

---

## Negative scenarios

### TC-06: Page load after the timeout does not revive the session

**Preconditions**
- User is logged in. Last counted activity at 00:00.

**Steps**
1. Wait until 31:00 with no counted activity.
2. Load a protected page.

**Expected result**
- The user is logged out. The page load at 31:00 is not treated as activity that restores the session.
- The user is redirected to login and protected content is not shown.

---

### TC-07: Form submit after the timeout is not processed

**Preconditions**
- User is logged in. Last counted activity at 00:00.
- A valid form on a protected page is open in the browser, filled in before the timeout.

**Steps**
1. Wait until 31:00 with no counted activity.
2. Submit the form.

**Expected result**
- The submit is not processed. No data is written to the server.
- The user is redirected to login.
- The submit does not restore the session.

Note: whether the filled-in data is kept for the user is an open question (see Q-07).

---

### TC-08: Logged-out user is not affected by the timer

**Preconditions**
- No user is logged in (anonymous browser session).

**Steps**
1. Load a public page.
2. Wait 30 minutes or more.
3. Load a public page again.

**Expected result**
- No logout event occurs and no logout message is shown. The user is simply anonymous, as before.
- Public pages load normally.

---

### TC-09: Non-counted activity does not reset the window (must not extend the session)

**Preconditions**
- User is logged in. Last counted activity at login (00:00).
- The page in use is open and is not reloaded.

**Steps**
1. Move the mouse, type into a text field that is not part of a form submit, and scroll the page at intervals between 00:00 and 30:00. Do not load a page or submit a form.
2. At 30:30, load a protected page.

**Expected result**
- The user is logged out at 30:30. Only page loads and form submits reset the timer, so the interactions in step 1 do not extend the session.

Note: the requirement names only page loads and form submits as activity. If the product intends to count other input, this test needs revising (see Q-01).

---

### TC-10: Failed form submit (validation error) and its effect on the timer

**Preconditions**
- User is logged in. Last counted activity at login (00:00).
- A protected form is open.

**Steps**
1. At 25:00, submit the form with an invalid value so that validation fails and the page is re-rendered with errors.
2. At 45:00 (20 minutes after the failed submit), load a protected page.

**Expected result**
- The user remains logged in at 45:00 if a submit counts as activity regardless of outcome. Expected result is provisional. The requirement says "form submit" counts, which suggests the submit counts even when it fails. Confirm before running (see Q-02).

---

### TC-11: Expired session cannot be used through an API call

**Preconditions**
- User is logged in and has a session token or cookie.
- The user has been idle for 30 minutes or more.

**Steps**
1. Wait until 31:00 with no counted activity.
2. Send a request that reads protected data, using the same session token or cookie.

**Expected result**
- The server refuses the request (for example 401 or redirect to login).
- No protected data is returned.

---

## Boundary scenarios

### TC-12: Idle time just under the limit (29:59)

**Preconditions**
- User is logged in. Last counted activity at 00:00.

**Steps**
1. Wait until 29:59 with no counted activity.
2. Load a protected page.

**Expected result**
- The user is still logged in. The page loads and the window restarts from 29:59.

---

### TC-13: Idle time at exactly 30:00

**Preconditions**
- User is logged in. Last counted activity at 00:00.

**Steps**
1. Wait until exactly 30:00 with no counted activity.
2. Load a protected page.

**Expected result**
- Provisional: the user is logged out. "After 30 minutes" is read as idle time of 30:00 or more. This depends on the inclusive or exclusive boundary, which is not stated (see Q-03). Mark as blocked until Q-03 is answered.

---

### TC-14: Idle time just over the limit (30:01)

**Preconditions**
- User is logged in. Last counted activity at 00:00.

**Steps**
1. Wait until 30:01 with no counted activity.
2. Load a protected page.

**Expected result**
- The user is logged out and redirected to login.

---

### TC-15: Page load at 29:59 restarts the full window

**Preconditions**
- User is logged in. Last counted activity at 00:00.

**Steps**
1. At 29:59, load a protected page.
2. Wait until 59:58 (29 minutes 59 seconds after the 29:59 load) with no counted activity, then load a protected page.
3. Wait until 90:00 (30 minutes 2 seconds after the 59:58 load) with no counted activity, then load a protected page.

**Expected result**
- At 59:58 the user is still logged in. That is more than 30 minutes after login, so the 29:59 load must have restarted the window.
- At 90:00 the user is logged out and redirected to login.

---

### TC-16: Form submit at 29:59 restarts the full window

**Preconditions**
- User is logged in. Last counted activity at 00:00.
- A valid form is available.

**Steps**
1. At 29:59, submit the valid form.
2. At 59:58, load a protected page.

**Expected result**
- The user remains logged in. The submit restarted the 30-minute window.

---

### TC-17: Many activities in quick succession do not stack

**Preconditions**
- User is logged in.

**Steps**
1. Between 10:00 and 10:05, perform 20 page loads or form submits in quick succession, the last one at 10:05.
2. At 40:03 (29 minutes 58 seconds after the last activity at 10:05), load a protected page.

**Expected result**
- The user is still logged in. The window is measured from the most recent activity only. The number of activities does not extend the timeout beyond 30 minutes after the last one.

---

### TC-18: Activity from one user does not affect another user's timer

**Preconditions**
- User A and User B are both logged in, in separate browsers.
- Both last had activity at 00:00.

**Steps**
1. At 20:00, User A loads a protected page.
2. At 31:00, User B loads a protected page.

**Expected result**
- User A's activity at 20:00 restarts only A's window. User A is still logged in at 31:00 (11 minutes since A's last activity).
- User B is logged out at 31:00, since B had no activity since 00:00.

---

## Open questions

Each question below is a gap or ambiguity in the requirement. The matching test case is marked as provisional or blocked. No behaviour has been invented beyond the working assumptions above.

**Q-01: What counts as activity?**
The requirement says "any page load or form submit". Does mouse movement, keyboard input, scrolling, touch, or clicking a link count? Do background AJAX calls, polling, or heartbeat requests count? Does a client-side (single-page app) navigation count as a page load? Relates to TC-09.

**Q-02: Do failed or rejected submits count?**
Does a form submit that fails validation count? Does a submit the server rejects (4xx or 5xx) count? Relates to TC-10.

**Q-03: Exact boundary**
Is the limit "idle for 30 minutes or more" or "idle for more than 30 minutes"? What is the required precision, to the second or to the minute? Relates to TC-13, TC-14.

**Q-04: Scope of the timer**
Is the timer per session, per browser tab, or per user across all devices? If a user is active in one tab and idle in another, does the idle tab log out? Does activity on one device keep another device's session alive?

**Q-05: Warning before logout**
Should the user see a warning or countdown before the logout, with an option to stay logged in? Is there a grace period? The requirement describes only the logout itself.

**Q-06: Post-logout experience**
Where does the user land after the logout (login page, a specific page)? Is a message shown (for example "logged out due to inactivity")? After logging in again, does the user return to the page they were on?

**Q-07: Unsaved form data**
If the user is logged out while a form is filled in, is the data discarded, kept in the browser, or saved as a draft? Relates to TC-07.

**Q-08: Where is the timeout enforced?**
Is the timeout enforced on the server (session expiry), in the browser (client timer), or both? If the client timer and the server disagree, which one wins? This test plan assumes the server is authoritative.

**Q-09: Sleep, hibernation and clock changes**
Does the 30 minutes count wall-clock time while the device sleeps or hibernates? What happens if the system clock or time zone changes during the session?

**Q-10: Remember-me and persistent sessions**
Does the 30-minute rule apply to sessions that use "remember me" or a persistent login cookie? If not, what replaces it?

**Q-11: Absolute session lifetime**
Is there a maximum session length regardless of activity (for example 8 or 12 hours)? If so, what is the interaction with the 30-minute idle rule?

**Q-12: Non-browser clients**
Do API clients, mobile apps, and tokens follow the same 30-minute idle rule? Do they count "page load" and "form submit" at all, or do they need a different activity definition?

**Q-13: Which pages are protected?**
Does the rule apply to every page that requires login, or only to some? Are there public pages where the timer must not run?

**Q-14: Configurability**
Is the 30-minute value fixed, or should administrators be able to change it? If changeable, what are the minimum and maximum values?

**Q-15: Logout effects on other sessions**
Does the automatic logout revoke the server session and any tokens immediately? Does it affect the user's other devices?
