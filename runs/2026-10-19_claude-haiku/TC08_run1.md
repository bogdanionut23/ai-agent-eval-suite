# TC08 - Test cases: date of birth age gate at registration

**Requirement:** To register, a user enters a date of birth. Only users aged 18 or older can create an account.

**Working assumptions (used only to write expected results; each is also listed under Open questions):**
- "Aged 18 or older" means the user has reached their 18th birthday on or before the current date.
- Age is calculated from the entered date of birth against the current date in the server's reference time zone (UTC assumed).
- Expected results for the boundary cases use the example "today" = 2026-10-19.
- A rejected registration creates no account record.

---

## Positive scenarios

### TC08-01 - Adult user registers with a valid date of birth
- **Preconditions:** Registration page is available. No account exists for the test email (test.adult01@example.com).
- **Steps:**
  1. Open the registration form.
  2. Enter a valid email address and a password that meets policy.
  3. Enter date of birth 1996-05-14 (age 30).
  4. Submit the form.
- **Expected result:** Account is created. The user is signed in or shown the normal post-registration step (for example, email verification). No age-related error is shown.

### TC08-02 - User who turned 18 more than one year ago registers
- **Preconditions:** As TC08-01.
- **Steps:**
  1. Enter date of birth 2000-01-01 (age 26).
  2. Complete the remaining required fields and submit.
- **Expected result:** Account is created.

### TC08-03 - Leap-day date of birth that gives an adult age
- **Preconditions:** As TC08-01.
- **Steps:**
  1. Enter date of birth 2000-02-29 (age 26 on 2026-10-19).
  2. Complete the remaining fields and submit.
- **Expected result:** The leap date is accepted as a valid date. Account is created.

---

## Boundary scenarios

### TC08-04 - Turns 18 today (exact 18th birthday, lower boundary, accepted)
- **Preconditions:** As TC08-01. Current date is 2026-10-19.
- **Steps:**
  1. Enter date of birth 2008-10-19 (exactly 18 years ago today).
  2. Complete the remaining fields and submit.
- **Expected result:** Account is created. The user is 18 "or older", so the boundary value is accepted.

### TC08-05 - Turned 18 yesterday (accepted)
- **Preconditions:** As TC08-01.
- **Steps:**
  1. Enter date of birth 2008-10-18 (18 years and 1 day).
  2. Complete the remaining fields and submit.
- **Expected result:** Account is created.

### TC08-06 - Turns 18 tomorrow (one day below the boundary, rejected)
- **Preconditions:** As TC08-01.
- **Steps:**
  1. Enter date of birth 2008-10-20 (age 17; 18th birthday is tomorrow).
  2. Complete the remaining fields and submit.
- **Expected result:** Registration is rejected with an age-requirement error. No account is created.

### TC08-07 - Age 17 years and 364 days, one day below the boundary, entered via a second input method
- **Preconditions:** As TC08-01. Current date is 2026-10-19. Applies only if the form offers more than one way to enter the date (for example, a date picker and a typed field).
- **Steps:**
  1. Enter date of birth 2008-10-20 (17 years and 364 days old today) using the second input method.
  2. Complete the remaining fields and submit.
- **Expected result:** Rejected with an age-requirement error. No account created. Confirms the boundary is applied the same way for every input method. If the form has only one input method, skip this test.

### TC08-08 - Date of birth entered as today (age 0, rejected)
- **Preconditions:** As TC08-01.
- **Steps:**
  1. Enter date of birth 2026-10-19.
  2. Submit.
- **Expected result:** Rejected with an age-requirement error. No account created.

### TC08-09 - Oldest plausible date of birth
- **Preconditions:** As TC08-01.
- **Steps:**
  1. Enter date of birth 1900-01-01.
  2. Submit.
- **Expected result:** Depends on open question Q6 (upper age limit). Under the working assumption (no upper bound), the age rule is satisfied and the account is created. If an upper bound is defined, the result must follow that rule.

---

## Negative scenarios

### TC08-10 - Under-18 user (age 17) cannot register
- **Preconditions:** Registration page is available. No account exists for test.minor01@example.com.
- **Steps:**
  1. Enter a valid email and password.
  2. Enter date of birth 2011-06-15 (age 15).
  3. Submit.
- **Expected result:** Registration is rejected with an age-requirement error. No account is created. The user stays on the form (or sees the agreed rejection screen).

### TC08-11 - Under-18 rejection does not create a partial account
- **Preconditions:** As TC08-10. Access to the user table or account API is available for verification.
- **Steps:**
  1. Perform TC08-10.
  2. Query for test.minor01@example.com in the user store and in any pending-registration or email-verification queue.
- **Expected result:** No user record, no pending registration, and no verification email is sent.

### TC08-12 - Under-18 check is enforced on the server, not only in the client
- **Preconditions:** Registration API endpoint is known. Client-side validation can be bypassed (for example, by calling the API directly with a request tool).
- **Steps:**
  1. Send a registration request to the API with date of birth 2011-06-15 and valid other fields.
  2. Inspect the response and the user store.
- **Expected result:** The API responds with an error (for example, 400 or 403 with an age-requirement code). No account is created.

### TC08-13 - Future date of birth is rejected
- **Preconditions:** As TC08-01.
- **Steps:**
  1. Enter date of birth 2026-10-20 (tomorrow).
  2. Submit.
