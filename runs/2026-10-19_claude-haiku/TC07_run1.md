# TC07: Test cases for automatic logout after 30 minutes of inactivity

**Requirement:** A logged-in user is logged out automatically after 30 minutes without activity. Any page load or form submit counts as activity.

**Scope of this document:** positive, negative and boundary scenarios derived only from the requirement text. Behaviour the requirement does not define is listed under Open questions, not assumed.

---

## Test cases

### TC07-01: Logout after exactly 30 minutes of no activity (positive)

- **Preconditions:** User is logged in. No page load or form submit has occurred since login. The session timer starts at the moment of the last activity (login counts as the first activity, see OQ-02).
- **Steps:**
  1. Log in and note the time T0.
  2. Do nothing for 30 minutes (no page load, no form submit).
  3. At T0 + 30:00, load any protected page or submit a form.
- **Expected result:** The user is logged out. The protected page is not shown; the user is redirected to the login page (or receives the logged-out response defined for the app). The session is no longer valid.

### TC07-02: No logout before 30 minutes (positive)

- **Preconditions:** User is logged in.
- **Steps:**
  1. Log in at T0.
  2. Wait until T0 + 29:00 without any activity.
  3. Load a protected page.
- **Expected result:** The user stays logged in and the page loads normally. The inactivity timer is reset by this page load.

### TC07-03: Page load resets the 30-minute timer (positive)

- **Preconditions:** User is logged in.
- **Steps:**
  1. Log in at T0.
  2. At T0 + 20:00, load a protected page.
  3. At T0 + 45:00 (25 minutes after the last load), load a protected page again.
- **Expected result:** At T0 + 45:00 the user is still logged in. The 30-minute window was counted from the page load at T0 + 20:00, not from login.

### TC07-04: Form submit resets the 30-minute timer (positive)

- **Preconditions:** User is logged in and is on a page with a form.
- **Steps:**
  1. Log in at T0 and open a page with a form.
  2. At T0 + 25:00, submit the form with valid data.
  3. At T0 + 50:00 (25 minutes after the submit), load a protected page.
- **Expected result:** The form submit is processed. The user is still logged in at T0 + 50:00.

### TC07-05: Form submit at 29:59 keeps the session alive (boundary, just inside)

- **Preconditions:** User is logged in.
- **Steps:**
  1. Log in at T0.
  2. At T0 + 29:59, submit a form.
  3. Load a protected page at T0 + 30:30 (30:30 after login, 31 seconds after the submit).
- **Expected result:** The submit counts as activity. The user is still logged in, because the window is measured from the submit at T0 + 29:59.

### TC07-06: Page load at 29:59 keeps the session alive (boundary, just inside)

- **Preconditions:** User is logged in.
- **Steps:**
  1. Log in at T0.
  2. At T0 + 29:59, load a protected page.
  3. Load another protected page at T0 + 59:58.
- **Expected result:** The user is still logged in at T0 + 59:58, since 29:59 after the first page load is inside the window.

### TC07-07: Inactivity at 30:01 causes logout (boundary, just outside)

- **Preconditions:** User is logged in. Last activity at T0.
- **Steps:**
  1. Do nothing from T0 until T0 + 30:01.
  2. Load a protected page at T0 + 30:01.
- **Expected result:** The user is logged out. The protected page is not served.

### TC07-08: Exact 30:00 boundary (boundary, on the limit)

- **Preconditions:** User is logged in. Last activity at T0. Behaviour at exactly 30:00 is not defined by the requirement (see OQ-03); this test records the agreed answer once known.
- **Steps:**
  1. Do nothing from T0 until T0 + 30:00.
  2. Load a protected page at T0 + 30:00.
- **Expected result:** Defined by the answer to OQ-03. Until then the test is blocked and must not be marked pass or fail.

### TC07-09: Form submit at 30:01 after last activity is not treated as activity (negative)

