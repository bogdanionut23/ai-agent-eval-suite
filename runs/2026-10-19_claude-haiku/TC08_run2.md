# TC08: Test cases for age-gated registration (DOB, 18+)

**Requirement:** To register, a user enters a date of birth. Only users aged 18 or older can create an account.

**Scope note:** Test cases below use only what the requirement states. Anything it leaves open is listed under Open questions and is not assumed in the expected results. Where an expected result depends on an open question, it is marked TBD with the question ID.

---

## Positive scenarios

### TC08-01: Adult user well above the threshold can register

- **Preconditions:** Registration form is available. No account exists for the test email.
- **Steps:**
  1. Open the registration form.
  2. Enter valid name, email and password.
  3. Enter a date of birth 30 years before today (for example 1996-10-19).
  4. Submit the form.
- **Expected result:** Account is created. User is informed of success (or redirected to the next step). No age-related error is shown.

### TC08-02: User who turns 18 today can register

- **Preconditions:** Current date is 2026-10-19 (or the test uses a controlled clock set to this date).
- **Steps:**
  1. Open the registration form.
  2. Enter valid name, email and password.
  3. Enter DOB 2008-10-19 (exactly 18 years before today).
  4. Submit the form.
- **Expected result:** Account is created. The user is treated as 18 years old on the birthday.

### TC08-03: User who turned 18 yesterday can register

- **Preconditions:** As TC08-02.
- **Steps:**
  1. Open the registration form.
  2. Enter valid credentials.
  3. Enter DOB 2008-10-18 (18 years and 1 day).
  4. Submit.
- **Expected result:** Account is created.

---

## Negative scenarios

### TC08-04: User who turns 18 tomorrow is rejected

- **Preconditions:** As TC08-02.
- **Steps:**
  1. Open the registration form.
  2. Enter valid credentials.
  3. Enter DOB 2008-10-20 (17 years and 364 days old).
  4. Submit.
- **Expected result:** Account is not created. An error is shown that the user does not meet the minimum age. No account record and no session are created.

### TC08-05: Child under 13 is rejected

- **Preconditions:** As TC08-02.
- **Steps:**
  1. Open the registration form.
  2. Enter valid credentials.
  3. Enter DOB 2020-01-01.
  4. Submit.
- **Expected result:** Account is not created. Age error is shown. Same outcome as TC08-04.

### TC08-06: DOB today (age 0) is rejected

- **Preconditions:** As TC08-02.
- **Steps:**
  1. Enter DOB 2026-10-19.
  2. Submit.
- **Expected result:** Account is not created. Age error is shown.

### TC08-07: Future date of birth is rejected

- **Preconditions:** As TC08-02.
- **Steps:**
  1. Enter DOB 2027-01-01.
  2. Submit.
- **Expected result:** Account is not created. A validation error about the date is shown (not a generic server error). The age rule is not the only reason it fails, so the message should indicate an invalid date (exact wording TBD per OQ-10).

### TC08-08: Empty DOB field is rejected

- **Preconditions:** As TC08-02.
- **Steps:**
  1. Enter valid name, email and password.
  2. Leave the DOB field empty.
  3. Submit.
- **Expected result:** Account is not created. The DOB field is flagged as required. Form is not submitted to the server or the server returns a required-field error.

### TC08-09: Non-existent calendar date is rejected

- **Preconditions:** As TC08-02.
- **Steps:**
  1. Enter DOB 2000-02-31.
  2. Submit.
- **Expected result:** Account is not created. Invalid-date error is shown. The system does not roll the date over to 2000-03-02 and accept it.

### TC08-10: Non-date text is rejected