- **Expected result:** Rejected with a validation error (date must be in the past). No account created. The age-requirement message is acceptable only if the product uses it for this case (see Q5).

### TC08-14 - Non-existent calendar date is rejected
- **Preconditions:** As TC08-01.
- **Steps:**
  1. Enter date of birth 2000-02-31.
  2. Submit.
- **Expected result:** Rejected with an invalid-date error. No account created. The system must not roll the date over to 2000-03-02 and accept it.

### TC08-15 - Non-date text in the date of birth field is rejected
- **Preconditions:** As TC08-01.
- **Steps:**
  1. Enter "abc" or "not a date" in the date of birth field.
  2. Submit.
- **Expected result:** Rejected with an invalid-date error. No account created. No server error (HTTP 500) is shown.

### TC08-16 - Empty date of birth is rejected
- **Preconditions:** As TC08-01.
- **Steps:**
  1. Leave the date of birth field empty.
  2. Fill all other fields and submit.
- **Expected result:** Rejected with a "date of birth is required" error. No account created.

### TC08-17 - Partially entered date of birth is rejected
- **Preconditions:** As TC08-01.
- **Steps:**
  1. Enter only the year (2000) or only day and month, depending on the input control.
  2. Submit.
- **Expected result:** Rejected with an incomplete-date error. No account created.

### TC08-18 - Age-rejected user who corrects the date of birth
- **Preconditions:** As TC08-01. Depends on Q9 (whether retries in the same session are allowed).
- **Steps:**
  1. Enter date of birth 2011-06-15 and submit. Confirm rejection (TC08-10).
  2. Without reloading the page, correct the date of birth to 1996-05-14 and submit again.
- **Expected result:** Under the working assumption, the second attempt with a valid date of birth creates the account. If the product blocks retries (see Q9), the second attempt must be rejected and the expected result changes accordingly. Either way, the rule must be documented and applied consistently.

### TC08-19 - Age rule is not bypassed by changing the date after the check
- **Preconditions:** Registration is split into steps (for example, DOB first, then email and password), or the DOB is stored and reused. Depends on Q7.
- **Steps:**
  1. Submit under-18 date of birth at step 1 and confirm it is rejected.
  2. Attempt to continue to step 2 (URL navigation or API call).
  3. Complete step 2 and submit.
- **Expected result:** Step 2 is not reachable, or the final submit is rejected. No account is created with a stored under-18 date of birth.

---

## Open questions

Each question below affects at least one test or expected result. Behaviour is not invented for these points.

1. **Reference date and time zone (Q1).** Which "today" is used to calculate age: the server's UTC date, or the user's local date? Example: a user born 2008-10-19 in UTC+10 may be 18 locally while still 17 in UTC. Which result is correct?
2. **Definition of "18 or older" (Q2).** Is the user adult on the 18th birthday itself (TC08-04 assumes yes), or only on the day after? The requirement does not say.
3. **Leap-day birthdays (Q3).** A user born on 29 February: do they turn 18 on 28 February or 1 March in non-leap years? This affects a date-of-birth-specific test (TC08-03 covers only the accepted case in a leap year).
4. **Upper age limit or plausibility checks (Q4).** Is there a maximum age (for example, 120 years)? Is 1900-01-01 (TC08-09) valid? The requirement only sets a minimum.
5. **Invalid versus age error messages (Q5).** For a future date or an impossible date, should the user see a generic validation error or the age-requirement message? The requirement does not define the error text or code.
6. **Required error message and content (Q6).** What wording is shown for an under-18 rejection? Must it avoid revealing the exact age threshold or the reason, for privacy or age-gate design reasons?
7. **Multi-step registration (Q7).** Is date of birth captured on the same screen as the other fields, or in an earlier step? This determines the state in TC08-19.
8. **Retries after rejection (Q8, Q9).** Can a user who was rejected try again in the same session or from the same device with a different date of birth? Are rejections counted or rate-limited? Without an answer, TC08-18 cannot be fully assessed.
9. **Storage and retention of date of birth (Q10).** Is the date of birth stored after registration? For how long? Can the user change it later? If it is changed to an under-18 date on an existing account, what happens?
10. **Other registration paths (Q11).** Do social login, SSO, invitation links, or admin-created accounts also require the date of birth check? Is there any exception (for example, parental consent or an organisation-approved account)?
11. **Scope of the rule (Q12).** Is 18 the age for all regions and product lines? Is any other age (for example, 13 or 16 for specific regions) relevant for the same form?
12. **Enforcement point (Q13).** Is server-side enforcement required (TC08-12 assumes yes)? Is the client-side check only for user experience?
13. **Date input format (Q14).** Is the field a date picker or free text? If free text, which format is expected (for example, DD/MM/YYYY versus MM/DD/YYYY)? Ambiguous input such as 03/04/2008 cannot be tested without this.
14. **Re-check on activation (Q15).** Is the age rule checked again at email verification or account activation, for example if the account was created before a rule change? The requirement mentions only account creation.
15. **Test data and environment (Q16).** Test cases use the fixed date 2026-10-19. Should test data be generated relative to the run date, or is a clock-mocking tool available so that boundary tests stay valid?