- **Preconditions:** User is logged in. Last activity at T0. The user has the form open in the browser.
- **Steps:**
  1. Do nothing from T0 until T0 + 30:01.
  2. Submit the form with valid data at T0 + 30:01.
- **Expected result:** The user has already been logged out, so the submit is rejected. The user is sent to the login page and the data is not saved. The late submit does not extend the session.

### TC07-10: Logged-out user is not logged back in by a late page load (negative)

- **Preconditions:** Session has expired after 30 minutes of inactivity.
- **Steps:**
  1. Let the session expire.
  2. Load a protected page directly by URL.
- **Expected result:** The protected page is not shown. The user is redirected to the login page. No protected content is returned.

### TC07-11: Expired session does not survive a back-navigation (negative)

- **Preconditions:** User was logged in, then the session expired after 30 minutes of inactivity.
- **Steps:**
  1. Open a protected page, navigate to a second page, then let the session expire.
  2. Click the browser Back button to a protected page that is cached in the browser.
  3. Attempt a form submit from that page.
- **Expected result:** The protected content is not usable. Any form submit is rejected and the user is redirected to login. No data is saved.

### TC07-12: Activity in one tab keeps the session alive in another tab (positive, multi-tab)

- **Preconditions:** User is logged in and has two tabs of the same application open. (Assumes the session is shared across tabs, see OQ-05.)
- **Steps:**
  1. Log in at T0 in tab A.
  2. At T0 + 20:00, load a page in tab B.
  3. At T0 + 45:00, load a page in tab A.
- **Expected result:** Tab A is still logged in. The activity in tab B counts for the session, so the 30-minute window is measured from T0 + 20:00.

### TC07-13: Logout in one tab applies to all tabs (negative, multi-tab)

- **Preconditions:** User is logged in in two tabs. Both tabs were last active at T0.
- **Steps:**
  1. Do nothing in either tab until T0 + 30:01.
  2. Load a page in tab B at T0 + 30:01 and observe the result.
  3. Load a page in tab A.
- **Expected result:** Tab B shows the logged-out state. Tab A also shows the logged-out state when next used. Neither tab shows protected content.

### TC07-14: Session timer starts at login, not at the first page view (positive)

- **Preconditions:** User is logged in.
- **Steps:**
  1. Log in at T0. Do not load any other page.
  2. Wait until T0 + 30:01.
  3. Load a protected page.
- **Expected result:** The user is logged out. Login itself counts as the starting point of the 30-minute window (see OQ-02 for whether login counts as activity).

### TC07-15: Repeated short activity over a long session keeps the user logged in (positive)

- **Preconditions:** User is logged in.
- **Steps:**
  1. Log in at T0.
  2. Load a protected page every 25 minutes for 2 hours (T0 + 25:00, T0 + 50:00, T0 + 75:00, and so on).
- **Expected result:** The user is never logged out during the 2 hours, because each gap between activities is under 30 minutes.

### TC07-16: Activity that is not a page load or form submit does not reset the timer (negative)

- **Preconditions:** User is logged in. The requirement lists only page load and form submit as activity. Other events are not defined as activity (see OQ-01).
- **Steps:**
  1. Log in at T0.
  2. From T0 + 10:00 to T0 + 30:01, only move the mouse, type in a text field without submitting, and scroll the page. Do not load any page or submit any form.
  3. Load a protected page at T0 + 30:01.
- **Expected result:** Per the requirement as written, the user is logged out, because mouse, keyboard and scroll events are not listed as activity. If the product decides these count, the expected result changes (see OQ-01). This test documents the current requirement and must be confirmed before it is marked pass or fail.

### TC07-17: Background AJAX or polling requests do not count as activity (negative)

- **Preconditions:** User is logged in on a page that polls the server in the background (for example every 60 seconds). Requirement treats only page load and form submit as activity (see OQ-01).
- **Steps:**
  1. Log in at T0 and open a page that polls in the background.
  2. Do nothing else. Let the polling run until T0 + 30:01.
  3. Observe the page at T0 + 30:01.