- **Preconditions:** As TC08-02.
- **Steps:**
  1. Enter DOB "abc" (or the field's equivalent, if it is a picker, skip this case and see TC08-19).
  2. Submit.
- **Expected result:** Account is not created. Invalid-date validation error is shown.

### TC08-11: Under-18 account creation is blocked on the server when client validation is bypassed

- **Preconditions:** A tool (such as an HTTP client or dev tools) can send the registration request directly. The request includes a valid email and password.
- **Steps:**
  1. Send a registration request with DOB 2015-06-01 directly to the registration endpoint, skipping the browser form.
  2. Inspect the response and the account store.
- **Expected result:** Server rejects the request with an age-related error (for example HTTP 4xx). No account record is created. Client-side checks alone are not relied on.

### TC08-12: Under-18 rejection leaves no stored DOB or account behind

- **Preconditions:** As TC08-02. Test email has never been used.
- **Steps:**
  1. Submit a registration with DOB 2015-06-01 (rejected).
  2. Query the user store for the test email.
- **Expected result:** No user record exists for the email. No partial record (for example an email-only row) is left behind. Whether the rejected DOB is kept in any log is TBD per OQ-7.

### TC08-13: Rejected user cannot register by re-submitting with a different DOB in the same session

- **Preconditions:** As TC08-02.
- **Steps:**
  1. Submit a registration with DOB 2015-06-01. Rejected.
  2. Without leaving the page, change DOB to 1990-01-01.
  3. Submit again.
- **Expected result:** TBD per OQ-4. The requirement does not say whether a retry is allowed. The test is kept so the team decides the expected outcome before execution.

---

## Boundary scenarios

### TC08-14: Birthday boundary at time of day (same calendar day, late evening)

- **Preconditions:** A clock can be set, or the test runs at a known time. The user's time zone and the server's time zone are known (see OQ-2).
- **Steps:**
  1. Set the clock to 2026-10-19 23:30 in the user's time zone.
  2. Enter DOB 2008-10-19. Submit.
  3. Repeat with the clock at 2026-10-19 00:30.
- **Expected result:** TBD per OQ-2. The result must be the same at both times if the rule uses calendar dates only.

### TC08-15: Leap-day birthday (DOB 29 February)

- **Preconditions:** As TC08-02.
- **Steps:**
  1. Enter DOB 2008-02-29 (user turns 18 on 2026-02-29, a date that does not exist in 2026).
  2. Submit on 2026-10-19.
- **Expected result:** TBD per OQ-5. The user is already 18 on 2026-10-19 under any reasonable reading, so acceptance is expected, but the rule for counting age from a leap day must be confirmed.

### TC08-16: Leap-day DOB, submitted on the day before the 18th birthday in a non-leap year

- **Preconditions:** Clock set to 2026-02-28.
- **Steps:**
  1. Enter DOB 2008-02-29.
  2. Submit.
- **Expected result:** TBD per OQ-5. Expected outcome depends on whether the 18th birthday is counted as 28 Feb or 1 Mar.

### TC08-17: Very old but valid DOB

- **Preconditions:** As TC08-02.
- **Steps:**
  1. Enter DOB 1900-01-01.
  2. Submit.
- **Expected result:** TBD per OQ-8. The age requirement is met, but a maximum age or a plausibility check may exist.

### TC08-18: Boundary on the date format year (four-digit year)

- **Preconditions:** As TC08-02.
- **Steps:**
  1. Enter DOB 08-10-19 (two-digit year, ambiguous).
  2. Submit.
- **Expected result:** TBD per OQ-9. The system must either reject the input or interpret it in one documented way. It must not silently accept a date that makes the user appear younger or older than intended.

### TC08-19: Date entry format and locale (ambiguous day/month order)

- **Preconditions:** As TC08-02. Test is run for each supported locale.
- **Steps:**
  1. Enter 05/03/2008 in the DOB field.
  2. Submit and compare the age the system computes.
- **Expected result:** TBD per OQ-3. Reading as 5 March 2008 gives age 18 on 2026-10-19 (accepted). Reading as 3 May 2008 also gives age 18 (accepted). Read as 3 May 2008 for a user meant to be 17, the outcome would differ, so the format must be fixed before this case can pass or fail.

---

## Open questions

These are not answered by the requirement. No behaviour is assumed for them above beyond the TBD markers.

1. **OQ-1, Reference date for "now":** Is the age checked against the server clock, the client clock, or the user's local date? Clock skew can change the result near the birthday.
2. **OQ-2, Time zone:** Which time zone defines the birthday boundary (user local, UTC, or a fixed company zone)? Does the same DOB give the same result for a user in another zone?
3. **OQ-3, Input format and locale:** What date format is used (picker, dd/mm/yyyy, mm/dd/yyyy, ISO)? Is the input a free-text field or a picker? Which locales are supported?
4. **OQ-4, Retry after rejection:** After an under-18 rejection, may the user change the DOB and try again in the same session or later? Is there a lock-out or a notice to the user?
5. **OQ-5, Leap-day birthdays:** When a user is born on 29 February, on which date do they reach 18 in non-leap years: 28 February or 1 March?
6. **OQ-6, Minimum age definition:** Does "aged 18 or older" mean 18 full years completed on the registration day (inclusive of the birthday), as assumed in TC08-02 and TC08-04? Is there any age at which registration is allowed for a parent or guardian?
7. **OQ-7, Data retention:** Is the DOB of a rejected user stored, logged, or sent to analytics? If yes, for how long? Does the same apply to accepted users?
8. **OQ-8, Maximum age and plausibility:** Is there an upper limit on DOB (for example 120 years) or a check against other data?
9. **OQ-9, Two-digit and partial years:** Are two-digit years accepted? If so, how is the century chosen?
10. **OQ-10, Error messages:** What is the exact wording for under-18, invalid date, future date and empty field errors? Must the under-18 error avoid disclosing the threshold (for example "you must be 18")?
11. **OQ-11, Required field:** Is DOB mandatory for every registration path (web, mobile, API, SSO, invitations)? Are SSO and invite-based sign-ups in scope for the same check?
12. **OQ-12, Users aged 13 to 17:** Is there a separate rule (for example parental consent, limited account, or a different flow) for minors? Is the requirement a full block, or does it include such flows?
13. **OQ-13, Changing DOB after registration:** Can a user edit their DOB later? If so, must the 18+ check run again on edit?
14. **OQ-14, Existing accounts:** Do accounts created before this rule are affected? Is there a migration or re-verification step?
15. **OQ-15, Age verification evidence:** Is the DOB only self-declared, or must it be verified by a document or third party? The requirement suggests self-declaration; this is not confirmed.
16. **OQ-16, Error handling on the server:** If the DOB check fails due to a system error (for example the clock service is down), should registration be blocked or allowed? Fail-open or fail-closed?

---

## Coverage summary

| Category | Test cases | Fully defined | TBD (open question) |
|---|---|---|---|
| Positive | TC08-01 to TC08-03 | 3 | 0 |
| Negative | TC08-04 to TC08-13 | 8 | 1 (TC08-13) |
| Boundary | TC08-14 to TC08-19 | 0 | 6 |

Boundary cases are mostly TBD because the requirement does not define the reference date, time zone, leap-day rule or format. They should be re-checked once OQ-1 to OQ-5 are answered.