- **Expected result:** Per the requirement as written, background polling does not reset the timer, so the user is logged out at about T0 + 30:00. If the background requests are meant to count, the expected result changes (see OQ-01).

### TC07-18: Form submit that fails validation still counts as activity (positive)

- **Preconditions:** User is logged in on a form page. Requirement says "form submit" counts; it does not say whether a rejected submit counts (see OQ-04). Assumed: a submit that reaches the server counts.
- **Steps:**
  1. Log in at T0 and open a form.
  2. At T0 + 25:00, submit the form with invalid data so validation fails.
  3. At T0 + 50:00, load a protected page.
- **Expected result:** The user is still logged in at T0 + 50:00, because the submit reached the server. Confirm the assumption against OQ-04 before marking pass.

### TC07-19: Form submit that is blocked on the client does not count as activity (negative)

- **Preconditions:** User is logged in on a form page. Client-side validation is enabled.
- **Steps:**
  1. Log in at T0 and open a form.
  2. At T0 + 25:00, click submit with a required field empty, so the browser blocks the submit before it is sent.
  3. Do nothing else until T0 + 30:01, then load a protected page.
- **Expected result:** No request is sent, so this is not a form submit. The user is logged out at T0 + 30:01. (Depends on OQ-04.)

### TC07-20: Page load of a non-protected page (negative)

- **Preconditions:** User is logged in. Some pages do not require login (for example a public help page). Requirement does not say whether these count (see OQ-01).
- **Steps:**
  1. Log in at T0.
  2. At T0 + 25:00, load a public page that does not require login.
  3. At T0 + 50:00, load a protected page.
- **Expected result:** Depends on OQ-01. If any page load counts, the user is still logged in at T0 + 50:00. If only protected page loads count, the user is logged out at T0 + 50:00 and the test records the outcome.

### TC07-21: Page load of a static asset does not count as activity (negative)

- **Preconditions:** User is logged in. Requirement says "page load", which may not include images, scripts or other resources loaded on a page (see OQ-01).
- **Steps:**
  1. Log in at T0.
  2. Let the browser load only static resources (for example by refreshing the image or script URL directly) at T0 + 25:00.
  3. At T0 + 30:01, load a protected page.
- **Expected result:** Depends on OQ-01. Under the most literal reading, the static asset request is not a page load and the user is logged out at T0 + 30:01.

### TC07-22: Session expiry is not silent data loss (negative)

- **Preconditions:** User is logged in and has typed a long text into a form but has not submitted it. The requirement does not say what happens to unsaved input (see OQ-06).
- **Steps:**
  1. Log in at T0. Open a form and type text into it.
  2. Do nothing until T0 + 30:01.
  3. Submit the form at T0 + 30:01.
- **Expected result:** The submit is rejected and the user is sent to login. Behaviour for the typed text is defined by OQ-06. Until then this test records the observed behaviour only.

### TC07-23: Logout after inactivity is not triggered by a page that is still open (negative)

- **Preconditions:** User is logged in and a page is open in the browser.
- **Steps:**
  1. Log in at T0 and leave the page open on screen.
  2. Wait until T0 + 30:01 without interacting with the page.
  3. Return to the page and check its state, without loading it again.
- **Expected result:** The page content is the same as before. Whether the user has been logged out is only visible on the next page load or form submit, unless OQ-07 defines a client-side logout or warning on screen. The test checks that the page does not show protected data after logout.

### TC07-24: Login again after an automatic logout works normally (positive)

- **Preconditions:** Session expired after 30 minutes of inactivity.
- **Steps:**
  1. Let the session expire.
  2. Log in again with valid credentials.
  3. Load a protected page.
- **Expected result:** The user is logged in and the protected page loads. The new session gets its own 30-minute timer.

### TC07-25: Invalid credentials after automatic logout (negative)

- **Preconditions:** Session expired after 30 minutes of inactivity.
- **Steps:**
  1. Let the session expire.
  2. Log in with an incorrect password.
- **Expected result:** Login fails with the standard error message. No session is created. The inactivity logic is not involved.

---

## Open questions

Each question is about behaviour the requirement does not define. None of the test cases above assume an answer to them without saying so.

- **OQ-01: What counts as "activity"?** The requirement names "any page load or form submit". It does not say whether these also count: mouse movement, keyboard input, scrolling, background AJAX or polling calls, API calls from the page, loading public (non-login) pages, loading static assets such as images or scripts, or navigation within a single-page app that does not reload the page. The word "any" may mean "any page load" (all pages, including public ones) or "any form submit" (all forms, including ones that fail validation). Affects TC07-16, TC07-17, TC07-20, TC07-21.

- **OQ-02: Does login count as activity?** TC07-14 assumes the 30-minute window starts at login. Confirm that login itself starts the timer and that the time of the login request is the reference point.

- **OQ-03: Exact boundary.** Is the user logged out at exactly 30:00 of inactivity, or only after more than 30:00 (30:00 is not "30 minutes without activity" if the user acted at 30:00)? Affects TC07-08 and the "inside" and "outside" boundary definitions in TC07-05 to TC07-07.

- **OQ-04: Form submit that is rejected or blocked.** Does a form submit that fails server-side validation count as activity? Does a submit that is blocked by client-side validation (no request sent) count? TC07-18 and TC07-19 assume the answers shown; both need confirmation.

- **OQ-05: Session scope across tabs and windows.** Is the inactivity timer shared by all tabs and windows of the same browser session, or is it per tab? If one tab is used, does that keep the other tabs alive? Affects TC07-12 and TC07-13.

- **OQ-06: Unsaved input.** When the session expires, what happens to data typed into a form that has not been submitted? Is the user warned first, is the input kept after re-login, or is it lost? The requirement is silent.

- **OQ-07: Warning before logout.** Should the user see a warning (for example a countdown or a "stay logged in" prompt) before the automatic logout? Is the logout forced on the client (the page changes without a request) or only enforced on the next request to the server?

- **OQ-08: What happens after logout.** Where is the user sent (login page, home page, a specific message)? Is a message shown explaining the logout? Is there a "return to the page you were on" after re-login?

- **OQ-09: Clock source.** Is the 30 minutes measured by the server clock, the client clock, or both? What happens if the client clock is wrong or the device goes to sleep and wakes up after 30 minutes? (A laptop that is closed for 40 minutes and then opened: is the user logged out on the next request?)

- **OQ-10: Remember-me and long-lived sessions.** Does the 30-minute inactivity rule apply to sessions created with "remember me" or similar options? Does it apply to SSO or API tokens, or only to the browser session?

- **OQ-11: Interaction with an absolute session limit.** Is there also a maximum session length (for example 8 hours) regardless of activity? If so, which limit wins?

- **OQ-12: Activity from different actors.** Does a request made by a background job or a different user on the same account count as activity? Unlikely, but the requirement does not exclude it.

- **OQ-13: Logout on other devices.** If the user logs out automatically on one device, does their session on other devices end too, or is each session timed separately?

- **OQ-14: Auditing.** Should the automatic logout be logged or audited, and is that part of the requirement?

---

## Summary

- 25 test cases: 14 positive, 9 negative, and 2 boundary cases (TC07-05 to TC07-08 cover the boundary, with TC07-08 blocked on OQ-03).
- The following tests depend on an open question and must not be marked pass or fail until it is answered: TC07-08 (OQ-03), TC07-12 and TC07-13 (OQ-05), TC07-16 and TC07-17 (OQ-01), TC07-18 and TC07-19 (OQ-04), TC07-20 and TC07-21 (OQ-01), TC07-22 (OQ-06).
- Counting note: the cases above are numbered TC07-01 to TC07-25. Some are mixed (for example TC07-05 is positive and boundary), so the positive/negative tally above is approximate. Treat the classification in each title as authoritative.
